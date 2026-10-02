# NOVA Skill Contract

**Status:** v1, normative

---

## Schema Enforcement

Every invariant in this document is enforced by [`skill-contract.schema.json`](skill-contract.schema.json) at registration time. Specifically:

| Invariant | Enforcement |
|---|---|
| #1 No implicit authority | `authorization.permissions` required, non-empty |
| #2 Trusted identity only | `ownership` required with owner_type/owner_id |
| #3 No direct rail access | `execution.provider` required, `rails` restricted to `cex` / `dex` |
| #4 Closed, validated parameters | input/output schemas must be `additionalProperties: false` and contain no remote `$ref` |
| #5 Side effects are explicit | `write` side_effect requires `idempotency: required` AND `approval.mode` in (`conditional`, `always`) |
| #6 Versioned behavior | `identity.version` must match SemVer pattern |
| #7 Auditable outcomes | `audit.include_standard_fields` must be `true` |
| #8 Evidence before enablement | `evaluation` requires non-empty routing, parameters, security, execution lists |

### Impact classification

Every manifest declares `impact.class`:
- `read` or `stateful_read` → no approval required
- `write`, `financial`, or `destructive` → approval required (`conditional` or `always`)

This is enforced by a `oneOf` constraint in the schema. A `write` skill with `approval.mode: "none"` fails validation.

### Backoff constraints

`retry.backoff` must satisfy:
- `initial_ms` in [50, 10000]
- `max_ms` in [100, 60000]
- `multiplier` in (1, 5] — exponential, not constant
- `max_ms >= initial_ms` (validated by the registry, since JSON Schema cannot compare siblings)

---

  
**Applies to:** every registered NOVA skill, regardless of agent, provider, or execution rail

This contract is the boundary between an agent's reasoning and an operation NOVA may perform. A skill is a versioned, schema-bounded capability; it is not an authorization grant. The registry validates the manifest, Governance authorizes each invocation, the Bridge executes it, and Evals verify its declared behavior.

The machine-readable manifest format is [`skill-contract.schema.json`](skill-contract.schema.json). A manifest is not publishable unless it validates against that schema and the semantic requirements below.

## Invariants

1. **No implicit authority.** Registration does not grant access. Governance evaluates the authenticated principal, tenant, requested permission, policy version, limits, and approval state on every invocation. Missing, stale, or indeterminate policy decisions deny execution.
2. **Trusted identity only.** The Bridge derives tenant, organization, and actor from authenticated server-side context. They are not accepted from model-generated skill input. Every invocation is checked against the skill's declared ownership scope; no wildcard tenant or cross-tenant fallback is allowed.
3. **No direct rail access.** Agents and models call skills, not CEX/DEX providers. The Bridge is the only execution path and applies tenant isolation, credential-vault access, provider routing, and audit recording.
4. **Closed, validated parameters.** Inputs and outputs are JSON objects described by JSON Schema Draft 2020-12. Reject unknown fields unless a schema explicitly defines a typed extension map. Validate inputs before authorization and again at the execution boundary; validate provider results before returning them.
5. **Side effects are explicit.** A skill declares whether it is read-only or writes state. Write operations require a caller-provided idempotency key, and retries must not duplicate an effect. Destructive or financially material actions declare approval requirements and applicable limits.
6. **Versioned behavior.** `skill_id` is stable and namespaced. A published version is immutable: any change to parameters, permissions, limits, approval behavior, provider operation, or observable result requires a new SemVer version. Deprecate old versions; do not silently replace them.
7. **Auditable outcomes.** Every attempt, including denials and failures, creates an audit event with the standard metadata below. Secrets, credentials, and unredacted sensitive values are never written to audit logs.
8. **Evidence before enablement.** All four evaluation groups are required and must pass the configured release threshold before a version can be enabled. Tests are versioned with the skill and run again for changes to the skill, provider adapter, policy, or model routing that can affect it.

## Manifest fields

| Field | Contract |
| --- | --- |
| `identity` | Stable namespaced `skill_id`, human-readable `name`, SemVer `version`, and lifecycle `status`. A version identifies one immutable behavior. |
| `ownership` | Declares who publishes/owns the skill and its `platform`, `tenant`, `organization`, or `actor` scope. Tenant-specific ownership includes its tenant ID. Runtime caller identity is separate and comes from trusted context. |
| `input`, `output` | JSON Schema Draft 2020-12 object schemas. They define required fields, types, bounds, enums, and sensitive-field handling. Schemas must not contain executable code or resolve remote references at runtime. |
| `authorization` | One or more namespaced permissions, plus an optional policy reference. Permissions are requests for Governance to evaluate, never capabilities embedded in the skill. |
| `risk` | Risk classification and a policy reference. Declared limits are machine-readable name/value/unit tuples; Governance remains responsible for enforcing them and any stricter tenant policy. |
| `approval` | `none`, `conditional`, or `always`. Conditional and always-required approval use a Governance policy reference. A skill cannot approve its own invocation. |
| `execution` | One or more rails, a provider and operation, and an explicit `read` or `write` side-effect classification. Write operations require idempotency-key support. |
| `retry` | Bounded attempts and retryable transient classes only. Never retry validation, authorization, policy, or approval denials. Writes retry only with the same idempotency key and provider support. |
| `audit` | Enables standard fields and may name additional metadata. Additional metadata must be redacted, bounded, and non-secret. |
| `evaluation` | Non-empty references for routing, parameter, security, and execution tests. Test fixtures must not contain production credentials or live-account authority. |

## Invocation requirements

The Bridge assigns a unique `request_id` and resolves the immutable skill version. The model may supply only fields declared in `input.schema`; tenant, organization, actor, credentials, policy decisions, and approval results are trusted server-side context, never model parameters.

For each attempt, the execution path is:

1. Resolve caller and tenant context; reject a mismatch with `ownership`.
2. Validate the request against the selected input schema.
3. Ask Governance to authorize every declared permission and enforce tenant policy, risk limits, and spending limits.
4. Obtain and verify required human approval before dispatch. Approval is bound to the request, skill version, parameters (or their stable digest), tenant, actor, and expiry; any material parameter change invalidates it.
5. Execute only through the declared provider operation and rail, using credentials fetched by the Bridge from the vault.
6. Validate the result against the output schema, record the outcome, and return a structured result.

On any denial, timeout, provider error, or invalid result, fail closed: do not fall back to a broader permission, another tenant, an undeclared provider operation, or an unapproved side effect. Return a stable error code and a safe message; keep provider secrets and sensitive internals out of the response.

## Retry and idempotency

`max_attempts` includes the first attempt. Retry only the declared transient classes, with bounded exponential backoff. Respect provider rate limits and any stricter provider retry-after value. Do not retry an ambiguous write outcome unless the provider can resolve the original idempotency key. Exhausted retries produce one final failed result and an audit event for each attempt.

## Audit baseline

Every invocation audit event records at minimum: `request_id`, timestamp, `skill_id`, skill version, tenant ID, organization ID when present, actor ID, authorization decision and policy version, risk decision, approval decision/reference, provider and operation, attempt number, outcome/error code, and model/prompt attribution when a model initiated or materially shaped the request. Record a redacted parameter summary or stable digest, not secrets or unrestricted raw payloads. Audit storage is append-only and access-controlled.

## Registration and release

The registry rejects a manifest if it fails JSON Schema validation, references unsupported schema drafts, omits any required eval group, declares an unbounded retry policy, or violates a semantic invariant in this document. A version is enabled only after security and execution tests pass in a non-production environment and required governance policies are configured. The registry stores the manifest, schema, eval references/results, publisher, and immutable version digest. Runtime invocations pin that digest for their full lifetime.

## Minimal manifest example

This example is illustrative; replace test references and policies with registered NOVA resources before enabling it.

```json
{
  "$schema": "./skill-contract.schema.json",
  "identity": {
    "skill_id": "market.ticker",
    "name": "Ticker",
    "version": "1.0.0",
    "status": "active"
  },
  "ownership": {
    "owner_type": "platform",
    "owner_id": "nova"
  },
  "input": {
    "schema": {
      "$schema": "https://json-schema.org/draft/2020-12/schema",
      "type": "object",
      "properties": {
        "symbol": { "type": "string", "minLength": 1, "maxLength": 32 }
      },
      "required": ["symbol"],
      "additionalProperties": false
    }
  },
  "output": {
    "schema": {
      "$schema": "https://json-schema.org/draft/2020-12/schema",
      "type": "object",
      "properties": {
        "symbol": { "type": "string" },
        "price": { "type": "number", "exclusiveMinimum": 0 },
        "observed_at": { "type": "string", "format": "date-time" }
      },
      "required": ["symbol", "price", "observed_at"],
      "additionalProperties": false
    }
  },
  "authorization": {
    "permissions": ["market.ticker.read"]
  },
  "risk": {
    "level": "low",
    "policy_ref": "policy.market-read.v1",
    "limits": []
  },
  "approval": {
    "mode": "none"
  },
  "execution": {
    "rails": ["cex"],
    "provider": "bybit",
    "operation": "market.ticker",
    "side_effect": "read",
    "idempotency": "not_applicable"
  },
  "retry": {
    "max_attempts": 3,
    "retry_on": ["timeout", "rate_limit", "transient_provider"],
    "backoff": {
      "initial_ms": 200,
      "max_ms": 2000,
      "multiplier": 2
    }
  },
  "audit": {
    "include_standard_fields": true,
    "additional_fields": []
  },
  "evaluation": {
    "routing": ["evals/market.ticker/routing/basic"],
    "parameters": ["evals/market.ticker/parameters/valid-and-invalid"],
    "security": ["evals/market.ticker/security/tenant-and-permission"],
    "execution": ["evals/market.ticker/execution/bybit-read"]
  }
}
```
