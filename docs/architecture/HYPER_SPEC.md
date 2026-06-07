# HYPER SPEC — Self-Service Recipe Schema Onboarding System 體系總綱

> Status: Draft（與 AAA Standard、PRD 同為 draft，可互相 grill 校準）
> 角色：本文件是 handoff 體系的**最上層**，同時是 ①願景宣告 ②文件地圖 ③對齊表。
> 它不取代任何子文件，只定義它們的範圍、關係、與調和狀態。
>
> 一句話定位：本系統是 **Self-Service Recipe Schema Onboarding System**，
> 實作 recipe schema 能力分層的 authoring 段（Layer 1~4 + Layer 5 前段），
> **不是重寫 AAA**。
>
> **標籤** `[blocking-open]` = 擋住特定 milestone/artifact 的未決決策，必須在指定交付點前解決。
> 目前 blocking-open：①ParserRegistry metadata 契約　②Submit skill 最小契約（見 C.3）。

---

## A. 願景層

### A.0 本系統定位

**Self-Service Recipe Schema Onboarding System** —— 讓 user（process engineer /
equipment owner）**自助** onboard recipe schema，把 schema 建立從 IT 移到 user
（shift-left）。

本系統**不是要重寫 AAA**。它實作 recipe schema 能力分層中的 **authoring 那幾層**，
產出可被既有 AAA / 既有提交工具消費的 schema package。AAA 能力分層（見 A.1）是
**參考框架**，界定本系統認領哪幾層、其餘層是環境。

### A.1 能力分層與認領範圍

```
┌──────────────────────────────────────────────────────────────────┐
│           AAA Recipe 能力分層（參考框架，見 REFACTOR_STANDARD） │
│                                                                    │
│  Layer 0  BB Communication                         ⬜ 環境（BB）    │
│           SECS/GEM · HSMS · transfer                               │
│  ════════════════════════════════════════════════════════════════ │
│  Layer 1  Recipe Body Format        🟦 認領 ◀ MVP focus: formatted；   │
│                                      target: formatted + XML（自描述）  │
│                                      unformatted 需 preparser 中繼=future│
│  Layer 2  Parser / Decoder Contract 🟦 認領                         │
│  Layer 3  Canonical Projection      🟦 認領      ┌─────────────────┐ │
│           （onboarding 期投影）                  │ 此區受認領區     │ │
│  Layer 4  Recipe Schema             🟦 認領（核心）│ guardrail 約束   │ │
│           （schema 規格 + URD 規格）              │ （見 A.2）       │ │
│  ┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄ │ （見 A.2）       │ │
│  Layer 5  Binding & Governance                   └─────────────────┘ │
│     ├ onboarding / authoring / IT gate  🟦 認領（前段）             │
│     └ release / rollback / golden / audit  ⬜ 環境（後段，既有 AAA）│
└──────────────────────────────────────────────────────────────────┘

  🟦 = Self-Service Onboarding System 認領並實作
  ⬜ = 環境（既有 AAA / BB，本系統不實作，但界定邊界）
  ════ = BB / AAA 責任邊界    ┄┄┄ = onboarding 與治理後段的切分線
```

**認領範圍** `[decided]`：本系統 = Layer 1~4 authoring 子集 + Layer 5 前段（onboarding/authoring/gate）。
Layer 3 為 **onboarding-time canonical projection**（非 AAA runtime canonical 物件）。
Layer 0（BB 通訊）與 Layer 5 後段（release/rollback/golden/audit）是環境，不實作（例外：Flow 4 **deploy 觸發**為薄 wrapper，執行仍委派 AAA/BB）。

> **能力 ≠ 模組** `[decided]`：本圖是**能力分層**（系統要做到的職責），不是模組分解。
> 一個能力可能對應多個模組、多個能力也可能共用一個模組。**模組怎麼切屬於落地層，
> 留給接手工程師**（見 HANDOFF 協作協議）。另有「資料流（Ingest→…→應用）」視角，
> 與能力分層的對應關係見 `ARCHITECTURE.md §4.1`（模組×功能×能力層）＋ `§4.2` 對折星型。

### A.2 Guardrail（認領區的最高約束）`[constraint]`

本系統認領的 Layer 1~4 必須守住下列 guardrail，違反即偏離設計：

1. **RawBody 必須是一等 domain object** — 保留原始 recipe body + 完整 metadata，
   供 re-parse / audit / parser 升級 / 重建。
2. **Schema / Binding / Governance 必須分離** —
   Schema 定義「長什麼樣」、Applicability Binding 定義「可用在哪」、Governance 定義「是否核准」。
   （注意：本系統做的是 **Semantic Binding**（標 item name），非 Applicability Binding；見 PRD §8 術語澄清）
3. **通訊載體不是 schema** — S7F26 / formatted SECS-II / XML / vendor file 都是
   **input carrier 或 body format**。AAA schema 必須透過 semantic binding 與 schema 抽象
   **獨立定義**，絕不把載體結構直接當 schema。

> **第三條 guardrail（ParserRegistry metadata-driven）屬 AAA 母架構層，不在本系統認領區。**
> ParserRegistry 可能已是既有 AAA 在 DB 的資料表；本系統提供 format/toolModel metadata
> 供其選 parser，不擁有 registry（見 C.3）。AAA Standard 仍要求它 metadata-driven，
> 但那是母架構的約束，非本系統的 guardrail。
>
> **但 ParserRegistry 所需的 metadata 契約是 package 輸出的 `[blocking-open]`**：雖然
> ParserRegistry 在邊界外，本系統**必須產出足夠 metadata 讓母 AAA / 既有 ParserRegistry
> 選對 parser**。此契約未解決前 package 無用。見 C.3。

### A.3 BB / AAA 責任邊界 `[constraint]`

| 領域 | Owner |
|------|-------|
| 與設備通訊、取得/遞送 recipe body（Layer 0） | BB |
| recipe 內容解讀、schema、驗證、比對、治理（Layer 1~5） | AAA（本系統實作其 authoring 段） |

關鍵規則：**通訊訊息（如 S7F26）是 input carrier，不是 schema。** 所有格式都必須先收斂成
**onboarding-time canonical recipe projection**，才能進行 schema validation、package generation、
IT gate 等 authoring-phase 動作。（此 projection 非 AAA runtime 的最終 canonical 物件。）

---

## B. 文件體系地圖

### B.1 文件清單與職責

| 文件 | 層級 | 職責 | 權威性 |
|------|------|------|--------|
| `HYPER_SPEC.md`（本檔） | 總綱 | 願景 + 地圖 + 對齊 | 體系入口 |
| `REFACTOR_STANDARD.md` | 願景層 | AAA 整體重構方向、guardrail、優先順序 | draft，可 grill |
| `PRD.md` | 執行層 | **本次實作**：schema onboarding toolkit | draft，可 grill；**唯一的執行分期來源** |
| `ARCHITECTURE.md` | 執行層 | toolkit 技術架構 | ✅ 已寫（§0–11）|
| `DECISIONS.md` | 橫貫 | decided/open/constraint + 怎麼 grill | ✅ 已重寫 |
| `HANDOFF.md` | 協議 | 標籤系統、grill 權限、詞彙、視角、角色、交棒流程 | ✅ 已寫 |
| `specs/*`（Schema/URD/VB Parser） | 落地 | 接手工程師填 | 待填 |

### B.2 層級關係

```
HYPER_SPEC（總綱：願景+地圖+對齊）
   │
   ├─ REFACTOR_STANDARD（願景層：整個 AAA 的 WHAT/WHY + guardrail + priority）
   │        │ 本次只實作其中「schema / parser / onboarding-time canonical projection 製作」這一塊
   │        ▼
   └─ PRD（執行層：onboarding toolkit 的 HOW + 唯一 Milestone 分期）
            │
            ├─ ARCHITECTURE（技術架構）
            ├─ DECISIONS（決策追蹤）
            ├─ HANDOFF（協作協議）
            └─ specs（落地細節）
```

### B.3 分期 / 優先序的單一來源原則 `[decided]`

- **AAA Standard 只提供「優先順序（priority）」**，不綁硬性 phase。
- **PRD 的 Milestone 是唯一的執行分期**——要 breakdown、要 review gate 的就是它。
- 兩者不一一對應；AAA priority 是願景級先後，PRD Milestone 是交付級排程。

---

## C. AAA Standard ↔ PRD 對齊表

> 記錄願景層與執行層的對應、重疊、衝突、調和狀態。
> 狀態：✅ 一致 ／ ⚠️ 已調和 ／ 🟡 待調和（open）

### C.1 範圍對應

| AAA Standard | PRD（toolkit）對應 | 狀態 |
|--------------|----------------|------|
| Layer 1 Recipe Body Format | PRD §5 格式支援 | ✅ PRD 是聚焦子集（formatted 為主、target formatted+XML） |
| Layer 2 Parser/Decoder Contract | PRD §8 RawBody→AST / parser adapter；§5 ParserRegistry metadata | ⚠️ ParserRegistry 在邊界外但 metadata 契約 `[blocking-open]`，見 C.3 |
| Layer 3 Canonical Recipe Projection | PRD §8 核心概念模型 | ✅ 已改名 Projection（onboarding 期投影，非 runtime 物件） |
| Layer 4 AAA Recipe Schema | PRD §8 Schema SSOT + Semantic Binding | ✅ 一致 |
| Layer 5 Binding & Governance（前段） | PRD §9 Flow 1/2/3；§11 驗證；§12 Milestone | ⚠️ 只到 onboarding/IT gate，不含 release/rollback/golden |

**結論**（layer-based，不綁 priority）：
**toolkit ≈ AAA Standard Layer 1~4 的 authoring 子集 + Layer 5 onboarding/gate 前段。**
Layer 0（BB 通訊）與 Layer 5 後段（release/rollback/golden/runtime governance）在邊界外。

### C.2 Guardrail 落實狀態

| Guardrail | PRD 狀態 | 調和 |
|-----------|---------|------|
| RawBody first-class | ✅ 已補進 PRD §8 | ✅ 已調和 |
| Schema/Binding/Governance 分離 | ⚠️ PRD 的 binding 是 **Semantic Binding**（標 name），非 AAA 的 **Applicability Binding**（可用在哪）——已加術語澄清區分（PRD §8） | ✅ 已調和（術語澄清後） |
| Communication carrier is not schema | ✅ PRD §3 已明確：S7F26 / formatted message 是 input carrier，不是 schema；本檔 A.2 已列為 guardrail | ✅ 已調和 |
| ParserRegistry metadata-driven | 屬 AAA 母架構層，非本系統 guardrail | ✅ 已調和（移出認領區，見 A.2 + C.3） |

### C.3 待調和項

**Blocking-Open（擋 package 輸出/交付）：**

| 項目 | 說明 | 處理 |
|------|------|------|
| **ParserRegistry metadata 契約** `[blocking-open]` | ParserRegistry 在邊界外（可能已是既有 AAA DB 資料表），但本系統**必須產出足夠 metadata 供其選 parser**。最小契約：`bodyFormat, toolType, toolModel, recipeType, schemaVersion, vendorFormatVersion, parserHint, strictMode/compatMode, sourceSystem, ppid, rawBodyChecksum` | **package format 凍結 / F2.5 package 輸出前必須解決**（PRD §5、§12 已載明） |
| **Submit skill 最小契約** `[blocking-open]` | 本系統與既有工具的邊界契約（input/output） | **F3.4 前須與既有工具確認**（PRD §10.1 已定最小契約） |

**Open（不擋交付）：**

| 項目 | 說明 | 處理 |
|------|------|------|
| **SEMI E172/E173** | PRD §3 列為待評估 | 🟡 需對照實際 XML 來源與既有 AAA 介面 |

> 已調和（移出待調和）：
> - **S7F26 ≠ schema / 載體不是 schema**：PRD §3 + 本檔 A.2 guardrail 3 已明文。
> - **Validation vs Comparison 分離**（Rule 7）：PRD §11 已明文。
> - **Semantic vs Applicability Binding** 同名異義：PRD §8 已加術語澄清。

---

## D. 給接手者的讀法

1. 先讀本檔（HYPER_SPEC）建立全局 → 知道 toolkit 在 AAA 大圖的位置。
2. 讀 `REFACTOR_STANDARD` 理解願景與 guardrail（哪些不可違反）。
3. 讀 `PRD` 知道本次實際要做什麼（唯一執行分期）。
4. 讀 `DECISIONS` 知道哪些可 grill、怎麼 grill。
5. C.3 待調和項是你接手後要優先釐清的。
