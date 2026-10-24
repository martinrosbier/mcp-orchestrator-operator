# Next Steps — From Documents to a Working Vertical Slice

This plan turns the current product definition into a small executable result while keeping high-impact choices visible and reversible.

## 1. Close the decisions that block implementation

Record each accepted decision as a concise ADR in `docs/adr/`.

1. **MCP version and SDK:** verify the current protocol revision and stable official TypeScript SDK from primary sources.
2. **Local configuration:** define the JSON schema for server definitions, environment-variable references, timeouts, and local state paths.
3. **First transport:** use stdio for the first vertical slice; add Streamable HTTP after the connection boundary is proven.
4. **First source adapter:** select one known client configuration format, document its known locations and field mapping, require user consent before inspecting it, and define linked, imported, and inspect-only behavior.
5. **Overlay precedence:** preserve upstream metadata, store organization overlays separately, and define how the effective catalog is rendered.
6. **Policy proposal format:** define stable identifiers, selectors, decisions, reasons, provenance, and expiry or review metadata.
7. **Approval boundary:** document that MOO confirmation governs MOO routing, while specialized executors such as `mcp-tool-control` may require an additional trusted-host approval.

Decisions 1–3 block the first executable slice. Decisions 4–7 can be finalized before their corresponding slices.

## 2. Build the first vertical slice

The first slice should prove this path:

```text
mooctl init
    -> find existing servers or paste a new server
    -> mooctl server add
       or mooctl source detect --client <client>
    -> save, link, import, or directly test the selected draft/candidate
    -> mooctl server test <saved-server>
    -> mooctl discover
    -> normalized tool list
```

Recommended behavior:

1. `mooctl init` previews and creates a local `.moo/` workspace.
2. The initializer offers to detect servers in a supported client or accept a pasted command, package invocation, URL, or configuration snippet.
3. `mooctl server add` recognizes pasted input and asks only for missing fields such as name and environment-variable references.
4. MOO parses executable and arguments separately, shows working-directory and package-download implications, and never invokes a shell by default.
5. MOO displays the resulting definition with secret values redacted and asks for confirmation to save it.
6. A separate confirmation authorizes package download or process execution for testing.
7. `mooctl server test` establishes the SDK-supported MCP lifecycle with a timeout, reports protocol/server information, and exits cleanly.
8. `mooctl discover` retrieves tools and prints their original names, descriptions, schemas, and provenance.
9. An integration test runs this flow against one deliberately small local demo server.

Acceptance criteria:

- A new user does not edit JSON or understand MCP transports to connect the first server.
- No resolved secret is stored in the project configuration or displayed in diagnostics.
- A failed connection returns an actionable error and leaves no orphaned child process.
- The discovered catalog is deterministic for unchanged server metadata.

## 3. Add detection and import as a beginner onboarding path

Implement:

```text
mooctl source detect --client <client>
mooctl source link --from <client> --dry-run
mooctl source link --from <client>
mooctl source import --from <client> --dry-run
mooctl source import --from <client>
mooctl source import --file <explicit-path>
```

The adapter must disclose the selected client's known locations before reading, identify its supported format and base directory, validate every entry, show unsupported fields, redact secret values, detect naming conflicts, preview normalized paths and exact additions, and write only after confirmation. Detection candidates are ephemeral unless the user links or imports them. It must not perform a broad filesystem scan, inspect unselected clients, or import all detected configurations. Testing a detected stdio or package-manager command requires separate confirmation.

## 4. Close the assessment-to-remediation loop

After discovery and the first assessment rules work, implement:

```text
mooctl finding show <finding-id>
mooctl remediation propose <finding-id>
```

The proposal generator chooses one of three outcomes:

- **Upstream recommendation:** fix the MCP server's schema, metadata, tool design, or implementation.
- **Organization overlay:** add ownership, classification, aliases, or advisory usage guidance without rewriting the server's declaration.
- **MOO policy:** enforce `allow`, `deny`, or `require_confirmation` at the MOO invocation boundary.

Release 0.1 produces a reviewable proposal artifact and explains its expected effect. It never edits the upstream server automatically. Policy testing and application arrive with governed invocation in 0.2.

## 5. Validate release 0.1

Run the complete 0.1 journey from a clean checkout:

- detect or add a server without editing JSON;
- discover multiple servers with partial-failure behavior;
- create deterministic snapshots and show drift;
- produce three to five evidence-backed findings;
- generate and review one upstream, overlay, or policy remediation proposal;
- produce terminal and JSON reports.

The test must prove that different observation timestamps for identical discovered content produce the same canonical snapshot fingerprint.

Do not make governed invocation or chat a condition for completing 0.1.

## 6. Release 0.2 — Prove enforcement with one safe and one risky call

Add an invocation service shared by CLI and later chat paths. Demonstrate:

- one permitted read-only call;
- one denied call;
- one confirmation-required call;
- argument validation before forwarding;
- fail-closed behavior when policy evaluation fails;
- redacted execution evidence.

Add `mooctl policy test --proposal <file>` and `mooctl policy apply --proposal <file>` in this release. Applying a proposal changes only MOO-owned local files, after a visible diff and confirmation.

Natural-language server instructions and organization guidance can inform the explanation shown to the operator, but cannot change the policy decision.

## 7. Release 0.3 — Add focused chat last

Once the application APIs exist, chat becomes another interface over them. It may answer questions about servers, tools, findings, policies, and drift, and may request an explicit invocation. Every invocation still passes through validation, policy, confirmation, routing, and audit. The model is never an authorization authority.

## Suggested local GitFlow sequence

Continue from `develop` with short-lived feature branches and small commits:

```text
feature/architecture-decisions
  docs: record initial architecture decisions

feature/cli-onboarding
  chore: initialize typescript tooling
  feat: initialize local workspace
  feat: add server from pasted input
  feat: test server connection

feature/discovery-slice
  feat: discover stdio server tools
  feat: normalize discovered tools
  test: cover discovery integration

feature/config-import
  feat: detect configured mcp servers
  feat: link external client configuration
  feat: preview client config import
  feat: import reviewed server definitions

feature/remediation-proposals
  feat: propose finding remediation

feature/governed-invocation # release 0.2
  feat: test policy proposals
  feat: apply reviewed policy proposal
  feat: route governed tool calls
```

Keep every commit local. Do not add a remote or push.
