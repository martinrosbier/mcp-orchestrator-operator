# Next Steps — From Documents to a Working Vertical Slice

This plan turns the current product definition into a small executable result while keeping high-impact choices visible and reversible.

## 1. Close the decisions that block implementation

Record each accepted decision as a concise ADR in `docs/adr/`.

1. **MCP version and SDK:** verify the current protocol revision and stable official TypeScript SDK from primary sources.
2. **Local configuration:** define the JSON schema for server definitions, environment-variable references, timeouts, and local state paths.
3. **First transport:** use stdio for the first vertical slice; add Streamable HTTP after the connection boundary is proven.
4. **First detection/import adapter:** select one known client configuration format, document its known locations and field mapping, and require user consent before inspecting it.
5. **Overlay precedence:** preserve upstream metadata, store organization overlays separately, and define how the effective catalog is rendered.
6. **Policy proposal format:** define stable identifiers, selectors, decisions, reasons, provenance, and expiry or review metadata.
7. **Approval boundary:** document that MOO confirmation governs MOO routing, while specialized executors such as `mcp-tool-control` may require an additional trusted-host approval.

Decisions 1–3 block the first executable slice. Decisions 4–7 can be finalized before their corresponding slices.

## 2. Build the first vertical slice

The first slice should prove this path:

```text
mooctl init
    -> find existing servers or paste a new server
    -> mooctl server add or mooctl detect
    -> mooctl server test <server>
    -> mooctl discover
    -> normalized tool list
```

Recommended behavior:

1. `mooctl init` previews and creates a local `.moo/` workspace.
2. The initializer offers to detect servers in a supported client or accept a pasted command, package invocation, URL, or configuration snippet.
3. `mooctl server add` recognizes pasted input and asks only for missing fields such as name and environment-variable references.
4. MOO displays the resulting definition with secret values redacted and asks for confirmation.
5. `mooctl server test` starts the server with a timeout, initializes an MCP session, reports protocol/server information, and exits cleanly.
6. `mooctl discover` retrieves tools and prints their original names, descriptions, schemas, and provenance.
7. An integration test runs this flow against one deliberately small local demo server.

Acceptance criteria:

- A new user does not edit JSON or understand MCP transports to connect the first server.
- No resolved secret is stored in the project configuration or displayed in diagnostics.
- A failed connection returns an actionable error and leaves no orphaned child process.
- The discovered catalog is deterministic for unchanged server metadata.

## 3. Add detection and import as a beginner onboarding path

Implement:

```text
mooctl import --from-file <explicit-path>
mooctl detect
mooctl import --detected
```

The adapter must disclose the selected client's known locations before reading, identify its supported format, validate every entry, show unsupported fields, redact secret values, detect naming conflicts, preview the exact additions, and write only after confirmation. It must not perform a broad filesystem scan, inspect unselected clients, or import all detected configurations. Testing a detected stdio command requires separate confirmation.

## 4. Close the assessment-to-remediation loop

After discovery and the first assessment rules work, implement:

```text
mooctl finding show <finding-id>
mooctl remediation propose <finding-id>
mooctl policy test --proposal <file>
mooctl policy apply --proposal <file>
```

The proposal generator chooses one of three outcomes:

- **Upstream recommendation:** fix the MCP server's schema, metadata, tool design, or implementation.
- **Organization overlay:** add ownership, classification, aliases, or advisory usage guidance without rewriting the server's declaration.
- **MOO policy:** enforce `allow`, `deny`, or `require_confirmation` at the MOO invocation boundary.

Applying a proposal changes only MOO-owned local files, after a visible diff and confirmation. It never edits the upstream server automatically.

## 5. Prove enforcement with one safe and one risky call

Add an invocation service shared by CLI and later chat paths. Demonstrate:

- one permitted read-only call;
- one denied call;
- one confirmation-required call;
- argument validation before forwarding;
- fail-closed behavior when policy evaluation fails;
- redacted execution evidence.

Natural-language server instructions and organization guidance can inform the explanation shown to the operator, but cannot change the policy decision.

## 6. Add focused chat last

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
  feat: preview client config import
  feat: import reviewed server definitions

feature/remediation-proposals
  feat: propose finding remediation
  feat: test policy proposals
  feat: apply reviewed policy proposal
```

Keep every commit local. Do not add a remote or push.
