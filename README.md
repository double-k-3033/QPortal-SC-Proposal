# Proposal to include QPortal Smart Contract

## Proposal

Allow the **QPortal Smart Contract** to be deployed on Qubic.

## Available Options

- **Option 0:** No, don't allow
- **Option 1:** Yes, allow

---

## What is QPortal?

**QPortal** is an on-chain DAO for holders of the **PORTAL** token, built on Qubic.

It provides:

- A **membership system** where PORTAL holders register to join the Portal DAO.
- A **proposal system** where members submit yes/no proposals for the community to decide on.
- **Token-weighted voting**, where each vote locks PORTAL as voting weight, so influence is tied to real holdings.
- A **revenue-sharing model** where execution fees are split between burning and QPORTAL shareholders, and refund fees are redistributed to PORTAL holders.

QPortal is designed to:

- Provide transparent, tamper-proof governance for the PORTAL community
- Tie voting power to actual PORTAL holdings (skin in the game)
- Generate sustainable revenue for shareholders and burn pressure for the network
- Resolve every proposal deterministically and automatically at epoch end

---

## QPortal Solution

QPortal introduces an **epoch-based governance system** with a built-in DAO:

1. A **membership registry** that any PORTAL holder can join by staking PORTAL.
2. **Yes/no proposals** that registered members submit, subject to per-user and per-epoch limits.
3. **Token-weighted voting**, where voters lock PORTAL behind a yes or no choice and can withdraw their vote while voting is open.
4. A **freeze window** (Monday 00:00 UTC → Wednesday 12:00 UTC) that blocks submissions and votes near the epoch boundary to prevent last-minute manipulation.
5. **Automatic resolution** at epoch end, accepting a proposal only when it meets both a minimum-PORTAL threshold and a majority of votes and PORTAL.
6. A **revenue-sharing model** where fees are split between burns and shareholders, and refund fees are redistributed to PORTAL holders.

In short:
**QPortal = On-chain DAO membership + token-weighted proposals & voting + automatic resolution + revenue sharing.**

---

## How It Works (Step by Step)

### 1. Membership (Register / Logout)

1. **Register**
   - Any PORTAL holder can register in the Portal DAO by calling `registerInPortalDAO`.
   - The caller pays the QU execution fee (20,000 QU) and must hold at least `QPORTAL_REGISTER_FEE` (5) PORTAL.
   - `QPORTAL_REGISTER_FEE` PORTAL is transferred to the contract as a registration stake.
   - Capacity: up to `QPORTAL_MAX_MEMBER` (4096) members.

2. **Logout**
   - A member can leave the DAO at any time via `logoutInPortalDAO`.
   - Locked voting PORTAL is reclaimed separately through `requestRefund`.

### 2. Proposals (By Members)

1. **Submit a Proposal**
   - A registered member submits a yes/no proposal (identified by an `id`) via `submitProposal`.
   - Limits: `QPORTAL_MAX_PROPOSAL_USER` (2) per member per epoch, `QPORTAL_MAX_PROPOSAL_EPOCH` (5) per epoch.
   - Submissions are blocked during the freeze window.

### 3. Voting (In / Out)

1. **Vote (submitInVote)**
   - A member casts a yes or no vote on an open proposal, locking `votingPortalAmount` PORTAL into the contract as voting weight.
   - Each member can vote once per proposal.

2. **Withdraw a Vote (submitOutVote)**
   - While the proposal is still open, a member can withdraw their vote, reversing the tally.
   - The PORTAL stays locked and is reclaimed via `requestRefund`.

3. **Freeze Window**
   - Voting and vote withdrawal are blocked from **Monday 00:00 UTC** to **Wednesday 12:00 UTC**.

### 4. Refunds

1. **Reclaim Locked PORTAL (requestRefund)**
   - Once a member has no active vote in the current proposal epoch, they can reclaim their locked PORTAL.
   - A `QPORTAL_REFUND_FEE` (5 PORTAL) is charged per refund and added to the PORTAL redistribution pool.

### 5. Epoch End → Automatic Resolution

At the end of each epoch, every proposal opened during the epoch is resolved:

- **Accepted** if `yesPortal + noPortal >= QPORTAL_MIN_TOTAL_VOTED_PORTAL` (100,000) **and** `yesPortal > noPortal` **and** `yesVotes > noVotes`.
- **Rejected** otherwise (including proposals that received no votes).

Then QU revenue is split (see Fee Structure), refund fees in PORTAL are redistributed per-share to PORTAL holders, and per-epoch state is reset.

---

## QPortal Key Features

| Feature | Description |
|---|---|
| **DAO Membership** | Any PORTAL holder can register (staking PORTAL) and participate in governance. |
| **Yes/No Proposals** | Registered members submit proposals, capped per user and per epoch. |
| **Token-Weighted Voting** | Votes lock PORTAL as weight; influence is proportional to committed PORTAL. |
| **Vote Withdrawal** | Voters can withdraw a vote while the proposal is open. |
| **Epoch Freeze Window** | Submit/vote disabled Mon 00:00 UTC → Wed 12:00 UTC to stop end-of-epoch manipulation. |
| **Automatic Resolution** | Proposals resolve deterministically at epoch end (min-PORTAL threshold + majority). |
| **Revenue Sharing** | QU fees split 25% burn / 75% shareholders; PORTAL refund fees redistributed to PORTAL holders. |
| **Refundable Stakes** | Locked voting PORTAL is reclaimable once no active vote remains. |

---

## Fee Structure

### Execution fee (QU)

Every procedure requires an invocation reward of at least `QPORTAL_EXECUTION_FEE` (**20,000 QU**). Underpayment is rejected (the sent amount is kept as revenue); overpayment is refunded. Collected QU accumulates in `epochRevenue`.

At epoch end, `epochRevenue` is split using `QPORTAL_TOTAL_FEE = 4`:

- **75%** (`QPORTAL_SHAREHOLDER_FEE = 3`) → distributed to the 676 QPORTAL shareholders via `qpi.distributeDividends()`.
- **25%** (`QPORTAL_BURN_FEE = 1`, computed as the exact remainder) → burned via `qpi.burn()`, refilling the execution-fee reserve.

### Refund fee (PORTAL)

Each refund charges `QPORTAL_REFUND_FEE` (**5 PORTAL**), accumulated in `portalEpochRevenue`. At epoch end, this pool is distributed per-share to all current PORTAL possessors, proportional to holdings.

---

## Contract Details

| Field | Value |
|---|---|
| Contract long name | QPortal |
| Share asset name (676 IPO shares) | `QPORTAL` |
| Governance token | `PORTAL` (asset name id `83843471265616`) |
| Contract index | 28 |
| Proposal epoch (N) | 213 |
| IPO epoch (N+1) | 214 |
| Construction / first-use epoch (N+2) | 215 |

> Recommended filename when submitting to `qubic/proposal/SmartContracts`:
> `2026-05-29-QPortal.md` (date-prefixed, matching the directory convention).

---

## Technical Implementation

The full implementation is available here:
[src/contracts/QPortal.h](https://github.com/double-k-3033/core/blob/feature/QPortal/src/contracts/QPortal.h)
