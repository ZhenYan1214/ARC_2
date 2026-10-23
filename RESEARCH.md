# Arc Microgrants Round 2 — 研究與選題報告

> 整理日期：2026-10-18。資料來源：DoraHacks 活動頁（第 1、2 輪）、studio.arc.io、docs.arc.io、developers.circle.com、arc.io blog、Circle 新聞稿，以及 **explorer.arc.io / rpc.mainnet.arc.io 的鏈上原始數據**（本報告自己查的）。
> 工作方法依照 Felix Rieseberg〈How to be more ambitious〉：Altitude（看目標背後的目標）、Depth（回到最原始的資料、用外部裁判驗證）、Range（跨領域、把名詞換成動詞）。

---

## 1. 比賽本身（Altitude：先看懂「評審真正要什麼」）

| 項目 | 內容 |
|---|---|
| 名額 / 獎金 | 8 個 × 500 USDC（總池 4,000），付在 Arc 上，非稀釋 |
| 時程 | 10/15 開放；**截止 11/1 23:59 ET = 台灣時間 11/2 12:59**（11/1 美國夏令時間結束，換算一致） |
| 審查方式 | **滾動審查，「越早交越早有答案」**，8 個名額可能在截止前就發完 |
| 報名人數 | 約 120 人（第 1 輪 1,145 人、20 個名額、不要求 Arc Studio） |
| 硬性要求 | 在 Arc Studio 寫＋測合約 → 部署 Arc **mainnet** → Studio 做的 app 有公開連結 → 至少 1 筆 mainnet USDC 經過你的合約與 app |
| 要交的東西 | Studio 專案 URL（`studio.arc.io/app/...`）、live app 連結、合約地址＋tx hash（explorer.arc.io）、public repo、**影片（展示 Studio 建構過程＋mainnet 交易）**、說明、GitHub/X/Farcaster 公開檔案 |
| 不合格 | 不是在 Arc Studio 做的、只有 testnet、mockup、簡報、已拿過 Circle/Arc 資助 |

**評分權重（決定策略）**

| 權重 | 項目 | 策略含義 |
|---|---|---|
| 30% | 真實 USDC 在 mainnet 流動 | 不要只有 1 筆自己打給自己的交易。設計成「讓真實的、不同的人」會把 USDC 打進來 |
| 25% | Arc Studio 用得多廣（合約、測試、app） | 全程在 Studio 內完成並錄影；用到 Studio 的特色（Tracing、Contracts panel、ERC-8004/x402 解析） |
| 25% | 可運作的產品 | 範圍要小、要能真的跑 |
| 20% | 能走多遠 | 要能接到 Circle Grant Program（優先領域：agentic、FX、P2P、treasury、預測市場、借貸） |

**目標背後的目標**：不是「做一個 app」，而是「成為 8 個之一 → 進 Circle Grant Program → 變成真的有人用的產品」。

---

## 2. Arc 與 Arc Studio 的現況

### Arc（Circle 的 L1）
- Mainnet **2026-09-16** 上線；chain ID 5042；RPC `https://rpc.mainnet.arc.io`
- USDC 是 gas（約 $0.001/筆 ERC-20 transfer）、確定性次秒最終性、Malachite BFT + 許可制驗證者（BlackRock、Visa、Mastercard、DTCC、ICE、Standard Chartered…）
- 已上線：USDC/EURC/cirBTC/USYC、CCTP V2（domain 26）、Gateway/Unified Balance、Nanopayments、x402 facilitator（`eip155:5042`）、App Kits、**ERC-8004 身分/聲譽/驗證註冊表**、Memo 合約、Multicall3From、StableFX（機構限定）
- **未上線**：Opt-in privacy（隱私）、AgentVM、Circle Paymaster（Arc 不在名單）
- **ERC-8183（Agentic Commerce 工作託管）官方參考合約只在 testnet**（mainnet 上 `0x0747…4583` 無 code，已用 RPC 確認）

### 開發地雷（Studio 生成的程式碼要自己檢查）
1. 原生 USDC 18 位小數，ERC-20 介面 `0x3600…0000` 是 6 位，同一份餘額 — 不要混用 `msg.value` 與 `balanceOf`
2. `maxFeePerGas` < 20 Gwei 的交易會被**靜默丟棄**
3. 轉給黑名單地址會 revert 且照扣 gas → 付款一律用 **pull / claim** 模式
4. `PREVRANDAO = 0`，沒有鏈上隨機數
5. Memo / Multicall3From 只支援 EOA（Safe、4337 錢包會 revert）
6. 時間戳非嚴格遞增，排序用 block number

### Arc Studio
- Circle 的雲端 AI coding agent：描述 → 寫 Solidity → 初步審查與測試 → 部署 → 生成 React app、live preview；有 Contracts panel、Tracing panel
- 能讀鏈上狀態、**解析 ERC-8004 agent、知道 agent 是否可透過 x402 付款**、可貼上既有合約地址生成 app、可從 GitHub 匯入、可推到 GitHub
- **一鍵部署只到 testnet**。要上 mainnet：用**你自己的 Circle Developer 錢包**（Studio 可在你同意下幫你建 API key 與 entity secret）。Studio 不會用自己的錢包部署 mainnet
- 每日免費額度；超過可下載專案用 Circle skills 在本地繼續
- CLI：`npm i -g @circle-fin/arc-studio-cli`，`arc-studio login`、`run --output json`、`apps --json` 可取得專案 URL；可註冊成 Claude Code 的 subagent
- **官方 demo 就是「USDC 營收分潤」**（Revenue Router）→ 單純的 splitter 會被視為抄範例

---

## 3. 鏈上原始數據（Depth：自己去看，而不是相信簡報）

查詢 explorer.arc.io Blockscout API（2026-10-18）：

- 總交易 5,684 萬筆、地址 340 萬、合約 172 萬、今天 67 萬筆；網路利用率約 1.7%
- 交易量前段班：Uniswap UniversalRouter（660 萬）、Permit2、各種 DEX router、SeaDrop、launchpad、memecoin
- **支付類應用幾乎是空的**（已驗證合約中最高的交易數）：

| 類別 | 最多交易的合約 | 交易數 |
|---|---|---|
| Payroll | WizPayPayrollMainnet | 38 |
| Vesting | — | 14 |
| Invoice | InvoiceRegistry | 7 |
| Stream | AccountantWithYieldStreaming | 5 |
| Subscription | — | 0 |
| Refund | PredgeRefundArbiter | 5 |
| Checkout | MercentaCheckout | 5 |
| ERC-8183 AgenticCommerce（他人自行部署） | 多份 | ≤17 |
| Circle 自家 PaymentSettlementV2（付款意圖＋退款） | — | 1,072 |
| StableFX FxEscrow | — | 2,720 |

- 很多本地法幣穩定幣已在 Arc（BRL、MXN、COP、ARS、CLP、ZAR、TRY、GBP、CHF、CAD），但持有人都極少
- 結論：**Arc 的活動被投機佔據，真正的「付款工作流」應用在 mainnet 上幾乎沒有人做成產品**。這是空白，也是評審（Circle）最想看到被填補的地方

---

## 4. Circle / Arc 明確說想要什麼

- **Request for Builders（2026-09）**：可程式化貿易流程（里程碑放款、爭議流程、條件結算）、無國界薪資、**成果市場（貼出目標＋USDC 賞金，驗證完成後放款）**、agent 金庫、參數型保險、家庭帳務、「有授權範圍的錢」
- Mainnet blog：「treasury workflows, escrow systems, and conditional payments, which Arc Studio is designed to help generate」
- Studio 發表文：跨境付款後台、按用量計費、agent 對 agent / 按算力付費、可程式化 payout
- Circle 的 ETHGlobal Arc 賞金：「conditional escrow with onchain dispute + automatic release」、「programmable payroll / vesting in USDC or EURC」

---

## 5. 往屆 Arc 黑客松作品（避免撞題）

- **lablab 2025-11 AI Agents on Arc**：Tiba（醫療帳單）、Arc Pay（agent 的 Apple Pay）、SOLRARC
- **lablab 2026-01 Agentic Commerce**：NewsFacts、AisaEscrow、VibeCard、RSoft Agentic Bank、Arcana
- **ETHGlobal HackMoney 2026**（155 隊，97% 有 agent）：arctan(x) FX DEX、Text-to-Chain、**ArcFlow（薪資金庫＋閒置收益）**、**Versus（AI 創作者按秒付費＋自動分潤）**；被點名的還有 Mand(ate) 里程碑發票託管、SmartRenovate AI 驗收工程託管、NOMA 全球薪資、sugarc SaaS 分潤
- **ETHGlobal Cannes 2026**：Onda（聽歌付 $0.01 給藝人）、PayMate、NanoCrawl、VEIL VPN
- **Microgrants 第 1 輪**：ArcSplit（splitter）、SettleRef（對帳）…（作品未公開）
- Circle 自己的觀察：「大多數人只是把熟悉的商業流程自動化」

→ **AI 判官的 escrow、一般 splitter、x402 paywall、agent 錢包、目錄網站都已經很擠。**

---

## 6. 其他鏈的競品現況

### A. 託管 / 條件付款 / 薪資 / vesting
- **成熟（但都是託管型、法幣優先）**：Deel（1 萬+ 承包商領穩定幣）、Rise（累計 $13.7 億；Trustpilot 抱怨提領卡 6 週、每筆 $10）、Toku、Gusto+zerohash、Remote+Stripe、Request Finance（只有付發票，沒有託管/里程碑/爭議）
- **加密原生正在退場**：**Sablier 2026-07 進入維護模式**（理由：用量崩、AI 讓串流容易被複製、非創投規模）、Superfluid TVL 約 $6M（2025-08 AutoWrap 故障讓 ENS DAO 的串流被清算）、Hedgey 被 Anchorage 收、Utopia 被 Coinbase 收掉、Coinshift 轉做收益
- **Escrow**：Circle Refund Protocol（開源、**未審計**、只支援 EOA）、Coinbase Commerce Payments Protocol（商家刷卡式授權/請款）、Kleros（太重、小額不適用）、Smart Invoice（老舊）、Trustless Work（Stellar）
- **痛點證據**：Arbitrum DAO 服務商等多簽付款約 6 週、每年花 $23–25 萬請簽署人；Upwork 客戶在第 14 天自動放款前要求「修改」拖延、仲裁每方 $291；DFINITY Supernova 得獎者 40 多天沒拿到獎金、贊助商「人間蒸發」；穩定幣付款沒有退款機制

### B. 分潤 / 共同金庫
- Safe（最大，但不是 splitter；nonce 佇列卡住、盲簽 → Bybit 被駭 $15 億）
- Splits.org（已轉型做企業金庫，**不支援 Arc**，創作者分潤體驗沒人顧）
- Squads（Solana）、Stripe Connect 穩定幣 payout（預覽中、費用高）、Zora（創作者那 50% 無法再分給合作者）
- **痛點**：「每筆都看得到，但沒有一筆有標籤」→ 年底對帳像考古；誰付的錢對不上發票

### C. 軟體對軟體 / agent 付款
- x402（Linux Foundation 治理；1.69 億筆，但**真實商業每天只有約 $2.8–4 萬、約一半是洗量**；**沒有退款**；8.6 萬個端點只有 8.56% 活著）
- Circle Nanopayments/Gateway（已上 Arc）、Stripe ACP / Tempo MPP、Google AP2（爭議不在範圍內）、ERC-8004、Virtuals ACP（「評估者問題」）
- **痛點**：付了錢沒交付也拿不回來；沒有「每個任務」的預算控管；多 agent 流水線收入無法自動分潤；一個月一萬次請求要怎麼開發票

### 跨主題共同空白
1. **退款 / 爭議 / 「錢一定會付」的保證** — 自由工作者、商家、agent 全都缺
2. **對帳 / 會計** — Arc 上沒有任何原生工具
3. 現有贏家靠託管型法幣通路，**非託管＋可程式化條件**的那一層沒人做好

---

## 7. 題目候選（依「真實需求 × 評分 × 10 天可行性」排序）

### 🥇 1. PrizeVault — 「獎金先鎖進鏈上，得獎秒到帳」
**一句話**：黑客松、懸賞、社群比賽的主辦方在活動開始前把獎金鎖進合約，所有人都看得到錢真的在；評審名單簽核後秒發給得獎者；若主辦方在截止日前沒公布，資金照**事先公開的規則**處理（例如平分給入圍者或轉到評審多簽），永遠不會悄悄退回主辦方。

- **痛點（有證據）**：獎金拖欠 40 天以上、贊助商失聯、Gitcoin Grants Stack 2025 年收掉、Talent/OnlyDust 退出；參賽者要「相信主辦方的善意」
- **為什麼是 Arc**：次秒最終性＝「宣布即到帳」；gas 是 USDC，得獎者不用另外準備 ETH；Memo 合約可把「比賽 ID / 名次」寫進每筆付款，方便對帳
- **對應主題**：條件付款（A）＋多名得獎者分配（B），三大主題命中兩個
- **評分優勢**：
  - 30% 真實 USDC：你可以**自己辦 2–3 場真的小型懸賞**（社群、實驗室、同學、X 上的 bug bounty），每場都有真實的、不同的人收到 USDC → 不是自己打給自己
  - 20% 潛力：Circle/Arc 自己就一直在辦黑客松、用 USDC 在 Arc 上發獎金 → 評審本身就是潛在客戶；可延伸成 grants / bounty 基礎設施（Gitcoin 留下的空白）
- **差異化**：一般 escrow 是一對一買賣；這是一對多、有截止日 fallback、有公開「已鎖定」證明的獎金專用設計
- **誠實的缺點**：市場比薪資小；需要你真的找得到 2–3 個願意用的小活動
- **⚠️ 10/18 複查後的修正**：
  - 黑客松作品裡已經有很多同類：JudgeBuddy（Hedera，ETHGlobal Cannes 2026）、ENSure（Rootstock）、De-Hack、BountyX（Arbitrum，USDC 多級獎金）、Trustless Work（Stellar 託管 API 專門列出 hackathon 用例）、OpenQ（ETHDenver 2023 的鏈上發獎）。**「獎金鎖鏈上」本身不是新點子**
  - 公開的延遲案例（TRON 黑客松 S3 超過 1 個月、Devpost 最多 60 天、Epic 約 8 個月、Code4rena/Cantina 要先 KYC）**主因多是 KYC、文件、付款處理商與溝通，而不是主辦方沒錢**。單純託管合約解決不了 KYC 那一段
  - Arc mainnet 上已驗證合約搜尋：prize / hackathon / contest / grant 都是 0 個；bounty 類最多 61 筆交易 → Arc 上仍是空白，但「空白」不等於「需要」
  - 結論：信心下調。若要做，價值主張必須改成「解決 KYC 之後的那一段」（地址收集、多簽簽核延遲、多名次分配、對帳證明），並且先找真實主辦方確認他們願意用

### 🥈 2. MilestonePay — 防拖延的里程碑託管（自由工作者跨境收款）
**一句話**：每個里程碑各自鎖款與設期限；工作者提交後開啟審查窗口，**客戶不回應就自動放款**（解決 Upwork 第 14 天拖延）；爭議只凍結那一個里程碑；仲裁人只能在付款方與收款方之間分錢、不能轉給第三人、費用有上限。
- **痛點**：Upwork 拖延與 $291 仲裁費、穩定幣無追索、Rise 卡款 6 週
- **對應**：RFB「里程碑放款、爭議流程、條件結算」＋ETHGlobal Arc 賞金原文
- **缺點**：黑客松裡最擁擠的題型（AisaEscrow、Mand(ate)、ARC Invoice、circlefin/arc-escrow 範例）；要做出明確差異（防拖延規則＋有限權仲裁＋發票連結＋對帳匯出），否則會被當作「熟悉流程」

### 🥉 3. AgentSplit — 讓 x402 的收款方是「分潤合約」
**一句話**：一個 orchestrator agent 對外賣組合服務（研究→摘要→排版），收到的 USDC 依 manifest 自動分給各個子 agent（用 ERC-8004 身分登記）；子步驟失敗可連鎖退款。
- **空白**：x402 的 `payTo` 只能填一個地址；找不到任何多 agent 流水線的分潤產品
- **評分優勢**：Studio 本身會解析 ERC-8004 與 x402 → 25% 的 Studio 使用分很好拿；最新穎，對齊 Circle Grant 的 agentic 優先領域
- **缺點**：現在真實需求偏少（x402 真實商業量小、一半是洗量）；技術風險最高；「真實的不同使用者」最難找

### 4. PayMandate — 「核准一次，自動按時付薪」
一次核准建立付款授權（收款人、頻率、每月上限、結束日），任何人都能觸發當期付款，超過上限或新收款人才要重新核准；用 Memo 寫入發票/薪資單號，匯出 CSV 對帳。填補 Sablier 退場、Arc 上薪資合約最多才 38 筆交易的空白。缺點：DAO 目前不在 Arc 上，真實使用者要從小團隊找。

### 5. Outcome Bounty（ERC-8183 上 mainnet）
RFB 點名的「成果市場」，官方參考合約只在 testnet。缺點：已有人自行部署多份 AgenticCommerce 到 mainnet，且與 1、3 重疊，可作為 1 的進階版本。

### ❌ 不建議當主軸
單純 splitter（Studio 官方 demo＋第 1 輪 ArcSplit）、一般 x402 paywall、只靠 AI 判官的 escrow、agent 錢包、目錄網站、隱私薪資（Arc 隱私未上線）、StableFX RFQ（機構限定）

---

## 8. 推薦與 14 天計畫

**推薦主題：#1 PrizeVault**（進階版可加入 #5 的「提交成果 → 驗收放款」做懸賞）。
理由：痛點有證據、你自己就在黑客松圈子裡（是真實使用者）、最容易產生「多個真實的人收到真實 USDC」（30% 那一項）、評審本身是潛在客戶（20% 那一項）、範圍小到 10 天做得完（25% 那一項）。

| 日期 | 工作 | 外部驗證（Depth：不讓模型自己打分數） |
|---|---|---|
| 10/18–19 | 註冊 Arc Studio＋Circle Developer Console；取得少量 Arc mainnet USDC（交易所提領到 Arc 或用 CCTP 跨鏈）；**先用最小合約試走一次 Studio → mainnet 部署流程** | explorer.arc.io 上看到合約 |
| 10/20–23 | 在 Studio 寫合約＋測試＋app（testnet 迭代），全程錄影 | Studio 測試全過；Tracing panel 看到正確事件 |
| 10/24 | 依第 2 節地雷清單審查程式碼 → 部署 mainnet；從 Studio 推 GitHub、前端上 Vercel | mainnet 第一筆真實 USDC |
| 10/25–27 | 辦 2–3 場真實小懸賞，讓不同的人收到 USDC | explorer 上多個獨立地址收款 |
| 10/27–28 | 剪影片、寫說明、提交（越早越好，滾動審查） | 用一個沒看過的人照說明操作一次 |

**最大風險**：Studio → mainnet 的部署流程（文件說法不一）。所以第 1 天就要驗證，不要等到最後。

---

## 9. 驗證狀態（哪些是親自確認、哪些是轉述）

**親自確認（讀原始頁面或查鏈上）**
- 第 1、2 輪 DoraHacks 頁面全文（規則、評分、截止、名額、報名人數）
- Arc Studio 官網 FAQ 全部展開（mainnet 需自己的 Circle developer 錢包、每日額度、支援 ERC-8004 / x402）
- docs.arc.io 首頁；Felix 的文章
- 鏈上：總量統計、交易量前 50 的合約、各類支付合約的交易數、AgenticCommerce 與 PaymentSettlementV2 原始碼
- RPC 確認 ERC-8004 三個註冊表、Memo、Multicall3From 在 mainnet 有 code；ERC-8183 參考合約地址在 mainnet 無 code
- 抽查：Arc Request for Builders 內容、Sablier 進入維護模式、x402 約 50% 為人造量、真實日成交約 $2.8–4 萬、平均 $0.32
- 獎金託管的競品與延遲原因（見 #1 的修正）

**轉述自研究 agent、未逐條核對**
- 往屆黑客松得獎名單與描述、各競品的用量數字與用戶抱怨（Rise、Upwork、Arbitrum DAO、Superfluid 事故等）、機構合作夥伴名單

**仍然未知（只能動手或問人才知道）**
- Arc Studio → mainnet 部署流程實際能否走通（文件說法互相矛盾）
- Arc Studio 的 preview 能不能當「公開連結」，還是要自己架 Vercel
- 第 1、2 輪實際交了哪些作品（Builds 頁目前隱藏）
- **最重要：任何一個題目都還沒有跟真實潛在使用者確認過需求**。桌上研究只能證明「有人抱怨過」，不能證明「有人會用你的東西」

---

## 10. 競爭者原始數據（2026-10-20 更新，這一節推翻了第 7 節的排序）

DoraHacks 作品列表雖然隱藏，但公開 API `/api/v1/hub/buidls?search=arc` 可查到 **486 個近期與 Arc 相關的作品**（ID 49119–50047，大多是第 1 輪投稿，最新幾個寫明「built in Arc Studio」應是第 2 輪）。完整清單：[research/dorahacks_arc_builds_2026-10-20.txt](research/dorahacks_arc_builds_2026-10-20.txt)

**主題擁擠度（一句話描述含關鍵字的作品數）**：agent 130、x402/按次付費 48、預測市場 39、發票 36、懸賞/獎金 32、託管 31、儲蓄/群組 30、打賞/創作者 22、自由工作者 21、分潤 19、借貸 19、結帳 18、訂閱 16、FX 16、薪資 15

**和我們候選題目直接撞的**：
- MilestonePay → **已有同名作品 #49998（Arc Studio 做的、已上 mainnet、支援 CCTP）**，另有 SafeDeal、Lurro、Heldby、Statio、Demo-first Escrow、ShortWorks、Arc InvoiceLock…
- PrizeVault → Ego（得獎者用 email 領獎）、ArcBounty ×2、BountyBox、Happy Bounty、EarnKit（成果市場）、Stubby（抽獎）、Oppor.fun
- AgentSplit / agent 付款 → AgentEscrow ×2、MonkePay（x402 收款進分潤 vault）、PaidRep（ERC-8183＋ERC-8004）、Leash ×3、AgentGuard、Beacon、SpendGate、Fuel…
- PayMandate → Standing、Standing Orders、Tock、Porter、Writ、Arc Prepaid
- 其他已有：債務抵銷（Setoff、Quits、Contraflow）、儲蓄互助會（Ajo、Potluck、Links、DEODES、GScoop）、群組記帳（Tab、Arclet、pottle）、對帳（Arc Ledger、Ledgerline、Arc Recon、ArcReceipt）

**從數據得到的結論（挑戰原本「找新點子」的假設）**：
1. **在主辦方建議的三個主題裡，已經找不到沒人做過的點子**。用「新穎」取勝不可行
2. 評分 55% 在「真實 USDC 流動（30%）＋可運作（25%）」。抽看的第 1 輪作品（如 ArcProof）坦白寫「同一個 EOA 付款與收款、沒有其他付款人」→ **大多數作品只有自測交易**。有「多個真實、獨立的使用者」就是最大的差異
3. 第 2 輪強制 Arc Studio，第 1 輪 486 件中一句話描述提到 Studio 的只有 4 件 → 第 1 輪大部分作品**不能直接轉投第 2 輪**，第 2 輪實際競爭比表面小（報名 161 人搶 8 名）
4. 地區切角幾乎空白：486 件中只有 1 件針對亞洲（印尼 QRIS），已知的地區型都在拉美與奈及利亞。**台灣/亞洲且你本人接觸得到真實使用者的情境**，是少數同時滿足「真實需求、有真實使用者、別人沒做」的方向
5. 新的勝出策略：**選一個你能找到 5–10 個真實使用者的具體社群 → 用最小的 Arc Studio app 解決他們一個具體的付款問題 → 讓他們在 mainnet 上真的付錢 → 早點交**

---

## 11. 去哪裡找真實使用者（2026-10-20）

前提：你身邊沒有現成社群，所以使用者必須來自「已經在 Arc 上、而且找得到」的人。

**找得到的人：Arc House**（community.arc.io，官方數據：7.5 萬名成員、1 萬名 Architects）
- 地區分會正在大學和城市辦 meetup，現場就有幾十個真實的人，例如：
  - 越南：Da Nang「Build Your First App with Arc Studio」10/23；An Giang 大學「From Campus to Mainnet」10/17
  - 孟加拉：每週 office hours、Build Club
  - 奈及利亞：Lagos 10/17、Enugu 10/24「Building Live with Arc Studio」
  - 巴西 roundtable、倫敦 meetup、ETHGlobal Mumbai 11/5–7
- 正在招募分會成員，11/1 前申請優先。APAC 優先市場：韓、港、日、星、越、印尼、巴基斯坦（**沒有台灣**；但規則允許以「語言或關係」加入最近的分會，例如香港）
- 10/20 新成立「Agentic Economy」群組，文章瀏覽數 12.5 萬

**用這條線驗證過的題目**
- 免費活動 no-show：PheedLoop 統計 1,070 場活動，免費活動中位數約 28% no-show，付費活動約 17%（[PheedLoop](https://pheedloop.com/blog/event-data-lab-report-5)）；Skift 引述的數字是 40–60%（[Skift](https://meetings.skift.com/2025/11/26/no-shows-create-stress-wreak-havoc-on-events/)）
- 前例：Kickback / BlockParty 在 Ethereum 上做過押金報名，2023 年收掉；Unlock 有「I'm Going」（[Unlock](https://unlock-protocol.com/blog/unlock-labs-adds--i-m-going--commitment-feature-to-events-onchain-ticketing-platform)）
- **Arc 上已有 Shwup（#49859）做得很完整**（押金報名、手機掃碼報到、no-show 押金分配），另有 Turnout（#49303，付費票託管）

**結論**
1. 換了切角，還是一樣：每個點子在 486 件作品裡都有近似品。「找新點子」這條路已經走到底
2. 真正稀缺的是**有人實際用**。第 1 輪作品大多只有自測交易
3. 所以選題順序應該倒過來：**先找到一群願意用的人（例如一個 Arc House 分會的下一場 meetup），再為他們做一個最小的 app**，而不是先做 app 再找人
4. 這需要你主動聯絡分會負責人或在 Arc House / Discord 發文。我可以幫你擬訊息，但不會代你發送

---

## 12. 不主動找使用者的前提下：重新評估（2026-10-20）

**重讀評分規則**：30% 那一項寫的是「Real USDC moving on Arc mainnet, shown with contract addresses and transaction hashes」，**沒有要求第三方使用者**。只要有自己完成的完整 mainnet 流程就符合（第 1 輪作品大多也是這樣）。所以不找使用者不會失格，只是少了一個差異點。

**沒有使用者時，哪裡還能拉開差距**
- 30%：自測交易每個人都有。唯一能再拉高的是讓 USDC **付給你控制不了的真實對象**
- 25% Studio 使用：用到 Studio 的特色功能（Build an agent、ERC-8004 解析、x402、Tracing panel），並全程錄影
- 25% 可運作：範圍小、真的能跑
- 20% 潛力：對齊 Circle 現在最推的方向

**一個不用找人的關鍵事實（已確認）**
- Circle 的 x402 facilitator 支援 Arc mainnet（`eip155:5042`，`exact`／`upto`）
- Circle Agent Marketplace（agents.circle.com/services）共 113 個服務、2,856 個 endpoint，**其中 71 個服務接受 Arc 付款**（例如 web search、加密市場數據、BlockRun、Arkham、CoinGecko）
- → 你的 agent 向這些服務付款，就是**真實 USDC 付給獨立第三方**，而且不需要聯絡任何人

**三個方向的評分**

| 方向 | 30% 真實 USDC | 25% Studio | 25% 可運作 | 20% 潛力 | 主要風險 |
|---|---|---|---|---|---|
| **A. 軟體對軟體：任務預算金庫＋向 Arc 上真實 x402 服務付款的 agent** | ★★★ 付給真實第三方 | ★★★ 用到 Build an agent、x402、ERC-8004 | ★★ 技術整合較多 | ★★★ Circle 第一優先（agentic） | agent 類作品最擁擠（130 件）；x402 付款方需 EOA |
| B. 條件付款（里程碑／vesting） | ★ 只有自己 | ★★ Studio 很擅長 | ★★★ 風險最低 | ★★ | 非常擁擠、看起來像「熟悉流程」 |
| C. 分潤 | ★ | ★★ | ★★★ | ★ | 是 Studio 官方 demo |

**推薦：A。** 具體形態：
- 使用者（就是你自己，真實使用）存入一筆 USDC 作為「這個任務的預算」，設定上限、允許的服務清單、截止時間
- 合約依預算撥款給 agent 的錢包；agent 向 Arc 上真實的 x402 服務買資料（例如搜尋＋市場數據）並產出結果
- 任務結束，**沒用完的錢自動退回**；每筆購買都有鏈上收據（可用 Memo 帶任務 ID）
- 解決的是已驗證的痛點：agent 付款沒有「每個任務」的預算控管、x402 付了就無法退款
- 差異化：同類作品（Leash、Bursar、AgentTab…）大多只有自測，或只付給自己架的假服務；A 的每筆交易都付給 Circle 市集上的真實服務

**誠實的限制**
- 沒有第三方使用者，得獎機率比有使用者時低；這只是在你給的前提下最好的選擇，不保證得獎
- 先確認技術可行：Studio 的 agent 能否在 mainnet 用 Circle facilitator 付款給市集上的 Arc 服務（第 1 天就要試）

**時程**
1. **10/20–21**：不管選哪個題目都要做的準備：註冊 Arc Studio＋Circle Developer Console、取得少量 Arc mainnet USDC、用最小合約試走 Studio → mainnet 部署
2. **10/21**：第 1 輪結果公布（官方說「所有決定 10/21 前發出」），看得獎作品校準評審偏好，必要時微調
3. **10/22–25**：在 Studio 建構（合約、測試、agent、app），全程錄影
4. **10/26–27**：mainnet 部署、真實付款、剪影片、提交（滾動審查，越早越好）

---

## 主要來源
- https://dorahacks.io/hackathon/arc-microgrants-2/detail
- https://dorahacks.io/hackathon/arc-microgrants/detail
- https://studio.arc.io （FAQ）
- https://docs.arc.io/llms.txt 、https://docs.arc.io/arc/references/evm-differences.md 、https://docs.arc.io/arc/references/contract-addresses.md
- https://www.arc.io/blog/the-unfinished-business-of-finance-machine-commerce-and-global-money （Request for Builders）
- https://www.arc.io/blog/introducing-arc-studio-the-onchain-coding-agent
- https://www.arc.io/blog/build-a-usdc-revenue-splitting-app-with-arc-studio-no-code-required
- https://www.arc.io/blog/meet-the-arc-track-winners-from-the-hackmoney-2026-hackathon-and-what-we-learned
- https://www.arc.io/blog/meet-the-arc-track-winners-from-ethglobal-cannes-hackathon-and-what-we-learned
- https://ethglobal.com/events/cannes2026/prizes/arc
- https://www.circle.com/blog/refund-protocol-non-custodial-dispute-resolution-for-stablecoin-payments
- https://blog.sablier.com （維護模式公告）、https://forkast.news/half-of-x402s-transaction-volume-is-artificial-the-infrastructure-layer-keeps-building-anyway/
- https://forum.arbitrum.foundation/t/wind-down-the-mss-transfer-payment-responsibilities-to-the-arbitrum-foundation/29279
- https://forum.dfinity.org/t/40-days-since-supernova-and-dfinity-still-has-not-paid-prizes-to-any-of-the-winners-edit-resolved-as-of-late-august/14786
- https://community.upwork.com/t5/Freelancers/Abuse-of-Upwork-14-days-automatic-payment-amp-Upwork-dispute/td-p/1508942
- https://explorer.arc.io/api/v2 （鏈上數據）
- 獎金託管競品：https://ethglobal.com/showcase/judgebuddy-nje6q 、https://ethglobal.com/showcase/ensure-cze9p 、https://hackquest.io/projects/DE-HACK 、https://arbitrum.hackquest.io/projects/Bountyx 、https://docs.trustlesswork.com/trustless-work/use-cases-unlocking-the-potential-of-smart-escrows/grants-bounties-and-hackathons
- 獎金延遲案例：https://forum.trondao.org/t/the-tron-grand-hackathon-season-3-winners-announcement/16091?page=3 、https://community.monday.com/t/hackathon-prizes/13240 、https://forums.unrealengine.com/t/unreal-challenge-animation/1175886?page=6
