# NOVA Spec

**Frozen architecture spec and skill contract for the NOVA platform.**

## Documents

- [`ARCHITECTURE.md`](ARCHITECTURE.md) — 11-section frozen architecture. Every layer, every rail, every repo.
- [`SKILL_CONTRACT.md`](SKILL_CONTRACT.md) — the normative contract every NOVA skill must implement.

## What NOVA is

A governed autonomous trading platform where AI agents reason, skills execute capabilities, bots operate continuously, and CEX/DEX actions pass through a controlled Bridge, policy, approval, audit, and evaluation layer.

## The Layers

| Layer | Job |
|---|---|
| AGENTS | reason |
| SKILLS | act |
| BOTS | operate |
| GOVERNANCE | control |
| BRIDGE | execute |
| EVALS | verify |
| CEX + DEX | provide the rails |

## Repos

| Repo | Purpose |
|---|---|
| [nova-global-keys-](https://github.com/rizbot4u/nova-global-keys-) | 9-service trading platform |
| [nova_mcp](https://github.com/rizbot4u/nova_mcp) | MCP bridge — policy, vault, audit |
| [nova-skills-registry](https://github.com/rizbot4u/nova-skills-registry) | Multi-tenant skill catalog |
| [nova-orchestrator](https://github.com/rizbot4u/nova-orchestrator) | Multi-agent delegation layer |
| [nova-evals](https://github.com/rizbot4u/nova-evals) | Evaluation + regression detection |
| [job-agent](https://github.com/rizbot4u/job-agent) | Human-in-the-loop job application agent |

## Status

**Frozen:** 2026-10-02
**Next review:** 2027-01-02

MIT
