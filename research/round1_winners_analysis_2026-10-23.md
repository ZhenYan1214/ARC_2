# 第 1 輪 20 個得獎作品：逐篇拆解與共同模式

> 2026-10-23 從 DoraHacks 讀全文整理（https://dorahacks.io/hackathon/arc-microgrants/winner）。
> 第 1 輪沒有公開評分標準。第 2 輪公開了：真實 USDC 30%、Arc Studio 25%、可運作 25%、潛力 20%。
> 下表依得獎順序排列。「機制」是作品自己一句話講的核心；「主網證據」是描述裡實際給的東西。

## 1. 逐篇拆解

| # | 作品 | 一句話機制 | 自己的合約 | 主網證據 | 為什麼是 Arc（他們的說法） | 真實第三方 | 描述篇幅與特色 |
|---|---|---|---|---|---|---|---|
| 1 | D20DAO | 給 app 和 agent 用的可驗證隨機數，x402 計費 | 有（VRF coordinator 等 3 個） | examples app、explorer、API | 費用用原生 USDC 付；x402 經 Gateway | 有（x402 API 對外開放） | 中篇；有 OpenAPI、agent 指南 |
| 2 | Mercurifi Launch | 一切都用美元計價的代幣發射台，售完的曲線同一筆交易開 Uniswap v4 池 | 有（5 個＋timelock，Sourcify 驗證） | 從某區塊開始已有真實代幣在交易 | gas 是 USDC，不用先買波動幣 | **有（真實使用者在發幣、交易）** | 長篇；寫「合約強制的性質」清單 |
| 3 | Oneliq | USDC 跨鏈統一平台 | 不明 | 只有網站 | — | 不明 | **39 個字**（反證：文字不是關鍵） |
| 4 | Shwup | 押金訂位：到場掃碼秒退，沒到的押金重新分配 | 有 | 「兩分鐘試用」步驟 | msg.value 一筆就訂位、退款不到一分錢、次秒最終性、掃碼器的 gas 用 USDC 一起撥 | 無（自測） | 長篇；Why Arc 用表格；74 個測試含不變量 |
| 5 | lpTOKEN.fun | Uniswap v4 LP 部位變成可交易的 ERC-20 | 有 | app、合約頁 | 「在 Arc 上跑完整產品」 | 有（接 Uniswap v4） | 短篇 |
| 6 | MergePay | GitHub issue 掛 USDC 懸賞，merge 就付，用 GitHub 的簽章當證明 | 有 | **已付出一筆真實懸賞**（issue #1 → PR #2，10 秒到帳） | modexp 讓 RSA 驗章約 1.3 美分；gas 是 USDC，新手不用買幣 | **有（GitHub、427 star 的 repo）** | 長篇；寫「你需要信任什麼」 |
| 7 | AgentPay Receipts | 把 Circle 市集的服務全部真的付過一輪，記錄交付、把收據錨定上鏈 | 有（24 行 ReceiptAnchor） | 12 筆付費、8 家服務商 | 94% 經 Gateway 批次結算，買方沒有自己的交易可指 | **有（8 家真實服務商）** | 中篇；**抓到真問題**（付了錢沒交貨） |
| 8 | Pigeonhole | 每張發票一個沒有私鑰的入金地址，任何人都能一筆掃回金庫 | 有（CREATE2 factory） | 3 筆「該讀的交易」、bench 25 次 | **SELFDESTRUCT 能搬原生 USDC，只有 Arc 做得到** | 無（自測） | 超長；有 /#/judge 評審路線、bench、「我們做錯的事」、「刻意不做的事」 |
| 9 | ArcMandate | 委託 agent 排程付款，合約限額，管理動作需後量子金鑰 | 有 | 兩筆排程付款（瀏覽器關著、服務重啟中間） | SLH-DSA precompile、USDC gas | 無 | 中篇 |
| 10 | Olinea | 只有後量子簽章能放款的個人金庫 | 有（factory） | 存入 0.20、放款 0.10 | 後量子 precompile | 無 | 中篇；「不用帳號、不用錢包就能試」 |
| 11 | Q2FA | 智慧帳戶加上後量子第二因子 | 有 | 真實存入與受保護的提領 | 後量子 precompile | 無 | 短篇；唯讀的「安全模擬」 |
| 12 | Doorstep | 由「指示」產生的 CREATE2 地址，交易所直接打錢也能入帳到對的人 | 有（無 owner） | 6 種情境各一筆（含逾期自動退款） | **送件機器人用 USDC 付 gas 再從款項拿回，不需要 ETH 和價格預言機** | 部分（交易所提領） | 長篇；SDK、API、MCP |
| 13 | Loan Shield | 預備金在貸款快被清算前自動救援 | 有（無 owner） | **真的救了 Borrow Kit 貸款兩次** | Borrow Kit／Morpho 在 Arc、USDC 既是貸款也是 gas | **有（Morpho Blue 市場）** | 短篇；Arc Foundry fork test 打真實主網狀態 |
| 14 | vet402 | Gateway 批次付款的買方收據，Merkle root 錨定上鏈 | 有 | 付了 11 個 Goldsky 端點 | Gateway 批次拿掉了逐筆收據 | **有（11 個真實端點）** | **很短**（約 120 字） |
| 15 | Writ | 定期付款：錢包出資、任何人觸發、只有離線後量子金鑰能改規則 | 有（無 owner、Sourcify 驗證） | 一張任何人都能執行的實際訂單 | 後量子 precompile；**觸發者的 gas 與小費用 USDC 從訂單退回，不需要伺服器** | 無 | 中篇；**「試著劫持這張訂單」按鈕：免費模擬 4 種攻擊全被拒** |
| 16 | ARCANUM | agent 錢包的鏈上政策引擎，超額轉人工 | 有（5 個） | 主網試營運 | USDC gas、Circle 錢包 | 無 | 長篇；收據可離線驗證 |
| 17 | Setoff | 多幣別債務在一個鏈上匯率淨額結算，要嘛全部結清、要嘛全部退回 | 有（Sourcify 驗證） | **成功的一輪＋失敗退款的一輪**，都在主網 | 原生 USDC 不用 approve、確定性最終性讓「全有或全無」可執行 | 部分（Chainlink） | 長篇；**「拒絕室」：17 個唯讀攻擊，15 個必須被拒** |
| 18 | A-Identity | 付款前先驗證 agent，自己的 agent 花費受合約限制 | 有（Sourcify） | 每一步都有 tx（含 Memo、Multicall3From、CCTP） | Circle 全套在同一條鏈、ERC-8004 第一天就在 | 部分 | 超長；「誠實要說的」一節、自己抓到的錯誤 |
| 19 | SiteCheck | 給 agent 用的 8 個 x402 付費工具 | **無**（只有收款地址） | 第一筆 Arc 付款 | 支援 Arc 當付款選項 | 開放給外部 | 中篇；坦白說「還沒有付費客戶」 |
| 20 | Arcurrent | 開放的 agent 託管合約＋有上限的 agent 金庫 | 有（2 個，已驗證） | agent 自己付了 $0.05 | — | 無 | 短篇 |

## 2. 評審明顯在意的東西（從 20 篇歸納）

### A. 主網上「自己的合約」＋可點的交易
- 19/20 有可以指出的鏈上產物（合約或付款）；至少 18 個有自己部署的合約
- 最強的寫法是「**生命週期每一步一個 tx 連結**」（A-Identity、Doorstep、Setoff、Pigeonhole）

### B. 失敗路徑也要在主網上演出來
- Setoff：「一輪成功，一輪失敗而且退款到 wei」
- Doorstep：「逾期自動退款」、「低於最低金額被取消」
- A-Identity：「超額付款被合約拒絕（附錯誤名稱）」
- → 只演快樂路徑不夠。**合約拒絕了什麼**，跟它做了什麼一樣重要

### C. 讓評審自己動手、不用花錢
- Writ：「Try to hijack this order」，4 種攻擊用唯讀模擬全部被拒
- Setoff：「refusal room」，17 個唯讀呼叫
- Pigeonhole：`/#/judge` 評審路線
- Olinea、A-Identity：不用帳號、不用錢包就能試
- → **評審只有幾分鐘，給他一個按鈕**

### D.「任何人都能觸發」＋用 USDC 付觸發者
- Writ：觸發者的 gas 加小費由訂單退回，不需要伺服器
- Doorstep：送件機器人用 USDC 付 gas，再從款項拿回（有上限），不需要 ETH 和價格預言機
- Loan Shield、Pigeonhole：任何人都能呼叫救援或掃回
- → 這是第 1 輪最有說服力的 **Arc 經濟學**：gas、款項、獎勵全部是同一種美元，所以不用預言機、不用第二種代幣

### E.「為什麼是 Arc」講到別的鏈做不到，或至少講清楚差別
- 最強：Pigeonhole（SELFDESTRUCT 搬原生 USDC）、後量子 precompile 那幾個
- 次強而且誠實：Setoff 寫「淨額計算在任何鏈都能跑，結算設計不行」
- → 不需要硬說「只有 Arc」，但要指出**具體哪個機制靠 Arc 才成立**

### F. 信任的形狀：不可升級、沒有 owner、拉取式付款
- Writ、Doorstep、Loan Shield、Olinea 都寫「no owner, no pause, no upgrade」
- Setoff、Shwup、Pigeonhole 都用 pull／credit 付款，避免一個被凍結的地址卡住全部
- Mercurifi 列「合約強制的性質，不是承諾」

### G. 誠實
- 「What we got wrong」（Pigeonhole）、「What is honest to say」（A-Identity）
- 「The four parties are wallets I control」（Setoff）、「No paying customers yet」（SiteCheck）
- 所有人都寫了「未經審計，小額」
- → 誠實不扣分，反而是可信度的來源

### H. 測試數量與種類寫出來
- 不變量、fuzz、fork test 打真實主網狀態（Loan Shield）、E2E 打主網（Pigeonhole）、CI 模擬每條資金路徑（Shwup）

### I. 接 Circle 自己的產品
- Gateway、x402、Agent Marketplace、Borrow Kit、Swap Kit、App Kit、CCTP、ERC-8004、Memo、Multicall3From
- Loan Shield 一次用了 Borrow Kit、Swap Kit、Arc Foundry

### J. 有具體的下一步，最好對到 Circle Grant
- Pigeonhole：「microgrant 會資助的 4 件事（6–8 週）」，以及「刻意不資助的」
- 第 2 輪評分的 20%「潛力」就是這一塊

## 3. 不是關鍵的東西

- **文字長短**：Oneliq 39 字、vet402 約 120 字都得獎
- **文字是否像 AI**：多數得獎作品明顯有 AI 協助，Mercurifi 甚至直接寫「AI-assisted reviews」
- **UI 華麗程度**：描述裡幾乎沒人提設計
- **有沒有真實使用者**：大多數也只有自測（Setoff、Shwup、Pigeonhole、Arcurrent…），但他們的**合約與證據完整**

## 4. 對第 2 輪（Arc Studio）的直接含義

| 第 1 輪模式 | 第 2 輪怎麼用 |
|---|---|
| 自己的合約＋每一步的 tx | 硬性要求：在 Studio 寫合約；30% 看真實 USDC |
| 失敗路徑也上主網 | 自動撥款、取消退款都在主網演一次 |
| 唯讀的「拒絕室」 | 在 Studio 做 app 時，一併做「試著把錢拿走」頁面 |
| 任何人觸發＋USDC 付觸發者 | 自動撥款任何人可觸發，由利息付小費 |
| 誠實與「做錯的事」 | 描述固定段落 |
| Circle 產品 | Morpho（Earn Kit 生態）、Memo、App Kit／CCTP 入金 |
