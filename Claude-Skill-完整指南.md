# 為 Claude 打造 Skill 的完整指南

## 目錄

- [介紹](#介紹)
- [第一章：基礎知識](#第一章基礎知識)
- [第二章：規劃與設計](#第二章規劃與設計)
- [第三章：測試與迭代](#第三章測試與迭代)
- [第四章：發佈與分享](#第四章發佈與分享)
- [第五章：模式與疑難排解](#第五章模式與疑難排解)
- [第六章：資源與參考](#第六章資源與參考)
- [附錄 A：快速檢查清單](#附錄-a快速檢查清單)
- [附錄 B：YAML Frontmatter](#附錄-byaml-frontmatter)
- [附錄 C：完整 Skill 範例](#附錄-c完整-skill-範例)

---

## 介紹

Skill 是一組指令——以簡單的資料夾形式打包——用來教導 Claude 如何處理特定的任務或工作流程。Skill 是自訂 Claude 最強大的方式之一。與其在每次對話中重複解釋你的偏好、流程和領域專業知識，Skill 讓你只需教導 Claude 一次，就能在每次使用時受益。

當你有可重複的工作流程時，Skill 特別強大：從規格生成前端設計、以一致的方法論進行研究、建立符合團隊風格指南的文件，或編排多步驟流程。它們能與 Claude 的內建能力（如程式碼執行和文件建立）良好配合。對於正在建構 MCP 整合的開發者，Skill 增加了另一個強大的層次，幫助將原始的工具存取轉化為可靠、優化的工作流程。

本指南涵蓋你建構有效 Skill 所需的一切——從規劃和結構到測試和發佈。無論你是為自己、團隊還是社群建構 Skill，全文都有實用的模式和真實世界的範例。

**你將學到：**

- Skill 結構的技術需求與最佳實踐
- 獨立 Skill 與 MCP 增強工作流程的模式
- 我們在不同使用情境中看到效果良好的模式
- 如何測試、迭代和發佈你的 Skill

**適用對象：**

- 希望 Claude 一致遵循特定工作流程的開發者
- 希望 Claude 遵循特定工作流程的進階使用者
- 希望在組織中標準化 Claude 使用方式的團隊

### 本指南的兩條路徑

建構獨立 Skill？請重點閱讀「基礎知識」、「規劃與設計」以及類別 1-2。增強 MCP 整合？「Skill + MCP」章節和類別 3 適合你。兩條路徑共享相同的技術需求，你可以選擇與你的使用情境相關的內容。

**你能從本指南獲得什麼：** 讀完後，你將能在一次坐下來的時間內建構出一個可運作的 Skill。預計使用 skill-creator 建構和測試你的第一個可運作的 Skill 大約需要 15-30 分鐘。

讓我們開始吧。

---

## 第一章：基礎知識

### 什麼是 Skill？

一個 Skill 是一個包含以下內容的資料夾：

- **SKILL.md**（必要）：帶有 YAML frontmatter 的 Markdown 指令
- **scripts/**（選用）：可執行程式碼（Python、Bash 等）
- **references/**（選用）：按需載入的文件
- **assets/**（選用）：輸出中使用的模板、字型、圖示

### 核心設計原則

#### 漸進式揭示（Progressive Disclosure）

Skill 使用三級系統：

- **第一級（YAML frontmatter）：** 始終載入 Claude 的系統提示中。提供剛好足夠的資訊，讓 Claude 知道每個 Skill 何時應被使用，而不需要將所有內容載入上下文。
- **第二級（SKILL.md 主體）：** 當 Claude 認為該 Skill 與當前任務相關時載入。包含完整的指令和指導。
- **第三級（連結檔案）：** Skill 目錄中附帶的額外檔案，Claude 可以在需要時選擇導航和發現。

這種漸進式揭示在維持專業知識的同時，最大限度地減少了 token 使用量。

#### 可組合性（Composability）

Claude 可以同時載入多個 Skill。你的 Skill 應該能與其他 Skill 良好配合，而非假設它是唯一可用的能力。

#### 可攜性（Portability）

Skill 在 Claude.ai、Claude Code 和 API 之間的運作完全相同。只需建立一次 Skill，它就能在所有介面上無需修改地運作，前提是環境支援該 Skill 所需的任何依賴項。

### 給 MCP 開發者：Skill + 連接器

> 💡 建構不需要 MCP 的獨立 Skill？請跳到「規劃與設計」——之後隨時可以回來。

如果你已經有一個運作中的 MCP 伺服器，最困難的部分已經完成。Skill 是上層的知識層——捕捉你已經知道的工作流程和最佳實踐，讓 Claude 能一致地應用它們。

#### 廚房比喻

MCP 提供專業廚房：工具、食材和設備的存取。
Skill 提供食譜：如何創造有價值成果的逐步指令。

兩者結合，使用者無需自己弄清楚每個步驟就能完成複雜任務。

#### 它們如何協同運作：

| MCP（連接性） | Skill（知識） |
|---|---|
| 將 Claude 連接到你的服務（Notion、Asana、Linear 等） | 教導 Claude 如何有效使用你的服務 |
| 提供即時資料存取和工具調用 | 捕捉工作流程和最佳實踐 |
| Claude **能做什麼** | Claude **應該怎麼做** |

#### 為什麼這對你的 MCP 使用者很重要

**沒有 Skill 時：**

- 使用者連接了你的 MCP 但不知道下一步該做什麼
- 出現「如何用你的整合做 X」的支援工單
- 每次對話都從零開始
- 因為使用者每次的提示方式不同而產生不一致的結果
- 使用者在真正的問題是工作流程指導時責怪你的連接器

**有 Skill 時：**

- 預建的工作流程在需要時自動啟動
- 一致、可靠的工具使用
- 最佳實踐嵌入每次互動中
- 降低你的整合的學習曲線

---

## 第二章：規劃與設計

### 從使用案例開始

在撰寫任何程式碼之前，識別你的 Skill 應該支援的 2-3 個具體使用案例。

**良好的使用案例定義：**

> **使用案例：專案 Sprint 規劃**
>
> **觸發條件：** 使用者說「幫我規劃這個 sprint」或「建立 sprint 任務」
>
> **步驟：**
> 1. 從 Linear 取得當前專案狀態（透過 MCP）
> 2. 分析團隊速率和產能
> 3. 建議任務優先排序
> 4. 在 Linear 中建立帶有適當標籤和估計的任務
>
> **結果：** 完整規劃的 sprint，任務已建立

**自問：**

- 使用者想要完成什麼？
- 這需要什麼多步驟工作流程？
- 需要哪些工具（內建或 MCP）？
- 應該嵌入哪些領域知識或最佳實踐？

### 常見 Skill 使用案例類別

在 Anthropic，我們觀察到三種常見的使用案例：

#### 類別 1：文件與資產建立

**用途：** 建立一致、高品質的輸出，包括文件、簡報、應用程式、設計、程式碼等。

**真實範例：** frontend-design skill（另見 docx、pptx、xlsx 和 ppt 的 Skill）

> 「建立獨特的、生產等級的前端介面，具有高設計品質。用於建構網頁元件、頁面、成品、海報或應用程式。」

**關鍵技巧：**

- 嵌入式風格指南和品牌標準
- 用於一致輸出的模板結構
- 完成前的品質檢查清單
- 不需要外部工具——使用 Claude 的內建能力

#### 類別 2：工作流程自動化

**用途：** 受益於一致方法論的多步驟流程，包括跨多個 MCP 伺服器的協調。

**真實範例：** skill-creator skill

> 「建立新 Skill 的互動式指南。引導使用者完成使用案例定義、frontmatter 生成、指令撰寫和驗證。」

**關鍵技巧：**

- 帶有驗證關卡的逐步工作流程
- 常見結構的模板
- 內建的審查和改進建議
- 迭代精煉循環

#### 類別 3：MCP 增強

**用途：** 增強 MCP 伺服器所提供工具存取的工作流程指導。

**真實範例：** sentry-code-review skill（來自 Sentry）

> 「使用 Sentry 的錯誤監控資料（透過其 MCP 伺服器），自動分析和修復 GitHub Pull Request 中偵測到的錯誤。」

**關鍵技巧：**

- 按順序協調多個 MCP 呼叫
- 嵌入領域專業知識
- 提供使用者原本需要指定的上下文
- 常見 MCP 問題的錯誤處理

### 定義成功標準

如何知道你的 Skill 是否運作正常？

這些是理想目標——粗略的基準而非精確的閾值。追求嚴謹，但接受會有基於感覺的評估成分。我們正在積極開發更健全的測量指導和工具。

#### 量化指標：

- **Skill 在 90% 的相關查詢上觸發**
  - 如何測量：執行 10-20 個應觸發你的 Skill 的測試查詢。追蹤它自動載入的次數 vs. 需要明確調用的次數。
- **在 X 次工具呼叫內完成工作流程**
  - 如何測量：比較啟用和未啟用 Skill 時的相同任務。計算工具呼叫次數和消耗的總 token 數。
- **每個工作流程 0 次失敗的 API 呼叫**
  - 如何測量：在測試執行期間監控 MCP 伺服器日誌。追蹤重試率和錯誤代碼。

#### 質化指標：

- **使用者不需要提示 Claude 下一步**
  - 如何評估：在測試期間，記錄你需要重新引導或澄清的頻率。向 Beta 使用者徵求回饋。
- **工作流程在無使用者修正的情況下完成**
  - 如何評估：執行相同的請求 3-5 次。比較輸出的結構一致性和品質。
- **跨會話結果一致**
  - 如何評估：新使用者能否在最少指導下首次嘗試就完成任務？

### 技術需求

#### 檔案結構

```
your-skill-name/
├── SKILL.md              # 必要 - 主要 Skill 檔案
├── scripts/              # 選用 - 可執行程式碼
│   ├── process_data.py   # 範例
│   └── validate.sh       # 範例
├── references/           # 選用 - 文件
│   ├── api-guide.md      # 範例
│   └── examples/         # 範例
└── assets/               # 選用 - 模板等
    └── report-template.md # 範例
```

#### 關鍵規則

**SKILL.md 命名：**

- 必須完全是 SKILL.md（區分大小寫）
- 不接受任何變體（SKILL.MD、skill.md 等）

**Skill 資料夾命名：**

- 使用 kebab-case：`notion-project-setup` ✅
- 不要用空格：`Notion Project Setup` ❌
- 不要用底線：`notion_project_setup` ❌
- 不要用大寫：`NotionProjectSetup` ❌

**不要有 README.md：**

- 不要在 Skill 資料夾內包含 README.md
- 所有文件都放在 SKILL.md 或 references/ 中
- 注意：透過 GitHub 發佈時，你仍然需要一個倉庫層級的 README 供人類使用者閱讀——見「發佈與分享」。

### YAML Frontmatter：最重要的部分

YAML frontmatter 是 Claude 決定是否載入你的 Skill 的依據。務必做好這部分。

**最小必要格式：**

```yaml
---
name: your-skill-name
description: 它做什麼。當使用者要求 [特定短語] 時使用。
---
```

這就是你開始所需的一切。

#### 欄位需求

**name（必要）：**

- 僅限 kebab-case
- 不要有空格或大寫
- 應與資料夾名稱匹配

**description（必要）：**

- 必須同時包含：
  - Skill 做什麼
  - 何時使用它（觸發條件）
- 1024 字元以內
- 不要有 XML 標籤（`<` 或 `>`）
- 包含使用者可能會說的特定任務
- 如果相關，提及檔案類型

**license（選用）：**

- 如果將 Skill 開源則使用
- 常見：MIT、Apache-2.0

**compatibility（選用）：**

- 1-500 字元
- 指示環境需求：例如目標產品、所需系統套件、網路存取需求等。

**metadata（選用）：**

- 任何自訂的鍵值對
- 建議：author、version、mcp-server
- 範例：

```yaml
metadata:
  author: ProjectHub
  version: 1.0.0
  mcp-server: projecthub
```

#### 安全限制

**Frontmatter 中禁止的內容：**

- XML 角括號（`< >`）
- 名稱中包含「claude」或「anthropic」的 Skill（保留用）

原因：Frontmatter 出現在 Claude 的系統提示中。惡意內容可能注入指令。

### 撰寫有效的 Skill

#### description 欄位

引用 Anthropic 的工程部落格：「這些 metadata……提供剛好足夠的資訊，讓 Claude 知道每個 Skill 何時應被使用，而不需要將所有內容載入上下文。」這是漸進式揭示的第一級。

**結構：**

`[它做什麼] + [何時使用它] + [關鍵能力]`

**良好 description 範例：**

```yaml
# 好 - 具體且可操作
description: 分析 Figma 設計檔案並生成開發交接文件。當使用者上傳 .fig 檔案、要求「設計規格」、「元件文件」或「設計轉程式碼交接」時使用。

# 好 - 包含觸發短語
description: 管理 Linear 專案工作流程，包括 sprint 規劃、任務建立和狀態追蹤。當使用者提及「sprint」、「Linear 任務」、「專案規劃」或要求「建立 ticket」時使用。

# 好 - 清晰的價值主張
description: PayFlow 的端到端客戶入門工作流程。處理帳戶建立、付款設定和訂閱管理。當使用者說「入門新客戶」、「設定訂閱」或「建立 PayFlow 帳戶」時使用。
```

**不良 description 範例：**

```yaml
# 太模糊
description: 幫助處理專案。

# 缺少觸發條件
description: 建立精密的多頁文件系統。

# 太技術化，沒有使用者觸發條件
description: 實作具有層級關係的 Project 實體模型。
```

#### 撰寫主要指令

在 frontmatter 之後，用 Markdown 撰寫實際指令。

**建議結構：**

根據你的 Skill 調整此模板。將括號中的部分替換為你的特定內容。

```markdown
---
name: your-skill
description: [...]
---

# 你的 Skill 名稱

## 指令

### 步驟 1：[第一個主要步驟]
清晰解釋會發生什麼。

```bash
python scripts/fetch_data.py --project-id PROJECT_ID
預期輸出：[描述成功的樣子]
```

（根據需要添加更多步驟）

## 範例

**範例 1：[常見情境]**

使用者說：「設定一個新的行銷活動」

動作：
1. 透過 MCP 取得現有活動
2. 使用提供的參數建立新活動

結果：活動已建立並附有確認連結

（根據需要添加更多範例）

## 疑難排解

**錯誤：[常見錯誤訊息]**
原因：[為什麼會發生]
解決方案：[如何修復]

（根據需要添加更多錯誤案例）
```

#### 指令的最佳實踐

**要具體且可操作**

✅ 好：

```
執行 `python scripts/validate.py --input {filename}` 來檢查資料格式。
如果驗證失敗，常見問題包括：
- 缺少必要欄位（將它們加入 CSV）
- 無效的日期格式（使用 YYYY-MM-DD）
```

❌ 差：

```
在繼續之前驗證資料。
```

**清楚引用附帶的資源**

```
在撰寫查詢之前，參考 `references/api-patterns.md` 了解：
- 速率限制指導
- 分頁模式
- 錯誤代碼和處理
```

**使用漸進式揭示**

保持 SKILL.md 專注於核心指令。將詳細文件移至 `references/` 並連結到它。（參見「核心設計原則」了解三級系統如何運作。）

**包含錯誤處理**

```markdown
## 常見問題

### MCP 連線失敗
如果你看到「Connection refused」：
1. 驗證 MCP 伺服器正在執行：檢查 Settings > Extensions
2. 確認 API 金鑰有效
3. 嘗試重新連線：Settings > Extensions > [你的服務] > Reconnect
```

---

## 第三章：測試與迭代

Skill 可以根據你的需求以不同的嚴謹程度進行測試：

- **在 Claude.ai 中手動測試** - 直接執行查詢並觀察行為。快速迭代，無需設定。
- **在 Claude Code 中腳本化測試** - 自動化測試案例，在變更間進行可重複的驗證。
- **透過 Skills API 程式化測試** - 建構針對定義測試集系統性執行的評估套件。

選擇與你的品質要求和 Skill 可見度相匹配的方法。小團隊內部使用的 Skill 與部署給數千名企業使用者的 Skill 有不同的測試需求。

> **專業提示：先在單一任務上迭代再擴展**
>
> 我們發現最有效的 Skill 建立者會在單一具有挑戰性的任務上迭代，直到 Claude 成功為止，然後將成功的方法提取為 Skill。這利用了 Claude 的上下文學習能力，比廣泛測試提供更快的訊號。一旦你有了可運作的基礎，再擴展到多個測試案例以獲得覆蓋率。

### 建議的測試方法

根據早期經驗，有效的 Skill 測試通常涵蓋三個領域：

#### 1. 觸發測試

**目標：** 確保你的 Skill 在正確的時機載入。

**測試案例：**

- ✅ 在明顯的任務上觸發
- ✅ 在改述的請求上觸發
- ❌ 不在無關的主題上觸發

**測試套件範例：**

應觸發：
- 「幫我設定一個新的 ProjectHub 工作區」
- 「我需要在 ProjectHub 中建立一個專案」
- 「為 Q4 規劃初始化一個 ProjectHub 專案」

不應觸發：
- 「舊金山的天氣如何？」
- 「幫我寫 Python 程式碼」
- 「建立一個試算表」（除非 ProjectHub Skill 處理試算表）

#### 2. 功能測試

**目標：** 驗證 Skill 產生正確的輸出。

**測試案例：**

- 產生有效的輸出
- API 呼叫成功
- 錯誤處理正常運作
- 邊界案例已涵蓋

**範例：**

```
測試：建立帶有 5 個任務的專案
前提：專案名稱「Q4 Planning」、5 個任務描述
當：Skill 執行工作流程
則：
- 專案已在 ProjectHub 中建立
- 5 個任務已建立並帶有正確屬性
- 所有任務已連結到專案
- 無 API 錯誤
```

#### 3. 效能比較

**目標：** 證明 Skill 比基準改善了結果。

使用「定義成功標準」中的指標。以下是比較可能的樣子。

**基準比較：**

| 沒有 Skill | 有 Skill |
|---|---|
| 使用者每次提供指令 | 自動執行工作流程 |
| 15 次來回訊息 | 僅 2 次澄清問題 |
| 3 次失敗的 API 呼叫需要重試 | 0 次失敗的 API 呼叫 |
| 消耗 12,000 token | 消耗 6,000 token |

### 使用 skill-creator Skill

skill-creator skill——可在 Claude.ai 的外掛目錄中取得，或下載用於 Claude Code——可以幫助你建構和迭代 Skill。如果你有 MCP 伺服器並知道你的前 2-3 個工作流程，你可以在一次坐下來的時間內建構和測試一個可運作的 Skill——通常在 15-30 分鐘內。

**建立 Skill：**

- 從自然語言描述生成 Skill
- 產生格式正確的帶有 frontmatter 的 SKILL.md
- 建議觸發短語和結構

**審查 Skill：**

- 標記常見問題（模糊的描述、缺少的觸發條件、結構問題）
- 識別潛在的過度/不足觸發風險
- 根據 Skill 的聲明目的建議測試案例

**迭代改進：**

- 在使用你的 Skill 並遇到邊界案例或失敗後，將這些範例帶回 skill-creator
- 範例：「使用此聊天中識別的問題和解決方案來改進 Skill 如何處理 [特定邊界案例]」

**使用方式：**

```
「使用 skill-creator skill 幫我為 [你的使用案例] 建構一個 Skill」
```

注意：skill-creator 幫助你設計和精煉 Skill，但不會執行自動化測試套件或產生量化評估結果。

### 根據回饋進行迭代

Skill 是活文件。計劃根據以下內容進行迭代：

**觸發不足的訊號：**

- Skill 在應該載入時沒有載入
- 使用者手動啟用它
- 關於何時使用它的支援問題

**解決方案：** 為 description 添加更多細節和細微差別——這可能包括關鍵字，特別是技術術語

**過度觸發的訊號：**

- Skill 在無關的查詢時載入
- 使用者停用它
- 對用途感到困惑

**解決方案：** 添加負面觸發條件，更加具體

**執行問題：**

- 結果不一致
- API 呼叫失敗
- 需要使用者修正

**解決方案：** 改進指令，添加錯誤處理

---

## 第四章：發佈與分享

Skill 讓你的 MCP 整合更加完整。當使用者比較連接器時，擁有 Skill 的那些提供了更快的價值路徑，讓你比僅有 MCP 的替代方案更具優勢。

### 當前發佈模式（2026 年 1 月）

**個人使用者如何取得 Skill：**

1. 下載 Skill 資料夾
2. 壓縮資料夾（如有需要）
3. 透過 Claude.ai 的 Settings > Capabilities > Skills 上傳
4. 或放入 Claude Code 的 Skills 目錄

**組織層級的 Skill：**

- 管理員可以在整個工作區部署 Skill（2025 年 12 月 18 日推出）
- 自動更新
- 集中管理

### 透過 API 使用 Skill

對於程式化的使用案例——如建構應用程式、代理或利用 Skill 的自動化工作流程——API 提供對 Skill 管理和執行的直接控制。

**關鍵能力：**

- `/v1/skills` 端點用於列出和管理 Skill
- 透過 `container.skills` 參數將 Skill 添加到 Messages API 請求
- 透過 Claude Console 進行版本控制和管理
- 與 Claude Agent SDK 配合建構自訂代理

**何時透過 API vs. Claude.ai 使用 Skill：**

| 使用案例 | 最佳介面 |
|---|---|
| 終端使用者直接與 Skill 互動 | Claude.ai / Claude Code |
| 開發期間的手動測試和迭代 | Claude.ai / Claude Code |
| 個人臨時工作流程 | Claude.ai / Claude Code |
| 應用程式程式化使用 Skill | API |
| 大規模生產部署 | API |
| 自動化管道和代理系統 | API |

注意：API 中的 Skill 需要 Code Execution Tool beta，它提供 Skill 運行所需的安全環境。

詳細實作請參見：

- Skills API Quickstart
- Create Custom Skills
- Skills in the Agent SDK

### 一個開放標準

我們已將 Agent Skills 發佈為開放標準。如同 MCP，我們相信 Skill 應該在工具和平台之間可攜——同一個 Skill 應該無論你使用 Claude 還是其他 AI 平台都能運作。話雖如此，有些 Skill 設計為充分利用特定平台的能力；作者可以在 Skill 的 compatibility 欄位中註明這一點。我們一直在與生態系統的成員合作制定標準，我們對早期的採用感到興奮。

### 目前建議的方法

首先在 GitHub 上託管你的 Skill，使用公開倉庫、清晰的 README（供人類訪客使用——這與你的 Skill 資料夾分開，Skill 資料夾不應包含 README.md）以及帶有截圖的使用範例。然後在你的 MCP 文件中添加一個章節，連結到該 Skill，解釋為什麼同時使用兩者很有價值，並提供快速入門指南。

**1. 託管在 GitHub**

- 開源 Skill 使用公開倉庫
- 帶有安裝說明的清晰 README
- 使用範例和截圖

**2. 在你的 MCP 倉庫中記錄**

- 從 MCP 文件連結到 Skill
- 解釋同時使用兩者的價值
- 提供快速入門指南

**3. 建立安裝指南**

```markdown
## 安裝 [你的服務] Skill

1. 下載 Skill：
   - 克隆倉庫：`git clone https://github.com/yourcompany/skills`
   - 或從 Releases 下載 ZIP

2. 在 Claude 中安裝：
   - 開啟 Claude.ai > Settings > Skills
   - 點擊「Upload skill」
   - 選擇 Skill 資料夾（已壓縮）

3. 啟用 Skill：
   - 開啟 [你的服務] Skill
   - 確保你的 MCP 伺服器已連線

4. 測試：
   - 問 Claude：「在 [你的服務] 中設定一個新專案」
```

### 定位你的 Skill

你如何描述你的 Skill 決定了使用者是否理解其價值並實際嘗試它。在撰寫關於你的 Skill 時——在 README、文件或行銷材料中——請牢記這些原則。

**聚焦於成果，而非功能：**

✅ 好：

> 「ProjectHub Skill 讓團隊能在幾秒內設定完整的專案工作區——包括頁面、資料庫和模板——而不是花 30 分鐘手動設定。」

❌ 差：

> 「ProjectHub Skill 是一個包含 YAML frontmatter 和 Markdown 指令的資料夾，它呼叫我們的 MCP 伺服器工具。」

**突出 MCP + Skill 的故事：**

> 「我們的 MCP 伺服器讓 Claude 存取你的 Linear 專案。我們的 Skill 教導 Claude 你的團隊的 sprint 規劃工作流程。兩者結合，實現 AI 驅動的專案管理。」

---

## 第五章：模式與疑難排解

這些模式來自早期採用者和內部團隊建立的 Skill。它們代表我們看到效果良好的常見方法，而非規範性的模板。

### 選擇你的方法：問題優先 vs. 工具優先

想像一下 Home Depot。你可能帶著一個問題走進去——「我需要修理一個廚櫃」——然後一位員工指引你找到正確的工具。或者你可能挑選了一把新電鑽，然後問如何將它用於你的特定工作。

Skill 的運作方式相同：

- **問題優先：**「我需要設定一個專案工作區」→ 你的 Skill 按正確順序編排正確的 MCP 呼叫。使用者描述成果；Skill 處理工具。
- **工具優先：**「我連接了 Notion MCP」→ 你的 Skill 教導 Claude 最佳的工作流程和最佳實踐。使用者有存取權限；Skill 提供專業知識。

大多數 Skill 傾向於其中一個方向。知道哪種框架適合你的使用案例有助於你選擇下面的正確模式。

### 模式 1：循序工作流程編排

**使用時機：** 你的使用者需要按特定順序進行的多步驟流程。

**範例結構：**

```markdown
## 工作流程：入門新客戶

### 步驟 1：建立帳戶
呼叫 MCP 工具：`create_customer`
參數：name、email、company

### 步驟 2：設定付款
呼叫 MCP 工具：`setup_payment_method`
等待：付款方式驗證

### 步驟 3：建立訂閱
呼叫 MCP 工具：`create_subscription`
參數：plan_id、customer_id（來自步驟 1）

### 步驟 4：發送歡迎郵件
呼叫 MCP 工具：`send_email`
模板：welcome_email_template
```

**關鍵技巧：**

- 明確的步驟順序
- 步驟之間的依賴關係
- 每個階段的驗證
- 失敗時的回滾指令

### 模式 2：多 MCP 協調

**使用時機：** 工作流程跨越多個服務。

**範例：設計轉開發交接**

```markdown
### 階段 1：設計匯出（Figma MCP）
1. 從 Figma 匯出設計資產
2. 生成設計規格
3. 建立資產清單

### 階段 2：資產儲存（Drive MCP）
1. 在 Drive 中建立專案資料夾
2. 上傳所有資產
3. 生成可分享的連結

### 階段 3：任務建立（Linear MCP）
1. 建立開發任務
2. 將資產連結附加到任務
3. 分配給工程團隊

### 階段 4：通知（Slack MCP）
1. 在 #engineering 發佈交接摘要
2. 包含資產連結和任務參考
```

**關鍵技巧：**

- 清晰的階段分離
- MCP 之間的資料傳遞
- 移至下一階段前的驗證
- 集中式錯誤處理

### 模式 3：迭代精煉

**使用時機：** 輸出品質透過迭代而改善。

**範例：報告生成**

```markdown
## 迭代式報告建立

### 初始草稿
1. 透過 MCP 取得資料
2. 生成初版報告
3. 儲存到臨時檔案

### 品質檢查
1. 執行驗證腳本：`scripts/check_report.py`
2. 識別問題：
   - 缺少的章節
   - 不一致的格式
   - 資料驗證錯誤

### 精煉循環
1. 處理每個識別的問題
2. 重新生成受影響的章節
3. 重新驗證
4. 重複直到達到品質閾值

### 定稿
1. 套用最終格式
2. 生成摘要
3. 儲存最終版本
```

**關鍵技巧：**

- 明確的品質標準
- 迭代改進
- 驗證腳本
- 知道何時停止迭代

### 模式 4：上下文感知的工具選擇

**使用時機：** 相同的成果，根據上下文使用不同的工具。

**範例：檔案儲存**

```markdown
## 智慧檔案儲存

### 決策樹
1. 檢查檔案類型和大小
2. 決定最佳儲存位置：
   - 大檔案（>10MB）：使用雲端儲存 MCP
   - 協作文件：使用 Notion/Docs MCP
   - 程式碼檔案：使用 GitHub MCP
   - 臨時檔案：使用本地儲存

### 執行儲存
根據決策：
- 呼叫適當的 MCP 工具
- 套用服務特定的 metadata
- 生成存取連結

### 向使用者提供上下文
解釋為什麼選擇該儲存方式
```

**關鍵技巧：**

- 清晰的決策標準
- 備用選項
- 選擇的透明度

### 模式 5：領域特定智慧

**使用時機：** 你的 Skill 添加了超越工具存取的專業知識。

**範例：金融合規**

```markdown
## 帶有合規的付款處理

### 處理前（合規檢查）
1. 透過 MCP 取得交易詳情
2. 套用合規規則：
   - 檢查制裁名單
   - 驗證管轄區許可
   - 評估風險等級
3. 記錄合規決策

### 處理
如果合規通過：
- 呼叫付款處理 MCP 工具
- 套用適當的反詐騙檢查
- 處理交易

否則：
- 標記為審查
- 建立合規案件

### 稽核軌跡
- 記錄所有合規檢查
- 記錄處理決策
- 生成稽核報告
```

**關鍵技巧：**

- 領域專業知識嵌入邏輯
- 行動前先合規
- 全面的文件記錄
- 清晰的治理

### 疑難排解

#### Skill 無法上傳

**錯誤：「Could not find SKILL.md in uploaded folder」**

原因：檔案未準確命名為 SKILL.md

解決方案：
- 重新命名為 SKILL.md（區分大小寫）
- 用 `ls -la` 驗證應顯示 SKILL.md

**錯誤：「Invalid frontmatter」**

原因：YAML 格式問題

常見錯誤：

```yaml
# 錯誤 - 缺少分隔符
name: my-skill
description: Does things

# 錯誤 - 未關閉的引號
name: my-skill
description: "Does things

# 正確
---
name: my-skill
description: Does things
---
```

**錯誤：「Invalid skill name」**

原因：名稱有空格或大寫

```yaml
# 錯誤
name: My Cool Skill

# 正確
name: my-cool-skill
```

#### Skill 不觸發

**症狀：** Skill 從不自動載入

**修復：**

修訂你的 description 欄位。參見「description 欄位」中的好/壞範例。

**快速檢查清單：**

- 是否太籠統？（「幫助處理專案」不會起作用）
- 是否包含使用者實際會說的觸發短語？
- 如果適用，是否提及相關的檔案類型？

**除錯方法：**

問 Claude：「你什麼時候會使用 [Skill 名稱] Skill？」Claude 會引用 description 回答。根據缺少的內容調整。

#### Skill 觸發過於頻繁

**症狀：** Skill 在無關的查詢時載入

**解決方案：**

1. 添加負面觸發條件

```yaml
description: CSV 檔案的進階資料分析。用於統計建模、迴歸、聚類。不要用於簡單的資料探索（改用 data-viz Skill）。
```

2. 更加具體

```yaml
# 太廣泛
description: 處理文件

# 更具體
description: 處理 PDF 法律文件進行合約審查
```

3. 澄清範圍

```yaml
description: PayFlow 電商付款處理。專門用於線上付款工作流程，不用於一般財務查詢。
```

#### 指令未被遵循

**症狀：** Skill 載入但 Claude 不遵循指令

**常見原因：**

1. **指令太冗長**
   - 保持指令簡潔
   - 使用項目符號和編號列表
   - 將詳細參考移至單獨檔案

2. **指令被埋沒**
   - 將關鍵指令放在最上面
   - 使用 `## Important` 或 `## Critical` 標題
   - 必要時重複關鍵要點

3. **語言含糊**

```markdown
# 差
確保正確驗證事物

# 好
重要：在呼叫 create_project 之前，驗證：
- 專案名稱非空
- 至少指派一名團隊成員
- 開始日期不在過去
```

**進階技巧：** 對於關鍵驗證，考慮附帶一個以程式化方式執行檢查的腳本，而不是依賴語言指令。程式碼是確定性的；語言解釋則不是。參見 Office Skill 中此模式的範例。

4. **模型「懶惰」** 添加明確的鼓勵：

```markdown
## 效能備註
- 花時間徹底完成這件事
- 品質比速度更重要
- 不要跳過驗證步驟
```

注意：將此添加到使用者提示中比放在 SKILL.md 中更有效。

#### MCP 連線問題

**症狀：** Skill 載入但 MCP 呼叫失敗

**檢查清單：**

1. 驗證 MCP 伺服器已連線
   - Claude.ai：Settings > Extensions > [你的服務]
   - 應顯示「Connected」狀態

2. 檢查認證
   - API 金鑰有效且未過期
   - 已授予適當的權限/範圍
   - OAuth token 已重新整理

3. 獨立測試 MCP
   - 請 Claude 直接呼叫 MCP（不使用 Skill）
   - 「使用 [服務] MCP 取得我的專案」
   - 如果這也失敗，問題在 MCP 而非 Skill

4. 驗證工具名稱
   - Skill 引用了正確的 MCP 工具名稱
   - 檢查 MCP 伺服器文件
   - 工具名稱區分大小寫

#### 大量上下文問題

**症狀：** Skill 似乎很慢或回應品質下降

**原因：**

- Skill 內容太大
- 同時啟用太多 Skill
- 所有內容被載入而非漸進式揭示

**解決方案：**

1. 優化 SKILL.md 大小
   - 將詳細文件移至 references/
   - 連結到參考而非內嵌
   - SKILL.md 保持在 5,000 字以下

2. 減少啟用的 Skill
   - 評估你是否同時啟用了超過 20-50 個 Skill
   - 建議選擇性啟用
   - 考慮為相關能力建立 Skill「套組」

---

## 第六章：資源與參考

如果你正在建構你的第一個 Skill，從 Best Practices Guide 開始，然後根據需要參考 API 文件。

### 官方文件

**Anthropic 資源：**

- Best Practices Guide
- Skills Documentation
- API Reference
- MCP Documentation

**部落格文章：**

- Introducing Agent Skills
- Engineering Blog: Equipping Agents for the Real World
- Skills Explained
- How to Create Skills for Claude
- Building Skills for Claude Code
- Improving Frontend Design through Skills

### 範例 Skill

**公開 Skill 倉庫：**

- GitHub: anthropics/skills
- 包含你可以自訂的 Anthropic 建立的 Skill

### 工具和實用程式

**skill-creator Skill：**

- 內建於 Claude.ai 並可用於 Claude Code
- 可從描述生成 Skill
- 審查並提供建議
- 使用方式：「幫我使用 skill-creator 建構一個 Skill」

**驗證：**

- skill-creator 可以評估你的 Skill
- 詢問：「審查此 Skill 並建議改進」

### 取得支援

**技術問題：**

- 一般問題：Claude Developers Discord 社群論壇

**錯誤回報：**

- GitHub Issues: anthropics/skills/issues
- 請包含：Skill 名稱、錯誤訊息、重現步驟

---

## 附錄 A：快速檢查清單

使用此檢查清單在上傳前後驗證你的 Skill。如果你想更快開始，使用 skill-creator Skill 生成初稿，然後過一遍此清單確保沒有遺漏。

### 開始之前

- [ ] 識別 2-3 個具體使用案例
- [ ] 識別所需工具（內建或 MCP）
- [ ] 審閱本指南和範例 Skill
- [ ] 規劃資料夾結構

### 開發期間

- [ ] 資料夾以 kebab-case 命名
- [ ] SKILL.md 檔案存在（完全正確的拼寫）
- [ ] YAML frontmatter 有 `---` 分隔符
- [ ] name 欄位：kebab-case，無空格，無大寫
- [ ] description 包含「做什麼」和「何時使用」
- [ ] 任何地方都沒有 XML 標籤（`< >`）
- [ ] 指令清晰且可操作
- [ ] 包含錯誤處理
- [ ] 提供範例
- [ ] 清楚連結參考資料

### 上傳之前

- [ ] 測試在明顯任務上觸發
- [ ] 測試在改述請求上觸發
- [ ] 驗證不在無關主題上觸發
- [ ] 功能測試通過
- [ ] 工具整合正常運作（如適用）
- [ ] 壓縮為 .zip 檔案

### 上傳之後

- [ ] 在真實對話中測試
- [ ] 監控觸發不足/過度觸發
- [ ] 收集使用者回饋
- [ ] 迭代 description 和指令
- [ ] 更新 metadata 中的版本

---

## 附錄 B：YAML Frontmatter

### 必要欄位

```yaml
---
name: skill-name-in-kebab-case
description: 它做什麼以及何時使用它。包含特定的觸發短語。
---
```

### 所有選用欄位

```yaml
name: skill-name
description: [必要的描述]
license: MIT                    # 選用：開源授權
allowed-tools: "Bash(python:*) Bash(npm:*) WebFetch"  # 選用：限制工具存取
metadata:                       # 選用：自訂欄位
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [project-management, automation]
  documentation: https://example.com/docs
  support: support@example.com
```

### 安全備註

**允許的：**

- 任何標準 YAML 類型（字串、數字、布林值、列表、物件）
- 自訂 metadata 欄位
- 長描述（最多 1024 字元）

**禁止的：**

- XML 角括號（`< >`）——安全限制
- YAML 中的程式碼執行（使用安全 YAML 解析）
- 以「claude」或「anthropic」前綴命名的 Skill（保留用）

---

## 附錄 C：完整 Skill 範例

有關展示本指南中模式的完整、生產就緒的 Skill：

- **Document Skills** - PDF、DOCX、PPTX、XLSX 建立
- **Example Skills** - 各種工作流程模式
- **Partner Skills Directory** - 查看來自各合作夥伴的 Skill，如 Asana、Atlassian、Canva、Figma、Sentry、Zapier 等

這些倉庫保持最新，並包含超出本文涵蓋範圍的額外範例。克隆它們，根據你的使用案例修改，並將其作為模板使用。
