# Circle company deep-dive (research agent, 2026-10-21) — key facts

Primary sources: Q2 2026 10-Q https://www.sec.gov/Archives/edgar/data/1876042/000187604226000248/crcl-20260630.htm ; Q2 call https://equibles.com/stocks/crcl/calls/2026-q2 ; Circle pressroom. Q3 results Nov 4 2026 (after deadline).

## Numbers (Q2 2026)
- Reserve income $667.7M (~95% of revenue); subscription & services $28.2M; **transaction revenue only $5.3M, down YoY**
- Distribution & transaction costs $410M (~61.5% of reserve income); RLDC margin 41%
- USDC circulation: $75.3B end-2025 → $77.0B Q1 → $73.3B Q2 → ~$72.1B Oct 19 (flat/down in 2026)
- Market share by supply ~24–27% (USDT ~59%); CFO: USDC ~70% of stablecoin *transaction* volume in June
- 100bp rate cut ≈ −$377M net/yr
- FY26 other revenue guidance raised to $310–330M — mostly ARC token presale (~$180M)
- **"USDC on platform" (Mint, corporate, and non-custodial Circle Wallets) = no yield share with Coinbase**: $12.4B, 19.5% of supply (vs 7.4% a year ago) → main margin lever
- USDC total supply on Arc (RPC, Oct 21): ~$777.7M (~1% of all USDC)
- Stock ~$82 (Oct 19); risks: Open USD (OUSD) consortium, rates, distributor yield demands, Morgan Stanley: agentic adoption thin (~$41.9K/day, ~$0.24/tx); CFO leaving end of Dec 2026

## Strategy
- Arc = "economic OS for the internet"; new revenue engines: CPN per-transaction fees + Arc transaction/infrastructure fees ("like AWS")
- ARC token: 807.5M sold at $0.30 ($242M); **buyers can demand repayment if no PoS by May 8 2028**; whitepaper ties fees to real usage → Circle needs real fee-paying usage on Arc
- Day-one Arc DEX volume ~82% meme launchpads
- CPN: $14.7B annualized end Q2, ~$23B by July 31, 175 FIs; monetization H2 2026; native on Arc
- Tazapay $400M acquisition (Singapore, $25B+ volume, ~60% stablecoins, closing 2027)
- Enterprise: SAP/Tereina, Kyriba, Volante
- Agent Stack (May 11 2026): 900+ paid services; USDC 99.3% of x402 settlement

## What leadership asks of developers
- **"Stranger test"** (H2 2026 goal): a published paid USDC agent service is discovered and paid by an independent agent; transaction evidence informs the next buyer. Gaps named: behavior-grounded reputation, validation, trusted discovery. https://www.circle.com/blog/building-the-open-agentic-economy
- Allaire: "agents want actual money"; priorities: Know-Your-Agent, discovery, reputation from real tx data, programmable spending policies; payments growth: cross-border, capital markets, agentic, stablecoin cards & QR POS in Asia/LatAm

## Developer pain (GitHub issues)
- Agent payment retries double-settle: circlefin/skills#50, arc-nanopayments#43, arc-fintech#50
- Gateway withdraw fails when agent spent all USDC (also its gas): arc-nanopayments#48
- Agent wallet spending policies may not cover x402 EIP-3009 signatures: agent-stack-starter-kits#9
- Sample-app security bugs: arc-nanopayments#29, arc-escrow#42, arc-commerce#47
- Refund Protocol unaudited, open reentrancy/over-withdraw reports unanswered since Feb 2026 (#9–12, #19–20)
- Mainnet friction: first-USDC chicken-and-egg, Foundry CREATE2 fails on 5042, eth_getLogs 10K-block cap, ~0.5 rps public RPC, explorer API behind Cloudflare (arc-node#454, #455, #453)

## Agent's ranked synthesis: what helps Circle
1. Recurring, non-speculative USDC payment flow on Arc (many small repeated payments between distinct addresses)
2. Balances held in Circle Wallets (margin lever)
3. Pass the "stranger test" (x402 service discovered & paid by independent agents, or buyer agent paying third-party Arc services + reputation evidence)
4. Reference-quality fixes for Circle's own unsolved patterns (idempotency, policy covering signed payments, gas-safe Gateway withdraw, refunds)
5. Onboarding first-time users (first-USDC problem)
6. Cross-border/Asia payouts with local stablecoins
7. Accounting-ready records (Memo, CSV)
