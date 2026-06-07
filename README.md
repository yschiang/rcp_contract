# Self-Service Recipe Schema Onboarding System — Handoff 包

> 交接給接手工程師 / Claude Code CLI。讓 user 自助 onboard recipe schema
> （shift-left：schema 建立從 IT 移到 user），IT 只做技術 gate。

---

## 定位（一句話）

> **E42 defines the recipe management _service context_; this project defines the recipe _payload schema system_.**

本專案做的是：**解析、驗證、normalize（投影成 canonical）** equipment message body（尤其 SECS **S7F26**）裡的 **recipe payload**，並定義描述這些 payload 的 **schema 系統**。

**不做**：transport protocol、完整 AAA 服務、recipe 生命週期。**SEMI E42 是 recipe 管理服務的*參考模型*，不是 recipe body payload 的 schema 定義來源。**（詳見 `CONTEXT §3.1` + ADR **D17**）

---

## Scope Boundary（in / out）

| 領域 | In scope | Out of scope |
|---|---|---|
| Recipe body 解析 | ✅ | — |
| Recipe schema YAML 定義 | ✅ | — |
| Recipe schema meta-model | ✅ | — |
| Vendor / tool-specific recipe 變體 | ✅（parsing profile）| — |
| SECS message body 取 payload | ⚠️ 只取 body 結構 | 完整 SECS / HSMS protocol stack |
| SEMI E42 生命週期對齊 | reference only | 完整 E42 AAA 實作 |
| Recipe upload / download 服務 | 只在當 test fixture 時 | production AAA 服務 |
| Recipe approval / change workflow | reference only | 企業 workflow 系統 |
| **Transport（HSMS / SECS-I 連線、session）** | — | **明確 out of scope** |

> 對應既有層模型：transport 在 `HYPER_SPEC A.1` 認領層**之下**、owner=BB（`REFACTOR_STANDARD §3.1`）；boundary 見 `HYPER_SPEC A.3`；E42 reference-only 見 `CONTEXT §3.1` / ADR D17。

---

## Spec Taxonomy → 現址（依關注點切，不開獨立 spec 檔）

本 repo 依 **WHAT / substrate / HOW** 切,spec 內容落在現有檔——**不另開檔**(避免重複漂移)。下表是**「內容現址」地圖**;**要產哪些／優先序／擋什麼 ＝ `PRD §12.0`(權威 deliverable);每份 spec 由哪個模組實作 ＝ `ARCHITECTURE §1.1`**。

| 概念 Spec | 範圍 | 現址 |
|---|---|---|
| **Recipe Schema Meta Spec** | 怎麼描述 schema：YAML tree、validation、versioning、mapping | `ARCHITECTURE §3.1` + `§4.3` + `CONTEXT §1.4` |
| **Recipe Body Spec** | S7F26 body / vendor payload：fields, nesting, type, unit, enum, default/null | `CONTEXT §1.2` + `ARCHITECTURE §3.1` |
| **URD Input Format Spec** | user authoring 輸入：**serialized 匯入**（sample + 3 欄線性標注 + convention + override，**D22**）| `PRD §8.1` + `DECISIONS #2` + `ARCHITECTURE §4.3`/§5 |
| **Runtime Parser Behavior Spec** | legacy runtime 對 schema-miss 等的**顯性行為**(逆向,非猜) | `CONTEXT §1.5` + `ARCHITECTURE §9`/§10 |
| **Vendor Adapter Spec** | vendor/tool/model parsing profile、例外、相容規則 | `CONTEXT §1.4.1` + `ARCHITECTURE §7`（compat）|
| Recipe Management Interface Profile | E42 lifecycle（**reference-only,不產出**）| `REFACTOR_STANDARD` + `HYPER_SPEC A.3` + ADR D17 |

> **本次 P1 交付**:Schema Meta／Recipe Body／URD Input／Runtime Behavior;**P2/future**:Vendor Adapter;**不產**:AAA Interface Profile(reference)。兩個 P1 前置依賴外部:URD Input **模型已決(D22 serialized 匯入)、僅欄位細節**靠會議 #2;Runtime Behavior 靠逆向 legacy parser(`DECISIONS BO-4`)。

---

## 術語

沿用既有 glossary（Schema SSOT / Semantic Binding / dispatch profile / RawBody / compat），外部 spec 詞不另立。**唯一要記的雷**：「normalize」= 投影成 Canonical Projection（`project`），**不是**改 schema 樹形（D16）。

---

## 文件地圖

**① 先看（定位 · pre-req）** — 建立全局再進正文
- `docs/architecture/HYPER_SPEC.md` — 體系總綱：能力分層、認領範圍、guardrail、AAA↔PRD 對齊。**必讀**。

**② 正文（主體 spec · 依序讀）**
- `docs/product/PRD.md` — **WHAT**：產品意圖、概念模型、user flow、分期、標準附錄 A.1
- `CONTEXT.md`（repo root）— **領域基底 + 決策**：parser 真實機制、dispatch 契約、metadata、ADR D1–D22、外部標準
- `docs/architecture/ARCHITECTURE.md` — **HOW**：資料流＋文件物件流（§2）、data model（§3）、模組三鏡頭（§4：catalog/拓樸/契約）、validation/package/模式/邊界（§5–§9）
- `docs/decisions/DECISIONS.md` — open / blocking-open 追蹤 + ADR 導航

**③ 特別用途（需要時查）**
- `docs/reference/REFACTOR_STANDARD.md` — 願景層參考框架（整個 AAA 方向 + guardrail）
- `HANDOFF.md`（repo root）— 協作協議：角色權責、grill 權限、review 流程

> 結構：`CONTEXT.md`/`README.md`/`HANDOFF.md` 在 repo root；其餘進 `docs/`（architecture / product / decisions / reference）。
> 線性讀法：① HYPER_SPEC → ② PRD → CONTEXT → ARCHITECTURE → DECISIONS。

---

## 一句話背景（parser 真實模型）

Parser 兩階段、schema-agnostic：先遞迴 parse SML 成 **AST「地形」(tree)**，再用 **flat schema「地圖」(offset)** 對齊貼語意。
痛點根源 = 用 flat 地圖描述 tree 地形。目標 = **SSOT 變成同構於 AST 的 tree**，flat CSV 降為 compiler 衍生輸出，**parser 不動**，user 線性標注習慣不變。
本系統認領 AAA 能力分層的 **Layer 1~4 + Layer 5 前段**（onboarding/authoring/gate）；governance 後段與 BB 通訊在邊界外。**不是重寫 AAA。**

---

## 核心約束（constraint）

1. 不改既有 VB parser core（只 wrap adapter，D2）
2. user 不碰 machine schema（用 URD 當 binding surface，D5；URD ＝ serialized 匯入 sample+線性標注，D22）
3. 單一 SSOT（CSV 是 YAML 衍生，D1；abstractor 結構保真不改樹形，D16）
4. 單一 sample（不做多範例歸納，D7）
5. 不依賴線上中心 DB（package-centric，D10）
6. **carrier ≠ schema**：S7F26 / SECS-II / XML / vendor file 是載體，schema 須經 semantic binding 獨立定義（guardrail 3 / AAA Rule 2）
7. **E42 reference-only**：不實作 E42 AAA 服務、不以 E42 當 body schema、transport out of scope（D17）

---

## 給 Claude Code 的起手式

```
請先讀 HYPER_SPEC → PRD → CONTEXT → ARCHITECTURE，確認理解：
(1) 定位：E42 = 管理服務脈絡；本專案 = recipe payload schema 系統（reference-only，D17）
(2) parser 真實模型：AST=地形 / schema=地圖；SSOT=tree、CSV 衍生、parser 不動
    URD 模型：serialized 匯入（sample + 線性標注），URD parser 按遍歷序綁回 AST（D22）
(3) 三條 guardrail + carrier≠schema + 核心約束
(4) 標籤系統（decided / open / constraint / blocking-open）與 grill 權限（HANDOFF）

未決事項見 DECISIONS（blocking-open #1 外部介面 owner 最關鍵）；決策理由見 CONTEXT §2（D1–D22）。
```
