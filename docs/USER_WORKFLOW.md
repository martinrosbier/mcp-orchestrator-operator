# User Workflow

This document describes how a user goes from an empty directory to a discovered and assessed MCP estate with reviewed remediation proposals. Governed invocation arrives in 0.2 and chat in 0.3.

The command names describe the intended product experience. They become executable incrementally as the implementation milestones land.

## 1. Initialize a MOO workspace

Users should not need to understand MOO's configuration schema before connecting their first server.

```text
mkdir company-mcp
cd company-mcp
mooctl init
```

The guided initializer should:

1. Explain which local files it will create.
2. Ask whether the workspace configuration may be committed to Git.
3. Create `moo.config.json` with no embedded credentials or active runtime policy.
4. Create explicit locations for reviewable project artifacts and ignored runtime state.
5. Add only cache, run, and temporary paths to `.gitignore`, after preview and confirmation.
6. Offer to find already configured MCP servers, paste a new server, import a selected file, or configure one manually.

Example generated configuration:

```json
{
  "$schema": "https://example.invalid/moo/config.schema.json",
  "version": 1,
  "sources": {},
  "servers": {},
  "overlays": {}
}
```

The final schema URL will be chosen when the project has a stable publication location. The implementation must not ship the placeholder URL.

The default first-run choice is designed for non-specialists:

```text
How would you like to connect MCP servers?

1. Find servers already configured on this computer
2. Paste a new MCP server URL, command, package, or configuration
3. Import a configuration file
4. Configure manually
```

Manual JSON editing is an advanced escape hatch, not the expected onboarding path.

## 2. Add a server interactively

```text
mooctl server add
```

The simplest path accepts one of four defined forms: an MCP URL, supported JSON configuration, explicit executable-plus-arguments input, or curated package-runner shorthand. MOO rejects pipes, redirection, substitutions, environment prefixes, and ambiguous quoting rather than guessing.

```text
Paste an MCP URL, command, package, or configuration:
> npx -y @company/example-mcp

MOO recognized a local stdio MCP server.

Name: example
Command: npx
Arguments: -y @company/example-mcp

Recognized adapter: npm package runner
Native executable: C:\Program Files\nodejs\node.exe
Runner module: C:\Program Files\nodejs\node_modules\npm\bin\npx-cli.js
Spawned arguments: ["<runner-module>", "-y", "@company/example-mcp"]
Working directory: C:\path\to\company-mcp
Network/install behavior: this package-manager command may download code
Launch definition fingerprint: sha256:launch-example
Environment variables passed: PATH, SYSTEMROOT, EXAMPLE_TOKEN

Save this definition? Yes
Download or execute it for a connection test? No
```

MOO converts recognized input into an executable and argument array without passing it through a shell, displays the actual native executable, runner module, and arguments it will spawn, and asks for confirmation. On Windows, `.cmd` and `.bat` shims are never presented as native no-shell execution; curated package-runner adapters resolve them to a reviewed native runtime and runner module. Downloading packages or starting the process requires a separate confirmation bound to the displayed launch-definition fingerprint. The child receives a minimal environment containing documented runtime essentials and explicitly selected variable names, never the full parent environment. If the input is ambiguous, MOO asks for the missing fields or rejects it.

Example interaction:

```text
Server ID: crm
Transport: stdio
Command: node
Arguments: ./servers/crm.js
Owner (optional): sales-platform
Environment: development

MOO will add:
crm -> node ./servers/crm.js

Test the connection before saving? Yes
```

For a remote server:

```text
Server ID: documents
Transport: streamable-http
URL: https://mcp.company.example/documents
Authorization: bearer token from environment variable
Environment variable: DOCUMENTS_TOKEN
```

MOO stores the reference, not its value:

```json
{
  "servers": {
    "crm": {
      "transport": "stdio",
      "command": "node",
      "args": ["./servers/crm.js"],
      "owner": "sales-platform",
      "environment": "development",
      "protocolEra": "auto"
    },
    "documents": {
      "transport": "streamable-http",
      "url": "https://mcp.company.example/documents",
      "headers": {
        "Authorization": "Bearer ${env:DOCUMENTS_TOKEN}"
      },
      "owner": "knowledge-platform",
      "environment": "production",
      "protocolEra": "auto"
    }
  }
}
```

Before writing, the add wizard can test its in-memory draft without first saving it. After saving, `mooctl server test <id>` performs the same connection check without modifying configuration.

## 3. Find and import existing MCP configuration

Many users already have servers configured in clients such as Claude, Codex, Cursor, or VS Code. MOO should offer consent-based detection through explicit client adapters rather than asking users to find and copy entries by hand.

```text
mooctl source detect --client claude
```

Example interaction:

```text
MOO can inspect known MCP client configuration locations.

Clients to inspect:
  [x] Claude
  [x] Codex
  [ ] Cursor
  [ ] VS Code

MOO will only read the disclosed configuration locations.
It will not copy credentials or start servers.

Continue? Yes
```

Results are selectable:

```text
Found 4 configured servers:

  [x] filesystem    Claude    stdio
  [x] github        Claude    stdio
  [x] company-crm   Codex     streamable-http
  [ ] postgres      Claude    missing POSTGRES_URL

Continue with selected servers? Yes
```

MOO then asks how to manage the selection:

```text
1. Link — re-read this client configuration and detect future changes
2. Import — copy reviewed definitions into this MOO workspace
3. Temporary discovery — build an inventory without retaining the definition; stdio execution still requires separate consent
```

Each retained server records source provenance and a management mode of `linked`, `imported`, or `moo-managed`. Linked sources are never modified by MOO. Removing a linked entry detaches it from the MOO workspace; it does not delete or change the source client's configuration.

Removing one server from a linked source creates a persistent exclusion so refresh does not silently re-add it. `mooctl source unlink <source>` detaches the entire source without modifying its external configuration.

A linked source has two states:

- **Observed:** the latest normalized definition read from the external client configuration.
- **Accepted:** the reviewed definition fingerprint permitted for testing and discovery.

MOO executes only the accepted definition. If the source, resolved executable, arguments, working directory, or credential references change after review, MOO refuses execution until the change is accepted.

If equivalent-looking definitions appear in multiple clients, MOO presents them as possible duplicates with their provenance. It never silently merges them. Source-definition drift is reported separately from capability drift. A linked change to a command, URL, working directory, or credential reference must be reviewed before MOO executes or adopts it.

For a direct, scriptable import, use an explicit client adapter:

```text
mooctl source import --from claude --dry-run
mooctl source import --from claude
mooctl source link --from claude
```

`mooctl source detect --client claude` is preview-only and does not create a persistent detection database. `source link` and `source import` independently re-detect and re-preview the selected client before writing. During `mooctl init`, detection and selection may occur in one interactive process. An explicitly selected configuration file remains supported:

```text
mooctl source import --file ./existing-mcp-config.json
```

Import behavior:

1. Show which clients and exact known locations will be inspected, and proceed only with consent.
2. Detect or ask for the source format.
3. Parse only supported server fields.
4. Show unsupported fields and conflicts.
5. Preserve environment-variable or credential-provider references where possible.
6. If the authorized source contains a literal credential, keep it in memory only while parsing and never log, display, snapshot, propose, or persist it.
7. Refuse to reuse a literal secret by default; mark the server non-runnable until an environment or approved credential-provider reference is supplied.
8. Preview the resulting MOO configuration and allow individual server selection.
9. Resolve relative commands and working directories against the source configuration's base directory and show the normalized result.
10. Test selected servers only with separate explicit approval, because stdio tests may download or execute code.
11. Write only after confirmation.

Client adapters such as `mooctl source import --from codex` or `--from claude` may be added after their paths, base-directory behavior, and formats are verified. MOO must not broadly scan a home directory, inspect unselected clients, import every detected server, or change another client's configuration silently.

After linking or importing, MOO offers to test the selected connections and gives actionable results:

```text
filesystem   connected        11 tools
github       connected        26 tools
company-crm  credential setup required

Next:
  1. Configure credentials for company-crm
  2. Discover connected servers
  3. Add another server
```

When a linked source changes later:

```text
mooctl source refresh claude
mooctl source diff claude
mooctl source accept claude
```

`refresh` records the observed definition without changing the accepted one. `diff` shows the exact normalized change and fingerprints. `accept` updates the accepted definition after review.

## 4. Discover the estate

```text
mooctl discover
```

MOO asks the official SDK to negotiate the supported protocol era, obtains server discovery metadata when available, follows every `tools/list` page within configured page, item, byte, and deadline limits, normalizes capabilities, and saves a snapshot. Repeated or malformed cursors produce a partial observation rather than an apparently complete catalog.

Example:

```text
Connected: 2/2 servers
Discovered: 17 tools

crm
  search_customer
  get_customer
  delete_customer

documents
  search_documents
  read_document

Catalog fingerprint: sha256:catalog-example
Snapshot: .moo/snapshots/sha256-catalog-example.json
Observation: .moo/runs/2026-10-24T15-30-00Z.json
```

MOO retains server-provided natural-language instructions as declared metadata with provenance. Instructions are untrusted guidance and never authorize execution.

## 5. Assess and understand findings

```text
mooctl assess
mooctl finding list
mooctl finding show finding-123
```

Example finding:

```text
MEDIUM  crm.delete_customer

Kind: heuristic
Profile: default

Potentially destructive capability has no explicit tool-specific disposition.

Evidence:
- Upstream tool name contains "delete".
- Upstream destructiveHint is true.
- The name and untrusted annotation suggest destructive behavior.
- Release 0.1 has no runtime invocation path; the proposal describes a future policy posture.

Recommended layer: policy proposal for release 0.2
Recommendation: propose explicit deny or confirmation for a future governed invocation.
Confidence: high
```

The finding does not prove the implementation is unsafe, destructive, or currently executable. Release 0.1 does not route tool calls, so this is a heuristic governance finding about the posture to adopt when governed invocation is introduced.

## 6. Turn a finding into a reviewed remediation

```text
mooctl remediation propose finding-123
```

MOO generates a proposal rather than silently editing configuration:

```json
{
  "proposalVersion": 1,
  "sourceFinding": "finding-123",
  "target": "crm.delete_customer",
  "sourceServer": "crm",
  "basedOnMetadataFingerprint": "sha256:metadata-example",
  "basedOnSchemaFingerprint": "sha256:schema-example",
  "basedOnCompleteRecordFingerprint": "sha256:record-example",
  "layer": "policy",
  "proposedRule": {
    "id": "confirm-crm-delete-customer",
    "match": {
      "capability": "crm.delete_customer"
    },
    "decision": "require_confirmation",
    "reason": "Customer deletion requires explicit operator confirmation"
  }
}
```

In 0.1, the user reviews the proposal as an artifact. MOO explains its expected effect but does not claim runtime enforcement:

```text
Expected 0.2 policy effect if the proposed default-deny model is adopted:
crm.search_customer  -> deny (proposed default policy)
crm.delete_customer  -> require_confirmation (proposed rule)
```

Beginning in 0.2, the user can test and apply the proposal to MOO-owned policy configuration:

```text
mooctl policy test --proposal moo/proposals/finding-123.json
mooctl policy apply --proposal moo/proposals/finding-123.json
```

Apply must show a diff and require confirmation. It does not alter the upstream MCP server. Until 0.2 exists, a policy proposal is evidence of recommended intent, not an enforced control.

## 7. Instructions, overlays, and policies

These mechanisms solve different problems.

| Mechanism | Purpose | Enforceable? | Source |
|---|---|---:|---|
| Server instructions | Help a model use a server effectively | No | Upstream MCP server |
| Tool descriptions and annotations | Describe inputs, behavior, and advisory properties | No | Upstream MCP server |
| MOO organization overlay | Add ownership, classification, aliases, and local guidance | No | Organization |
| MOO policy | Decide visibility and invocation | Yes inside MOO beginning in 0.2 | Organization |
| Trusted execution approval | Authorize an exact controlled operation | Yes for supported executor | Trusted host, such as `mcp-tool-control` integration |

An organization overlay might look like:

```json
{
  "overlays": {
    "crm.delete_customer": {
      "owner": "customer-platform",
      "classification": ["destructive", "customer-data"],
      "guidance": "Use only after retention and legal-hold checks.",
      "displayDescription": "Permanently delete an eligible customer record."
    }
  }
}
```

MOO preserves the original upstream metadata next to this overlay. It must never claim the server supplied organization-owned guidance.

Natural-language instructions can help a model choose correctly, but a model can misunderstand or ignore them. Therefore, instructions cannot implement deny, approval, credential, or data-access controls.

## 8. Invoke a capability — planned for 0.2

```text
mooctl tool call crm.search_customer --args '{"email":"person@example.com"}'
```

MOO resolves the capability, validates arguments, evaluates policy, routes the call, returns the upstream result, and records redacted execution evidence.

For a confirmation-required operation:

```text
Decision: require_confirmation
Rule: confirm-crm-delete-customer
Reason: Customer deletion requires explicit operator confirmation

Continue? [y/N]
```

This confirmation is a MOO gateway control. It does not guarantee that a remote server implements transactional or replay-safe side effects.

For `mcp-tool-control`, MOO's allow or confirmation decision does not replace its trusted-host approval. The trusted approval controller remains unavailable to model-facing clients and MOO chat.

## 9. Explore through chat — planned for 0.3

```text
mooctl chat
```

Chat can explain catalog records, findings, diffs, overlays, and policy decisions. It can request an invocation, but all calls use the same validation and policy pipeline as `mooctl tool call`.

Example:

```text
You: Why can't I call crm.delete_customer?

MOO: The capability is classified as destructive and matches
the confirm-crm-delete-customer rule. Interactive operator confirmation
is required. Its server instruction is advisory and cannot override policy.
```

## 10. Repeat and detect drift

```text
mooctl discover
mooctl snapshot diff
mooctl assess
```

MOO reports added, removed, changed, and unknown capabilities. A capability is removed only when its server was completely enumerated in both compared observations. If a server failed or pagination was partial, its current state is unknown; last-known capabilities may be shown only as historical context. Existing overlays and policies are checked against the new fingerprints, and stale rules or suppressions are surfaced for review.

## Estate visibility vocabulary

MOO must label evidence precisely:

- **Configured:** found in a supported source or MOO configuration.
- **Reachable:** successfully contacted and discovered during a specific run.
- **Governed:** an invocation was routed through MOO policy, beginning in 0.2.
- **Observed usage:** an invocation was seen through MOO or an explicit telemetry integration.

Finding a server in Claude, Codex, Cursor, or another client does not prove that anyone uses it. MOO must not claim organization-wide usage visibility unless it has corresponding invocation or telemetry evidence.

## Project and runtime state

MOO separates reviewable configuration from disposable local state:

```text
moo.config.json       committed project configuration
moo.profile.json      optionally committed quality profile
moo/proposals/        reviewable proposals, optionally committed
.moo/cache/           ignored discovery cache
.moo/runs/            ignored observation records and diagnostics
.moo/reports/         ignored by default; export explicitly for CI
.moo/snapshots/       ignored by default; explicit sanitized export/commit opt-in
```

Canonical catalog snapshots contain no timestamps or run IDs. A separate discovery observation records when the run occurred, which servers were reachable, timing, failures, and the referenced canonical snapshot fingerprint.

## Protocol basis

The current MCP specification allows a modern server to return optional natural-language `instructions` from `server/discover`. The specification describes those instructions as guidance for LLMs; they are not an authorization primitive. Tool security remains the responsibility of clients and servers, including confirmation, input validation, access control, timeouts, and audit behavior.

- MCP discovery: https://modelcontextprotocol.io/specification/2026-07-28/server/discover
- MCP tools and security considerations: https://modelcontextprotocol.io/specification/2026-07-28/server/tools
