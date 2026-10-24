# MOO — MCP Operations & Orchestration

## Product Requirements Document

**Status:** Draft 0.1  
**Date:** 2026-10-24  
**License intent:** Apache-2.0  
**Delivery model:** Free, open source, local-first, and self-hostable

## 1. Product definition

MOO is an open-source MCP capability operations and orchestration workbench. It connects to an organization's Model Context Protocol servers, creates a normalized inventory of their capabilities, assesses their quality and risk, recommends concrete improvements, and provides a controlled interface for exploring and invoking approved tools.

The product has three pillars:

1. **Connect and map** — connect to MCP servers and build a trustworthy, versioned capability inventory.
2. **Assess and improve** — identify quality, consistency, governance, and interoperability problems and recommend evidence-backed remediation.
3. **Orchestrate and govern** — let users explore the estate conversationally, invoke selected tools through deterministic routing, and eventually export or enforce policy decisions.

> **Connect. Assess. Orchestrate.**

## 2. Problem statement

Organizations adopting MCP accumulate servers and tools created by different teams and vendors. The resulting capability estate may contain:

- Duplicated or overlapping tools.
- Inconsistent names, descriptions, schemas, and error behavior.
- Missing ownership and lifecycle information.
- Misleading or absent safety annotations.
- Excessively broad inputs or permissions.
- Untracked schema and behavior changes.
- Tools exposed to agents that should not see or invoke them.
- Unknown usage, reliability, latency, and operational value.
- Multiple gateways or clients without a common evidence model.

Existing gateways increasingly solve routing, authentication, allowlisting, and audit logging. MOO's initial differentiation is capability intelligence: determining whether the capabilities behind those gateways are coherent, well-designed, appropriately governed, changing safely, and useful in practice.

## 3. Product principles

1. **Evidence over unsupported scoring.** Findings must explain what was observed and why it matters.
2. **Deterministic behavior first.** Ordinary validation and policy logic must not depend on an LLM.
3. **Declared data is not trusted evidence.** Tool descriptions and annotations are claims that may be incomplete or incorrect.
4. **Connect without becoming infrastructure.** The MVP connects to real MCP servers but is not a production gateway, identity provider, or secrets manager.
5. **Explicit invocation before autonomous orchestration.** Users select or approve tool calls; multi-step autonomous planning is deferred.
6. **Local-first and private by default.** A hosted service must never be required for core functionality.
7. **Interoperate instead of replacing.** Reuse official MCP SDKs, conformance tooling, gateways, policy engines, and OpenTelemetry.
8. **Minimum viable abstraction.** Introduce interfaces only when real implementations justify them.
9. **Safe degradation.** Failure to assess or enrich a capability must not be represented as proof that it is safe.
10. **Measurable utility.** Retain rules only when they are protocol-mandated, security-relevant, operationally useful, or supported by an organizational profile.

## 4. Target users

### Primary users

- AI platform engineers managing multiple MCP integrations.
- MCP server developers improving tool definitions and behavior.
- Application-security engineers reviewing MCP capabilities.
- Internal developer-platform teams maintaining service ownership and deployment standards.

### Secondary users

- Agent developers selecting reliable tools.
- Enterprise architects defining capability standards.
- Governance and compliance teams requiring traceable evidence.
- Consultancies performing repeatable MCP estate assessments.

## 5. MVP scope

The MVP is a TypeScript CLI with a reusable application core. A lightweight interactive chat or REPL may be included, but no web dashboard is required.

### 5.1 Connect and map

The system must:

- Provide a guided `mooctl init` workflow that creates a valid local project configuration without requiring users to know its schema.
- Provide `mooctl server add`, `mooctl server test`, `mooctl server list`, and `mooctl server remove` workflows.
- Let users add stdio servers by command and arguments and HTTP servers by URL, protocol era, and secret references.
- Preview and validate configuration changes before writing them.
- Import server definitions from a user-selected MCP client configuration file through explicit, format-aware adapters. Imports must be previewed and must not copy resolved credentials.
- Load server definitions from a version-controlled YAML or JSON file.
- Connect to multiple MCP servers using the official TypeScript SDK.
- Support local stdio and Streamable HTTP transports.
- Support environment-referenced headers or bearer credentials without persisting resolved secrets.
- Discover tools from each configured server.
- Record connection failures without discarding successful results from other servers.
- Normalize discovered tools into a canonical internal representation.
- Namespace tools to prevent collisions across servers.
- Save deterministic JSON catalog snapshots.
- Compare snapshots and classify relevant changes.

Resources and prompts may be recorded as server capability counts, but their full assessment is not required in the MVP.

### 5.2 Assess and improve

The system must:

- Run deterministic, independently testable assessment rules.
- Distinguish protocol failures, confirmed findings, heuristic warnings, and organization-specific requirements.
- Assess descriptions, schemas, annotations, naming, ownership, risky parameters, outputs, and common operational weaknesses.
- Detect exact duplicates and structural similarity candidates.
- Present semantic overlap only as a review candidate, never as confirmed equivalence.
- Apply a version-controlled organizational quality profile.
- Attach evidence, severity, confidence, and remediation to every finding.
- Support documented suppressions with reasons and optional expiration dates.
- Generate terminal, JSON, and SARIF-compatible reports.
- Produce concrete suggested changes where correction is deterministic.
- Exit with configurable status codes for CI use.
- Convert suitable findings into reviewable local remediation proposals, including metadata overlays, policy rules, or upstream code/configuration suggestions.
- Never silently apply a finding recommendation to an upstream MCP server.

### 5.3 Orchestrate and govern

The MVP must provide a narrow proof of orchestration:

- List and search normalized capabilities.
- Resolve a canonical capability identifier to its originating server.
- Validate invocation arguments against the discovered input schema.
- Apply a simple default-deny allow/deny policy before execution.
- Forward an explicitly selected call to the correct upstream server.
- Preserve the upstream MCP result and error distinction.
- Record redacted invocation metadata: capability fingerprint, decision, status, and latency.
- Require explicit confirmation for destructive or high-risk capabilities.

The MVP must not perform autonomous multi-step planning.

### 5.4 Remediation layers

MOO must distinguish three ways to improve or constrain a capability:

1. **Upstream correction:** fix the MCP server's source, schema, descriptions, annotations, or authorization. This is the preferred durable solution when the organization controls the server.
2. **Organization overlay:** attach organization-owned classification, ownership, guidance, aliases, or improved display metadata without claiming the upstream server supplied it. Overlays must preserve original discovered values and provenance.
3. **Enforceable policy:** define visibility and invocation decisions such as allow, deny, or require confirmation. Policies are evaluated by MOO and cannot be replaced by natural-language instructions.

MOO may ingest the optional natural-language `instructions` supplied by modern MCP servers during discovery. Those instructions are untrusted advisory content for model usage and must not affect authorization or safety decisions.

For suitable findings, MOO should support a reviewed workflow such as:

```text
mooctl finding show <finding-id>
mooctl remediation propose <finding-id>
mooctl policy test --proposal <file>
mooctl policy apply --proposal <file>
```

`remediation propose` must explain which layer is recommended and why. `policy apply` modifies only MOO-owned local configuration, requires an explicit review or confirmation, and must never modify an upstream server.

### 5.5 Conversational exploration

Users should be able to ask:

- Which servers and tools are connected?
- Which capabilities can delete or modify customer data?
- What changed since the previous snapshot?
- Which tools appear to overlap?
- Which tools violate the organization's profile?
- How should a particular schema or description be improved?
- Can this explicitly selected tool be invoked with these arguments?

The conversational layer must use structured catalog and assessment data as its source of truth. Execution must pass through the same validation, policy, confirmation, and audit path used by CLI calls.

An LLM provider must be optional. Discovery, assessment, reporting, policy, and direct invocation must work without one.

## 6. Canonical concepts

### Server record

- Configured stable server ID.
- Display name and optional owner.
- Environment and tags.
- Transport configuration with secret references, never resolved values.
- Discovered protocol and capability metadata.
- Discovery timestamp and outcome.

### Capability record

- Logical capability ID: configured server ID plus original tool name.
- Original and namespaced names.
- Description and annotations.
- Normalized input schema.
- Output schema when supplied.
- Source server and provenance.
- Discovery timestamps.
- Metadata, schema, and semantic fingerprints.
- Ownership and organization-supplied classifications.

Logical identity and version fingerprints must remain separate. A description edit must not create a new logical capability, while an input-schema change must remain detectable.

### Assessment finding

- Stable rule ID and version.
- Affected server or capability.
- Category and severity.
- Finding kind: protocol, confirmed, heuristic, or profile.
- Confidence.
- Machine-readable and human-readable evidence.
- Expected and observed conditions.
- Recommended remediation.
- Suppression state and rationale.

### Governance decision

- Principal or local actor.
- Capability ID and fingerprint.
- Decision: allow, deny, or require confirmation.
- Matching policy rule and reason.
- Decision timestamp.

### Execution evidence

- Invocation ID and trace correlation when available.
- Capability ID and schema fingerprint.
- Policy decision.
- Start time, duration, and result classification.
- Upstream error classification.
- No raw arguments or results by default.

## 7. Initial command surface

```text
mooctl config validate
mooctl init
mooctl import --from-file <client-config>
mooctl server add
mooctl server test <server>
mooctl server list
mooctl server remove <server>
mooctl discover
mooctl servers
mooctl tools
mooctl assess
mooctl diff [snapshot-a] [snapshot-b]
mooctl findings
mooctl finding show <finding-id>
mooctl remediation propose <finding-id>
mooctl policy check <capability>
mooctl policy test --proposal <file>
mooctl policy apply --proposal <file>
mooctl call <capability> --args <json>
mooctl chat
```

Commands must support machine-readable output where appropriate and must not mix protocol traffic with diagnostic output.

## 8. Quality profiles

Profiles define organizational expectations without pretending every preference is part of MCP. A profile may configure:

- Required ownership and environment metadata.
- Naming conventions.
- Description expectations.
- Required parameter descriptions and schema constraints.
- Required structured output for selected capability classes.
- Risk-classification requirements.
- Rules enabled, disabled, or severity-adjusted.
- CI finding thresholds.
- Documented exceptions.

Profiles must be versionable and included in report provenance.

## 9. Security and trust boundaries

- MCP server definitions are security-sensitive configuration.
- Starting a stdio server executes a local command and must be treated as code execution.
- Remote endpoints, descriptions, schemas, results, and errors are untrusted.
- Credentials must be externally referenced and never persisted in snapshots.
- Remote connections need explicit timeouts, size limits, redirect behavior, and TLS expectations.
- Tool arguments must be validated before forwarding.
- Visibility filtering and execution authorization are separate checks.
- Protocol compliance must never be presented as proof of safety.
- Logs and reports must redact likely secrets.
- Invocation arguments and results must not be stored by default.
- A failed or unavailable policy check must fail closed for invocation.
- High-risk execution requires explicit confirmation in the MVP.
- Natural-language server instructions and organization guidance are never authorization controls.
- A MOO gateway decision to allow a call does not prove that the upstream server controls its side effects safely.
- For operations delegated to `mcp-tool-control`, MOO authorization must not replace its separate trusted-host approval, replay protection, or controlled-write guarantees.
- Model-facing chat and MCP clients must never receive access to `mcp-tool-control`'s trusted approval controller.

## 10. Non-functional requirements

### Compatibility

- Use the current stable official MCP TypeScript SDK.
- Document supported MCP protocol eras and SDK versions.
- Maintain integration fixtures for supported transports.
- Surface unsupported or malformed behavior clearly.

### Reliability

- One server failure must not abort estate-wide discovery.
- Connection and call timeouts must be configurable.
- Spawned stdio processes must be cleaned up reliably.
- Snapshots must be deterministic for identical inputs.

### Performance

- Discovery should operate concurrently with configurable limits.
- The CLI should remain responsive for at least 100 servers and 5,000 tools using synthetic fixtures if needed.
- Expensive semantic analysis must not be required for baseline assessment.

### Privacy

- Core operation works offline except for explicitly configured remote servers.
- Telemetry is opt-in.
- Reports show which fields were redacted or omitted.

### Maintainability

- Enable strict TypeScript settings.
- Use explicit domain types and structured errors.
- Keep transport, normalization, assessment, policy, invocation, and CLI responsibilities separable in one modular codebase.
- Avoid microservices and framework-heavy abstractions.

## 11. Testing requirements

- Unit tests for normalization and canonical serialization.
- Unit tests for every assessment rule, including counterexamples.
- Fingerprint stability and drift-classification tests.
- Exact and structural duplicate-detection tests.
- Policy allow, deny, confirmation, and fail-closed tests.
- Routing and argument-validation tests.
- Process cleanup, timeout, malformed response, and partial-discovery tests.
- Integration tests using real local MCP interactions.
- Tool-name collision tests.
- Tests proving unauthorized tools cannot be invoked directly.
- Snapshot and report-format tests.
- End-to-end demonstration of discovery, a finding, a rejected call, and an approved call.

## 12. Demonstration environment

Provide three local MCP servers with intentionally varied quality:

- CRM server with read and destructive customer operations.
- Documents server with strong descriptions and schemas.
- Operations server with vague descriptions, broad objects, collisions, and inconsistent annotations.

The demo must show multi-server discovery, a normalized catalog, an actionable finding, an overlap candidate, snapshot drift, a permitted call, a denied or confirmation-required call, and conversational explanation of a finding.

## 13. Explicit non-goals for the MVP

- Production-grade gateway or reverse proxy.
- Autonomous planning and multi-step workflows.
- New identity provider or OAuth authorization server.
- Credential vault or token broker.
- Custom general-purpose policy language or engine.
- Enterprise SSO or multi-tenancy.
- Kubernetes operator.
- Web dashboard.
- Vector database or mandatory embeddings.
- Hosted SaaS dependency.
- Long-term metrics warehouse.
- Complete resources and prompts assessment.
- Automatic modification of upstream servers.
- Claiming security certification or proof of tool safety.
- Reimplementing controlled local-write planning, approval consumption, replay prevention, filesystem containment, or stale-state protection already owned by `mcp-tool-control`.

## 14. Proposed module boundaries

```text
src/
  config/          Configuration and secret references
  upstreams/       MCP connections, transports, lifecycle, discovery
  catalog/         Records, normalization, fingerprints, snapshots
  assessment/      Rules, findings, profiles, suppressions
  comparison/      Drift, duplicates, overlap candidates
  policy/          Minimal decisions and future export adapters
  invocation/      Validation, authorization, confirmation, routing, evidence
  reporting/       Terminal, JSON, SARIF-compatible output
  chat/            Optional natural-language interface over application APIs
  cli/             Commands and user interaction
examples/
tests/
docs/
```

These are package boundaries inside a modular monolith, not services. Do not create empty modules before their behavior exists.

## 15. Delivery sequence

1. Establish repository standards, toolchain, CI, and architecture decisions.
2. Implement guided initialization, add one local server, and display discovered tools.
3. Connect to multiple servers and preserve partial failures.
4. Normalize the catalog and create stable snapshots.
5. Add fingerprints and snapshot diffing.
6. Implement the rule engine and initial deterministic rules.
7. Add profiles, suppressions, and reports.
8. Add duplicate and structural-overlap candidates.
9. Add minimal policy decisions and explicit routed calls.
10. Add reviewed finding-to-overlay and finding-to-policy remediation workflows.
11. Add the focused chat/REPL interface.
12. Add optional `mcp-tool-control` integration without crossing its trusted-host boundary.
13. Complete demonstration servers and end-to-end tests.
14. Validate against real estates before expanding runtime functionality.

## 16. MVP acceptance criteria

The MVP is complete when:

- Three configured servers can be discovered in one run.
- A new user can initialize configuration and add or test a server without hand-writing JSON.
- An existing supported client configuration can be imported with a preview and without copying secrets.
- Tool-name collisions do not overwrite or ambiguously identify capabilities.
- Repeated discovery creates stable fingerprints.
- Meaningful schema changes appear in deterministic diffs.
- At least ten useful deterministic rules are implemented and tested.
- Findings contain evidence, severity, confidence, and remediation.
- A quality profile changes enforcement without source changes.
- A suitable finding can produce a reviewable overlay or policy proposal with provenance.
- Reports serve humans and CI automation.
- An allowed call routes to the correct server.
- A denied tool cannot be invoked through any CLI or chat path.
- A high-risk tool requires confirmation.
- Chat answers questions from structured catalog and finding data.
- Core functionality works without an LLM or hosted service.
- Setup and demonstration instructions work from a clean checkout.

## 17. Validation criteria

Before committing to a production gateway or formal standard, validate that:

- At least three real MCP estates can be assessed.
- High-severity findings achieve at least 70% maintainer acceptance in pilots.
- At least half of deterministic remediation suggestions are useful.
- Snapshot comparison catches a meaningful regression.
- One organization requests CI integration.
- MOO reveals information not already supplied by the organization's gateway or scanner.

If organizations care only about access control and vulnerability scanning, integrate with or contribute to existing gateway/scanner projects instead of expanding into a competing platform.

## 18. Future direction

- Gateway and registry adapters.
- OPA or Cedar policy-data export.
- OpenTelemetry runtime correlation.
- Usage, reliability, latency, and unused-capability analysis.
- Optional semantic overlap analysis.
- AI-assisted description and schema proposals.
- Downstream MCP gateway interface.
- Approval workflows using existing protocol mechanisms.
- Portable capability-evidence and quality-profile specifications.

Standardization should begin with implementation-independent JSON schemas for capability records, findings, profiles, fingerprints, and evidence. Wire-level MCP extensions should wait for demonstrated interoperability needs across multiple implementations.

## 19. Open decisions

- Supported protocol-version matrix for the first release.
- Package manager and Node.js support policy.
- YAML versus JSON for human-authored configuration.
- JSON Schema normalization and fingerprint canonicalization.
- Files only versus SQLite for local history.
- Chat provider abstraction and whether chat ships in 0.1 or 0.2.
- Confirmation UX for interactive and non-interactive execution.
- Initial rules and required evidence for confirmed versus heuristic findings.
- Final project, package, and CLI naming after namespace and trademark checks.
- Which MCP client configuration formats are supported by the first import adapters.
- The exact merge rules and precedence for original metadata, organization overlays, and server-provided instructions.
- Whether `mcp-tool-control` integration is demonstrated in 0.1 or the next milestone.
