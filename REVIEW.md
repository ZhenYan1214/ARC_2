# 第 2 輪計畫審查與改版決定

> 2026-10-23。審查 v0.1（Accrue）的 README、REPORT、SPEC、RESEARCH，並對照：
> - 第 1 輪 20 個得獎作品（[research/round1_winners_analysis_2026-10-23.md](research/round1_winners_analysis_2026-10-23.md)）
> - Tieout 的檢討（[LESSONS_ROUND1.md](LESSONS_ROUND1.md)）
> - 第 2 輪的評分規則
>
> 結果已寫進 [SPEC.md](SPEC.md) v0.2 和 [BUILD_PLAN.md](BUILD_PLAN.md)。

---

## 結論

**題目保留，設計大改。**

「託管的錢在 Morpho 生息，沉默就撥款」的方向是對的，它補上了 Tieout 的三個弱點：
- 有自己的合約
- 有真實第三方（Morpho）
- 接到了 Circle 自己指出的缺口（Refund Protocol 的「currently unproductive」）

第 1 輪得獎的 Loan Shield 也是接上 Arc 上真實的 Morpho 市場，證明評審吃這一套。

但 v0.1 有四個問題：撞名撞題、範圍太大、Arc 的論點不夠、給評審的證據形式太弱。

---

## 問題 → 改法

| # | v0.1 的問題 | 依據 | v0.2 的改法 |
|---|---|---|---|
| 1 | **撞名也撞機制**：第 1 輪已有「Accrue on Arc」（#49851），也有「審核沉默照付」 | DoraHacks 資料集 | 改名 **Interim**（DoraHacks 查無同名）；差異點只講「等待中生息＋利息付小費給觸發者＋撥款方拿不走已提交的錢」 |
| 2 | **範圍太大**：仲裁、爭議、退回次數、三種利息規則、待領補撥、CSV，Studio 一次生成風險高 | 可運作佔 25%；Tieout 輸在範圍太廣；得獎作品多是一件事做到很深 | MVP 只有 8 個函式；其餘列進 roadmap |
| 3 | **利息的角色模糊**：2% APY 在小額時看起來像噱頭 | Writ、Doorstep 用 USDC 付觸發者，是第 1 輪最有說服力的 Arc 經濟學 | 利息有明確用途：**付給替受款方按下逾期撥款的人**。gas、本金、利息、小費都是同一種美元，不需要預言機和第二種代幣 |
| 4 | **「為什麼是 Arc」不夠具體** | Tieout 的教訓：不要硬說只有 Arc；Setoff 的寫法是「算術哪裡都能跑，結算設計不行」 | 只寫成立的四點，並主動承認「生息託管別的鏈也能做」 |
| 5 | **有 owner 白名單** | Writ、Doorstep、Loan Shield 都強調 no owner、no upgrade | 允許的金庫在建構時寫死，合約沒有 owner |
| 6 | **給評審的證據只有交易表** | Writ 的劫持模擬、Setoff 的拒絕室、Pigeonhole 的 /#/judge：評審只有幾分鐘，要給他按鈕 | 新增**拒絕室**（9 種攻擊＋1 個對照組，唯讀、不用錢包）和**證明頁** |
| 7 | **主網流程只演快樂路徑** | Setoff、Doorstep 都在主網演出失敗與退款 | 主網流程包含：陌生錢包逾期撥款並拿到小費、期限後退款、退回一次再核可；總共 20 筆以上 |
| 8 | **還沒開始做**，時程已落後 | 滾動審查，8 個名額 | BUILD_PLAN：10/24 冒煙測試，10/28 交件 |
| 9 | **最大的技術風險沒驗證**：testnet 沒有 Morpho，Studio 到 mainnet 也還沒走過 | SPEC v0.1 §0 | BUILD_PLAN 第 0 步：最小合約在主網存取 1 USDC，沒過就不做 app |
| 10 | 文件前後不一致：`cancel` 兩種簽名、redeem 和 withdraw 混用、REPORT 寫 4% APY、章節順序 | 審查 | SPEC 重寫；REPORT 改名、改成 2%、§6 移到附錄 |

---

## 刻意不做的事（以及原因）

| 不做 | 原因 |
|---|---|
| 仲裁人、爭議 | 會讓合約變複雜，評審也看膩了 AI 判官和仲裁託管 |
| 多種利息歸屬規則 | 一種就能講清楚故事；其他放 roadmap |
| 「待領」補撥機制 | 改成直接 revert、任何人都能重試，少一個狀態 |
| Arc Memo | 合約撥款不能用 Memo（只支援 EOA 包裝），放進 roadmap 的「撥款方入金帶補助編號」 |
| agent／x402 方向 | 第 2 輪的研究已確認 agent 類太擠（130 件），而且 PayOnce 的每個組成都有人做過 |

---

## 還沒解決、只能動手才知道的

1. Studio 部署到 mainnet 的實際流程（第 0 步）
2. Studio 的 preview 能不能當公開連結，或要另外放 Vercel
3. Studio 能不能 fork mainnet 跑測試（可以的話就加 fork test，Loan Shield 的做法）
4. 示範金額下利息多久才會在 6 位小數出現：預估 6 USDC 放一天約 0.0003，所以補助 A 的逾期撥款排在隔天
