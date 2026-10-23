# Arc Microgrants 第 1 輪檢討：Tieout 為什麼沒得獎

> 整理日期：2026-10-23。第 1 輪：20 個名額 × 500 USDC，截止 10/15，結果已公布，Tieout 沒有得獎。
> 依據：Tieout 的完整開發紀錄、DoraHacks 公布的 20 個得獎作品全文（逐篇讀過），以及 ARC_2 的 562 件作品資料集。
> 「原因」都是根據得獎作品歸納出來的推論。主辦方沒有給個別回饋，所以這不是評審的原話。

---

## 1. 我們交了什麼

**Tieout**：Arc 上 USDC 的對帳工具。
- 把每筆付款／收款對到帳本上它結清的那一筆，抓出對不上的
- 錢包餘額對鏈上滾算到 wei
- 匯出 Xero、QuickBooks、ISO 20022 camt.054 的分錄

| 項目 | 內容 |
|---|---|
| 技術 | 只用 Arc 預先部署的 `Memo` 和 `Multicall3From`。**沒有自己的合約**、沒有後端 |
| 主網證據 | 一個虛構市集「PromptBazaar」的 9 個情境，7 筆真實主網交易，手續費 0.018 USDC，錢包都是自己的 |
| 交付內容 | 線上網站、開源 repo（58 個測試）、memo 規格、架構圖等 8 張圖、13 張表格圖、1:42 的英文旁白影片 |
| 描述 | 改了三版：完整版約 2,300 字、精簡版約 1,270 字，第三版改成第一人稱並加入開發過程 |

---

## 2. 得獎的 20 個作品長什麼樣

| 類型 | 數量 | 作品 |
|---|---|---|
| AI agent 付款與 x402 | 8 | D20DAO、AgentPay Receipts、vet402、ARCANUM、A-Identity、SiteCheck、Arcurrent、ArcMandate |
| 後量子簽章（Arc 新上線的 SLH-DSA precompile） | 4 | Olinea、Q2FA、Writ、ArcMandate |
| DeFi | 4 | Mercurifi、lpTOKEN、Loan Shield、Oneliq |
| 付款基礎設施 | 3 | Pigeonhole、Doorstep、Setoff |
| 消費者應用 | 2 | Shwup、MergePay |

（ArcMandate 同時屬於兩類）

**共同點**
1. **至少 18／20 在主網部署了自己的合約**。連最接近我們的 AgentPay Receipts 也部署了一個 24 行的 ReceiptAnchor 合約
2. **「為什麼一定要 Arc」講的是「別的鏈做不到」**，不只是「在 Arc 上比較順」：
   - Pigeonhole：SELFDESTRUCT 能搬動原生 USDC
   - Olinea、Q2FA、Writ：後量子簽章 precompile
   - MergePay：用 modexp 驗證 GitHub 的 RSA 簽章
3. **接到真實的外部系統**：

| 作品 | 接到的真實系統 |
|---|---|
| MergePay | 在真的 GitHub repo 付出懸賞 |
| Loan Shield | 救了真的 Borrow Kit／Morpho 貸款兩次 |
| AgentPay Receipts | 付錢給 Circle Agent Marketplace 上 8 家真實服務商，抓到「付了錢沒交貨」 |
| vet402 | 付了 11 個真實的 API 端點 |
| Doorstep | 交易所提領直接入帳 |

4. **用了 Circle 正在推的產品**：Gateway、x402、Agent Marketplace、Borrow Kit、Swap Kit、App Kit、CCTP、ERC-8004
5. **文字好壞不是關鍵**：Oneliq 整篇描述只有 39 個字也得獎；很多得獎作品的文字比我們更像 AI 寫的

---

## 3. 我們輸在哪裡（依重要程度排序）

| # | 原因 | 說明 |
|---|---|---|
| 1 | **沒有自己的合約** | 我們把「no custom contract」包裝成優點，但投稿表單問的是「contract address or transaction hash」。在評審眼裡，這可能讀成「沒有在 Arc 上部署東西」。**這是設計方向的判斷錯誤** |
| 2 | **「為什麼是 Arc」太弱** | 對帳在任何鏈都能做。我們的描述還寫了 Stellar、Solana Pay、Tempo 早就有類似機制，等於自己承認不是只有 Arc 能做 |
| 3 | **只有自己模擬** | PromptBazaar 是虛構市集，所有錢包都是自己的，沒有任何真實的外部對象 |
| 4 | **沒接 Circle 的新產品** | 只用了 Memo 和 Multicall3From。評審是 Circle，自然偏好幫自家新產品長生態的作品 |
| 5 | **題材不在風口** | 對帳是真需求，但它是後台、月底、會計的工作。這一輪偏好 AI agent 和安全題材 |
| 6 | **範圍太廣** | 付款、請款、對帳、驗證、六種匯出、對帳單模式什麼都做。得獎作品多是一件事做到很深 |
| 7 | **沒有給評審「動手」的入口** | 得獎作品常有「不用錢包就能試」的按鈕：Writ 的劫持模擬、Setoff 的拒絕室、Pigeonhole 的 /#/judge。Tieout 有一鍵 demo，但沒有讓評審親手確認「合約會拒絕什麼」 |

（投稿後 repo 改名讓舊網址變 404 的事，表單已經更新，不是落選原因。）

**值得注意的反證**：AgentPay Receipts 和 vet402 本質上也是「對帳」，兩個都得獎。所以對帳這個方向本身沒錯，差在它們：
- 綁上了 Circle 最熱門的產品（Gateway 小額支付、agent 付款）
- 用真錢實測，抓出了真問題
- 在主網放了合約當證據

---

## 4. 我們做對的事（下次要保留）

- 主網上真的跑過，每個情境都有 explorer 連結
- 把開發中踩到的坑寫成「Findings」，可以回饋給 Arc 文件：
  - 一筆 ERC-20 轉帳會有兩條 Transfer log
  - 原生 Send 只出現在 EIP-7708 log
  - 收款的 memo 由付款方簽名
  - gas 低於 20 gwei 會被靜默丟棄
- 誠實寫出限制和不能保證的事
- 投稿前把網站每個功能實際點過一遍，因此抓到手續費 0.018343 對 0.018344 的顯示錯誤
- 影片的旁白逐句對照畫面核實（抓到「invoice ID」其實是 payment ID）

---

## 5. 過程中的教訓

1. **太晚研究得獎模式。** 我們是先做完產品，比賽結束後才回頭讀得獎作品。應該在選題前，就先讀往屆和同期的得獎作品
2. **把時間花在包裝，而不是證據上。** 我們花很多時間在 UI 設計、動畫、21 張圖、三版描述，但沒有花時間讓真實的外部對象參與
3. **投稿後絕對不要改 repo 名稱或網址。** GitHub Pages 不會自動轉址，所有對外連結會一次失效。這次雖然及時補救，但風險很高
4. **投稿後要用無痕視窗把每個連結點一遍**，包括 DoraHacks 頁面、表單、影片、網站、repo
5. **描述的寫法**：用第一人稱，一個人就寫 I；少用口號式句子；放「做的過程中壞掉什麼、怎麼修」；最後放一段自己的話
6. **第一個情境不要虛構**：如果一定要模擬，至少讓一部分交易對象是真實第三方，例如真實的協議或服務

---

## 6. 下次投稿前的檢查清單

- [ ] 在主網部署**自己的合約**，表單填合約地址
- [ ] 用一句話講出「為什麼只有 Arc 能做」，而且不要在描述裡替競品背書
- [ ] 至少一部分 USDC 流向**你控制不了的真實對象**（協議、服務、真人）
- [ ] 用到 Circle 正在推的產品，並在描述裡點名
- [ ] 範圍收斂成一個一句話說得清的機制
- [ ] 證據表：合約地址、每筆交易 hash 和 explorer 連結
- [ ] 影片少於 3 分鐘：先講問題，再展示真實交易
- [ ] 投稿後不改名、不換網址；用無痕視窗檢查所有連結
- [ ] 確認 DoraHacks 上的狀態是「已提交到這個比賽」
- [ ] 早點交（滾動審查）
