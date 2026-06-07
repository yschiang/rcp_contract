# AAA Refactor Standard
## Recipe Format, Schema Hierarchy, and BB/AAA Responsibility Boundary

> 定位：本文件是**願景層**，定義整個 AAA 重構方向與 guardrail。
> 它**不是執行分期**——真正的執行分期見 `PRD.md` 的 Milestone。
> 本文件的「Recommended Refactor Scope」應讀作**優先順序（priority）**，非硬性 phase。
> 體系關係見 `HYPER_SPEC.md`。本文件與 PRD 同為 draft，可被 grill、互相校準。

## 1. Purpose

This document defines the refactor direction for AAA recipe management.

The goal is to separate:

```text
Equipment communication
from
Recipe content governance
```

AAA shall not be refactored as an equipment communication platform.
AAA shall be refactored as a recipe governance platform.

---

## 2. Core Position

```text
BB owns recipe communication.

AAA owns recipe content governance.
```

This means:

| Area | Owner |
|---|---|
| Communicate with equipment | BB |
| Acquire recipe body from equipment | BB |
| Deliver recipe body to equipment | BB |
| Interpret recipe body content | AAA |
| Define recipe schema | AAA |
| Validate recipe content | AAA |
| Compare recipe versions | AAA |
| Approve / release / rollback recipe | AAA |
| Maintain recipe audit and golden baseline | AAA |

---

## 3. Responsibility Boundary

### 3.1 BB Responsibility

BB is responsible for the equipment-facing communication layer.

This includes:

```text
SECS/GEM communication
HSMS / equipment session handling
Formatted process program transfer
Unformatted process program transfer
Recipe upload / download / list / delete execution
Equipment ACK / error / timeout handling
Tool-side recipe inventory acquisition
Technical transfer log
Equipment-specific communication behavior
```

BB shall provide AAA with:

```text
Recipe body
Recipe body format
PPID
Equipment identity
Tool model / equipment type metadata
Technical transfer status
Checksum / size / timestamp where available
Communication result
```

BB does not own recipe business meaning.

---

### 3.2 AAA Responsibility

AAA is responsible for the recipe content and governance layer.

This includes:

```text
Recipe identity model
PPID mapping
Raw recipe body storage
Recipe body format classification
Recipe schema definition
Parser / decoder rule
Canonical recipe object
Parameter model
Validation rule
Comparison rule
Approval workflow
Release control
Rollback control
Golden recipe baseline
Recipe audit trail
Tool capability binding
```

AAA shall not treat equipment communication messages as recipe schema.

For example:

```text
S7F26 is a formatted recipe input carrier.
It is not the AAA canonical recipe schema.
```

---

### 3.3 Boundary Principle

BB owns **recipe communication**.
AAA owns **recipe content governance**.

BB shall provide recipe body and technical metadata based on the equipment-supported communication format, such as formatted process program, unformatted process program, XML file, text file, binary file, or vendor-specific package.

AAA shall not use the communication message itself as the recipe schema.
AAA shall classify the received recipe body by format, parse it through the corresponding schema, and convert it into the AAA canonical recipe object for validation, comparison, approval, release, and audit.

---

## 4. Recipe Standard Support Hierarchy

AAA shall support recipe management through a layered structure.

```text
Layer 0: Equipment Communication Standard
Layer 1: Recipe Body Format
Layer 2: Parser / Decoder Contract
Layer 3: AAA Canonical Recipe Object
Layer 4: AAA Recipe Schema
Layer 5: Binding and Governance
```

---

### 4.1 Layer 0 — Recipe Acquisition / Communication Channel

This layer defines how recipe data is acquired from or delivered to equipment, or
otherwise brought into AAA.

Owner: **BB**（設備通訊路徑）／ **AAA / Onboarding Tool**（manual / 非設備路徑）

Examples:

| Acquisition Category | Description | Owner | AAA Position |
|---|---|---|---|
| SECS/GEM Process Program | Standard equipment communication model | BB | AAA consumes result |
| Formatted Process Program | Structured process program body | BB | AAA parses body |
| Unformatted Process Program | Raw / vendor-specific process program body | BB | AAA stores and decodes body |
| Vendor communication interface | Vendor-specific transfer method | BB | AAA consumes normalized body |
| Manual import | File uploaded by user / engineer | **AAA / Onboarding Tool**（非 BB） | AAA treats as recipe body source |

BB owns the **equipment communication** acquisition paths. Manual / non-equipment
acquisition is **not** owned by BB. AAA does not directly own equipment communication standards.

---

### 4.2 Layer 1 — Recipe Body Format

This layer defines the physical or logical format of the recipe content received by AAA.

Owner: **AAA**

AAA shall support the following body format categories:

| Format Category | Description | AAA Requirement |
|---|---|---|
| Formatted SECS-II | Structured list-based process program body | Convert to canonical object |
| Unformatted / Binary | Raw PPBODY, binary, or vendor-specific body | Store raw body and decode through parser |
| XML | Vendor or fab-defined XML recipe | Validate through XSD or AAA schema mapping |
| JSON / YAML | Structured recipe representation | Validate through AAA schema |
| CSV / Text | Table-based or position-based recipe body | Parse through mapping rule |
| Vendor File Package | Recipe represented by multiple files or archive | Register manifest, checksum, and parser |
| Manual Engineering Import | Manually uploaded recipe body | Must follow same parsing and validation flow |

Key rule:

```text
Recipe body format is not equal to recipe schema.
```

The body format only tells AAA how the recipe is represented.
It does not tell AAA whether the recipe is valid, approved, comparable, or releasable.

---

### 4.3 Layer 2 — Parser / Decoder Contract

This layer converts recipe body into an AAA-readable structure.

Owner: **AAA**

AAA shall maintain a parser registry.

```text
Recipe Body
  → Format Classification
  → Parser / Decoder Selection
  → Parsed Recipe Structure
  → Canonical Recipe Object
```

Parser selection shall be based on:

```text
Body format
Tool type
Tool model
Recipe type
Schema version
Vendor format version if available
```

Parser output must include:

```text
Parsed parameters
Parameter path
Parameter value
Parameter type
Unit where available
Source location in raw body
Parsing warning / error
Unsupported field
Checksum / hash
```

Parser rules shall not be hard-coded into AAA core logic.

Recommended pattern:

```text
AAA Core
  → Parser Registry
    → Format Parser
    → Tool Model Parser
    → Recipe Type Parser
```

---

### 4.4 Layer 3 — AAA Canonical Recipe Object

This layer defines the internal AAA representation of recipe content.

Owner: **AAA**

> **與 onboarding 系統的用詞對齊**（對齊 PRD / HYPER_SPEC）：本層的 **AAA Canonical Recipe
> Object** 是 **AAA runtime 的最終物件**（治理、比對、release、audit 的對象，屬 AAA 母系統）。
> Self-Service Onboarding System 在 authoring 期產出的是 **Canonical Recipe Projection**
> ——由 RawBody + Schema 投影出的 onboarding 期表示，用於 schema validation 與 package
> generation。**Projection 不是 runtime Object**；它在提交後由 AAA 收斂 / 對應成 runtime
> canonical object。兩者是同一條收斂鏈上的不同階段（onboarding-time → runtime），非同義詞、非衝突。

All recipe body formats shall converge into one canonical model.

Minimum canonical object:

```text
Recipe ID
PPID
Recipe version
Recipe type
Tool type
Tool model
Schema version
Raw body reference
Parsed parameter list
Validation result
Comparison result
Approval status
Release status
Checksum / hash
Audit trail
```

Minimum parameter model:

```text
Parameter name
Parameter path
Data type
Unit
Value
Default value
Required / optional
Allowed range
Allowed enum
Compare policy
Validation policy
Source location in raw body
```

Key rule:

```text
Formatted, unformatted, XML, JSON, text, binary, and vendor package recipes
must all be converted into the AAA canonical recipe object before governance.
```

---

### 4.5 Layer 4 — AAA Recipe Schema

This layer defines what a valid recipe should look like.

Owner: **AAA**

Recommended schema hierarchy:

```text
Recipe Schema Family
  → Tool Type Schema
    → Tool Model Schema
      → Recipe Type Schema
        → Schema Version
```

Example:

```text
Metrology Recipe Schema
  → Overlay Tool
    → Vendor Model A
      → Production Recipe
        → schema-v2026.06
```

Each recipe schema shall define:

```text
Supported input body format
Parser rule
Required fields
Optional fields
Parameter path
Parameter type
Unit
Allowed range
Allowed enum
Default behavior
Null behavior
Overflow behavior
Validation rule
Comparison rule
Ignore rule
Security classification
Export rule if applicable
```

Important separation:

```text
Schema defines recipe structure.

Binding defines where the recipe can be used.

Governance defines whether the recipe is approved.
```

Do not overload schema with all applicability rules.

---

### 4.6 Layer 5 — Binding and Governance

This layer defines where a recipe can be used and whether it is officially allowed.

Owner: **AAA**

Binding shall cover:

```text
Tool type
Tool model
Equipment ID
Chamber / module where applicable
Product / route / step where applicable
Capability constraint
Allowed recipe type
Allowed release scope
```

Governance shall cover:

```text
Draft
Parsed
Validated
Reviewed
Approved
Released
Delivered
Verified
Retired
Rollback
```

AAA shall maintain the official state of recipe governance.

BB may report delivery result and tool-side existence, but it shall not decide whether the recipe is approved or valid.

---

## 5. Standard Support Position

### 5.1 AAA Standard Position

AAA shall align with recipe management principles such as:

```text
Recipe identification
Recipe uniqueness
Recipe version control
Recipe content validation
Recipe comparison
Variable parameter governance
Pre-release / pre-execution consistency check
Recipe approval and audit
Golden recipe baseline
```

AAA shall define and maintain the fab-level recipe body and schema specification.

---

### 5.2 BB Standard Position

BB shall align with equipment communication standards and vendor interface specifications.

This includes:

```text
Formatted process program transfer
Unformatted process program transfer
Equipment-side recipe inventory
Equipment-side recipe deletion
Technical transfer result
Equipment communication error handling
```

BB shall normalize communication results for AAA consumption.

---

## 6. Refactor Design Rules

### Rule 1 — AAA is not an equipment communication system

AAA shall not be designed around raw equipment messages.

### Rule 2 — S7F26 is not the AAA schema

Formatted process program data is an input carrier.
AAA schema must be defined independently.

### Rule 3 — AAA must be format-agnostic

AAA shall support multiple recipe body formats through format classification, parser registry, and schema mapping.

### Rule 4 — All formats must converge to one canonical model

**Content-level governance decisions**（validation、comparison、approval、release）shall be
performed on the **canonical recipe object**.

**RawBody remains the audit and reconstruction baseline**——it is the immutable source for
re-parse, audit trail, dispute handling, and canonical reconstruction. Governance decides on
canonical; audit/reconstruction traces back to RawBody.

### Rule 5 — Raw recipe body must be preserved

AAA shall preserve raw recipe body for:

```text
Audit
Re-parse
Parser upgrade
Dispute handling
Vendor troubleshooting
Golden baseline reconstruction
```

### Rule 6 — Schema and binding must be separated

Schema defines the recipe structure.
Binding defines where the recipe can be used.

### Rule 7 — Validation and comparison must be separated

Validation decides whether a recipe is legal.
Comparison decides whether the difference between recipes is acceptable.

---

## 7. Recommended Refactor Priority（優先順序，非硬性分期）

> 本節**不是執行分期（phase）**，只表達「該做的事情的建議先後順序與相依」。
> 真正的執行分期見 `PRD.md` 的 Milestone（那是本次實際在做的 onboarding toolkit）。
> 下列 Priority 1~4 是「優先級」，不承諾時程，亦不與 PRD Milestone 一一對應。

### Priority 1 — Recipe Governance Core（地基，最優先）

Build or refactor the following core objects:

```text
RawRecipeBody (RecipeBody — first-class immutable raw body + capture metadata)
RecipeSchema
CanonicalRecipe (runtime canonical object; onboarding 期對應 Canonical Recipe Projection)
ParserRegistry
ValidationRule
CompareRule
RecipeVersion
RecipeAuditTrail
```

### Priority 2 — Format Support（隨工具優先級擴增）

Add recipe body support by category:

```text
Formatted SECS-II
Unformatted / Binary
XML
JSON / YAML
CSV / Text
Vendor File Package
Manual Import
```

> 注意：Format support 是**架構需求**，不是一次性交付範圍（見 §8.1）。

### Priority 3 — Governance Capability（內容模型穩定後）

Implement:

```text
Schema validation
Recipe comparison
Golden recipe baseline
Release approval
Rollback
Tool capability binding
Recipe audit
```

### Priority 4 — BB Integration（核心穩定後）

After AAA core model is stable, refine the integration contract with BB.

The integration contract should be based on recipe body, metadata, and technical result, not raw equipment message structures.

---

## 8. Grill Review

### 8.1 Risk 1 — Scope Is Too Large

The architecture target is correct, but format support should not be treated as one single delivery scope.

Recommended delivery positioning:

| Category | Delivery Position |
|---|---|
| Formatted | Priority 1 concrete support |
| Unformatted / Binary | Priority 1 **data model must preserve RawBody**; actual parser/preparser support is later priority based on tool need |
| XML / Text / CSV | Priority 2 extension |
| Vendor Package | Priority 2/3, based on tool priority |
| JSON/YAML | Internal canonical/config format unless externally required |
| Manual Import | Can be early, but must follow the same parser and validation flow |

Key conclusion:

```text
Format support is an architecture requirement, not a one-time delivery scope.
```

---

### 8.2 Risk 2 — Parser Registry Becomes If-Else Logic

Parser Registry must be metadata-driven.

Parser selection should use:

```text
bodyFormat
toolType
toolModel
recipeType
schemaVersion
vendorFormatVersion
```

Without metadata-driven selection, Parser Registry is only a renamed hard-code structure.

> **視角區分**（對齊 PRD / HYPER_SPEC）：**ParserRegistry 是完整 AAA 的 guardrail**（母架構必守）。
> 對 **onboarding toolkit** 而言，ParserRegistry **可能在其 ownership 之外**，但 toolkit 產出的
> **package metadata 必須滿足母 AAA ParserRegistry 的契約**（emit 足夠 metadata 供其選 parser）。
> 此 metadata 契約對 toolkit 是 `[blocking-open]`（見 PRD §5 / HYPER_SPEC C.3）。

---

### 8.3 Risk 3 — Schema Hierarchy Becomes Too Deep

Keep schema hierarchy focused:

```text
Recipe Schema Family
  → Tool Type
    → Tool Model
      → Recipe Type
        → Schema Version
```

Do not expand schema hierarchy into:

```text
Fab
 → Phase
   → Product
     → Route
       → Step
         → Tool
           → Recipe
```

Schema defines structure.
Binding defines applicability.

---

### 8.4 Risk 4 — Formatted SECS-II Is Misread as Business Structure

Formatted SECS-II gives a structured carrier, not necessarily a business-level recipe model.

Key rule:

```text
SECS-II structure is not equal to recipe semantic structure.
```

Formatted recipe still requires parser and schema mapping before becoming the AAA canonical recipe object.

---

### 8.5 Risk 5 — Validation and Comparison Are Mixed

Validation and comparison must be separated.

| Rule Type | Question |
|---|---|
| Validation | Is this recipe legal? |
| Comparison | Is the difference between two recipes acceptable? |

Example:

```text
Pressure must be 10-20 torr.
```

This is validation.

```text
Pressure delta <= 0.1 torr can be ignored.
```

This is comparison.

---

### 8.6 Risk 6 — Raw Body Is Treated as an Attachment

Raw Recipe Body must be a first-class domain object.

Minimum fields (original / capture metadata only):

```text
bodyId
bodyFormat
encoding
rawBodyReference
checksum
size
source
sourceTime
sourceSystem
toolModel
ppid
captureAdapterVersion / ingestionVersion (if normalized by adapter)
```

Note: `parserVersion` and `schemaVersion` describe how RawBody is later **interpreted**,
not intrinsic properties of the raw body. They belong to
**ParseResult / Canonical Recipe Object / Projection / Package metadata**, not RawBody.

Reason:

```text
Parser will change.
Schema will change.
Vendor format will change.
Audit will trace history.
Recipe dispute may happen.
```

Without raw body, AAA loses reconstruction capability.

---

### 8.7 Risk 7 — Approval Workflow Is Overbuilt Too Early

Priority 1 should stabilize the recipe content model first:

```text
RecipeBody
CanonicalRecipe
RecipeSchema
ParserRegistry
ValidationRule
CompareRule
RecipeVersion
```

Approval, release, and rollback can initially reuse the existing flow or start with a minimal state model.

Understanding recipe content is more important than building a sophisticated workflow too early.

---

## 9. Final Statement

AAA shall be refactored as the governance system for recipe content, independent from equipment communication format.

BB owns recipe communication and technical transfer execution.
AAA owns recipe body classification, schema, parser, canonical model, validation, comparison, approval, release, rollback, audit, and golden baseline governance.

Recipe body formats such as formatted SECS-II, unformatted binary, XML, JSON/YAML, CSV/text, vendor package, and manual import shall all converge into the AAA canonical recipe object before governance actions are applied.（onboarding 期先產 Canonical Recipe **Projection**，提交後收斂成此 runtime object；見 §4.4 用詞對齊。）

The success of this refactor depends on three design guardrails:

```text
1. RecipeBody must become a first-class domain object.
2. ParserRegistry must be metadata-driven.
3. Schema, Binding, and Governance must be clearly separated.
```

If these guardrails are maintained, AAA can evolve into a scalable recipe governance platform.
If not, AAA will become a mixture of S7F26 parser, vendor parser, and approval workflow logic, creating a new legacy system.
