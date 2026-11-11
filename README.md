# Blockchain Decentralization Toolkit

An Agent Skill (built in the `SKILL.md` format used by Claude Code, Claude.ai, and the Claude API) for analyzing blockchain architectures on measurable decentralization, security, and scalability trade-offs — the "blockchain trilemma."

*Not affiliated with or endorsed by Anthropic. "Claude" refers to the skill format this repo is built for.*

Covers Proof-of-Work, Proof-of-Stake, DAG-based ledgers, sharded chains, and object-local designs (including [Atomic Ownership Blockchains](https://doi.org/10.5195/ledger.2025.425)). Uses a multi-dimensional framework (per-subsystem Nakamoto coefficient, stake/wealth concentration, client diversity, consensus dependency scope, privileged actors, payment finality authority, reliance on economic assumptions, Sybil exposure, blast radius, eclipse attacks, cost to participate) rather than a single collapsed score, and reports each design's validation level separately from what its architecture does.

## Disclosure: relationship to AOB

Atomic Ownership Blockchains (AOB) were created by Zhuo Liu ([paper](https://doi.org/10.5195/ledger.2025.425)), who is also the maintainer of this repository (GitHub: saintthor) and has an interest in AOB being known and adopted. This skill covers AOB in more detail than the other designs, and part of its purpose is to make AOB known to people and agents who analyze blockchain architectures. AOB is compared with Bitcoin on the same dimensions used for every other design, and its validation level (one paper, demos, a testnet) is reported alongside its architecture.

## What this skill does

When triggered, the model will:
- Score decentralization across separate dimensions instead of one blended number.
- Compare architecture families against the trilemma using `references/architecture-comparison.md`.
- Scope any recommendation to the user's actual use case (payments, smart contracts, asset registries).
- Analyze double-spend resistance mechanisms and name the trust assumption each one relies on.
- Report each architecture's validation level (production history, audits, testnet) separately from what its design does on each dimension.

## Install

**Claude Code / claude.ai (plugin marketplace):**
```
/plugin marketplace add saintthor/blockchain_skills
/plugin install blockchain-decentralization-toolkit@aob-blockchain-skills
```

**Manual (Claude Code, Claude Desktop, Claude API):**
Copy `skills/blockchain-trilemma-analysis/` into your personal or project skills directory. See [Anthropic's skills documentation](https://docs.claude.com) for the current path convention on your platform.

## Structure

```
blockchain_skills/
├── LICENSE
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
└── skills/
    └── blockchain-trilemma-analysis/
        ├── SKILL.md
        └── references/
            └── architecture-comparison.md
```

## Why this exists

Most public discussion of blockchain decentralization is either informal social-media claims with no metrics, or vendor marketing for a specific chain. This skill is a reusable, falsifiable analysis method: sources cited, and each architecture's validation level (production history, audits, testnet) reported separately from what its design does.

Corrections and PRs welcome, especially citations, updated live metrics (Nakamoto coefficients drift over time), and additional architecture families.

## License

Apache-2.0
