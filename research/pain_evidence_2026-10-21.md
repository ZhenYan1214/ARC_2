# First-person pain evidence (research agent, 2026-10-21) — key facts
Reddit blocked (403); stackexchange 0 hits; used GitHub, HN Algolia, dev.to, PTT, Arc RPC.

## P1 Ambiguous settlement: paid but got no data / paid twice (strongest)
- Facilitator timeout (5–10s) < Base confirmation (10–28s): users debited, no data (tx cited) https://github.com/x402-foundation/x402/issues/1062
- 502 at t+5s, settled on-chain at t+11.9s https://github.com/x402-foundation/x402/issues/2294
- Reference client makes a fresh nonce per 402 → two valid payments for one purchase https://github.com/x402-foundation/x402/issues/3438
- Idempotency requests #452 #808 #832; no "pending" settlement state #3208
- Circle GatewayClient.withdraw() reports success as failure https://github.com/circlefin/arc-node/issues/203
- Plus circlefin skills#50, arc-fintech#50, arc-nanopayments#43
- Arc builds partially overlapping: SettleOnce #49775, Consigna #49860, Radar Agent #49626
- Arc angle: deterministic sub-second finality removes the timeout race

## P2 Paid for empty/wrong/out-of-scope response, no recourse
- #508 (paywall fraud), #3500 (user said <$0.10, agent paid $5), #3260 (no validation hook), #2887 (dispute layer, 131 comments)
- Heavily saturated on Arc; the motif already wins elsewhere

## P3 Runaway spend from retry loops
- HN: 47519532, 47425731, 47427761, 47220611; circlefin/arc-node#490 asks per-call/daily caps and no auto-retry on ambiguous outcome
- Saturated on Arc (Leash ×3, AgentTab, SpendGate…)

## P4 Agent purchases cannot be booked
- Receipt lacks supplier identity → AP can't book or recover VAT https://github.com/x402-foundation/wg-identity/issues/22
- No settlement timestamp #3085; SDK strips amount #2807
- Arc ledgers exist but address-level; per-purchase ledger with payee identity (ERC-8004) + delivery hash only partly covered

## P5 Sellers have almost no real buyers
- Three sellers (5–20 Oct): 40,400 unpaid quotes, 23 paid calls, 6 paying wallets, $0.075 revenue https://dev.to/alexar76/three-x402-sellers-compared-notes-the-routes-answer-nobody-comes-back-to-buy-5897
- 265 paid routes, $0.00 in 30 days https://dev.to/hal_gobvan_16a285d49bda97/if-youre-building-an-ai-agent-that-pays-for-apis-the-unpaid-listing-problem-and-how-to-debug-your-4753
- Bazaar indexing failures #2156, #3045 (180 comments), #3638
- Arc: by Sep 21, 82 of 171 Circle facilitator tx were PayGate's own tests https://github.com/circlefin/arc-x402-circle-wallets/issues/1 ; facilitator signer nonce 505 total (agent RPC check)
- No Arc build found that is a real task-driven buyer of third-party Arc services

## P6 Seller risk: serving before settling
- #1169, #451, #1886, #259 — settle-then-serve gateway looks unsaturated on Arc

## Freelancers / groups / Asia
- Freelancer evidence weak & old (HN 39237354, 45435658, 42063819, 48804842; Bitwage KYC delay)
- Group pooling: no new evidence; saturated
- Taiwan PTT: pain is off-ramp (MAX holds, XREX 6-month withdrawal lock, Bitfinex source-of-funds) — not solvable by an Arc app

## P7 CCTP into Arc stuck
- 900 USDC frozen 3+ weeks, attestation contradictory https://github.com/circlefin/arc-node/issues/200 ; Arc burns take 2–6h to attest #428; #434; skills#42

## Ranking (sum of 7 scores, 1–5 each)
1 P1 (30) · 2 P5 (29) · 3 P4 (27) · 4 P6 (26) · 5 P3 (25) · 6 P2 (24) · 7 P7 (24) · 8 Asia off-ramp evidence (21)
Agent synthesis: P1+P5+P4 combine into one product — a buyer agent paying real third-party Arc services, one-order-one-payment contract, receipts with payee ERC-8004 identity, settlement block/time, amount, response hash; flags paid-but-undelivered; refunds unspent budget.
