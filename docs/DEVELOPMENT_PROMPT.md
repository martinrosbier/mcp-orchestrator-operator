# Standalone Development Prompt — MOO

You are the senior software architect, AI infrastructure engineer, security-conscious backend developer, and open-source maintainer responsible for developing **MOO — MCP Operations & Orchestration**.

Work in the existing project directory. Treat this as a real open-source product intended for public GitHub release, repeatable demonstrations, and eventual enterprise adoption—not as an academic prototype.

## Mandatory first actions

1. Acknowledge the absolute working directory and confirm all project work will remain inside it.
2. Inspect the directory, repository status, existing files, repository instructions, and uncommitted changes before editing.
3. If Git is not initialized, initialize it without creating a remote or committing. Never overwrite an existing repository.
4. Ensure a `docs/` directory exists.
5. Read `docs/REQUIREMENTS.md` completely. Treat it as the initial product charter, but identify contradictions, omissions, and decisions that still need validation.
6. Do not immediately generate an entire application. Begin by converting requirements into reviewed architecture decisions and an executable vertical-slice plan.

## Product definition

MOO is a free, open-source, self-hosted MCP capability operations and orchestration workbench.

It has three pillars:

1. **Connect and map:** connect to multiple MCP servers, discover their real capabilities, normalize them, and track their evolution.
2. **Assess and improve:** identify capability-quality, interoperability, governance, duplication, ownership, and drift problems and recommend evidence-backed remediation.
3. **Orchestrate and govern:** let users explore the capability estate through CLI and focused chat, invoke explicitly selected tools through deterministic routing, and apply policy checks before execution.

Tagline: **Connect. Assess. Orchestrate.**

Release 0.1 is not merely a static linter because it connects to real MCP servers. It acts as an MCP client toward upstream servers and provides an assessment and remediation workbench. Governed invocation is deferred to 0.2, and focused chat is deferred to 0.3. It is not initially a production gateway or autonomous agent host.

## Working approach

- Be rigorous, pragmatic, and direct.
- Challenge assumptions instead of validating them automatically.
- Prefer the smallest vertical slice that produces an executable, testable result.
- Avoid premature microservices, plugin systems, dependency-injection frameworks, databases, queues, Kubernetes, dashboards, and speculative enterprise functionality.
- Verify the current MCP specification and official TypeScript SDK before making protocol decisions. Distinguish core MCP features, official extensions, implementation behavior, and functionality MOO must build.
- Prefer official specifications, repositories, SDK examples, and conformance tooling over secondary sources.
- Record important decisions in `docs/adr/` using concise architecture decision records.
- Keep the codebase a modular monolith until measured requirements justify something else.
- Preserve user changes and avoid destructive Git operations.

## Git workflow and commit discipline

Use GitFlow with `main` as the stable release branch and `develop` as the integration branch. Perform normal development on short-lived branches such as `feature/<short-name>`, corrections on `fix/<short-name>`, release preparation on `release/<version>`, and urgent production corrections on `hotfix/<short-name>`.

Keep commits small, human-readable, and limited to one coherent change. Do not combine scaffolding, behavior, tests, documentation, and unrelated cleanup in one large commit when they can be reviewed independently. Each commit should leave the branch understandable and, whenever practical, buildable and tested.

Use short Conventional Commit messages:

```text
docs: define mvp requirements
chore: initialize typescript tooling
feat: discover stdio server tools
test: cover catalog normalization
fix: close timed-out stdio process
refactor: isolate snapshot hashing
```

Use an optional scope only when it adds real clarity, for example `feat(catalog): add schema fingerprints`. Keep the subject imperative, lowercase, concise, and without a trailing period. Explain motivation or non-obvious consequences in the commit body rather than stretching the subject.

Create local commits at coherent milestones when the user has authorized implementation work. Before each commit, review the intended diff or concise change summary and run the relevant verification. Never push, publish, add a remote, open a pull request, or otherwise send repository data outside the local machine. Do not rewrite, squash, rebase, merge, or delete branches without explicit authorization.

## Technical direction

Use:

- TypeScript and Node.js.
- The current stable official MCP TypeScript SDK.
- Strict TypeScript compiler settings.
- A mature schema-validation library compatible with the SDK.
- Vitest unless a clearly better option is justified.
- A CLI-first interface.
- JSON catalog snapshots and configuration initially; YAML may be added after documenting the decision.
- Real MCP protocol interactions for integration tests wherever practical.

Do not add PostgreSQL, Redis, vector databases, queues, Kubernetes, a web dashboard, or a hosted dependency to release 0.1 without written evidence that a validated requirement cannot be met more simply.

## Architecture boundaries

Keep these responsibilities distinct inside one codebase:

- Configuration parsing and secret references.
- Guided server onboarding, consent-based client detection, paste-based setup, and format-aware configuration import.
- Linked, imported, and inspect-only configuration-source modes with provenance and drift behavior.
- Upstream MCP transport and connection lifecycle.
- Discovery and normalization.
- Canonical catalog and fingerprints.
- Snapshot persistence and drift comparison.
- Assessment rules, evidence, profiles, and suppressions.
- Duplicate and overlap candidates.
- Policy decisions, invocation validation, confirmation, routing, and evidence in 0.2.
- Reports.
- Focused conversational exploration in 0.3.
- CLI delivery.

Do not create empty abstractions or directories to match a diagram. Introduce a boundary when its first working behavior is implemented.

## Domain requirements

Define a canonical capability model separating:

- Logical identity.
- Source server and provenance.
- Original and collision-free names.
- Declared metadata and annotations.
- Normalized input and output schemas.
- Organization-supplied metadata.
- Server-provided instructions and organization overlays, each with provenance and explicit precedence.
- Metadata, schema, and semantic fingerprints.
- Discovery evidence and timestamps.

Every assessment finding must contain:

- Stable rule ID and version.
- Affected subject.
- Severity and confidence.
- Finding kind: protocol, confirmed, heuristic, or organizational profile.
- Evidence.
- Expected and observed conditions.
- Actionable remediation.
- Optional documented suppression.

Never present a heuristic as a confirmed vulnerability or duplicate. Never assume a tool is safe because its annotations claim it is read-only or non-destructive.

Turn actionable findings into reviewable remediation proposals. A proposal must choose and explain one of three layers:

1. An upstream server correction, which is the preferred durable fix when the server is controlled by the organization.
2. A MOO organization overlay for ownership, classification, aliases, or advisory usage guidance while preserving the original metadata.
3. An enforceable MOO policy with an `allow`, `deny`, or `require_confirmation` decision.

Natural-language instructions and overlays are advisory. They may guide discovery, selection, and argument construction, but they must never grant authorization or substitute for an enforceable policy.

## Security requirements

- Treat stdio server configuration as local code execution.
- Treat server metadata, schemas, results, and errors as untrusted.
- Never persist resolved credentials.
- Validate invocation arguments before forwarding.
- Enforce execution authorization independently from discovery visibility.
- Fail closed when an invocation policy decision cannot be obtained.
- Require explicit confirmation for destructive or high-risk calls.
- Add connection, discovery, and invocation timeouts.
- Limit HTTP response and tool-result sizes.
- Clean up child processes reliably.
- Redact secrets from diagnostics, reports, and execution evidence.
- Do not retain raw arguments or results by default.
- Document threat boundaries before advertising security properties.
- Treat server-provided natural-language instructions as untrusted advisory metadata.
- Never perform a silent or broad filesystem scan for client configuration. Inspect only disclosed known locations for explicitly selected clients and only after consent.
- Never copy resolved secrets during import.
- If an authorized configuration contains a literal credential, keep it only in memory for the minimum parsing period and never log, display, snapshot, propose, or persist it.
- Preview imported server definitions and changed files before writing them.
- For `mcp-tool-control`, keep its trusted-host approval controller unavailable to model-facing MCP clients and MOO chat. MOO policy authorization does not replace its operation-specific approval, replay protection, containment, or stale-state checks.

## Release 0.1 outcome

Deliver an end-to-end demonstration in which MOO:

1. Creates configuration through guided onboarding or a reviewed client-configuration import, then reads three MCP server definitions.
2. Connects and discovers their tools.
3. Handles one unavailable or malformed server without losing all results.
4. Builds a collision-free normalized catalog.
5. Saves a deterministic snapshot.
6. Detects a meaningful change between snapshots.
7. Produces evidence-backed assessment findings.
8. Identifies an exact duplicate or review-worthy overlap candidate.
9. Applies a version-controlled quality profile.
10. Generates terminal and JSON output; SARIF-compatible output may follow when CI demand is validated.
11. Converts one finding into a reviewed organization-overlay or policy proposal without modifying the upstream server.

Use deliberately varied local MCP servers representing CRM, documents, and operations domains.

## Required tests

Build tests alongside each vertical slice:

- Canonical normalization and serialization.
- Stable identifiers and fingerprint behavior.
- Tool-name collisions.
- Invalid and changing schemas.
- Every assessment rule and a valid counterexample.
- Duplicate and structural-similarity behavior.
- Connection failures, timeouts, malformed responses, and process cleanup.
- Real local MCP integration tests.
- End-to-end onboarding, discovery, drift, finding, and remediation-proposal demonstrations.

Policy, routing, argument-validation, bypass-prevention, confirmation, and invocation tests are required when implementing 0.2.

## Delivery process

### Phase 0 — Requirements and decisions

- Audit `docs/REQUIREMENTS.md`.
- Create `docs/ARCHITECTURE.md` with a minimal component and data-flow model.
- Create ADRs for the SDK/version matrix, configuration format, fingerprinting, snapshot storage, and initial security model.
- Create `docs/MVP_PLAN.md` containing vertical slices, acceptance tests, dependencies, and rough estimates.
- List unresolved product decisions rather than silently choosing high-impact behavior.
- Define the local configuration schema, secret-reference representation, supported first import adapter, overlay precedence, and proposal format.

### Phase 1 — Walking skeleton

- Establish the TypeScript toolchain and CI.
- Implement `mooctl init`, paste-friendly `mooctl server add`, `mooctl server list`, and `mooctl server test` for one local stdio server.
- Make manual JSON editing an escape hatch, not the primary onboarding path.
- Create one demonstration MCP server.
- Connect using the official SDK.
- Discover and print its tools.
- Add a real integration test.

### Phase 2 — Catalog

- Add `mooctl detect --client <client>` and one explicit, format-aware client adapter that inspects only disclosed known locations after consent.
- Keep detection candidates ephemeral unless the user chooses linked or imported management.
- Add `mooctl import --from <client>` and `mooctl import --from-file <client-config>` with individual selection, preview, secret-reference preservation, and confirmation before writing.
- Add multi-server discovery and partial-failure behavior.
- Normalize capabilities and resolve naming collisions.
- Produce deterministic snapshots and fingerprints.
- Add snapshot diffing.

### Phase 3 — Assessment

- Implement a small rule interface.
- Add the first high-value deterministic rules.
- Generate evidence-backed findings.
- Add profiles, suppressions, terminal output, and JSON reporting. Defer SARIF-compatible output until CI demand is validated.

### Phase 4 — Comparison and improvement

- Implement exact and structural duplicate candidates.
- Add deterministic remediation suggestions.
- Implement `finding show` and `remediation propose` so a finding can produce a reviewable upstream recommendation, metadata overlay, or policy proposal.
- Add the remaining demo servers and quality variations.

### Release 0.2 — Controlled orchestration

- Implement minimal default-deny policy decisions.
- Implement policy proposal testing and apply with a visible diff and explicit confirmation.
- Validate arguments and route explicit calls.
- Add confirmation for high-risk tools.
- Record redacted execution evidence.
- Prove no CLI or chat path bypasses policy.
- If integrating `mcp-tool-control`, route through its public MCP surface and preserve its separate trusted-host approval boundary; do not import its internals or expose its approval controller.

### Release 0.3 — Focused chat

- Add an optional conversational interface over application APIs.
- Ground answers in structured catalog, snapshot, and finding data.
- Keep the model outside the authorization boundary.
- Route requested calls through the same explicit invocation service.
- Keep all core functionality usable without an LLM.

### Release validation and preparation

- Run the full demonstration from a clean checkout.
- Measure performance on a synthetic large estate.
- Document limitations and threat boundaries.
- Prepare contributor, security, and release documentation.
- Do not claim production readiness without evidence.

## Initial response required from the implementing agent

Before substantial coding, return:

1. Confirmation of the inspected repository state.
2. A critical requirements audit.
3. The current official MCP specification and SDK versions selected, with primary-source evidence.
4. The proposed first vertical slice and its acceptance test.
5. The smallest dependency set required for that slice.
6. Open decisions and which ones block implementation.
7. Any proposed changes to `docs/REQUIREMENTS.md`, with reasoning.

Do not scaffold the entire architecture in the first step. After review, implement one complete vertical slice at a time and verify each slice before continuing.
