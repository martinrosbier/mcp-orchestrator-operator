# MOO — MCP Operations & Orchestration

## Product Requirements Document

**Status:** Draft 0.1  
**Date:** 2026-10-24  
**License intent:** Apache-2.0  
**Delivery model:** Free, open source, local-first, and self-hostable

## 1. Product definition

MOO is an open-source MCP capability operations and orchestration workbench. Release 0.1 creates a workspace-scoped evidence view from explicitly enrolled Model Context Protocol sources; it must never imply that one workspace represents the organization's complete estate. MOO normalizes discovered capabilities, assesses their quality and risk, recommends concrete improvements, and later provides a controlled interface for exploring and invoking approved tools.

The product has three pillars:

1. **Connect and map** — connect to MCP servers and build a trustworthy, versioned capability inventory.
2. **Assess and improve** — identify quality, consistency, governance, and interoperability problems and recommend evidence-backed remediation.
3. **Orchestrate and govern** — let users explore the estate conversationally, invoke selected tools through deterministic routing, and eventually export or enforce policy decisions.

> **Connect. Assess. Orchestrate.**

## 2. Problem statement

Organizations adopting MCP accumulate servers and tools created by different teams and vendors. The resulting capability estate may contain:

- Duplicated or overlapping tools.
- Inconsistent names, descriptions, schemas, and declared error contracts.
- Missing ownership and lifecycle information.
- Misleading or absent safety annotations.
- Excessively broad inputs or permissions.
- Untracked schema and declared-contract changes, plus runtime behavior changes where invocation or telemetry evidence is available.
- Tools exposed to agents that should not see or invoke them.
- Unknown usage, reliability, latency, and operational value when runtime evidence is available.
- Multiple gateways or clients without a common evidence model.

Existing gateways increasingly solve routing, authentication, allowlisting, and audit logging. MOO's initial differentiation is capability intelligence: determining whether the capabilities behind those gateways are coherent, well-designed, appropriately governed, changing safely, and useful in practice.

## 3. Product principles

1. **Evidence over unsupported scoring.** Findings must explain what was observed and why it matters.
2. **Deterministic behavior first.** Ordinary validation and policy logic must not depend on an LLM.
3. **Declared data is not trusted evidence.** Tool descriptions and annotations are claims that may be incomplete or incorrect.
4. **Connect without becoming infrastructure.** Release 0.1 connects to real MCP servers but is not a production gateway, identity provider, or secrets manager.
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

- Builders and "vibe coders" already using MCP-enabled applications who want to understand and improve their setup without learning configuration formats.
- Agent developers selecting reliable tools.
- Enterprise architects defining capability standards.
- Governance and compliance teams requiring traceable evidence.
- Consultancies performing repeatable MCP estate assessments.

## 5. Release roadmap and 0.1 scope

Release 0.1 is a TypeScript CLI with a reusable application core. It ends at connection, cataloging, assessment, drift detection, and reviewed remediation proposals. Governed invocation is planned for 0.2 and focused chat for 0.3. No web dashboard is required.

### 5.1 Connect and map

The system must:

- Provide a guided `mooctl init` workflow that creates a valid local project configuration without requiring users to know its schema.
- Make consent-based detection of MCP servers already configured in supported clients the default onboarding choice.
- Inspect only disclosed, documented configuration locations for clients selected by the user; never perform a silent or broad filesystem scan.
- Provide `mooctl server add`, `mooctl server test`, `mooctl server list`, and `mooctl server remove` workflows.
- Accept only defined input grammars: an MCP URL, a supported JSON configuration, an explicit executable-plus-arguments form, or a curated package-runner shorthand.
- Reject shell operators, redirection, substitutions, environment-variable prefixes, and ambiguous quoting rather than guessing how to parse them.
- Parse pasted commands into an executable and argument array without invoking a shell by default.
- Show the exact executable, arguments, working directory, and known package-download or network implications before any test.
- Require a separate confirmation before downloading packages, starting a process, or testing an imported stdio command.
- Let advanced users add stdio servers by command and arguments and HTTP servers by URL, protocol era, and secret references.
- Preview and validate configuration changes before writing them.
- Import server definitions from a user-selected MCP client configuration file through explicit, format-aware adapters. Imports must be previewed and must not copy resolved credentials.
- Preserve environment-variable and credential-provider references when importing. If credentials cannot be reused, import the definition with a clear credential-setup or missing-credential action.
- Require confirmation before testing a detected stdio server because doing so executes its configured local command.
- Support three source modes: linked configuration that can be re-read for drift, imported definitions copied into MOO, and temporary non-retained discovery. Temporary discovery still requires separate execution consent for stdio servers.
- Record source provenance and management mode for every server definition.
- Keep sensitive or machine-specific source paths out of portable reports and committed configuration; store a redacted or adapter-relative locator where possible.
- Resolve relative commands and working directories against the source configuration's documented base directory before previewing an import.
- Report source-definition drift separately from discovered capability drift. A linked change to a command, URL, working directory, or credential reference must be reviewed before MOO executes or adopts it.
- Keep the last observed linked definition separate from the last accepted definition. Testing and discovery must use only the accepted normalized definition.
- Bind execution approval to the accepted definition fingerprint and reject execution if the source or resolved launch definition changed after review.
- Treat removal according to management mode: detach linked definitions, delete only MOO-managed or imported definitions, and never edit an external client configuration.
- Persist per-source selections and exclusions so refreshing a linked source does not silently re-add a deliberately removed entry. `source unlink` detaches the entire external source.
- Load server definitions from version-controlled JSON in 0.1. YAML is explicitly deferred until a demonstrated user need justifies another parser and representation.
- Connect to multiple MCP servers using the official TypeScript SDK.
- Support local stdio and Streamable HTTP transports.
- Support environment-referenced headers or bearer credentials without persisting resolved secrets.
- Discover tools from each configured server.
- Follow MCP pagination until completion, subject to configurable page, item, byte, and deadline limits; detect repeated or malformed cursors.
- Record connection failures without discarding successful results from other servers.
- Record per-server enumeration completeness. A failed or partial enumeration must be classified as unknown or unobserved and must never create false removal findings.
- Normalize discovered tools into a canonical internal representation.
- Namespace tools to prevent collisions across servers.
- Save deterministic JSON catalog snapshots.
- Compare snapshots and classify relevant changes.

Resources and prompts may be recorded as server capability counts, but their full assessment is not required in 0.1.

### 5.2 Assess and improve

The system must:

- Run deterministic, independently testable assessment rules.
- Distinguish protocol failures, confirmed findings, heuristic warnings, and organization-specific requirements.
- Assess descriptions, schemas, annotations, naming, ownership, risky parameters, declared output schemas, and common operational weaknesses.
- In 0.1, assess only declared contracts, configuration, reachability, and discovery evidence. Runtime behavior, result quality, reliability, and latency require later invocation or telemetry evidence.
- Detect exact duplicates and structural similarity candidates.
- Present semantic overlap only as a review candidate, never as confirmed equivalence.
- Apply a version-controlled organizational quality profile.
- Attach evidence, severity, confidence, and remediation to every finding.
- Support documented suppressions with reasons and optional expiration dates.
- Generate terminal and JSON reports. Add SARIF-compatible output after CI demand is validated.
- Produce concrete suggested changes where correction is deterministic.
- Exit with configurable status codes for CI use.
- Convert suitable findings into reviewable local remediation proposals, including metadata overlays, policy rules, or upstream code/configuration suggestions.
- Never silently apply a finding recommendation to an upstream MCP server.

### 5.3 Planned controlled invocation — 0.2

Release 0.2 should provide a narrow proof of orchestration:

- List and search normalized capabilities.
- Resolve a canonical capability identifier to its originating server.
- Validate invocation arguments against the discovered input schema.
- Apply a simple default-deny allow/deny policy before execution.
- Keep visibility decisions (`visible` or `hidden`) separate from invocation decisions (`allow`, `deny`, or `require_confirmation`).
- Forward an explicitly selected call to the correct upstream server.
- Preserve the upstream MCP result and error distinction.
- Record redacted invocation metadata: structured capability identity, complete-record and schema fingerprints, decision, status, and latency.
- Require explicit confirmation for destructive or high-risk capabilities.

Release 0.2 must not perform autonomous multi-step planning.

### 5.4 Remediation layers

MOO must distinguish three ways to improve or constrain a capability:

1. **Upstream correction:** fix the MCP server's source, schema, descriptions, annotations, or authorization. This is the preferred durable solution when the organization controls the server.
2. **Organization overlay:** attach organization-owned classification, ownership, guidance, aliases, or improved display metadata without claiming the upstream server supplied it. Overlays must preserve original discovered values and provenance.
3. **Enforceable policy:** propose visibility and invocation decisions such as allow, deny, or require confirmation. Runtime evaluation begins in 0.2; natural-language instructions can never replace policy.

MOO may ingest the optional natural-language `instructions` supplied by modern MCP servers during discovery. Those instructions are untrusted advisory content for model usage and must not affect authorization or safety decisions.

For suitable findings, MOO should support a reviewed workflow such as:

```text
mooctl finding show <finding-id>
mooctl remediation propose <finding-id>
```

`remediation propose` must explain which layer is recommended and why. In 0.1, proposals are reviewable artifacts and are not runtime enforcement. Policy testing and application arrive with governed invocation in 0.2. No remediation command may modify an upstream server automatically.

Every proposal must record the source server and the basis fingerprints relevant to its layer: schema fingerprints for schema corrections, metadata fingerprints for descriptions or overlays, source-definition fingerprints for connection recommendations, and complete-record fingerprints when multiple dimensions matter. Testing or applying a stale proposal in 0.2 must fail closed and require regeneration or an explicit reviewed migration.

### 5.5 Planned conversational exploration — 0.3

Users should be able to ask:

- Which servers are configured or reachable, and which tools were discovered?
- Which capabilities can delete or modify customer data?
- What changed since the previous snapshot?
- Which tools appear to overlap?
- Which tools violate the organization's profile?
- How should a particular schema or description be improved?
- Can this explicitly selected tool be invoked with these arguments?

The conversational layer must use structured catalog and assessment data as its source of truth. Execution must pass through the same validation, policy, confirmation, and audit path used by CLI calls.

An LLM provider must be optional. Discovery, assessment, reporting, policy, and direct invocation must work without one.

In 0.3, server instructions, descriptions, errors, and results must enter model context only as size-bounded, provenance-labeled untrusted data. They must never be spliced into privileged prompts. Model-produced calls and proposals require deterministic schema validation, and model output can never satisfy human confirmation.

## 6. Canonical concepts

### Server record

- Configured stable server ID.
- Display name and optional owner.
- Environment and tags.
- Transport configuration with secret references, never resolved values.
- Discovered protocol and capability metadata.
- Configuration source, source adapter, and management mode: linked, imported, or MOO-managed.
- Last observed normalized definition and fingerprint for linked-source drift detection.
- Last accepted normalized definition and fingerprint used for testing and discovery.
- Review status when observed and accepted definitions differ.

### Capability record

- Logical capability identity as a structured tuple of configured server ID and original tool name.
- Reversibly encoded CLI/display identifier; dotted display strings must never be used as an ambiguous identity or hashing input.
- Original and namespaced names.
- Description and annotations.
- Normalized input schema.
- Output schema when supplied.
- Source server and provenance.
- Metadata, schema, and structural fingerprints. A deterministic hash must not be described as semantic analysis.
- Ownership and organization-supplied classifications.

Logical identity and version fingerprints must remain separate. A description edit must not create a new logical capability, while an input-schema change must remain detectable.

### Catalog snapshot

- Canonical server and capability content serialized deterministically.
- No timestamps, run IDs, latency, or transient failure data in the fingerprinted content.
- Separate metadata, schema, and complete-record fingerprints with documented canonicalization.

### Discovery observation

- Observation timestamp and run identifier.
- Per-server reachability, protocol era, duration, and failure classification.
- Per-server enumeration completeness: complete, partial, failed, or not attempted.
- Reference to the deterministic catalog snapshot fingerprint.
- Stored outside the canonical content so repeated identical discovery produces the same catalog fingerprint.

Snapshot comparison must report removal only when the relevant server was completely enumerated in both observations. Otherwise the state is unknown or unobserved. The last-known catalog may be displayed for context but must not be presented as current evidence.

### Assessment finding

- Stable rule ID and separately recorded rule version.
- Stable finding key derived from the rule ID (not its version), structured subject identity, and deterministic evidence key, so the same issue can be tracked across rule-pack upgrades.
- Versioned finding occurrence containing the rule version, observation ID, and basis fingerprints. Suppressions target the stable finding key and may optionally constrain a rule-version range or expiry.
- Affected server or capability.
- Category and severity.
- Finding kind: protocol, confirmed, heuristic, or profile.
- Confidence.
- Machine-readable and human-readable evidence.
- Expected and observed conditions.
- Recommended remediation.
- Suppression state and rationale.
- Basis fingerprints and suppression matching/expiration metadata.

### Report provenance manifest

Every report must include:

- MOO version and build identifier.
- Report schema version.
- Rule-pack IDs and versions.
- Quality-profile and project-configuration fingerprints.
- Source-adapter versions.
- Exact MCP SDK package/version once per report, plus the negotiated protocol era and revision for every server observation.
- Catalog snapshot fingerprint and discovery observation ID.
- Finding counts by status, including suppressed, skipped, and failed rules.

### Governance decision — planned for 0.2

- Execution context, including kind, label, and whether it is authenticated.
- Capability ID and fingerprint.
- Decision: allow, deny, or require confirmation.
- Matching policy rule and reason.
- Decision timestamp.

### Execution evidence — planned for 0.2

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
mooctl source detect --client <client>
mooctl source link --from <client> [--dry-run]
mooctl source import --from <client> [--dry-run]
mooctl source import --file <client-config>
mooctl source refresh <source>
mooctl source diff <source>
mooctl source accept <source> [--server <id>]
mooctl source unlink <source>
mooctl server add
mooctl server test <server>
mooctl server list
mooctl server remove <server>
mooctl discover
mooctl tool list
mooctl assess
mooctl snapshot diff [snapshot-a] [snapshot-b]
mooctl finding list
mooctl finding show <finding-id>
mooctl remediation propose <finding-id>
```

Planned 0.2 commands include `policy init`, `policy check`, `policy test`, `policy apply`, and `tool call`. `mooctl chat` is planned for 0.3.

Commands must support global `--json` and `--no-input` behavior where applicable. Machine-readable data goes to stdout, diagnostics go to stderr, and stable documented exit codes identify failure classes. When required consent is unavailable under `--no-input`, the command must refuse safely. A generic `--yes` must never implicitly authorize package download or process execution.

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

### Initial 0.1 rule pack

The first release implements these independently testable rules, each with positive and counterexample fixtures:

1. Missing or blank tool description as a confirmed metadata finding; vague or low-information descriptions require a separately identified heuristic rule.
2. Broad object input with no declared properties or constraints, or unrestricted additional properties, using explicit deterministic predicates.
3. Collision after canonical name normalization.
4. Risky-name and safety-annotation mismatch, reported only as a heuristic.
5. Missing ownership, reported as an organization-profile finding.

Structural overlap remains a comparison candidate and does not count as one of the five assessment rules.

## 9. Security and trust boundaries

- MCP server definitions are security-sensitive configuration.
- Starting a stdio server executes a local command and must be treated as code execution.
- Pasted or imported command text must not be passed through a shell unless an explicit advanced mode is designed, documented, and separately confirmed.
- Package-manager invocations may download and execute third-party code and require a distinct, informed confirmation.
- Resolve and display the executable path where possible, and document PATH lookup, symlink, package-manager shim, and Windows/POSIX launcher behavior.
- Spawn stdio servers with a minimal environment containing only documented runtime essentials and explicitly selected variable names. Do not inherit unrelated parent-process secrets.
- Remote endpoints, descriptions, schemas, results, and errors are untrusted.
- Sanitize terminal control characters and ANSI escape sequences from untrusted output. Bound schema depth, item counts, rendered diagnostics, links, and payload sizes.
- Credentials must be externally referenced and never persisted in snapshots.
- Streamable HTTP rejects credentials embedded in URLs, verifies TLS certificates and hostnames, disables redirects by default, and never forwards an authorization header across origins. If redirects are explicitly enabled, each hop is capped, resolved, and revalidated against the endpoint policy before following it.
- Proxies require explicit configuration and are recorded in redacted observation provenance. Connect, read, overall, response-size, and stream limits must have secure finite defaults and configurable upper bounds.
- Loopback and private-network endpoints remain usable for enterprise MCP, but must be explicitly configured; DNS results are revalidated at connection time to prevent a reviewed public target from resolving to a disallowed address class.
- Tool arguments must be validated before forwarding.
- Visibility filtering and execution authorization are separate checks.
- Protocol compliance must never be presented as proof of safety.
- Logs and reports must redact likely secrets.
- Invocation arguments and results must not be stored by default.
- A failed or unavailable policy check must fail closed for invocation.
- High-risk execution requires explicit confirmation when governed invocation is introduced in 0.2.
- Natural-language server instructions and organization guidance are never authorization controls.
- Client detection may inspect only known locations for explicitly selected clients, after showing the intended scope and receiving consent.
- Detection and import must never copy resolved credentials, modify another client's configuration, or start a detected stdio server without separate confirmation.
- A MOO gateway decision to allow a call does not prove that the upstream server controls its side effects safely.
- For operations delegated to `mcp-tool-control`, MOO authorization must not replace its separate trusted-host approval, replay protection, or controlled-write guarantees.
- Model-facing chat and MCP clients must never receive access to `mcp-tool-control`'s trusted approval controller.

## 10. Non-functional requirements

### Compatibility

- Before implementation, record the exact official MCP TypeScript SDK package/version, Node.js matrix, supported protocol eras, and lockfile policy in an ADR; do not depend on an unpinned notion of "current stable."
- Document supported MCP protocol eras and SDK versions.
- Maintain integration fixtures for supported transports.
- Surface unsupported or malformed behavior clearly.

### Reliability

- One server failure must not abort estate-wide discovery.
- Connection and call timeouts must be configurable.
- Spawned stdio processes must be cleaned up reliably.
- Snapshots must be deterministic for identical inputs.
- Observation metadata must not alter canonical catalog fingerprints.

### Performance

- Discovery should operate concurrently with configurable limits.
- On the release reference environment (4 logical CPU cores, 8 GiB RAM, supported Node.js LTS, and local SSD), local processing of a synthetic complete catalog containing 100 servers and 5,000 tools must emit progress within 1 second, complete normalization, assessment, and snapshot serialization within 10 seconds, complete a snapshot diff within 5 seconds, and remain below 512 MiB peak RSS. Network discovery time is excluded and reported separately.
- Expensive semantic analysis must not be required for baseline assessment.
- Phase 0 must define a reproducible reference environment and budgets for normalization, assessment, snapshot serialization/diff, peak memory, and progress reporting. Network discovery time is measured separately from local processing.

### Privacy

- Core operation works offline except for explicitly configured remote servers.
- Telemetry is opt-in.
- Reports show which fields were redacted or omitted.
- Literal credentials encountered while parsing an explicitly authorized source must remain in memory only for the minimum processing period and must never be logged, displayed, snapshotted, proposed, or persisted by MOO.
- Literal secrets found in linked or imported configurations are not reusable by default. The server remains non-runnable until the user supplies an environment or approved credential-provider reference.

### Project and runtime state

- `moo.config.json` contains commit-friendly project configuration.
- `moo.profile.json` contains an optionally committed quality profile.
- `moo/proposals/` contains reviewable remediation artifacts and may be committed.
- `.moo/cache/` and `.moo/runs/` are disposable local state and ignored by default.
- `.moo/reports/` is ignored by default and exported explicitly for CI or review.
- `.moo/snapshots/` is ignored by default because capability metadata may be sensitive. Retention and sanitized export or commit are explicit opt-ins; canonical content remains deterministic regardless of filename or observation time.

### Evidence vocabulary

- **Configured** means a server definition was found in a supported source or MOO configuration.
- **Reachable** means MOO successfully contacted and discovered it during a specific run.
- **Governed** means an invocation passed through MOO policy, beginning in 0.2.
- **Observed usage** means MOO saw an invocation directly or received it through an explicit telemetry integration.
- Configuration detection alone must never be presented as evidence of actual usage.

### Usability

- A non-specialist should reach a useful first catalog within five minutes for a supported existing client configuration.
- Connecting the first server must not require JSON editing or prior knowledge of MCP transports.
- Every onboarding failure should explain the next corrective action.
- Advanced protocol and configuration terminology should be progressively disclosed.

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
- Process cleanup, timeout, malformed response, and partial-discovery tests.
- Integration tests using real local MCP interactions.
- Tool-name collision tests.
- Snapshot and report-format tests.
- Tests proving timestamps and run metadata do not change canonical snapshot fingerprints.
- Tests proving pasted commands are executed as an executable plus arguments without implicit shell interpretation.
- Tests proving unrelated parent-process environment secrets are not inherited by stdio servers.
- Tests for hostile terminal metadata, ANSI escapes, oversized and deeply nested schemas, and malformed or repeated pagination cursors.
- Tests proving incomplete server enumeration produces unknown status rather than false removal.
- Tests for structured capability identity and reversible CLI encoding.
- End-to-end demonstration of onboarding, discovery, drift, a finding, and a reviewed remediation proposal.

Policy, routing, argument-validation, direct-call bypass, confirmation, and fail-closed invocation tests become required in 0.2.

## 12. Demonstration environment

Provide three local MCP servers with intentionally varied quality:

- CRM server with read and destructive customer operations.
- Documents server with strong descriptions and schemas.
- Operations server with vague descriptions, broad objects, collisions, and inconsistent annotations.

The 0.1 demo must show multi-server discovery, a normalized catalog, an actionable finding, an overlap candidate, snapshot drift, and a reviewed remediation proposal.

## 13. Explicit non-goals for release 0.1

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
  reporting/       Terminal and JSON output; later SARIF-compatible output
  chat/            Optional natural-language interface over application APIs
  cli/             Commands and user interaction
examples/
tests/
docs/
```

These are package boundaries inside a modular monolith, not services. Do not create empty modules before their behavior exists.

## 15. Delivery sequence

1. Establish repository standards, toolchain, CI, and architecture decisions.
2. Implement guided initialization, paste-based server addition, and display discovered tools.
3. Connect to multiple servers and preserve partial failures.
4. Normalize the catalog and create stable snapshots.
5. Add fingerprints and snapshot diffing.
6. Implement the rule engine and initial deterministic rules.
7. Add profiles, suppressions, and reports.
8. Add duplicate and structural-overlap candidates.
9. Add reviewed finding-to-overlay and finding-to-policy proposal workflows.
10. Complete demonstration servers and end-to-end tests.
11. Validate against real estates.
12. In 0.2, add minimal policy decisions, explicit routed calls, and optional `mcp-tool-control` integration.
13. In 0.3, add the focused chat/REPL interface.

## 16. Release 0.1 acceptance criteria

Release 0.1 is complete when:

- Three configured servers can be discovered in one run.
- A new user can initialize configuration and add or test a server without hand-writing JSON.
- A user can consent to detection in at least one supported MCP client and selectively import discovered server definitions.
- An existing supported client configuration can be imported with a preview and without copying secrets.
- Tool-name collisions do not overwrite or ambiguously identify capabilities.
- Repeated discovery creates stable fingerprints.
- Meaningful schema changes appear in deterministic diffs.
- Three to five useful deterministic rules are implemented and tested.
- Findings contain evidence, severity, confidence, and remediation.
- A versioned quality profile changes findings, severities, or CI thresholds without modifying an upstream server.
- A suitable finding can produce a reviewable overlay or policy proposal with provenance.
- Every proposal records the basis fingerprints required by its remediation layer, and stale-basis tests cover schema, metadata, source-definition, and complete-record proposals.
- Reports serve humans and CI automation.
- Core functionality works without an LLM or hosted service.
- Setup and demonstration instructions work from a clean checkout.

## 17. Validation criteria

Before committing to a production gateway or formal standard, validate that:

- At least three independently operated real MCP estates can be assessed, with at least five participating maintainers in total and at least 100 human-reviewed finding decisions across the pilots.
- High-severity findings achieve at least 70% maintainer acceptance in pilots.
- At least half of a minimum sample of 20 deterministic remediation suggestions are useful.
- Snapshot comparison catches a meaningful regression.
- One organization requests CI integration.
- MOO reveals information not already supplied by the organization's gateway or scanner.

For these gates, a finding is "accepted" only when a maintainer judges its evidence correct and its issue actionable without a substantive correction. A remediation is "useful" only when a maintainer would adopt it or says it materially shortens the work needed to resolve the finding. The pilot protocol must preserve these definitions, recruit at least two maintainers per estate where practical, and define sampling before results are collected. It must measure correctness and false-positive rate per rule, remediation usefulness, time to first catalog, time to triage, meaningful regressions found, and unique information beyond existing tools. Labels and denominators must be preserved with the results.

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

No MOO-internal schema should be presented as a standard until at least two independent producers or consumers have implemented it, concrete interoperability failures are documented, compatibility tests and versioning rules exist, a neutral schema namespace is available, and a public RFC/change process is defined.

## 19. Open decisions

- Supported protocol-version matrix for the first release.
- Package manager and Node.js support policy.
- Evidence required before adding YAML or another human-authored configuration format after 0.1.
- JSON Schema normalization and fingerprint canonicalization.
- Files only versus SQLite for local history.
- Chat provider abstraction for the planned 0.3 release.
- Confirmation UX for interactive and non-interactive execution.
- Severity, confidence, and evidence thresholds for the selected initial five-rule pack, including calibration criteria for confirmed versus heuristic findings.
- Final project, package, and CLI naming after namespace and trademark checks.
- Which MCP client configuration formats are supported by the first import adapters.
- Whether the first supported source defaults to linked, imported, or temporary non-retained mode.
- How duplicate definitions found across multiple clients are identified without silently merging distinct servers.
- The exact merge rules and precedence for original metadata, organization overlays, and server-provided instructions.
- Whether MCP OAuth-client support belongs in 0.2 or a later release; 0.1 may require external credential references.
- Whether `mcp-tool-control` integration is demonstrated in 0.2 or a later milestone.
- Exact numeric Streamable HTTP timeout, size, stream, and redirect caps within the secure behavior required above.
- Whether release benchmark budgets need additional platform-specific profiles beyond the defined reference environment.
- Final name and namespace clearance is a pre-alpha publication gate because it controls package names, schema URLs, and public documentation identity.
