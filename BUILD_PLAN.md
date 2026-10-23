# Interim：執行計畫（2026-10-23 起）

> 截止：**台灣時間 11/2 12:59**；滾動審查、8 個名額。**目標 10/28 交件**。
> 規格見 [SPEC.md](SPEC.md)。
> 分工：
> - **你**：操作 Arc Studio、Circle Developer 錢包、所有主網交易與簽名、錄影、上傳、投稿
> - **我（Claude）**：寫 prompt、審查 Studio 產出的程式碼、整理證據表、寫描述與影片腳本、檢查所有連結

---

## 時程

| 日期 | 步驟 | 完成標準 |
|---|---|---|
| **10/24 上午** | 第 0 步：冒煙測試 | explorer 上看得到「存進 Morpho」和「從 Morpho 取出」各一筆，而且是從**你在 Studio 做的合約**送出 |
| 10/24 下午–10/25 | 第 1、2 步：合約＋測試 | Studio 測試全過，包含 §5 拒絕室的每一種錯誤 |
| 10/25–10/26 | 第 3 步：app、拒絕室、證明頁 | testnet 上完整流程能跑 |
| 10/26 | 第 4 步：程式碼審查 → 主網部署 → 跑第一批交易 | 補助 A、B 建立完成 |
| 10/27 | 第 4 步收尾（補助 A 的逾期撥款要隔天才有利息可當小費）、第 5 步上線 | 20 筆以上交易，證明頁讀得到 |
| 10/27–10/28 | 第 6、7 步：影片與描述 | 無痕視窗檢查所有連結 |
| **10/28** | **投稿** | DoraHacks 顯示已提交 |
| 10/29–11/1 | 緩衝 | — |

**全程錄影**：從第 0 步的第一個 Studio 對話就開始錄螢幕。影片必須拍到 Studio 裡的建構過程（Studio 使用佔 25%）。

---

## 第 0 步：冒煙測試（不過就不要往下做）

目的：一次驗證四件事
- Studio 能不能部署到 mainnet
- ERC-4626 的外部呼叫寫得對不對
- 6 位小數換算對不對
- explorer 上看起來是什麼樣子

**預檢結果（2026-10-23，我用 RPC、Morpho API 和本地 Foundry fork 查過）**
- 兩個金庫都是 **Morpho Vault V2**，不是舊的 MetaMorpho。V2 的 `maxDeposit`／`maxWithdraw`／`maxRedeem` **永遠回傳 0**，而且四個 gate 都是 0（任何人都能存）。所以合約**絕不能**用 `max*` 做檢查，否則一定 revert
- Steakhouse Prime USDC `0xbeef0016…7298`：$511 萬，**100% 可提領**，淨 APY 約 **3.4%**；Galaxy USDC `0x8E357432CC12ff425c36432F312968aEb16112AF`：$9,490 萬，可提領 $1,080 萬，約 3.4%。（Morpho API 上另有一個同名的「Steakhouse Prime USDC」`0x3d1D…`，只有 $1，**地址要一個字一個字對**）
- Fork 測試（用替身取代 Arc 的原生 USDC 預編譯 `0x1800…0000`／`0x1800…0001`）：存 1 USDC → 立刻值 0.999999（捨入少 1 單位）→ 一天後 1.000094 → 取 0.4、再全部贖回，兩個金庫都通過
- **會影響第 1 步**：存入後馬上把**全部本金** `withdraw` 出來會 revert（需要的份額比持有的多 1 單位）。單一里程碑、或最後一個里程碑在利息還沒蓋過捨入前就撥款，就會卡住
- 成本：gas price 20 gwei；`deposit` 約 32 萬 gas ≈ 0.0064 USDC。第 0 步準備 **1.05 USDC** 就夠

**貼給 Arc Studio：**
```
Build a minimal contract "MorphoProbe" to test depositing USDC into an ERC-4626 vault on Arc.

- USDC is the ERC-20 at 0x3600000000000000000000000000000000000000 (6 decimals). Never use msg.value.
- The mainnet vault is a Morpho Vault V2. Its maxDeposit/maxWithdraw/maxRedeem always return 0 by design, so never call or check them; just call deposit/withdraw/redeem and let them revert if they must.
- Constructor takes the vault address and checks vault.asset() == USDC.
- deposit(uint256 assets): transferFrom the caller, approve the vault, vault.deposit(assets, address(this)), record shares for the caller.
- withdrawAll(): vault.redeem(shares of caller, caller, address(this)).
- view valueOf(address) = vault.convertToAssets(shares[address]).
- Use SafeERC20 and ReentrancyGuard. Emit events.

For tests on testnet, also create MockERC4626 (OpenZeppelin ERC4626 over testnet USDC) with simulateYield(uint256 assets) that transfers extra USDC into the vault. Override maxDeposit/maxMint/maxWithdraw/maxRedeem to return 0, like Morpho Vault V2.
Write tests: deposit, simulated yield increases valueOf, withdrawAll returns principal plus yield, and withdrawing the full deposited amount with withdraw() right after depositing reverts because of rounding (redeem of all shares does not).
```

**上 mainnet**（用你的 Circle Developer 錢包）：建構參數填 Steakhouse Prime `0xbeef0016cb2Fd5C352ea7CA08a9f54739DFa7298`
1. 先對 USDC `approve` MorphoProbe 1 USDC
2. 呼叫 `deposit(1000000)`
3. 呼叫 `withdrawAll()`

**把三個 tx hash 貼給我**，我用 RPC 確認 Morpho 的 Deposit／Withdraw 事件和金額都對。

**失敗時的退路**
- Studio 不能部署 mainnet：把 Studio 專案推到 GitHub，在本地用 Circle skills 部署。依規則，合約仍然要是在 Studio 寫、在 Studio 測過的
- Steakhouse 出問題：改用 Galaxy USDC

---

## 第 1 步：合約

**貼給 Arc Studio：**
```
Build "Interim": milestone-based USDC grant escrow on Arc. While a milestone waits, its USDC earns yield in an ERC-4626 vault (Morpho on mainnet, a mock on testnet). If the funder does not respond in time, ANYONE can release the milestone and is paid a small capped tip out of the accrued yield, never out of principal.

Contract InterimGrants (no owner, no pause, not upgradeable):
- constructor(address usdc, address[] allowedVaults): allowed vaults are fixed forever; each must have asset() == usdc.
- USDC is the ERC-20 at 0x3600000000000000000000000000000000000000 (6 decimals). Never use msg.value.
- Grant { funder, recipient, vault, shares, principalLeft, reviewWindow, tipCap, ref (bytes32), closed, milestones[] }
- Milestone { amount, deadline, status (Pending, Submitted, Released, Refunded), submittedAt, evidenceHash, rejected (bool) }

Functions:
1. createAndFund(recipient, vault, amounts[], deadlines[], reviewWindow, tipCap, ref) by funder: transferFrom the total, vault.deposit, record shares and principalLeft. Revert VaultNotAllowed / BadMilestones.
2. submit(id, i, evidenceHash) by recipient: Pending -> Submitted, set submittedAt. Revert NotRecipient / WrongStatus.
3. approve(id, i) by funder: Submitted (or Pending) -> Released; withdraw amount from vault; credit recipient. Revert NotFunder / WrongStatus.
4. reject(id, i, reasonHash) by funder: Submitted -> Pending, only once per milestone. Revert NotFunder / WrongStatus / AlreadyRejected.
5. releaseOverdue(id, i) by anyone: requires Submitted and block.timestamp >= submittedAt + reviewWindow. tip = min(tipCap, accruedYield(id)); paid = _take(id, amount + tip); credit recipient min(paid, amount) and the caller the rest (principal is paid in full first, so the tip can only come from yield). Revert WrongStatus / ReviewWindowOpen.
6. refund(id, i) by funder: requires Pending (never submitted, or rejected and not resubmitted) and block.timestamp > deadline. paid = _take(id, amount), credited to the funder. Revert NotFunder / WrongStatus / DeadlineNotPassed.
7. close(id) by anyone after every milestone is Released or Refunded: _take(id, type(uint256).max) redeems all remaining shares to the funder's credit; afterwards shares == 0. Revert NotFinished / AlreadyClosed.
8. withdraw() by anyone: pull own credit. Revert NothingToWithdraw.
Views: accruedYield(id) = max(0, vault.previewRedeem(shares) - principalLeft), grant(id), milestone(id, i), credit(addr).

The mainnet vaults are Morpho Vault V2: maxDeposit/maxMint/maxWithdraw/maxRedeem always return 0 by design. Never call or check any max* function.

Accounting: every vault exit goes through ONE internal function _take(id, assets) returns (uint256 paid):
- s = grant.shares; if s == 0 return 0.
- if assets >= vault.previewRedeem(s): paid = vault.redeem(s, address(this), address(this)); grant.shares = 0.
- else: burned = vault.withdraw(assets, address(this), address(this)); grant.shares = s - burned; paid = assets.
- Wrap the vault call in try/catch and revert VaultIlliquid() on failure (anyone can retry later).
Why: right after a deposit the shares are worth 1 unit less than the principal because of rounding, so withdraw(fullPrincipal) reverts until yield covers it. Redeeming all shares instead pays the recipient the principal minus at most a couple of units.
approve: paid = _take(id, amount), credited to the recipient. In approve/releaseOverdue/refund, principalLeft -= amount (nominal). Events for payouts carry both amount and paid.
Payouts only credit balances (pull pattern), because USDC transfers to blocklisted addresses revert on Arc.
Use SafeERC20, ReentrancyGuard, custom errors with the exact names above, and emit an event with id and i for every state change.
Time checks: use >= and < explicitly; Arc timestamps are non-decreasing, not strictly increasing.
```

---

## 第 2 步：測試

**貼給 Arc Studio：**
```
Write tests for InterimGrants with MockERC4626.
Make the mock behave like Morpho Vault V2: maxDeposit/maxMint/maxWithdraw/maxRedeem return 0; _decimalsOffset() returns 12 (18-decimal shares); add setIlliquid(bool) that makes withdraw and redeem revert. In setUp, have another address deposit and call simulateYield with an uneven amount (e.g. 1234567) so the share price is not exactly 1, otherwise rounding is never exercised.
- Happy paths: create and fund, submit, approve, withdraw.
- Rounding: a one-milestone grant approved in the same block it was funded must not revert, and the recipient receives at least amount - 2. Two milestones approved back to back with no yield: the first pays exactly amount, only the last may be short. With yield, the principal is paid exactly and close() leaves shares == 0.
- Overdue release by a third address after the review window; the tip equals min(tipCap, accrued yield); with zero yield the tip is 0 and principal is untouched.
- Refund of a never-submitted milestone after its deadline; close sends the remaining yield to the funder.
- One test per refusal, asserting the exact custom error: refund on Submitted (WrongStatus), refund before deadline (DeadlineNotPassed), second reject (AlreadyRejected), releaseOverdue before the window (ReviewWindowOpen), approve by a stranger (NotFunder), submit by a stranger (NotRecipient), double release (WrongStatus), non-allowed vault (VaultNotAllowed), withdraw with no credit (NothingToWithdraw), vault liquidity too low via setIlliquid(true) (VaultIlliquid), and a retry after setIlliquid(false) succeeds.
- Boundary: releaseOverdue exactly at submittedAt + reviewWindow succeeds; one second earlier fails.
- Invariant/fuzz: sum of released amounts + refunded amounts + principalLeft == funded for every grant (nominal); every paid >= amount - 2 and paid <= amount + tip while the vault has no loss; the sum of all credits <= the contract's USDC balance; after close, shares == 0.
```

**我會檢查的**：SPEC §3.5 的 7 條 Arc 檢查（特別是程式碼裡不能出現任何 `max*`）、所有取款是否都走 `_take`、份額的捨入方向、小費是否真的不會動到本金、每個錯誤名稱是否和拒絕室對得上。

---

## 第 3 步：App、拒絕室、證明頁

**貼給 Arc Studio：**
```
Build the Interim app on top of InterimGrants:
1. Create grant: recipient, milestones (amount + deadline), review window, tip cap, vault picker showing current APY. Two transactions: USDC approve, then createAndFund.
2. Public grant page /grant/:id: locked amount, current value, accrued yield, milestone timeline with status and live countdown to the end of the review window, and an explorer link (https://explorer.arc.io/tx/...) for every event.
3. Recipient view: submit evidence (paste a URL, store keccak256 of it), a "Release it yourself" button that appears when the review window has passed (calls releaseOverdue and shows the tip), withdraw.
4. Funder view: approve, reject (shows how many rejections are left), refund past-deadline milestones.
5. Page /try-to-take-it ("The funder has no function that takes back submitted money. Try."): runs read-only eth_call simulations against the deployed contract for each attempt below and shows the decoded custom error. No wallet needed, nothing is signed:
   funder refunds a Submitted milestone; funder refunds before deadline; second reject; releaseOverdue before the window; approve from a stranger; submit from a stranger; release twice; create with a non-allowed vault; withdraw with no credit; plus one control that must succeed: releaseOverdue on an overdue milestone, showing the tip it would pay.
6. Page /proof: a table of every mainnet transaction of the contract read from its events: action, grant, milestone, amount, gas paid in USDC, link.
Read-only pages must work without connecting a wallet.
```

---

## 第 4 步：主網流程（20 筆以上，包含失敗與退款路徑）

錢包：**F**＝撥款方、**R**＝受款方、**S**＝陌生人（三個都是你控制的，描述裡要誠實寫出來）。R 和 S 各需要約 0.02 USDC 當 gas，由你自己轉過去。

**補助 A**：6 USDC，3 個里程碑（各 2 USDC），審核期 10 分鐘，小費上限 0.01，M3 的期限設成建立後 20 分鐘

| # | 誰 | 動作 | 要證明的事 |
|---|---|---|---|
| 1 | F | USDC approve | — |
| 2 | F | `createAndFund` | **錢進了 Morpho**（真實第三方） |
| 3 | R | `submit` M1 | — |
| 4 | F | `approve` M1 | 核可即撥款 |
| 5 | R | `submit` M2 | — |
| — | — | **等到隔天**（讓利息累積到 6 位小數看得到） | — |
| 6 | **S** | `releaseOverdue` M2 | **撥款方沉默，陌生人替受款方撥款，並從利息拿到小費** |
| 7 | F | `refund` M3（從未提交、已過期） | 退款路徑 |
| 8 | 任何人 | `close` | 剩下的利息給撥款方 |
| 9–11 | R、F、S | `withdraw` | pull 模式 |

**補助 B**：2 USDC，1 個里程碑
| # | 誰 | 動作 | 要證明的事 |
|---|---|---|---|
| 12 | F | USDC approve | — |
| 13 | F | `createAndFund` | — |
| 14 | R | `submit` | — |
| 15 | F | `reject` | 退回一次 |
| 16 | R | `submit`（修改後） | — |
| 17 | F | `approve` | — |
| 18 | 任何人 | `close` | — |
| 19–20 | R、F | `withdraw` | — |

再加上第 0 步的 3 筆，總共約 23 筆。**每筆的 hash 都貼給我**，我用 RPC 逐筆核對事件與金額，整理成 README 的證據表。補助 B 留一個進行中的里程碑也可以，讓拒絕室有東西可以模擬「審核期還沒到」。

---

## 第 5 步：上線與 repo

- 從 Studio 推到 GitHub（public）；app 部署到公開網址（Studio preview 能不能當公開連結，第 0 步順便確認；不行就用 Vercel）
- **網址一旦填進表單就不再改**（repo 名稱、網域都不改）
- README 我來寫，包含證據表、架構圖、拒絕室連結、Studio 專案網址

---

## 第 6 步：影片（< 3 分鐘，英文旁白＋字幕，我做腳本和剪輯）

| 時間 | 畫面 | 說什麼 |
|---|---|---|
| 0:00 | 標題 | 補助款等里程碑時錢什麼都沒做；受款方等審核等幾個月 |
| 0:15 | Circle Refund Protocol 那段話 | Circle 自己說託管的錢「currently unproductive」 |
| 0:25 | **Studio**：prompt → 合約 → 測試全過 → Tracing panel 裡的 Morpho deposit | 這一切都是在 Arc Studio 做的 |
| 1:05 | app：建立補助、錢進 Morpho、公開頁的利息在跳 | 先鎖錢沒有成本 |
| 1:25 | 受款方提交 → 倒數結束 → **S 錢包按下「Release it yourself」**，拿到小費 | 沒人回應也會撥款，按的人從利息拿小費 |
| 1:50 | **拒絕室**：一排紅色錯誤名稱 | 撥款方沒有任何函式能拿走已提交的錢 |
| 2:10 | 證明頁、explorer | 20 筆以上主網交易，存入與取出都經過 Morpho |
| 2:30 | 結尾：網址、Studio 專案 | — |

---

## 第 7 步：描述（照第 1 輪得獎作品的結構，英文）

```
Interim
Grant money that isn't dead money: it earns in Morpho while a milestone waits, pays out the moment it's approved, and if the funder goes silent, anyone can release it and is tipped from the interest.

Circle's Refund Protocol notes that escrowed funds are "currently unproductive" and suggests routing them into lending protocols. Interim does that for milestone grants, on Arc.

Try it in two minutes (no wallet)
- Try to take the money: <app>/try-to-take-it — every way a funder could claw back submitted money, simulated against the live contract and refused by name.
- A live grant: <app>/grant/1 — locked amount, interest so far, the milestone the funder never answered and the stranger who released it.
- Every mainnet transaction: <app>/proof

Built with Arc Studio
Studio project: https://studio.arc.io/app/...
<What Studio wrote, how it was tested, which panels I used, what I had to fix by hand.>

Proof on Arc mainnet
| Step | Tx |   (approve / overdue release by a third wallet with tip / refund after deadline / close / withdraw …)
Contract: <address> (verified)

How it works
<createAndFund → submit → approve or releaseOverdue → refund → close → withdraw>

Why Arc
USDC is the gas, the principal, the interest and the tip, so "release it if nobody answers" pays for itself with no oracle and no second token. Finality is deterministic, so approval is payment. Morpho USDC vaults have real liquidity on Arc.
(Escrow with yield could be built elsewhere; on Arc the waiting money, its interest and the cost of enforcing the deadline are the same dollars.)

What the contract enforces
No owner, no pause, no upgrade. Allowed vaults fixed at deploy. The funder has no function for submitted money. Tips come only from yield. All payouts are pull.

Tests
<count; refusals; invariants>

What I got wrong
<real bugs found while building — fill in during the build>

Honest limits
The three wallets are mine. Demo amounts earn fractions of a cent; the review window is 10 minutes for the demo, 7 days in practice. Morpho vault risk. Not audited.

What a microgrant would fund
<4 concrete items, 6–8 weeks>

About me
<first-year master's student in Taiwan; second Arc build; what I learned from round 1>
```

---

## 投稿前檢查清單

- [ ] Studio 專案網址（`studio.arc.io/app/...`）
- [ ] live app 公開連結（無痕視窗打得開，拒絕室不用錢包也能跑）
- [ ] 合約地址＋至少一筆交易的 explorer 連結；README 有完整證據表
- [ ] public repo
- [ ] 影片：拍到 Studio 建構過程＋主網交易
- [ ] 簡短描述：做什麼、怎麼用 Arc Studio 做的
- [ ] GitHub／X／Farcaster 公開檔案
- [ ] 所有連結用無痕視窗點過一次；投稿後不改網址
- [ ] DoraHacks 顯示已提交到「Arc Microgrants: Built with Arc Studio」
