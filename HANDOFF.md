# HANDOFF — 協作協議（標籤系統 · 詞彙 · 視角 · 角色）

> 目的：定義**怎麼讀整套文件、怎麼 grill、誰拍板**。所有產品/架構文件（PRD、ARCHITECTURE、
> DECISIONS、CONTEXT）共用本檔的標籤、詞彙與視角約定，不各自重述。

---

## 1. 標籤系統（可變動性 + grill 權限）

文件中每個關鍵敘述帶一個標籤，定義它的「可變動性」與 grill 權限：

| 標籤 | 意義 | 工程師可否 grill | grill 後流程 |
|------|------|-----------------|-------------|
| `[success-metric]` | 成功指標 | ❌ 不可動（PM 保留定義權） | — |
| `[constraint]` | 紅線約束 | ⚠️ 可質疑但門檻極高 | 回到 PM 重拍 |
| `[decided]` | 已拍板設計 | ✅ 可 grill（你是 collaborative design 一員） | 提案 → 回到 PM 重拍 |
| `[open]` | 未決 | ✅ 主場 | 解了更新 DECISIONS；重大者知會 PM |
| `[blocking-open]` | **阻塞性未決**：擋住特定 milestone/feature 的決策 | ✅ 主場（但優先） | **必須在指定交付點前解決** |

---

## 2. 決策分類（判斷「什麼是綁定的」）

標籤回答「能不能 grill」；分類回答「這是哪種性質的敘述」。兩者搭配讀：

| 分類 | 意義 | 對應標籤傾向 |
|------|------|-------------|
| **Product Commitment**（產品承諾） | 產品對使用者的承諾，動它要回 PM | `[success-metric]` / `[constraint]` |
| **Architecture Constraint**（架構約束） | 守住設計完整性的紅線，違反即偏離 | `[constraint]` |
| **Implementation Proposal**（實作提案） | 目前的做法建議，工程師可換更好的 | `[decided]`（可 grill） |
| **Open Decision**（待決） | 尚未決定，工程師主場 | `[open]` |
| **Blocking Open Decision**（阻塞待決） | 未決且擋住交付 | `[blocking-open]` |

範例：
- 「user 不編輯 machine-readable schema」→ Product Commitment / Constraint
- 「RawBody 是一等不可變物件」→ Architecture Constraint
- 「S7F26 是 carrier 不是 schema」→ Architecture Constraint
- 「package user/IT 分區」→ Implementation Proposal
- 「IM/郵件通知」→ Implementation Proposal
- 「E172/E173 適用性」→ Open Decision
- 「ParserRegistry metadata 契約」→ **Blocking Open Decision**

---

## 3. 詞彙表（Glossary）

第一次讀請先掃一遍；後文遇到專有詞可回查。

| 詞 | 一句話定義 |
|----|-----------|
| **RawBody** | 機台原始 recipe body（SML/XML/binary）+ 完整 metadata，不可變的一等物件 |
| **URD** | 機台 recipe 實例（sample）+ user 線性標注 = 人操作的 semantic binding surface；MVP 為 **serialized 匯入**（D22，Model B；UI inline 標注為 future Model A）|
| **URD parser** | 解 URD：parser-adapter（sample → AST）+ urd-binder（user 線性標注**按深度優先遍歷序**綁回 AST 節點 → URD-樹），D22 |
| **AST** | URD parser 從 URD 的 sample 遞迴解出的純結構樹（syntactic「地形」），無語意 |
| **Schema (SSOT)** | 從 URD 抽象出的 machine-readable tree，唯一真相來源 |
| **Canonical Recipe Projection** | onboarding 期由 RawBody + Schema 投影出的語意表達，與序列化無關；**非** AAA runtime 的最終 canonical 物件 |
| **CSV** | Schema 編譯出的衍生產物，相容既有 parser，無人手碰 |
| **Semantic Binding** | user 把 item name 標到結構上（本系統做的，Layer 4） |
| **Conformance check** | 驗 recipe 是否符合抽象出的 schema（validation 的一種） |
| **Package** | 交付單位 zip：RawBody + Schema(YAML) + CSV + metadata + report，分 user/IT 區 |
| **地形 / 地圖** | 地形=AST（tree，真實結構）；地圖=flat schema（offset 導航指令） |

---

## 4. 三視角聲明（避免混淆）

整套文件用**三種不同視角**描述系統，它們是**不同軸，不要互相套對**：

1. **能力分層（Layer 0~5）** — 系統有哪些職責、認領哪幾層（見 HYPER_SPEC）。回答「邊界在哪」。
2. **資料流（RawBody → AST → Canonical Projection → Application）** — 一筆 recipe 怎麼被轉換（見 PRD §8 / ARCHITECTURE §2）。回答「資料怎麼流」。
3. **User Story（Flow 1/2/3 的 story）** — 使用者怎麼一步步達成價值（見 PRD §9）。回答「人怎麼用」。

> 三者可對應但**非一一對應**。例如「能力分層的 Layer 2 Parser」不等於「user flow 的某一步」——
> 一個是職責、一個是操作。讀的時候分清楚現在在哪個視角。能力↔資料流的精確對照見 `ARCHITECTURE §4.1`。

---

## 5. 文件協作角色與權責

| 角色 | 持有 | 不持有 |
|------|------|--------|
| **PM + Architect**（上游） | 成功指標定義權、最終拍板權、`[success-metric]`/`[constraint]` 的變更權 | — |
| **接手 Architect + Developer** | collaborative design 一員：可 grill `[decided]`/`[open]`、產出 spec 與 backlog、解 open | 不可單方變更 `[success-metric]`；`[constraint]` 須回 PM 重拍 |

**權責切分**（呼應 PRD §6.1 的系統使用者角色）：文件協作角色（誰能改文件決策）與系統使用者
persona（User process engineer / IT gatekeeper）是不同層面，勿混淆。

---

## 6. Review / grill 流程

1. 工程師對 `[decided]`/`[open]` 提 grill → 帶替代提案與理由。
2. `[open]` 解決 → 更新 `DECISIONS.md`（決策理由補一條 `CONTEXT §2` ADR）；重大者知會 PM。
3. `[decided]` 要改 → 提案回 PM 重拍；拍板後同步 PRD + 對應檔。
4. `[blocking-open]` → 優先處理，必須在其指定交付點（milestone/feature）前解決，否則擋交付。
5. **保持章節號穩定**：搬移內容留 redirect stub，避免炸掉跨檔 inbound link。
