<!-- solve-problem:6d668efd-166c-43ee-a2b8-d711dc203501 -->
# Paying for speculative validation work

A [Tangle Solve problem](https://solve.tangle.tools/problems/6d668efd-166c-43ee-a2b8-d711dc203501).

## What is the problem?

In blockchains, validators (block builders, sequencers) must perform **speculative computation**: they simulate or partially execute transactions—e.g., running the EVM to check a bundle's validity, backrun opportunities, or assess whether a transaction reverts—before deciding what to include in a block. Transactions that are *not* included still consumed real validator CPU/memory/bandwidth. Today this work is **uncompensated and unmetered**, creating an asymmetric resource-exhaustion game: an attacker can spam mempools or order-flow auctions with junk bundles that are expensive to evaluate but cost the attacker nothing if dropped (the setting of Yaish et al.'s Speculative DoS attacks on Ethereum).

The problem: **design a fee/metering mechanism that (a) measures or boundsably accounts for the simulation and bundle-evaluation work a proposer performs for transactions that are never included, and (b) charges for it—without enabling fabricated-work claims (a proposer inventing work to extract fees) or fee theft (charging users for work done on transactions they never authorized or that were dropped for the proposer's own strategic reasons).**

Formal setting: users submit signed transactions/bundles with fee parameters; a proposer chooses a subset to simulate and include; inclusion requires the transaction's effective gas/fees to cover execution. The open variable is the **off-inclusion evaluation cost** function c(·) and the payment rule. Constraints: incentive-compatibility for both sides (attacker can't get free evaluation work; proposer can't overclaim), soundness (some form of verifiability or cryptographic/statistical accounting of work actually performed), and DoS-resistance. A full formalization—including the exact claim/attest/dispute game—does not yet exist in the literature; producing one is part of the problem.

## Why does it matters?

Solving this closes a real DoS vector on Ethereum mainnet and all order-flow-auction chains (MEV-Boost builders, sequencers), where bundle evaluation is the dominant unreimbursed cost. It also unlocks principled multiresource fee markets (CPU, memory, bandwidth—not just gas), letting validators operate profitably at higher load. Analogous results would transfer to serverless platforms and any system where a service provider performs unbillable speculative work on unaccepted requests.

## What's already known?

- **Yaish et al. (USENIX Security 2024)** demonstrate the attack concretely: attackers force Ethereum proposers/builders to burn gas-equivalent simulation on transactions that revert or are excluded, at near-zero attacker cost. This establishes the problem is live, not hypothetical.
- **ConditionalExhaust, MemPurge, GhostTX (2024)** (verify—these appear to be attack tools/works, not defensive mechanisms) sharpen the threat model but do not propose metering or payment.
- **Multiresource fee markets** (e.g., work on multi-dimensional fees in Ethereum's roadmap; Roughgarden's transaction fee mechanism design) price *included* execution but assume excluded work is free.
- **Related theory:** pricing of failed auctions, bidding with refunds, serverless cost-recovery models. None handle the adversarial "charge for work on a transaction you yourself chose to drop" double-edged problem.

No published mechanism credibly meters off-inclusion work while resisting fabricated-claim attacks. Naïve solutions (attestation by proposer) fail to overclaiming; user-prepaid bonds fail because users won't pay for inclusion they didn't get; commit-reveal timing fails on latency.

## What would count as solving it?

Any of:
1. **A mechanism (with proof):** a payment rule + verifiable accounting scheme with provable incentive compatibility—no profitable deviation for spammers (bounded free evaluation work) and for proposers (no profitable fabricated-work claims). Formal game-theoretic statement and proof, or impossibility result showing no such mechanism exists under stated assumptions.
2. **A bounded-exposure result:** a protocol guaranteeing validator speculative work per unconfirmed transaction is bounded by O(inclusion-comparable cost), e.g., via bounded simulation depth, prechecks, or tiered admission.
3. **An implemented, evaluated prototype** (e.g., on a builder like mev-geth or a fork) demonstrating the accounting is verifiable and overhead is acceptable.
4. **An impossibility/tradeoff theorem:** characterizing exactly what must be sacrificed (latency, decentralization, trust assumptions) to price speculative work.

## How to start

1. Read Yaish et al. (2024) fully—especially their cost model for proposer simulation—then write the formal game: players, action spaces (submit/simulate/include/claim), payoffs. The missing formalization is the core gap.
2. Identify the two adversarial directions precisely: (a) spammer amortizing free evaluation; (b) proposer inflating claims. Sketch why each naïve payment rule fails—this gives the design constraints.
3. Look at verifiable computation / attestation literature (TEE-based metering is the pragmatic path; SGX-style remote attestation lets a builder prove work counts) and at auction theory on losing-bid cost recovery.
4. Main obstacle: verifying work on transactions that were dropped—there's no on-chain artifact. Consider optimistic schemes with fraud proofs over an evaluation log, or bounding work a priori so claims can't exceed a hard cap (transforms the metering problem into a admission-control problem).

## Key references

- **Yaish, G., et al. (2024). "Speculative Denial-of-Service Attacks in Ethereum." USENIX Security 24.** https://www.usenix.org/conference/usenixsecurity24/presentation/yaish — Demonstrates attackers can force Ethereum proposers to expend substantial simulation effort on never-included transactions at negligible attacker cost; establishes the attack model this problem must defend against.
- **Roughgarden, T. (2020/2021). "Transaction Fee Mechanism Design."** — Foundational game-theoretic framework for blockchain fees (truthfulness, off-chain agreements); the natural formal scaffolding, though it ignores excluded-transaction costs.
- **ConditionalExhaust, MemPurge, GhostTX (2024).** — Attack-oriented follow-ups sharpening the threat model (verify citations; details not independently confirmed here).

## Warnings

- The census record marks this **"engineering"** — a purely theoretical mechanism without overhead measurements will undersell; pair proofs with implementation numbers.
- The "fee theft" constraint is subtle: charging users for work on transactions the *proposer strategically dropped* (e.g., to favor a counter-bidder) must be distinguished from work on genuinely reverting spam. Mechanisms that ignore this will be rejected as unfair.
- Don't reinvent the failed naïve fixes: user prepaid bonds for submission, and simple proposer attestations are both known-inadequate (overclaiming / unwillingness to pay).
- The 2024 "sharpening" works (ConditionalExhaust etc.) are cited but unverified—locate and confirm them before relying on their content.

---
*`BCGT-000294` · flagged for review*

## Acceptance guidance

Any of:
1. **A mechanism (with proof):** a payment rule + verifiable accounting scheme with provable incentive compatibility—no profitable deviation for spammers (bounded free evaluation work) and for proposers (no profitable fabricated-work claims). Formal game-theoretic statement and proof, or impossibility result showing no such mechanism exists under stated assumptions.
2. **A bounded-exposure result:** a protocol guaranteeing validator speculative work per unconfirmed transaction is bounded by O(inclusion-comparable cost), e.g., via bounded simulation depth, prechecks, or tiered admission.
3. **An implemented, evaluated prototype** (e.g., on a builder like mev-geth or a fork) demonstrating the accounting is verifiable and overhead is acceptable.
4. **An impossibility/tradeoff theorem:** characterizing exactly what must be sacrificed (latency, decentralization, trust assumptions) to price speculative work.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/6d668efd-166c-43ee-a2b8-d711dc203501?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000294.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000294)
