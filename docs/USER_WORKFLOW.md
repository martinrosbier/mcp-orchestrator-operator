# User Workflow

This document describes how a user goes from an empty directory to a connected, assessed, and policy-controlled MCP estate.

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
3. Create `moo.config.json` with no embedded credentials.
4. Create `.moo/` for generated snapshots, reports, proposals, and local state.
5. Add recommended ignored local-state paths to `.gitignore` only after preview and confirmation.
6. Offer to find already configured MCP servers, paste a new server, import a selected file, or configure one manually.

Example generated configuration:

```json
{
  "$schema": "https://example.invalid/moo/config.schema.json",
  "version": 1,
  "servers": {},
  "overlays": {},
  "policies": {
    "defaultDecision": "deny",
    "rules": []
  }
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

The simplest path accepts material copied from MCP setup documentation:

```text
Paste an MCP URL, command, package, or configuration:
> npx -y @company/example-mcp

MOO recognized a local stdio MCP server.

Name: example
Command: npx
Arguments: -y @company/example-mcp

This command will execute local code.
Review and test it? Yes
```

MOO converts recognized input into a normalized definition, displays it, and asks for confirmation. If the input is ambiguous, MOO asks only for the missing fields instead of exposing the complete configuration schema.

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

Before writing, MOO displays a diff. `mooctl server test <id>` performs protocol negotiation and discovery without modifying configuration.

## 3. Find and import existing MCP configuration

Many users already have servers configured in clients such as Claude, Codex, Cursor, or VS Code. MOO should offer consent-based detection through explicit client adapters rather than asking users to find and copy entries by hand.

```text
mooctl detect
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

Import selected servers? Yes
```

The equivalent scriptable command may be:

```text
mooctl import --detected
```

It must still require an explicit client scope and provide preview or dry-run behavior. An explicitly selected configuration file remains supported:

```text
mooctl import --from-file ./existing-mcp-config.json
```

Import behavior:

1. Show which clients and exact known locations will be inspected, and proceed only with consent.
2. Detect or ask for the source format.
3. Parse only supported server fields.
4. Show unsupported fields and conflicts.
5. Preserve environment-variable or credential-provider references where possible.
6. Never resolve or copy credential values.
7. Mark connections that require a new sign-in or missing environment variable.
8. Preview the resulting MOO configuration and allow individual server selection.
9. Test selected servers only with explicit approval, because stdio tests execute local commands.
10. Write only after confirmation.

Convenience commands such as `mooctl import --from codex` or `--from claude` may be added after their paths and formats are verified. MOO must not broadly scan a home directory, inspect unselected clients, import every detected server, or change another client's configuration silently.

After import, MOO offers to test the selected connections and gives actionable results:

```text
filesystem   connected        11 tools
github       connected        26 tools
company-crm  sign-in required

Next:
  1. Sign in to company-crm
  2. Discover connected servers
  3. Add another server
```

## 4. Discover the estate

```text
mooctl discover
```

MOO connects to each configured server, negotiates the supported protocol era, obtains server discovery metadata when available, lists tools, normalizes capabilities, and saves a snapshot.

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

Snapshot: .moo/snapshots/2026-10-24T15-30-00Z.json
```

MOO retains server-provided natural-language instructions as declared metadata with provenance. Instructions are untrusted guidance and never authorize execution.

## 5. Assess and understand findings

```text
mooctl assess
mooctl findings
mooctl finding show finding-123
```

Example finding:

```text
HIGH  crm.delete_customer

Potentially destructive capability has no matching execution policy.

Evidence:
- Upstream tool name contains "delete".
- Upstream destructiveHint is true.
- MOO found no matching policy rule.

Recommended layer: enforceable policy
Recommendation: add a deny or require-confirmation rule.
Confidence: high
```

The finding does not prove the implementation is unsafe. It proves that a capability declared as destructive currently lacks a matching MOO policy.

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

The user reviews and tests it:

```text
mooctl policy test --proposal .moo/proposals/finding-123.json
```

Example test output:

```text
crm.search_customer  -> deny (default policy)
crm.delete_customer  -> require_confirmation (proposed rule)
```

The user can then apply the proposal to MOO-owned configuration:

```text
mooctl policy apply --proposal .moo/proposals/finding-123.json
```

Apply must show a diff and require confirmation. It does not alter the upstream MCP server.

## 7. Instructions, overlays, and policies

These mechanisms solve different problems.

| Mechanism | Purpose | Enforceable? | Source |
|---|---|---:|---|
| Server instructions | Help a model use a server effectively | No | Upstream MCP server |
| Tool descriptions and annotations | Describe inputs, behavior, and advisory properties | No | Upstream MCP server |
| MOO organization overlay | Add ownership, classification, aliases, and local guidance | No | Organization |
| MOO policy | Decide visibility and invocation | Yes inside MOO | Organization |
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

## 8. Invoke a capability

```text
mooctl call crm.search_customer --args '{"email":"person@example.com"}'
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

## 9. Explore through chat

```text
mooctl chat
```

Chat can explain catalog records, findings, diffs, overlays, and policy decisions. It can request an invocation, but all calls use the same validation and policy pipeline as `mooctl call`.

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
mooctl diff
mooctl assess
```

MOO reports added, removed, and changed capabilities. Existing overlays and policies are checked against the new fingerprints, and stale rules or suppressions are surfaced for review.

## Protocol basis

The current MCP specification allows a modern server to return optional natural-language `instructions` from `server/discover`. The specification describes those instructions as guidance for LLMs; they are not an authorization primitive. Tool security remains the responsibility of clients and servers, including confirmation, input validation, access control, timeouts, and audit behavior.

- MCP discovery: https://modelcontextprotocol.io/specification/2026-07-28/server/discover
- MCP tools and security considerations: https://modelcontextprotocol.io/specification/2026-07-28/server/tools
