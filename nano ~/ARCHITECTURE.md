# NOVA — Architecture Spec v1

**Frozen:** 2026-10-02
**Next review:** 2027-01-02
**Status:** Locked. Every section below is frozen unless a bug requires change.

---

NOVA is a governed autonomous trading platform where AI agents reason, skills execute capabilities, bots operate continuously, and CEX/DEX actions pass through a controlled Bridge, policy, approval, audit, and evaluation layer.

---

## The Layers (what each one does)

| Layer | Job |
|---|---|
| **AGENTS** | reason |
| **SKILLS** | act |
| **BOTS** | operate |
| **GOVERNANCE** | control |
| **BRIDGE** | execute |
| **EVALS** | verify |
| **CEX + DEX** | provide the rails |

---

## 01_PLATFORM

```
01_PLATFORM
│
├── Identity
│   ├── Users
│   ├── Organizations
│   ├── Tenants
│   ├── Actors
│   └── Roles / Permissions
│
├── Orchestrator
│   ├── Intent detection
│   ├── Agent routing
│   ├── Skill routing
│   ├── Parameter extraction
│   ├── Ambiguity handling
│   └── Rejection handling
│
└── AI Model Layer
    ├── Model adapter
    ├── Prompt / version control
    ├── Model attribution
    └── Fallback / retry
```

---

## 02_AGENTS

```
02_AGENTS
│
├── Trading Agent
├── Portfolio Agent
├── Risk Agent
├── Research Agent
├── Strategy Agent
├── Bot Agent
├── Wallet / Web3 Agent
└── Jobs / Operations Agent
```

---

## 03_SKILLS

```
03_SKILLS
│
├── MARKET
│   ├── ticker
│   ├── candles
│   ├── orderbook
│   ├── funding
│   ├── open_interest
│   ├── instruments
│   └── market_status
│
├── ACCOUNT
│   ├── balance
│   ├── positions
│   ├── margin
│   ├── account_info
│   └── transaction_history
│
├── TRADING
│   ├── order
│   ├── cancel_order
│   ├── amend_order
│   ├── close_position
│   ├── leverage
│   ├── take_profit
│   ├── stop_loss
│   └── ...
│
├── PORTFOLIO
│   ├── exposure
│   ├── allocation
│   ├── rebalance
│   ├── pnl
│   └── performance
│
├── RISK
│   ├── risk_check
│   ├── position_size
│   ├── exposure_limit
│   ├── drawdown
│   ├── liquidation
│   └── risk_policy
│
├── BOTS
│   ├── create
│   ├── configure
│   ├── start
│   ├── stop
│   ├── pause
│   ├── resume
│   ├── status
│   └── delete
│
├── RESEARCH
│   ├── fetch_url
│   ├── summarize
│   ├── market_research
│   └── data_analysis
│
├── WEB3 / DEX
│   ├── wallet
│   ├── token_info
│   ├── token_balance
│   ├── transfer
│   ├── swap
│   └── transaction
│
└── GOVERNANCE
    ├── authorize
    ├── approve
    ├── policy_check
    ├── audit
    └── tenant_check
```

---

## 04_EXECUTION

```
04_EXECUTION
│
├── CEX
│   └── Bybit
│       ├── Market
│       ├── Account
│       ├── Orders
│       ├── Positions
│       ├── WebSocket
│       └── Broker / Tier-3
│
├── DEX
│   ├── Base
│   ├── Wallet
│   ├── Token
│   ├── Transfer
│   └── Swap
│
└── NOVA BRIDGE
    ├── Skill execution
    ├── Tenant isolation
    ├── Vault
    ├── Bridge authentication
    ├── Provider routing
    ├── Retry / failure
    └── Execution audit
```

---

## 05_BOTS

```
05_BOTS
│
├── DCA
├── Grid
├── Momentum
├── Mean Reversion
├── Breakout
├── Trend Following
├── Rebalancing
├── Arbitrage
├── Signal / Webhook
└── Custom Agent Bot
```

---

## 06_GOVERNANCE

```
06_GOVERNANCE
│
├── Authentication
├── Authorization
├── Tenant Isolation
├── RBAC
├── Skill Permissions
├── Risk Policies
├── Spending Limits
├── Human Approval
├── Credential Vault
├── Immutable Audit
├── Model Attribution
└── Policy Versioning
```

---

## 07_EVALUATION

```
07_EVALUATION
│
└── NOVA EVALS
    │
    ├── Routing
    │   ├── trading
    │   ├── jobs
    │   ├── research
    │   ├── unrelated
    │   └── ambiguous
    │
    ├── Skills
    │   ├── ticker
    │   ├── balance
    │   ├── order
    │   ├── wallet
    │   └── transfer
    │
    ├── Security
    │   ├── wrong tenant
    │   ├── unauthorized skill
    │   ├── invalid skill
    │   ├── disabled skill
    │   └── approval required
    │
    ├── Execution
    │   ├── Bybit
    │   ├── Web3
    │   └── failure / retry
    │
    ├── Agent Behavior
    │   ├── correct agent
    │   ├── correct skill
    │   ├── correct parameters
    │   ├── rejection
    │   └── recovery
    │
    ├── Bots
    │   ├── creation
    │   ├── configuration
    │   ├── lifecycle
    │   └── execution
    │
    └── Regression
        ├── previous run
        ├── current run
        ├── delta
        └── automatic threshold
```

---

## 08_REGISTRY

```
08_REGISTRY
│
├── Skill Registry
├── Skill Versions
├── Agent Registry
├── Bot Registry
├── Policies
├── Model Registry
└── Capability Metadata
```

---

## 09_FRONTEND

```
09_FRONTEND
│
├── Dashboard
├── Trading Terminal
├── Bots
├── Agents
├── Skills
├── Portfolio
├── Risk
├── Approvals
├── Audit
└── System Health
```

---

## 10_OPERATIONS

```
10_OPERATIONS
│
├── Docker / Services
├── Deployment
├── Logging
├── Monitoring
├── Health Checks
├── Backups
├── Secrets
└── Incident Recovery
```

---

## 11_DOCUMENTATION

```
11_DOCUMENTATION
│
├── Architecture
├── API
├── Skills
├── Agents
├── Bots
├── Governance
├── Security
├── Evaluation
├── Deployment
└── Product
```

---

## The Skill Contract

Every skill in the platform must implement this contract. This is what allows NOVA to safely grow to 100+ skills.

```
SKILL
│
├── identity
│   ├── skill_id
│   ├── name
│   └── version
│
├── ownership
│   ├── tenant
│   ├── organization
│   └── actor
│
├── input
│   └── JSON schema
│
├── output
│   └── JSON schema
│
├── authorization
│   └── permissions
│
├── risk
│   ├── risk_level
│   └── limits
│
├── approval
│   └── required?
│
├── execution
│   ├── CEX
│   ├── DEX
│   └── provider
│
├── retry
│   └── policy
│
├── audit
│   └── required metadata
│
└── evaluation
    ├── routing test
    ├── parameter test
    ├── security test
    └── execution test
```

---

## Example Flow (Phase 4)

```
User:
"Run a BTC DCA strategy with my risk limit."
             ↓
        Trading Agent
             ↓
      Strategy Skills
             ↓
        Risk Agent
             ↓
        Governance
             ↓
       Approval Policy
             ↓
        NOVA Bridge
             ↓
           Bybit
```

---

## Project Status

```
NOVA PROJECT

Foundation              ██████████  DONE
CEX / Bybit             ██████████  DONE
DEX / Web3              ██████████  DONE
Bots                    ██████████  DONE
Skill Registry          ██████████  DONE
Orchestrator            ██████████  DONE
Bridge                  ██████████  DONE
Vault / Governance      ██████████  DONE
Agent routing           ██████████  DONE
NOVA Evals              ████████░░  FINALIZE
100-skill expansion     ███░░░░░░░  NEXT
Product tree            ███████░░░  FREEZE NOW
Documentation           █████░░░░░  FINAL PASS
```

---

## Repos

| Repo | Purpose |
|---|---|
| [nova-global-keys-](https://github.com/rizbot4u/nova-global-keys-) | 9-service trading platform |
| [nova_mcp](https://github.com/rizbot4u/nova_mcp) | MCP bridge — policy, vault, audit |
| [nova-skills-registry](https://github.com/rizbot4u/nova-skills-registry) | Multi-tenant skill catalog |
| [nova-orchestrator](https://github.com/rizbot4u/nova-orchestrator) | Multi-agent delegation layer |
| [nova-evals](https://github.com/rizbot4u/nova-evals) | Evaluation + regression detection |
| [job-agent](https://github.com/rizbot4u/job-agent) | Human-in-the-loop job application agent |

---

MIT
