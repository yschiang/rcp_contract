## Plan Mode

- Make the plan extremely concise. Sacrifice grammar for the sake of concision.
- At the end of each plan, give me a list of unresolved questions to answer, if any.

## Domain Docs
- Single-context layout: `CONTEXT.md` at the repo root (domain glossary + ADRs D1–D22 embedded in §2; not split into per-decision files).
- Specs are not standalone files — content lives in existing docs by concern; see README "Spec Taxonomy" map (authoritative deliverable list = `PRD §12.0`).

## Response Style
- Be concise. Start with the conclusion.
- Use project domain terms from CONTEXT.md.
- Do not re-explain known terms unless asked.
- Prefer tables for comparison and decision records.

## Required Reading
Before design, spec, or code changes, read:
1. `CONTEXT.md` — domain glossary + ADRs (D1–D22 in §2)
2. `docs/architecture/ARCHITECTURE.md` — HOW
3. `docs/architecture/HYPER_SPEC.md` — capability layering / guardrails / claim scope
4. `docs/product/PRD.md` — WHAT / scope / deliverables (§12.0)
5. `docs/decisions/DECISIONS.md` — open / blocking-open tracking

## Design Discipline
- Do not hide domain rules in code.
- Express domain knowledge in Converter Spec / Rule Inventory first.
- Change runtime code only when the rule language cannot express required behavior.

## Output Rules
For architecture work, produce:
- Problem
- Decision
- Alternatives considered
- Trade-offs
- Impacted modules
- Test strategy