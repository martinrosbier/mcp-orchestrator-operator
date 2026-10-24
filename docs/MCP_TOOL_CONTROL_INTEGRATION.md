# `mcp-tool-control` Integration Boundary

## Purpose

Beginning no earlier than release 0.2, MOO may connect to `mcp-tool-control` as an optional MCP server. The integration demonstrates how estate-wide discovery, assessment, routing, and governance compose with an executor that owns stronger transactional safeguards for local filesystem writes.

The products remain independently useful:

- MOO owns cross-server catalog, assessment, overlays, policy, routing, and operational visibility.
- `mcp-tool-control` owns inspect/plan/apply semantics, trusted-host approval, approval consumption, replay prevention, filesystem containment, stale-state checks, and its execution audit.

## Required call path

```text
operator or model
    -> MOO invocation request
    -> MOO argument validation and policy decision
    -> public mcp-tool-control MCP tool
    -> trusted host reviews and approves the exact operation
    -> mcp-tool-control validates and applies it
    -> correlated evidence returns to MOO
```

An MOO `allow` or `require_confirmation` result is necessary for routing but is not sufficient authorization for `mcp-tool-control` to apply a write. Its separate trusted-host approval remains mandatory.

## Prohibited integration shortcuts

MOO must not:

- expose the trusted approval controller to model-facing MCP clients or chat;
- synthesize approval credentials from model output;
- import private `mcp-tool-control` modules to bypass its public boundary;
- claim that MOO confirmation provides replay prevention, containment, or stale-state protection;
- place `mcp-tool-control` on every invocation path or make it a mandatory dependency.

## Catalog and assessment behavior

MOO discovers `mcp-tool-control` through the same protocol path as any other server and records original metadata with provenance. Organization overlays may add ownership, classifications, aliases, or usage guidance without rewriting the source declaration.

Assessment recommendations should distinguish:

- guidance that belongs in the upstream server;
- organization-specific metadata that belongs in a MOO overlay;
- routing controls that belong in MOO policy;
- transactional guarantees that remain the executor's responsibility.

## Audit correlation

Where supported, MOO should generate a correlation identifier and propagate it through the public invocation request. MOO records its policy decision and routing evidence. `mcp-tool-control` records its trusted approval and execution evidence. Reports may correlate the two records but must not imply that one replaces the other.

## Planned 0.2 MOO-owned acceptance criteria

- MOO can discover and assess the public tools without privileged controller access.
- A plan or inspection operation can be routed under MOO policy.
- MOO never exposes, synthesizes, or forwards privileged approval-controller access to a model-facing client.
- MOO cannot route around the executor's public MCP boundary.
- MOO and executor evidence can be correlated without storing secrets or raw sensitive payloads by default.

## Upstream compatibility prerequisites

- The selected `mcp-tool-control` version documents trusted-host approval, replay protection, containment, and stale-state behavior.
- Its own conformance tests pass independently of MOO.
- MOO records the exact upstream version and advertised capabilities used by the integration test.

## End-to-end integration evidence

- A write remains impossible until the trusted host approves the exact operation.
- Replaying an approval is rejected by `mcp-tool-control`.
- The evidence identifies which guarantees were observed end to end without implying that MOO implements the executor-owned safeguards.
