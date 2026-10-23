# Interim：規格書（v0.2，2026-10-23）

> **一句話**：補助款在等里程碑的期間放在 Arc 上的 Morpho 金庫生息；核可的當下就撥款；審核方逾期不回應，**任何人**都能替受款方按下撥款，按的人從利息裡拿一點小費。
>
> v0.2 是依第 1 輪 20 個得獎作品的拆解（[research/round1_winners_analysis_2026-10-23.md](research/round1_winners_analysis_2026-10-23.md)）和 Tieout 的檢討（[LESSONS_ROUND1.md](LESSONS_ROUND1.md)）大改的版本。和 v0.1 的差異見 §10。

---

## 0. 已驗證的事實（2026-10-21，查原始資料）

| 項目 | 結果 | 對設計的影響 |
|---|---|---|
| Arc 上 Morpho USDC 金庫 | Steakhouse Prime USDC `0xbeef0016cb2Fd5C352ea7CA08a9f54739DFa7298`：$510 萬，**全部可提領**；Galaxy USDC：$8,980 萬，可提領 $1,570 萬（Morpho API） | 預設 Steakhouse Prime，Galaxy 為第二個允許的金庫 |
| 實際收益 | 淨 APY 約 **2.0%** | 利息是附加價值，用來付自動撥款的小費，不是主賣點 |
| 介面 | 都是標準 ERC-4626；`asset()` = `0x3600…0000`（USDC 的 ERC-20 介面，6 位小數），份額 18 位小數 | 全程只用 6 位小數的 ERC-20 介面，不碰 `msg.value` |
| Morpho 在 Arc | TVL $3.01 億、借出 $1.81 億（DefiLlama） | 真實、有流動性的第三方 |
| Morpho 在 Arc testnet | **不存在** | Studio 的測試用 mock ERC-4626；**真正接 Morpho 只能在 mainnet 驗證**（見 BUILD_PLAN 第 0 步） |
| Arc Studio | 一鍵部署只到 testnet；mainnet 要用你自己的 Circle Developer 錢包 | 第 0 步就要走通 |
| Circle 自己的說法 | Refund Protocol 文件的「Considerations」：託管資產「currently unproductive」，建議未來接借貸協議 | 描述的第二句就引用這段 |
| 撞名與撞題 | 第 1 輪已有「Accrue on Arc」（#49851：ERC-8183 託管＋審核沉默也付款），所以**改名 Interim**（DoraHacks 查無同名）；「託管＋生息」在 562 件作品裡**沒有人做**（DoraHacks API 2026-10-23 再查一次：Morpho 相關只有儲蓄、保險、儀表板） | 差異點寫在描述第一段 |

---

## 1. 問題與使用者

**使用者**：用 USDC 發補助的人。例如 Circle Developer Grants、DAO 的補助計畫、黑客松與懸賞主辦方，以及拿這些錢的開發者。

**痛點（都有來源）**
- **受款方等錢等很久**：Arbitrum 小額補助延遲 3–7 個月才發放；DFINITY Supernova 得獎者等了 40 多天
- **撥款方不願意先把錢全部鎖住**：錢鎖在託管裡什麼都不做（Circle 自己也承認），所以撥款方傾向一次一次撥，受款方只能「相信對方會付」
- **審核拖延沒有成本**：Arbitrum 服務商等多簽付款約 6 週；Upwork 客戶會在第 14 天自動放款前要求修改來拖延

**Interim 的答案（三句話）**
1. **撥款方先把整筆鎖進去，錢照樣在 Morpho 生息**。先鎖錢不再有機會成本
2. **受款方看得到錢真的在**，而且審核期一過，**不用等任何人**就能拿到
3. 處理「沒人回應」的那個人，**用這筆錢等待期間賺的利息付他小費**。不需要伺服器，也不需要另一種代幣

---

## 2. 為什麼是 Arc（只寫真的成立的）

| Arc 的特性 | Interim 怎麼用 | 換到別的鏈會怎樣 |
|---|---|---|
| **USDC 同時是 gas、本金、利息、小費** | 逾期撥款的小費直接從 USDC 利息付，觸發者付的 gas 也是 USDC，**帳一定對得起來** | 觸發者要用 ETH 付 gas，小費要用預言機換算，或另外準備一池 ETH（Doorstep 也提出同一點） |
| **確定性次秒最終性** | 核可或逾期撥款的那一刻就是最終結果 | 要等確認數，「秒撥」只是 UI 上的說法 |
| **Morpho USDC 金庫在 Arc 上有真實流動性**（Circle Earn Kit 接的就是它） | 等待中的錢存進真實的第三方協議 | — |
| **USDC 轉帳可能因黑名單而 revert** | 所有付款都先記帳，再由本人領取（pull），一個被凍結的地址只會卡住自己 | — |

**不宣稱的事**：生息託管在 Base 上也做得出來。我們的論點是：**在 Arc 上，等待中的錢、它賺的利息、處理拖延的成本，全部是同一種美元**，所以「沒人回應也會撥款」可以自給自足。

---

## 3. 合約：`InterimGrants`（單一合約）

### 3.1 信任的形狀（寫在描述裡）
- **沒有 owner、沒有 pause、不可升級**。允許的金庫在建構時寫死（Steakhouse Prime、Galaxy），之後不能改
- **撥款方沒有任何函式**可以拿回「已提交」里程碑的錢；只能拿回「從未提交且已過期」的
- **小費有上限**：建立時設定，只能從利息付，**永遠不動本金**
- **所有付款都是 pull**：先記入 `credit[addr]`，本人呼叫 `withdraw()` 領取

### 3.2 資料結構
```
Grant {
  funder, recipient
  vault            // 建立時選定，必須是允許的金庫之一
  shares           // 這筆補助持有的金庫份額
  principalLeft    // 尚未撥出、也尚未退回的本金（USDC，6 位小數）
  reviewWindow     // 秒；demo 用 10 分鐘，正式建議 7 天
  tipCap           // 每次逾期撥款付給觸發者的小費上限（USDC）
  ref              // bytes32，外部補助編號（例如 Circle grant ID 的 hash）
  closed
  milestones[]     // { amount, deadline, status, submittedAt, evidenceHash, rejected }
}
status: Pending → Submitted → Released
                 ↘（從未提交且過了 deadline）Refunded
         Submitted →（reject，限一次）→ Pending
```

### 3.3 函式（MVP，只有這些）

| 函式 | 誰能呼叫 | 做什麼 | 失敗時的錯誤名稱 |
|---|---|---|---|
| `createAndFund(recipient, vault, amounts[], deadlines[], reviewWindow, tipCap, ref)` | 撥款方 | 用 `transferFrom` 拉進總額，`vault.deposit`，記錄份額 | `VaultNotAllowed`、`BadMilestones` |
| `submit(id, i, evidenceHash)` | 受款方 | 狀態從 Pending 變 Submitted，開始審核倒數 | `NotRecipient`、`WrongStatus` |
| `approve(id, i)` | 撥款方 | 從金庫取出該里程碑的金額，記到受款方 | `NotFunder`、`WrongStatus` |
| `reject(id, i, reasonHash)` | 撥款方 | 退回 Pending，**每個里程碑只能退回一次** | `NotFunder`、`WrongStatus`、`AlreadyRejected` |
| `releaseOverdue(id, i)` | **任何人** | 審核期已過、沒被核可也沒被退回 → 撥給受款方；觸發者拿 `min(tipCap, 目前利息)` | `WrongStatus`、`ReviewWindowOpen` |
| `refund(id, i)` | 撥款方 | 從未提交而且過了 deadline 的里程碑 → 本金退回撥款方 | `NotFunder`、`WrongStatus`、`DeadlineNotPassed` |
| `close(id)` | 任何人 | 所有里程碑都結束後，剩下的利息記給撥款方 | `NotFinished`、`AlreadyClosed` |
| `withdraw()` | 任何人 | 領取自己被記帳的金額 | `NothingToWithdraw` |
| `accruedYield(id)`、`grant(id)`、`milestone(id, i)` | view | 給 app 和拒絕室用 | — |

- 金庫流動性不足（`vault.maxWithdraw` 不夠）時直接 revert `VaultIlliquid`，**任何人都可以之後再試**。MVP 不做「待領」
- 不做仲裁人、爭議分配、多種利息規則，這些列進 roadmap

### 3.4 利息與份額會計
- `accrued(id) = vault.convertToAssets(shares) − principalLeft`（小於 0 時視為 0）
- 撥款或退款：`burned = vault.withdraw(amount, this, this)`，然後 `shares -= burned`、`principalLeft -= amount`
- 小費：同樣用 `withdraw(tip)` 取出，只扣份額、不扣 `principalLeft`，所以**一定來自利息**
- 不變量：`已撥出 + 已退回 + principalLeft == 存入總額`；`convertToAssets(shares) >= principalLeft`（除非金庫本身虧損，此時 close 時照實分配）

### 3.5 Arc 特有的檢查（Studio 產生的程式碼要逐條確認）
1. 只用 ERC-20 介面 `0x3600…0000`（6 位小數），**絕不使用 `msg.value`**
2. 送出的交易 `maxFeePerGas` ≥ 20 Gwei，否則會被靜默丟棄
3. 付款一律 pull；直接轉帳給黑名單地址會 revert 而且照扣 gas
4. 時間戳不保證嚴格遞增：判斷寫 `>=`／`<` 要明確
5. ReentrancyGuard、SafeERC20；ERC-4626 的捨入方向要偏向合約（存入向下、取出向上）
6. 每個狀態變化都 emit 事件，帶 `id` 和 `i`。app 的時間軸、證明頁、拒絕室都靠它

---

## 4. App（在 Arc Studio 裡做）

| 頁面 | 給誰 | 重點 |
|---|---|---|
| **建立補助** | 撥款方 | 受款方、里程碑（金額、期限）、審核天數、小費上限、選金庫（顯示目前 APY） |
| **補助公開頁** `/grant/:id` | 所有人 | 「錢真的在」：已鎖金額、目前價值、已累積利息、每個里程碑的狀態與倒數、每筆交易的 explorer 連結 |
| **受款方動作** | 受款方 | 提交成果（貼上連結 → 算 hash）；**審核期過了就出現「自己按下撥款」按鈕**；領款 |
| **撥款方動作** | 撥款方 | 核可、退回（顯示「只剩 0 次」）、對過期未提交的退款 |
| **拒絕室** `/try-to-take-it` | **評審** | 不用錢包、不花錢：用 `eth_call` 對主網合約模擬「把錢拿走」的各種方式，每一個都顯示合約回的錯誤名稱（見 §5） |
| **證明頁** `/proof` | 評審 | 主網上每一筆交易的表格：動作、金額、gas（USDC）、tx 連結；直接從合約事件讀取 |

---

## 5. 拒絕室（評審的兩分鐘）

學 Writ 的「Try to hijack this order」和 Setoff 的「refusal room」。對**主網上的真實補助**做唯讀模擬，不簽名、不花錢：

| # | 嘗試 | 期望結果 |
|---|---|---|
| 1 | 撥款方想拿回**已提交**的里程碑（`refund`） | `WrongStatus` |
| 2 | 撥款方在期限前想退款 | `DeadlineNotPassed` |
| 3 | 撥款方第二次退回同一個里程碑 | `AlreadyRejected` |
| 4 | 審核期還沒到，有人想強制撥款 | `ReviewWindowOpen` |
| 5 | 陌生人想核可 | `NotFunder` |
| 6 | 陌生人想冒充受款方提交 | `NotRecipient` |
| 7 | 同一個里程碑撥兩次 | `WrongStatus` |
| 8 | 建立補助時指定一個不在白名單的金庫 | `VaultNotAllowed` |
| 9 | 沒有餘額的人想領錢 | `NothingToWithdraw` |
| ✓ | 對照組：一個已經逾期的里程碑，任何人呼叫 `releaseOverdue` | **成功**（模擬結果顯示小費金額） |

頁面標題：「撥款方沒有任何函式可以拿走已提交的錢。試試看。」

---

## 6. 測試計畫

**Studio（testnet，mock 金庫）**
- `MockERC4626`（OpenZeppelin，底層 testnet USDC），加一個 `simulateYield(assets)`
- 單元測試：每個函式的成功路徑；§5 的每一種拒絕各一個測試；小費只來自利息，沒有利息時小費為 0；金庫流動性不足時 revert
- 不變量（fuzz）：`撥出 + 退回 + principalLeft == 存入`；`credit` 總和 == 合約持有的 USDC
- 時間：審核期邊界用 `>=`；重複的時間戳

**Mainnet fork（加分項，Loan Shield 的做法）**：如果 Studio 或本地 Foundry 能 fork Arc mainnet，用真實的 Steakhouse 金庫跑一次 deposit／withdraw 測試

**Mainnet（真實 USDC，小額）**：見 [BUILD_PLAN.md](BUILD_PLAN.md) 第 4 步，目標 **20 筆以上**，而且**包含失敗與退款路徑**

---

## 7. 評分對應

| 權重 | 項目 | Interim 拿分的證據 |
|---|---|---|
| 30% | 真實 USDC 在 mainnet | 證明頁與 README 的交易表（20 筆以上）；每筆存入與取出都經過**真實的 Morpho 金庫**；包含逾期撥款（由另一個錢包觸發並拿到小費）和期限後退款 |
| 25% | Arc Studio 使用 | 合約、mock 金庫、測試、app、拒絕室全部在 Studio 完成；影片拍到 Contracts panel、測試結果、Tracing panel 裡的 deposit 和 withdraw；Studio 專案網址 |
| 25% | 可運作的產品 | 公開 app：建立 → 存入 → 提交 → 核可或逾期撥款 → 領款，全部能跑；拒絕室評審自己點得到 |
| 20% | 潛力 | 補上 Circle 自己列出的 Refund Protocol 缺口；Circle Developer Grants 本身就是依里程碑撥 USDC；延伸到預付承包商薪資、營造保留款；描述裡寫「microgrant 會資助的 4 件事」 |

---

## 8. 風險與對策

| 風險 | 對策 |
|---|---|
| Studio 無法部署到 mainnet，或 ERC-4626 外部呼叫有問題 | BUILD_PLAN 第 0 步先用最小合約實測；沒過就不做 app |
| Morpho 金庫風險（壞帳、暫時流動性不足） | 只允許兩個高流動性金庫；不足時 revert，可以重試；app 與描述清楚揭露風險 |
| demo 金額小，利息幾乎是 0，小費也接近 0 | 誠實寫出 demo 的利息；用整個計畫的規模說明經濟性（$1M 計畫鎖 6 個月 ≈ $1 萬）；demo 保證至少一次小費 > 0（金額稍大、放久一點） |
| 審核期 demo 用 10 分鐘 | 描述寫明「正式建議 7 天，demo 縮短是為了在主網演完整流程」 |
| 和託管、里程碑作品撞題 | 描述第一段只講差異：等待中生息＋逾期由任何人撥款、利息付小費＋撥款方拿不走已提交的錢 |
| 法規（GENIUS Act 禁止發行方付息） | 收益來自 DeFi 借貸，不是 USDC 發行方；app 內標示 |

---

## 9. Roadmap（不在 MVP，寫在描述的「下一步」）
1. 仲裁人：只能在雙方之間分配，費用有上限
2. 利息歸屬規則：給受款方、按比例分
3. 用 Arc Memo 在撥款方的入金交易帶上補助編號，對帳可以直接匯入會計系統（Tieout 的經驗）
4. App Kit／CCTP：撥款方從其他鏈直接入金
5. 延伸場景：預付承包商薪資、營造保留款

---

## 10. v0.1 → v0.2 改了什麼、為什麼

| 改動 | 原因（出處） |
|---|---|
| Accrue → **Interim** | 第 1 輪已有 Accrue on Arc（#49851），而且也有沉默自動撥款 |
| `autoRelease` → `releaseOverdue`，**觸發者從利息拿小費** | Writ、Doorstep 用 USDC 付觸發者，是第 1 輪最有說服力的 Arc 經濟學；也讓利息有了明確用途 |
| 新增**拒絕室**頁面 | Writ、Setoff、Q2FA 都讓評審免費試攻擊，評審只有幾分鐘 |
| 移除 owner 白名單，改成建構時寫死 | Writ、Doorstep、Loan Shield 都強調 no owner、no upgrade |
| 砍掉仲裁人、爭議、多種利息規則、待領補撥、CSV | 範圍要小才能穩定運作（可運作 25%）；第 1 輪 Tieout 輸在範圍太廣 |
| `createProgram` 加 `fund` 合併成 `createAndFund` | 少一個狀態、少一筆交易、少一種錯誤 |
| `reject` 限一次 | 防止撥款方無限拖延 |
| 主網目標 15 → **20 筆以上**，而且一定包含失敗與退款路徑 | Setoff、Doorstep 的證據都包含失敗路徑 |
| 新增證明頁 | A-Identity、Setoff 都有可點的證據頁 |
| 「為什麼是 Arc」改成只寫成立的 | Tieout 的教訓：不要硬說只有 Arc 能做 |
