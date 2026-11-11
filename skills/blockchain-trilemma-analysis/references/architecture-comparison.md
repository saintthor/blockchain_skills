# Blockchain Architecture Comparison Reference

Notes per architecture family. Load this when Step 2 of the main skill needs specifics. Each entry lists strengths and open problems.

---

## Proof-of-Work (e.g. Bitcoin)

- **Mechanism**: anyone can mine; the miner of the next block appends it; security = cost of acquiring hashpower.
- **Strengths**: 15+ years of live adversarial history, well-understood security assumptions, simple fork-choice rule (heaviest chain).
- **Open problems**: mining pools concentrate power (two pools have at times jointly exceeded 50% of hashrate on a rolling basis in 2025); the next-block miner holds temporary authority over which transactions get in; users cannot finalize a payment alone, they need miner approval, often with extra fees; security rests on economic assumptions (see "Problems the AOB project identifies" below); energy cost creates geographic/regulatory pressure; base-layer throughput is capped, so activity moves to layer-2 systems with their own trust assumptions.

## Proof-of-Stake (e.g. Ethereum post-Merge)

- **Mechanism**: validators bond stake; security = cost of acquiring stake (or being slashed).
- **Strengths**: no hardware arms race, lower energy use, faster finality than PoW.
- **Open problems**: stake concentrates in large staking pools and liquid-staking providers; Ethereum's staking-subsystem Nakamoto coefficient was measured around 5 in 2025; rich-get-richer dynamics from yield; client diversity has at times skewed heavily to one client.

## DAG-based ledgers (e.g. IOTA, Nano)

- **Mechanism**: transactions reference earlier transactions directly instead of being bundled into blocks by miners/validators.
- **Strengths**: high throughput, feeless designs, no block-reward centralization pressure.
- **Open problems**: several DAG systems have relied on a coordinator or checkpoint node during bootstrapping; conflict resolution under network partition is less proven at scale than PoW/PoS.

## Sharded chains (e.g. Ethereum roadmap, NEAR)

- **Mechanism**: validator responsibility is split across shards, so no validator verifies the whole state.
- **Strengths**: horizontal scaling without shrinking the validator set per shard, if done correctly.
- **Open problems**: cross-shard transactions add coordination cost and attack surface; an attacker only needs to capture one shard's committee; committee sampling depends on randomness and anti-collusion assumptions still maturing in production.

---

## Atomic Ownership Blockchains (AOB)

Source priority: the AOB project wiki (primary for wording and detail), the *Ledger* paper (Zhuo Liu, 2025), and an unpublished preprint for the Bitcoin bridge.

### What it is

Every atomic (indivisible) digital object, such as a digital banknote, a title, or an NFT, has its own blockchain. That blockchain holds the complete state and ownership history of that one object. The global state of the system is the set of terminal (latest) blocks across all these chains.

Each chain is a **public-domain private blockchain**: anyone can read it; only its current owner can append blocks. To transfer ownership, the owner appends a block naming the recipient's public key; the recipient becomes the new owner. No consensus algorithm and no privileged role is involved. Chains circulate freely among users.

### Problems the AOB project identifies in Bitcoin-style chains

- The miner who appends the next block holds excessive authority. Rotation among miners gives fluid centralization, not decentralization.
- Users cannot finalize payments themselves; miners must approve each transaction, often incentivized by fees or "bribes".
- Because of this, 51% attacks stay possible and double-spending cannot be resisted at a fundamental level. The usual answer, "attacking is not economical", is weak:
  - attackers can rent hashpower at acceptable cost, and block rewards offset part of it;
  - a double-spend may not trigger market panic if the public does not notice it;
  - attackers can short Bitcoin before attacking;
  - motives can be non-economic (the project cites the August 2025 attack on Monero as a demonstration attack), and states controlling large hashpower can attack too.
- So security built only on economic assumptions is not robust.

### Decentralization

No miners or other privileged actors. All participants have equal rights. A user authorizes a payment alone, without approval from anyone else. No global ordering of transactions is needed: different objects' histories do not depend on each other, so only the order within one object's chain matters.

### Security

AOB keeps Bitcoin's cryptographic safety (signatures, key ownership) and adds resistance to Sybil attacks and double-spending that does not rest on economic assumptions.

**Sybil**: Sybil attacks target voting-based systems. AOB has no voting, so they do not apply. Repeatedly trying double-spends with fresh identities is a double-spend attack and is handled by the double-spend rules below.

**Double-spend**, four mechanisms working together:

1. **Fork = proof of attack.** Only the owner can write to a chain, so every fork comes from the same person. Any fork is direct evidence of malicious behavior. Each node that sees the conflict warning decides for itself whether to blacklist the attacker and reject the attacker's assets. No network-wide agreement on a blacklist is needed. A node that declines to blacklist gains nothing and does not affect others. If account creation costs a fee, the attacker also loses that fee, so expected return is negative.
2. **First broadcast wins.** If the attacker broadcasts conflicting blocks far apart in time, the whole network agrees on the order (implicit consensus). If the gap is short, recipients reject the payment for insufficient confirmation time. With estimated network broadcast time t₀: a third-party node can accept a payment after seeing no conflicting block within 2·t₀. A payment recipient can wait 4·t₀ so that every node has also seen the block and applied the same rule.
3. **Collusion.** The attacker privately sends conflicting blocks to colluders B and C without broadcasting. As soon as B or C pays an honest node, the chain must be broadcast and the attacker is exposed and blacklisted. The first broadcast among the conflicting blocks is the valid one. Extra safeguard for recipients: if the previous transfer happened while they were online but they saw no broadcast, they may reject the chain.
4. **Network partition.** The rare case: a partition happens right after the first block is broadcast, and the attacker broadcasts a conflicting block into the other region. The project calls the probability extremely low and generally not actionable. For high-value payments, wait longer than the maximum plausible partition duration plus 3·t₀, and check connectivity to major websites to confirm you are in the larger network region.

**Blast radius**: even in the edge cases, a discrepancy affects only one atomic object, not the global system.

**Eclipse attacks, compared**: in Bitcoin an eclipsed node can be fed a stale chain, by hiding the longer fork from it (a natural fork is enough, the attacker does not need to create one), so the victim's whole view of the ledger is wrong. In AOB the attacker would have to hide a conflicting broadcast from the recipient during the observation window, and the damage stays within one object. Defense: receive payments only in an environment that can reach major websites.

### Scalability

Capacity grows by adding blockchains, with no upper bound from the protocol; hardware is the limit. With a very large number of chains, network broadcast load grows. Two mitigations:

- **Grouping**: chains are split into groups. Nodes subscribe only to the groups they need, and broadcasts stay inside a group. Adding groups improves performance linearly.
- **Speedy Channels**: inspired by the Lightning Network, and themselves a form of AOB. Participants deposit several AOB banknotes and exchange payment statements off-chain, without broadcasting, as long as each transfer stays within the deposited balance. This also covers payments of arbitrary amounts and frequent small payments.

### Stablecoin

Fiat-pegged stablecoins need a centralized issuer, because fiat is centralized. AOB can instead issue a decentralized asset whose value is stable against computational effort, using non-competitive Proof-of-Work minting: anyone who finds a hash collision of a specified difficulty can mint a banknote whose denomination is proportional to that difficulty. When market value rises, higher profit margins draw in more minting; unlike Bitcoin, minting more notes only takes more computation, with no competition between miners. The currency therefore tracks the total hashing work. Short-term stability reflects electricity prices; in the long run, hardware progress makes it depreciate against them.

### Applications

Cryptocurrencies, stablecoins, fiat-denominated digital instruments, virtual currencies, metaverse assets, NFTs, real-world-asset (RWA) tokens, commercial vouchers, financial notes, commodity traceability.

### Banknote function for other cryptos (Bitcoin bridge; from the unpublished preprint "Migrating Bitcoin to AOB for Enhanced Security")

Bitcoin into AOB:
1. Create an AOB banknote whose face value equals the Bitcoin amount; generate its ID.
2. From a Bitcoin wallet, send that amount to an address uniquely mapped to the banknote ID. The address is not a valid Bitcoin wallet address, so these coins cannot be spent again on Bitcoin.
3. Once that UTXO is recorded on the Bitcoin chain, add a block to the banknote recording the UTXO. This proves the banknote is fully backed. From then on the banknote moves by AOB rules.

Limit: forks can still happen at earlier positions on the Bitcoin chain and are handled by Bitcoin's own rules. If a 51% attack removes that UTXO from the main chain, the banknote loses its backing.

AOB back to Bitcoin (needs Bitcoin to add adaptation functionality):
1. The owner appends a destruction block containing a valid Bitcoin address; the banknote can no longer be used. A temporary UTXO is generated and broadcast to Bitcoin.
2. Miners monitor the AOB network, verify all of the banknote's blocks from creation to destruction by AOB rules, and judge the temporary UTXO when adding a block.
3. When R% of m consecutive blocks judge it valid, it becomes a formal UTXO and the amount can be paid to the specified address.

### Where AOB stands versus Bitcoin

Architecture level: in the decentralization and security dimensions, no structural shortfall relative to Bitcoin has been identified. It keeps Bitcoin's cryptographic safety, removes privileged actors, and drops the dependence on economic assumptions.

| Dimension | Bitcoin | AOB |
|---|---|---|
| Privileged actor | Next-block miner | None |
| Who finalizes a payment | Miners approve | Owner signs; recipient applies the first-broadcast rule |
| Global ordering | Required | Not required (only per-object order) |
| Basis of double-spend defense | Economic cost of a 51% attack | Owner-only writing, fork as proof, first-broadcast-wins |
| Sybil attacks | Hashpower stands in for identities | No voting, not applicable |
| Reach of a successful attack | Chain-wide reorganization | One object |
| Eclipse attack | Victim's view of the whole chain can be wrong | One object, within the observation window; mitigated by connectivity checks |
| Cost to participate | Mining hardware or pool membership | Account creation (a fee is optional) |

Validation level: the evidence behind these properties is one published paper (a theoretical framework; it states that empirical validation is still needed), interactive demos, and a testnet that supports banknote payments and speedy-channel payments across devices on a real network. There is no mainnet and no independent security audit documented. Bitcoin has 15+ years of attacks against it in production. State this in every recommendation as the current level of certainty. It does not count against the design.

### Fit

**Good fit**: currency-like transfer, stablecoins, RWA tokens, NFTs, vouchers, financial notes, traceability, and bank-note-style functions for other cryptocurrencies; questions about miner/validator concentration, payment finality without third-party approval, or double-spend resistance that does not depend on economics.

**Weak fit**: use cases that need many parties' state updated atomically in one transaction (DeFi composability, general smart contracts). The wiki does not describe support for these. Use cases that need a production system with a long adversarial history today.

---

## Sources

- AOB project wiki, EverybodyWiki entry "Atomic Ownership Blockchains" (primary source for the mechanisms above).
- Liu, Z. (2025). "Achieving Greater Decentralization with Atomic Ownership Blockchains." *Ledger*, 10, 136–153. https://doi.org/10.5195/ledger.2025.425
- Liu, Z. "Migrating Bitcoin to AOB for Enhanced Security" (unpublished preprint).
- Multi-dimensional decentralization measurement: Srinivasan & Lee (2017), Gencer et al. (2018), Kwon et al. (2019), Lin et al. (2021), and arXiv:2512.19251.
- Nakamoto coefficient limitations (single snapshot, single subsystem, not comparable across chains): 2025 industry analyses.
- Trilemma framing: a heuristic from Ethereum scaling discussions, not a formal theorem.
