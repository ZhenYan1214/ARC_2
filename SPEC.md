# Accrue — 規格書（v0.1，2026-10-21）

> 一句話：**分期撥款的資金在等待里程碑的期間，放在 Arc 上的 Morpho 金庫生息；里程碑一通過就秒撥，審核方逾期不回應就自動撥款。**
> 背景與證據見 [REPORT.md](REPORT.md) §7–8。

---

## 0. 驗證結果（2026-10-21，都是親自查的原始資料）

| 項目 | 結果 | 對設計的影響 |
|---|---|---|
| Arc 上 Morpho USDC 金庫 | Galaxy USDC：TVL $8,980 萬、可立即提領 $1,570 萬；Keyrock Prime USDC：$7,510 萬、可提領 $1,570 萬；**Steakhouse Prime USDC（`0xbeef0016cb2Fd5C352ea7CA08a9f54739DFa7298`）：$510 萬、全部可提領**（Morpho API） | 預設用 Steakhouse Prime（流動性 100%），Galaxy 作為備選 |
| 實際收益 | **淨 APY 約 2.0%**（三個主要金庫都是），不是我先前估的 4% | 利息是附加好處，不是主賣點；定位要改（見 §1） |
| 介面 | 鏈上 `eth_call` 確認三個金庫都是標準 ERC-4626：`asset()` = `0x3600…0000`（Arc USDC 的 ERC-20 介面，6 位小數），份額 18 位小數，1 份 ≈ 1.0006 USDC | 合約直接呼叫 `deposit`／`redeem`；全程只用 6 位小數的 ERC-20 介面，不碰 `msg.value` |
| Morpho 在 Arc 上的規模 | TVL $3.01 億、借出 $1.81 億（DefiLlama） | 真實、有流動性的第三方協議 |
| Circle Earn Kit | TypeScript SDK（App Kit 的一部分），用 `exploreVaults()` 找金庫、份額換算；非託管；同鏈存入無 Earn 手續費；**合約不能直接呼叫** | 合約直接接 ERC-4626；app 端用 Earn Kit 做金庫探索與 APY 顯示（API key 只能放在後端） |
| Morpho 在 Arc testnet | **不存在**（Morpho API 不支援 5042002；該地址在 testnet 無 code） | Studio 裡（testnet）部署一個模擬 ERC-4626 金庫來測試；上 mainnet 時把金庫地址換成真的 Morpho，合約程式碼不變 |
| Arc Studio | 官方文件寫 testnet 部署；官網 FAQ 說可以用你自己的 Circle developer 錢包部署到 mainnet | 你說已經準備好了；mainnet 部署方式以你實測為準 |
| Circle 自己的說法 | Refund Protocol 文件的「Considerations」：託管中的資產「currently unproductive」，並建議未來可以把資金轉進借貸協議 | Accrue 做的就是 Circle 自己寫下的下一步，這是對評審最強的一句話 |

---

## 1. 定位：為什麼是「分期撥款（grants）」這個場景

APY 只有 2%，所以場景必須「金額大、時間長」，利息才有意義。也要「使用者已經在用 USDC」，而且「除了利息之外還有別的痛點」。

| 場景 | 期間 | 典型金額 | 已經在用 USDC？ | 除了利息之外的痛點（有證據） | Arc 上的競爭 | 評審關聯 |
|---|---|---|---|---|---|---|
| **里程碑補助／資助計畫** | 3–12 個月 | $1 萬–50 萬 | **是**（Circle Grants、DAO grants、黑客松獎金都用 USDC 發） | Arbitrum 小額補助**延遲 3–7 個月**才發放（[論壇](https://forum.arbitrum.foundation/t/arbitrumdaos-biggest-minigrants-program-ever-fiasco-summary/26920)）；多簽簽核一次等約 6 週、每年花 $23–25 萬請簽署人；Arbitrum DAO 自己把閒置資金投入代幣化國債（STEP，[The Block](https://www.theblock.co/post/353631/arbitrum-dao-us-treasurys)） | 弱：ClearFund（q2）、AllotRa（q2），都沒有生息、沒有防拖延 | **極高**：Circle Developer Grants 本身就是「依里程碑撥 USDC」 |
| 預付承包商薪資 | 1–6 個月 | $1–10 萬 | 部分（Deel 已有 1 萬+ 承包商領穩定幣） | 承包商怕被欠薪 | 擠（批次薪資 13 件、Quincena） | 中 |
| 營造保留款 | 6–12 個月 | $1–100 萬 | 否 | 總包倒閉，保留款就拿不回來 | Earnback（第 2 輪 Studio 作品，q5） | 低 |
| 租屋押金 | 1–2 年 | $1–5 千 | 否 | 美國多州法律要求押金計息給房客（麻州、紐約、佛州、馬里蘭…，[整理](https://www.baselane.com/resources/state-security-deposit-interest-rate-rules)） | Handova（Lagos） | 低 |

**結論：主場景定為「里程碑補助」**，預付薪資與保留款列為同一套合約的延伸場景。

**核心價值主張（按重要性排序）**
1. **撥款方**：先把整筆經費鎖進去，**幾乎沒有機會成本**，錢照樣在生息。這消除了撥款方不願預先撥款的理由
2. **受款方**：看得到「錢真的在」（鏈上公開），審核方拖延時**自動撥款**，不會再等 3–7 個月
3. **利息歸屬**在建立時就寫死（給撥款方、給受款方或按比例分），雙方都看得到

---

## 2. 為什麼要用 Accrue（對照現有作品）

| | 一般託管／里程碑作品（約 62 件） | Accrue |
|---|---|---|
| 等待中的資金 | 0% | Morpho 約 2% 淨 APY |
| 審核方拖延 | 無保護，或只有截止日 | 提交後開啟審核期，**不回應就自動撥款** |
| 鏈上交易對象 | 自己付給自己 | **每筆存入／贖回都經過真實第三方（Morpho）** |
| 用到的 Circle 產品 | USDC | USDC＋Earn Kit＋（選用）Memo |
| 和 Circle 的關係 | 無 | 補上 Circle 自己列出的 Refund Protocol 限制 |
| 第 2 輪 Studio 作品中有真實第三方的 | 0／45 | 有 |

---

## 3. 合約設計

### 3.1 角色
- **Funder（撥款方）**：建立計畫、存入經費、審核里程碑
- **Recipient（受款方）**：提交里程碑成果
- **Arbiter（仲裁人，選用）**：爭議時只能在撥款方與受款方之間分配，不能轉給第三人，費用有上限
- **Anyone（任何人）**：審核期過了之後，可以觸發自動撥款（keeper 友善，gas 是 USDC）

### 3.2 資料結構
```
Program {
  funder, recipient, arbiter
  vault            // ERC-4626 地址，必須在白名單內
  shares           // 這個計畫持有的金庫份額
  principalLeft    // 尚未撥出的本金（USDC，6 位小數）
  yieldRule        // TO_FUNDER | TO_RECIPIENT | SPLIT(bps)
  reviewWindow     // 例如 7 天
  milestones[]     // { amount, dueDate, status, evidenceHash, submittedAt }
}
status: PENDING → SUBMITTED → (APPROVED | AUTO_RELEASED | REJECTED → PENDING | DISPUTED → RESOLVED)
```

### 3.3 函式
| 函式 | 誰 | 做什麼 |
|---|---|---|
| `createProgram(recipient, vault, milestones, yieldRule, reviewWindow, arbiter)` | funder | 建立計畫，檢查金庫在白名單內、`asset()` 必須是 USDC |
| `fund(programId)` | funder | 從 funder 拉 USDC（ERC-20，6 位）→ `vault.deposit` → 記錄份額 |
| `submit(programId, i, evidenceHash)` | recipient | 提交成果的 hash，審核期開始 |
| `approve(programId, i)` | funder | 贖回並撥款 |
| `reject(programId, i, reasonHash)` | funder | 退回修改；每個里程碑限制退回次數，避免無限拖延 |
| `autoRelease(programId, i)` | anyone | 審核期過了、沒被退回 → 撥款 |
| `dispute(programId, i)` / `resolve(programId, i, bpsToRecipient)` | 任一方 / arbiter | 爭議：凍結該里程碑；仲裁人只能分給這兩方 |
| `cancel(programId)` | funder | 只能取消「尚未提交、而且已過 dueDate」的里程碑，取回本金 |
| `close(programId)` | anyone | 全部結束後，依 `yieldRule` 分配剩餘利息 |
| `claim()` | 任何收款人 | **pull 模式**領取應得款項（避免黑名單地址卡住整個流程） |

### 3.4 利息會計
- `accrued = vault.convertToAssets(shares) − principalLeft`
- 每次撥款只贖回該里程碑的本金（`vault.withdraw(amount)`），利息留在金庫繼續滾，`close()` 時再一次分配
- 若 `maxWithdraw` 不足（金庫流動性暫時不夠）：記為「待領」，之後任何人都可以 `settlePending()` 補撥，**不會整筆 revert**

### 3.5 Arc 特有的檢查（Studio 生成的程式碼要逐條確認）
1. 只用 ERC-20 介面 `0x3600…`（6 位小數），**絕不混用 `msg.value`**（原生 18 位）
2. 交易 `maxFeePerGas` ≥ 20 Gwei，否則會被靜默丟棄
3. 付款一律 pull 模式；轉給黑名單地址會 revert 而且照扣 gas
4. 時間判斷用 `block.timestamp` 但不能假設嚴格遞增；排序用區塊號
5. ReentrancyGuard；金庫白名單由 owner 設定，建立後不可更換
6. ERC-4626 首次存入的通膨攻擊：金庫是既有的 Morpho 大金庫，風險低，但 mock 金庫測試時仍要涵蓋

---

## 4. App（在 Arc Studio 裡做）

| 頁面 | 給誰 | 內容 |
|---|---|---|
| 建立計畫 | 撥款方 | 填受款方、里程碑（金額、日期）、利息規則、審核天數；用 Earn Kit 顯示可選金庫與目前 APY |
| 計畫公開頁 | 所有人 | 「錢真的在」：已鎖金額、目前價值、已累積利息、每個里程碑狀態、所有交易的 explorer 連結 |
| 受款方頁 | 受款方 | 提交成果（上傳 → 計算 hash）、審核倒數、自動撥款按鈕、領款 |
| 審核頁 | 撥款方 | 核可、退回、爭議 |
| 匯出 | 雙方 | CSV：日期、里程碑、金額、利息、tx hash |

---

## 5. 測試計畫

**Studio（testnet）**
- 部署 `MockERC4626`（OpenZeppelin ERC4626，底層用 testnet USDC），加一個「注入收益」函式模擬利息
- 單元測試：建立、存入、提交、核可、自動撥款、退回上限、爭議分配、取消、利息分配三種規則、流動性不足時記為待領、黑名單收款人改用 claim、重入
- 不變量：`撥出總額 + 剩餘本金 = 存入總額`；利息永遠 ≥ 0 時才分配

**Mainnet（真實 USDC，小額）**
- 金庫：Steakhouse Prime USDC
- 腳本：建立 1–2 個計畫（例如 20 USDC、3 個里程碑）→ 存入 Morpho → 提交 → 核可一個、讓一個**自動撥款**、退回一個再核可 → close 分配利息
- 目標：**≥ 15 筆 mainnet 交易**，每筆附 explorer 連結做成表格；其中存入、贖回是與 Morpho 的真實交易

---

## 6. 評分對應

| 權重 | 項目 | Accrue 的證據 |
|---|---|---|
| 30% | 真實 USDC 在 mainnet | 合約地址、≥15 筆 tx 表格；存入、贖回都經過真實的 Morpho 金庫 |
| 25% | Arc Studio 使用 | 合約、mock 金庫、測試、app 都在 Studio 完成；錄影展示 Contracts panel 與 Tracing panel 中的 deposit／withdraw 事件 |
| 25% | 可運作的產品 | 公開 app 連結：建立 → 存入 → 提交 → 撥款，完整流程能跑 |
| 20% | 潛力 | 補上 Circle 自己列出的缺口；對齊 Circle Grant 的 treasury 優先領域；延伸到預付薪資與保留款；可提交為 Circle skill 或範例 |

---

## 7. 風險與對策

| 風險 | 對策 |
|---|---|
| 2% 利息在小額時很少 | 主打「撥款方預先鎖錢沒有成本」與「防拖延自動撥款」，利息是附加；用年化試算表展示大額案例 |
| Morpho 金庫風險（壞帳、暫時流動性不足） | 白名單只收流動性高的金庫；`maxWithdraw` 不足時記為待領；介面明確揭露風險；之後可接 Stepwell／Rider 類保險 |
| Studio 對 ERC-4626 外部呼叫的熟悉度未知 | 第一個 Studio 對話就先做「mock 金庫＋存入、贖回」的最小原型 |
| 法規（GENIUS Act 禁止的是發行方付息） | 收益來自 DeFi 借貸，不是 USDC 發行方；app 內清楚標示 |
| 與 ClearFund、Earnback、MilestonePay 撞題 | 差異點寫在 README 首段：生息＋防拖延自動撥款＋真實第三方交易 |

---

## 8. 時程（截止：台灣時間 11/2 12:59；滾動審查，越早越好）

| 日期 | 工作 |
|---|---|
| 10/22 | Studio：mock 金庫＋核心合約＋單元測試 |
| 10/23 | Studio：完成所有函式與測試；開始做 app |
| 10/24 | Studio：app 完成，在 testnet 跑完整流程 |
| 10/25 | 依 §3.5 逐條審查程式碼 → 部署 mainnet → 小額真實流程 ≥ 15 筆 |
| 10/26 | 前端上線、README（含證據表）、架構圖 |
| 10/27 | 影片（< 3 分鐘）、提交 |

---

## 9. 給 Arc Studio 的第一個 prompt（英文，可直接貼上）

```
Build "Accrue": milestone-based USDC grant escrow on Arc where funds waiting for milestones earn yield in an ERC-4626 vault.

Contracts:
1. MockERC4626: OpenZeppelin ERC4626 over Arc USDC (ERC-20 at 0x3600000000000000000000000000000000000000, 6 decimals), plus an admin-only `simulateYield(uint256 assets)` that transfers extra USDC into the vault, for testnet only.
2. AccrueGrants:
   - createProgram(recipient, vault, milestoneAmounts[], milestoneDueDates[], yieldRule {TO_FUNDER, TO_RECIPIENT, SPLIT bps}, reviewWindowSeconds, arbiter). Vault must be on an owner-managed allowlist and vault.asset() must equal USDC.
   - fund(programId): pull total USDC from funder via transferFrom, then vault.deposit; track shares and principalLeft per program.
   - submit(programId, i, evidenceHash) by recipient starts the review window.
   - approve(programId, i) by funder; reject(programId, i, reasonHash) with a max of 2 rejections per milestone.
   - autoRelease(programId, i) callable by anyone after the review window if not rejected.
   - dispute/resolve: arbiter can only split the milestone amount between funder and recipient (bps).
   - cancel(programId, i) by funder only for milestones never submitted and past due date.
   - close(programId): after all milestones are final, distribute accrued yield = convertToAssets(shares) - principalLeft per yieldRule.
   - All payouts credit a pull balance; recipients call claim(). If vault.maxWithdraw is insufficient, record as pending and allow settlePending() later.
   - Never use msg.value; only the 6-decimal ERC-20 USDC interface. Use ReentrancyGuard and SafeERC20. Emit events for every state change with programId and milestone index.
Tests: full lifecycle, auto-release, rejection limit, dispute split, cancel, all three yield rules with simulated yield, insufficient vault liquidity -> pending, reentrancy, invariant (released + principalLeft == funded).
App: create program (show vault APY), public program page (locked amount, current value, accrued yield, milestone timeline, explorer links), recipient page (submit evidence hash, countdown, auto-release button, claim), funder review page, CSV export.
```
