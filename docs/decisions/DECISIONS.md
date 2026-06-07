# DECISIONS — 決策追蹤（open / blocking-open）

> 本檔追蹤**未決 / 進行中**決策 + 怎麼解。
> **已拍板決策 + 理由在 `CONTEXT.md §2`（ADR D1–D22），本檔不複製**（§4 只放導航）。
> 領域背景見 `CONTEXT.md §1`；分期見 `PRD.md §12`；架構見 `ARCHITECTURE.md`。

## 標籤 / 狀態

`[blocking-open]` 擋交付 · `[open]` 不擋 · `[deferred→M1]` 延後 spike｜🔴 卡住 / 🟡 待決 / 🟢 已決 / ⚪ 擱置

---

## 1. Blocking-Open（擋交付，須在指定點前解決）

### BO-1 · ParserRegistry metadata 契約 🟡 — 擋 **F2.5（package 輸出）**
- **狀況**：ParserRegistry 在邊界外（可能是既有 AAA DB 表），但 package 必須 emit 足夠 metadata 供其選 parser。
- **最小契約**（= metadata 層 C，見 `CONTEXT §1.7`）：`bodyFormat, toolType, toolModel, recipeType, schemaVersion, vendorFormatVersion, parserHint, strictMode/compatMode, sourceSystem, ppid, rawBodyChecksum`。
- **怎麼解（取得路徑階梯）**：**A** 有現成 spec → 取得直接對照；**B** 有 owner 無 spec → 排訪談 / 逆向；**C** 無 owner → 風險升級、package 格式暫不凍、用上列最小契約（11 欄）佔位。落地 = 確認既有 AAA / ParserRegistry owner 後取得或逆向實際欄位與對應規則。**= 會議 #1**。
- **未解前**：package format 不視為穩定。

### BO-2 · Submit skill 欄位對應 🟡 — 擋 **F3.4（submit wrapper）**
- **狀況**：shape 已 decided（`ARCHITECTURE §8.1` / PRD §10.1），但**欄位如何對應既有工具的實際介面**待確認。
- **怎麼解**：與既有提交工具 owner 確認；取得路徑同 BO-1 階梯（A 有 spec 取得對照／B 有 owner 無 spec 訪談逆向／C 無 owner 風險升級 + 最小契約佔位）。**= 會議 #1**。

### BO-3 · Deploy skill 介面 🟡 — 擋 **F3.5（deploy 觸發 wrapper, Flow 4）**
- **狀況**：shape 已 decided（PRD §10.2），但**確切 deploy 介面如何對應既有 AAA/BB**待確認（與 BO-2 submit 同性質的邊界對接）。deploy 執行/release 治理在邊界外，本系統只觸發。
- **怎麼解**：與既有 AAA/BB deploy 服務 owner 確認。**= 會議 #1**（與 BO-1/BO-2 同一「外部介面 owner」議題）。

### BO-4 · Runtime Parser 行為契約 🟡 — 擋 **strict 安全性論證（single-sample 安全前提）**
- **狀況**：「parser 不動 + 單一 sample」默默壓在 legacy runtime 行為上，但**從未顯性化**。關鍵：撞到 schema 未涵蓋的結構時，runtime **fail-loud（可補）還是 silent-wrong（默默產錯 recipe）**？後者則 single-sample strict 不安全。
- **要產**：Runtime Parser Behavior Spec（逆向 legacy VB parser ②），回答 7 點（schema-miss / type / missing / extra / count / range / binary，見 `CONTEXT §1.5`）。
- **怎麼解**：逆向既有 VB parser + M1 spike 實測（同級於 #4 待查事實①）。
- **未解前**：strict「單一 sample 可信交付」論證不成立。

---

## 2. 會議待決（本批 · 會後回填）

> 建議與議程排序見下表 + 註記；決議拍板後回填「結論」欄 + 對應檔（`CONTEXT §2 ADR` / `ARCHITECTURE`）。

| # | 議題 | 我的建議 | 結論（會後填）| owner | deadline |
|---|------|---------|--------------|-------|----------|
| #1 | 外部介面 spec owner（BO-1 + BO-2 + BO-3）| 點名 owner + deadline，用最小契約當起點 | ☐ | | |
| #2 | URD binding 內容（欄位細節）| 欄位：3 欄 + 選擇性 override 欄（**模型/載體軸已決＝D22：URD 為 serialized 匯入、Excel 過渡→UI**；本項剩欄位內容）| ☐ | | |
| #3 | Projection 內部表示（= 議題2）| 與 AST **分開**（syntactic / semantic），MVP 薄一層 | ☐ | | |
| #4 | parser / validator | parser 延後 spike（D14）→ 派人查 2 事實；validator **新建** | ☐ | | |

- **#2 選項枚舉**（建議已選，列全供取捨追蹤）：欄位 (i) 3 欄全自動推 ／ **(ii) 3 欄 + 選擇性 override 欄【建議】** ／ (iii) 更豐富標注 schema。**載體/模型軸已由 D22 決**：URD＝serialized 匯入（sample＋線性標注），(a) Excel/CSV 過渡 → (b) 專用 UI〔future inline 標注〕，即原 (c)。
- **#3 選項枚舉**：**A 分開**（AST syntactic + Projection semantic，利跨格式 compare、多一層）**【建議；MVP 先薄一層】** ／ B 共用（少一層，compare 被 syntactic 污染）。
- **#4 待查事實**：① VB Phase ① 能否乾淨叫出**只拿樹**；② authoring **stack**（決定 α/β/γ，見 D14）。
- **#4 α/β/γ 選型**（**方向已收斂 = 自建**；機制仍 `[deferred→M1]`，兩層之分見 `CONTEXT §2 · D14 連動註記`）：
  - **α** 直接用舊 parser AST 當內部表示 — 前提：VB① 能乾淨只回樹；代價：綁 VB runtime + 繼承 runtime 形狀（與 D16/D20 衝突）
  - **β** 從零自建 — 乾淨可測；代價：重實作二十年 edge case + 無 oracle
  - **γ** 自建 + 舊 parser① 當 test oracle — **領先候選**；gate = 上述兩 spike 事實（①②）
- 回填後：#2 → `ARCHITECTURE §4.4 擴充點` + `§3.1`；#3 → `§3 Projection`；皆補一條 `CONTEXT §2 ADR`。
- **排序理由**（議程優先序，原會議 brief §D）：#1 最急（**唯一擋交付且 pod 外**）；#2 blast radius 最廣（擋 authoring 建置 + shift-left 成敗）；#3 data model 地基（architect 主場、對齊即可）；#4 已半決（確認 + 派工）。

---

## 3. Open（不擋交付）

**Schema / abstractor**
- 語意 binding 歧義案例的**完整列舉 + override UX**（mixed run、optional branch、repeated group 局部差異、同義異 label）— 需真實 SML
- **單一 sample 缺「未出現分支」**的風險與處理
- `[open]` **schemaVersion draft↔frozen 語意**（authoring 期 schema 未定版，`CONTEXT §1.7`）
- `[open]` **tool family 結構相似度 → 未來 family-level 空間**（需問 operator/engineer）：同一 tool family 的 recipe 結構是否高度相似？**MVP 不做 family 抽象**——逐案單一 sample onboard、dispatch 僅節點級（C/NC/L…），不選 family-level parser。**若**回饋為「同 family 很像」，未來可開 family-level schema 重用 / dispatch profile 庫（接 D19 future-reuse、本節 `dispatch profile 住哪`）。先留未來空間，不擴 MVP 範圍。
- 🟢 ~~DCPL/TP 動態節點型別~~ → **已決 D18**：禁 dynamic parsing，tree 維 closed-world（`CONTEXT §2`）
- 🟢 ~~罕用 TAG-based（CPL/PVL/TC）支援~~ → **已決 D19**：MVP 不支援，碰撞家族塌成 C/NC（`CONTEXT §2`）；唯 **future 重用時 dispatch profile 住哪**（package 內嵌 vs schema-family 庫）仍 `[open]`
- `[open]` **csv-inferred 節點的 UI 標示方式**：compat→strict bootstrap（升級閘門乙路徑）推得的節點如何在 authoring UI / IT gate 標示其 `provenance=csv-inferred`（配 `ARCHITECTURE §3.1`/`§7.1`、D21）
- `[open]` **`ARCHITECTURE §3.1` `kind` enum vs D18/D19 對帳** — **需工程師嚴謹對帳**：node model 的 `kind` enum 仍列 `CPL/PVL/TC/DCPL/TP`（D19 MVP 砍 / D18 禁），節末設計約束註已說不支援但 enum 未修。對帳須區分 binary `TP` vs dynamic `TP`、D19 future-reuse 是否保留標記而非刪除、是否有 compiler/abstractor/validator 或他處引用。

**Compiler / parser**
- `[deferred→M1]` **AST 取得機制 α/β/γ**（D14；invariant 已鎖，γ 領先）
- `[open]` **compiler 的 OFFSET 雙語意 + B 混合樹**處理（`CONTEXT §1.4.1`）
- `[open]` **enhanced parser adapter 確切邊界**

**Validation / 慣例**
- `[open]` **validation rule code 的 namespace 編碼慣例**（registry 正本 `ARCHITECTURE §5`）
- 既有提交工具「**模擬正確性**」的驗證面向（需調查，PRD §11#4）

**範圍 / 標準 / 後期**
- 格式分布 **80/15/5 inventory** 驗證（PRD §5.1 working assumption）
- **SEMI E172 (SEDD) / E173 (SMN)** 適用性（需對照實際 XML 來源與既有 AAA 介面）
- **unformatted 的 Python preparser 中繼**路徑（VB/Python 兼容、時程）— future
- 現役 schema **主動維護 + DB schema 版本管理**（後期）
- 現役**版本仲裁**（去中心化代價）
- `[open→deferred]` **批次 migration 工具（DB 內既有 CSV+RawBody 大量轉 YAML）：現階段不做。** 理由：csv-inferred 推斷風險正比規模放大；與 **D9**（Projection ≠ runtime canonical）同源——不同階段該用不同形狀的物件，逐筆 bootstrap 需 user 補差/確認語意，批次無法保證正確。future 視需求再評估。

---

## 4. 已決 → 見 `CONTEXT.md §2`（導航，不複製）

D1 SSOT=tree(YAML)·CSV衍生 ｜ D2 不動 VB·wrap adapter ｜ D3 strict/compat+升級閘門 ｜ D4 MVP=formatted ｜ D5 Semantic Binding L2 ｜ D6 歧義不靜默猜 ｜ D7 單一 sample ｜ D8 RawBody 一等不可變 ｜ D9 Projection≠runtime canonical ｜ D10 package-centric ｜ D11 Validation/Comparison 分離 ｜ D12 Parser ① 可 map-free 分離 ｜ D13 dispatch hybrid 推斷 ｜ D14 AST 機制延後 spike ｜ D15 metadata 三層 ｜ D16 abstractor 結構保真（Schema 同構於 AST） ｜ D17 E42 reference-only（payload schema 系統） ｜ D18 禁 dynamic parsing（DCPL/TP）→ closed-world SSOT ｜ D19 MVP 不支援 TAG-based 尾巴（CPL/PVL/TC）→ 碰撞塌成 C/NC ｜ D20 既有 CSV table = 對外相容契約（序列化 byte-identical） ｜ D21 CSV 可逆當地圖：CSV+RawBody bootstrap 骨架樹（compat→strict 加速旁路） ｜ D22 URD=user serialized 輸入（sample+線性標注），URD parser 按遍歷序綁回 AST（MVP=匯入 Model B，後期 UI inline）

---

## 5. 歷史議題對照（舊版 3 議題 → 現狀）

| 舊議題 | 現狀 |
|--------|------|
| 議題1：tree SSOT 怎麼餵 parser（X compiler→flat / Y 改 parser）| 🟢 **已決 = D1**（選 X，compiler 衍生 flat，parser 不動）|
| 議題2：canonical 與 AST 是否同一棵樹 | 🟡 **= §2 #3 Projection 內部表示**（傾向分開）|
| 議題3：operator 編輯介面過渡（Excel / UI）| 🟡 **併入 §2 #2 URD 載體**（傾向 Excel 過渡 → UI）|
