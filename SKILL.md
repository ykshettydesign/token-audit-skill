---
name: figma-token-audit
description: "Use this skill whenever a user wants to audit, validate, review, or check their Figma Variables or Design Tokens setup against industry best practices. Triggers include: \"audit my design tokens\", \"validate my Figma variables\", \"check my token setup\", \"review my design system variables\", \"is my variable structure correct?\", \"does my Figma setup follow best practices?\", \"check my DTCG/token JSON\", or any time the user shares a Figma file, token JSON export, or variable list and wants it evaluated. Also triggers when a user is unsure whether their token architecture is scalable, correctly structured, or engineer-handoff-ready. Always use this skill before providing any token audit or governance feedback — it contains the full scoring rubric, check categories, and output format required to produce a thorough, actionable audit report."
---
 
# Figma Design Token & Variable Audit Skill
 
A structured audit framework for evaluating a Figma Variable system against
industry best practices. Produces a scored report with pass/fail checks,
severity ratings, and prioritized remediation steps.
 
---
 
## Overview
 
This skill guides the agent through a six-domain audit of a Figma Variable
system. Each domain maps to the six pillars of a well-governed design token
architecture. The audit produces:
 
- A **scored report** (0–100) with per-domain breakdowns
- **Pass / Warning / Fail** status per check
- **Severity labels** (Critical · Major · Minor) for every finding
- **Prioritized remediation steps** sorted by impact
---
 
## How to Run the Audit
 
### Step 1 — Gather Input
 
Ask the user to provide one or more of the following. More sources = higher
confidence audit.
 
| Input Source | What It Unlocks |
|---|---|
| Figma file link (view access) | Full variable collection names, modes, scopes |
| Screenshot(s) of Variable panel | Collection structure, naming, mode count |
| Exported token JSON (W3C Design Tokens / DTCG or Tokens Studio) | Naming depth, reference chains, value types |
| CSS / JS token output | Engineering alignment, naming parity |
| List of variable names (pasted text) | Naming convention analysis |
 
If none are provided, prompt: *"To audit your token system I'll need at least
one of: a Figma file link, screenshots of your Variables panel, or an exported
token JSON. Which can you share?"*
 
### Step 1b — Determine Evaluation Method
 
Depending on the input provided, the agent may evaluate checks via:
 
| Method | When to Use | API / Tool |
|---|---|---|
| **Figma MCP / Plugin API** | User provides a Figma file link and MCP is connected | `figma.variables.getLocalVariables()`, `figma.variables.getLocalVariableCollections()`, node `boundVariables` |
| **REST API** | Agent has Figma API token access | `GET /v1/files/:key/variables/local`, `GET /v1/files/:key/variables/published` |
| **JSON inspection** | User provides exported token JSON (DTCG / Tokens Studio) | Parse and analyze the JSON directly |
| **Manual / screenshot** | User provides screenshots or pasted text | Visual inspection of provided evidence |
 
If a check cannot be verified with the available inputs, mark it as
⚠️ **WARN** with the note: *"Unable to verify — requires [specific input]."*
Do not guess PASS when evidence is absent.
 
### Step 2 — Run All Six Domain Audits
 
Work through each domain in order. For each check, mark it as:
 
- ✅ **PASS** — Fully meets best practice
- ⚠️ **WARN** — Partially meets or has exceptions
- ❌ **FAIL** — Does not meet best practice
### Step 3 — Score and Summarize
 
Calculate scores per domain and overall. Write a prioritized action plan.
 
---
 
## Domain 1: Architecture & Layer Separation
 
**Required input:** Figma file access (MCP or REST API) or token JSON export.
Screenshots of the Variable panel are acceptable at reduced confidence.
 
**Goal:** Verify that the two-layer Primitive → Semantic architecture is
correctly implemented and that components reference only semantic tokens.
 
### Checks
 
| ID | Check | Severity |
|---|---|---|
| A1 | A dedicated Primitive (Global) collection exists and is separate from semantic collections | Critical |
| A2 | Primitive tokens contain only raw values (hex, px, unitless numbers) — never aliases | Critical |
| A3 | Semantic tokens reference primitives via alias, not hardcoded values | Critical |
| A4 | No component in the Figma file applies a Primitive token directly to a layer property | Major |
| A5 | Semantic tokens communicate intent, not appearance (e.g. `color/interactive/primary` not `color/blue`) | Major |
| A6 | The alias chain is max 2 levels deep: Primitive → Semantic. No Semantic → Semantic → Semantic chains unless intentional | Minor |
 
### How to Evaluate
 
- In the Variable panel, inspect a semantic token. Its value should show a
  chip referencing another variable, not a raw hex.
- Spot-check 5 random components: right-click a fill → "Edit variable" →
  confirm it is a semantic token.
- In JSON, check: does every leaf node in the semantic collection have a
  `$value` that begins with `{` (alias notation)?
**Via MCP / API:** Use `figma.variables.getLocalVariables()` to list all
variables. For each variable, check `resolvedType` and whether `valuesByMode`
contains alias references (`type: "VARIABLE_ALIAS"`) or raw values. Use
`figma.variables.getLocalVariableCollections()` to confirm separate Primitive
and Semantic collections. Inspect component nodes' `boundVariables` to verify
A4 — that no layer references a Primitive variable directly.
 
### Scoring (Domain 1)
 
- A1–A3 each worth 10 pts (Critical)
- A4–A5 each worth 7 pts (Major)
- A6 worth 4 pts (Minor)
- **Domain max: 48 pts → normalized to /20**
---
 
## Domain 2: Naming Convention & Taxonomy
 
**Required input:** Variable name list (any source — Figma access, JSON export,
pasted text, or screenshots). This domain can be fully evaluated from names alone.
 
**Goal:** Verify that token names are consistent, scalable, and
self-documenting.
 
### Checks
 
| ID | Check | Severity |
|---|---|---|
| N1 | All tokens follow a hierarchical path convention using a consistent separator (forward slash recommended) | Critical |
| N2 | No abbreviations are used (background not bg, interactive not int, disabled not dis) | Major |
| N3 | Names are all lowercase with no spaces or camelCase | Major |
| N4 | State is always the final segment: `default`, `hover`, `active`, `focus`, `disabled`, `error` | Major |
| N5 | Primitive tokens use numeric scale suffixes (e.g. `color/blue/500`, `spacing/4`) | Minor |
| N6 | No tokens are named after their raw value (e.g. `color/ffffff` or `spacing/16px` are anti-patterns) | Critical |
| N7 | Component-scoped tokens include the component name as a path segment | Minor |
 
### How to Evaluate
 
- Export or paste the full variable name list. Scan for: mixed separators
  (hyphens and slashes coexisting at the same level), camelCase, abbreviations,
  color hex values in names, numbers with units.
### Naming Red Flags (auto-fail patterns)
 
```
btn-bg              → abbreviation + non-standard separator
#0D7377             → raw value in name
blueColor500        → camelCase
color/FFFFFF        → uppercase
text_primary        → underscore separator
spacing/16px        → unit embedded in name
```
 
### Scoring (Domain 2)
 
- N1, N6 each worth 10 pts (Critical)
- N2, N3, N4 each worth 7 pts (Major)
- N5, N7 each worth 4 pts (Minor)
- **Domain max: 49 pts → normalized to /20**
---
 
## Domain 3: Collection Structure & Mode Governance
 
**Required input:** Figma file access (MCP or REST API) for full evaluation.
Screenshots of the Variable panel showing collection names and modes are
acceptable at reduced confidence. Token JSON alone cannot verify mode setup.
 
**Goal:** Verify that collections are organized logically and that Modes are
implemented correctly to enable theming, density switching, and platform
variants.
 
### Checks
 
| ID | Check | Severity |
|---|---|---|
| C1 | Color and Spacing are in separate collections (different mode boundaries require separation) | Critical |
| C2 | Primitives collection has only the single default mode (raw values do not vary by theme) | Critical |
| C3 | Semantic Color collection has at minimum a Light and Dark mode | Major |
| C4 | A Default/Fallback mode exists in every collection that has modes | Major |
| C5 | Collection names are prefixed or numbered for sort order clarity (e.g., `01 · Primitives`) | Minor |
| C6 | Each collection contains only one "type" of token — no mixing of color tokens into a spacing collection | Major |
| C7 | Mode names are consistent across collections (not "Dark" in one and "dark-mode" in another) | Minor |
| C8 | No single collection has more than 5 modes (beyond this, split into sub-collections) | Minor |
| C9 | Component-specific tokens are isolated in their own collection or group, not mixed into global semantic | Minor |
| C10 | Variable groups (folders) within each collection mirror the path hierarchy implied by token names | Major |
| C11 | If primitives and semantics are in separate Figma libraries, semantic tokens correctly reference published primitive library variables with no duplicated primitives | Major |
 
### How to Evaluate
 
- List all collection names and their mode names.
- For each collection: identify the property types it contains. Flag any
  collection containing both fill-type and size-type tokens.
- Check: does the Primitives collection have more than one mode? It should have only the single default mode.
**Via MCP / API:** Use `figma.variables.getLocalVariableCollections()` to list
all collections with their `modes` array and `variableIds`. Check `modes.length`
for the Primitives collection (should be 1). Cross-reference variable
`resolvedType` to verify no collection mixes color and number types.
 
### Collection Health Matrix
 
For each collection found, fill in:
 
```
Collection Name | Type (Primitive/Semantic/Component) | Modes | Issue?
```
 
### Scoring (Domain 3)
 
- C1, C2 each worth 10 pts (Critical)
- C3, C4, C6, C10, C11 each worth 7 pts (Major)
- C5, C7, C8, C9 each worth 4 pts (Minor)
- **Domain max: 71 pts → normalized to /20**
---
 
## Domain 4: Scoping & Visibility Configuration
 
**Required input:** Figma file access (MCP or REST API). Scoping information
is **not** available in token JSON exports or screenshots — those checks must
be marked WARN with a note. This domain requires live Figma file inspection.
 
**Goal:** Verify that variables are scoped to only the relevant Figma
properties, reducing noise for designers and preventing misuse.
 
### Checks
 
| ID | Check | Severity |
|---|---|---|
| S1 | Primitive tokens are hidden from pickers (zero scopes, or "Hide from publishing" approach) | Critical |
| S2 | Color tokens are not surfaced in spacing/size inputs | Critical |
| S3 | Spacing tokens are not surfaced in color pickers | Critical |
| S4 | Border radius tokens are scoped to CORNER_RADIUS only | Major |
| S5 | Stroke width tokens are scoped to STROKE_WIDTH only | Major |
| S6 | Opacity tokens are scoped to OPACITY only | Major |
| S7 | Fill color tokens are separated by intent: Fill scope for backgrounds, Stroke scope for borders | Minor |
| S8 | Z-index, duration, easing tokens are fully hidden (for documentation only — not applicable in Figma layers) | Minor |
| S9 | Boolean variables (show/hide, feature flags) are scoped appropriately and not exposed in color or size pickers | Major |
| S10 | String variables (font family, icon names) are scoped to their intended property (e.g., FONT_FAMILY) | Major |
 
### How to Evaluate
 
- In Figma, open any component and click on a fill property. Open the
  variable picker. Check: do spacing tokens appear? If yes, S2 = FAIL.
- Open a spacing/padding property picker. Check: do color tokens appear?
  If yes, S3 = FAIL.
- Check the Variable editor for any Primitive variable. Its "Scope" section
  should show no checkboxes enabled, or it should not appear in the picker at
  all.
**Via MCP / API:** Use `figma.variables.getLocalVariables()` — each variable
object has a `scopes` array. Verify Primitive variables have `scopes: []`.
Check that color variables do not include size-related scopes and vice versa.
Note: setting zero scopes hides a variable from property pickers but it
remains visible in the Variable editor and can still be applied manually.
For library consumers, "Hide from publishing" provides stronger isolation.
 
### Scoping Reference Table
 
Use this as the expected state to compare against:
 
| Token Category | Expected Figma Scopes |
|---|---|
| Primitives (all types) | None (hidden) |
| Semantic fill/background | FRAME_FILL, SHAPE_FILL |
| Semantic text color | TEXT_FILL |
| Semantic stroke/border | STROKE_COLOR |
| Semantic spacing/gap | GAP |
| Semantic padding | PADDING |
| Border radius | CORNER_RADIUS |
| Stroke width | STROKE_WIDTH |
| Opacity | OPACITY |
| Font size | FONT_SIZE |
| Font weight | FONT_WEIGHT |
| Boolean (show/hide, flags) | LAYER_VISIBILITY or component props only |
| String (font family) | FONT_FAMILY |
| String (other) | Scoped per intent or hidden |
| Z-index / duration / easing | None (hidden) |
 
### Scoring (Domain 4)
 
- S1, S2, S3 each worth 10 pts (Critical)
- S4, S5, S6, S9, S10 each worth 7 pts (Major)
- S7, S8 each worth 4 pts (Minor)
- **Domain max: 73 pts → normalized to /20**
---
 
## Domain 5: Unsupported Properties Handling (Hybrid Strategy)
 
**Required input:** Figma file access (MCP or REST API) for full evaluation.
Screenshots of the Styles panel are acceptable for partial checks. Token JSON
alone cannot verify Figma Style configuration.
 
**Goal:** Verify that properties Figma Variables do not natively support
(Typography bundles, Gradients, Effects) are handled via a consistent hybrid
Styles + Variables strategy, and that Style names align with token naming.
 
### Checks
 
| ID | Check | Severity |
|---|---|---|
| H1 | Figma Text Styles exist for all typography use cases (headings, body, code, etc.) | Major |
| H2 | Text Style names mirror the token naming convention (e.g. `Typography/Heading/H1/Default`) | Critical |
| H3 | Individual typographic values (size, weight, line-height) that ARE supported as variables are defined as variables (not only as Styles) | Minor |
| H4 | Figma Effect Styles are used for shadow/elevation definitions | Major |
| H5 | Effect Style names mirror elevation token paths (e.g. `Shadow/Elevation/100`) | Critical |
| H6 | Gradient Styles exist for gradient use cases and are named semantically | Major |
| H7 | No hardcoded typography or shadow values exist in components — they all reference a Style or Variable | Major |
| H8 | A written migration note exists (in a cover page or annotation) for properties expected to graduate to Variables in future Figma releases | Minor |
 
### How to Evaluate
 
- Open the Styles panel (4-dot grid icon). Count Text Styles, Effect Styles,
  and Color Styles for gradients.
- Check Style names against the token taxonomy: do they use the same path
  separator and segment vocabulary?
- In a component, inspect a text layer: is the font size hardcoded or does it
  reference a Text Style + (ideally) a font-size Variable?
### Scoring (Domain 5)
 
- H2, H5 each worth 10 pts (Critical)
- H1, H4, H6, H7 each worth 7 pts (Major)
- H3, H8 each worth 4 pts (Minor)
- **Domain max: 56 pts → normalized to /20**
---
 
## Domain 6: Engineering Handoff Alignment
 
**Required input:** Token JSON export and/or CSS/JS output for full evaluation.
Figma file access supplements with variable descriptions (E8) and code syntax
(E10). Some checks (E5, E6, E9) require asking the user about their workflow.
 
**Goal:** Verify that the token system is structured for lossless translation
to code — that names align with CSS/JSON conventions and a viable export
pipeline exists.
 
### Checks
 
| ID | Check | Severity |
|---|---|---|
| E1 | Token names use only lowercase letters, numbers, and forward slashes — no characters invalid in CSS custom property names | Critical |
| E2 | Token names, when converted (`/` → `-`, prepend `--`), produce valid and readable CSS custom properties | Critical |
| E3 | A W3C Design Tokens (formerly DTCG) compatible JSON export exists or a plan exists to generate one | Major |
| E4 | Alias chains are preserved in the JSON export (semantic tokens reference primitive names, not resolved values) | Critical |
| E5 | The token file is (or is planned to be) version-controlled in a Git repository | Major |
| E6 | A build tool (Style Dictionary or equivalent) is identified for transforming tokens to platform-specific outputs | Major |
| E7 | Deprecated tokens are marked (via naming suffix `/deprecated` or a separate collection) rather than deleted immediately | Minor |
| E8 | Token descriptions/annotations are filled in for all semantic tokens in Figma (visible in "Edit variable" → Description field) | Minor |
| E9 | Breaking changes (renames, deletions) have a documented versioning and communication strategy | Major |
| E10 | Variables have Web, iOS, and/or Android Code Syntax configured where applicable | Minor |
 
### How to Evaluate
 
- Take a sample of 10 token names and run the CSS mapping test manually.
- If a JSON export is available, verify: do semantic token values look like
  `{color.blue.500}` (alias) or like `#0D7377` (resolved)? Aliases are correct.
- Ask the user: "Is your token JSON stored in version control? Which tool
  transforms it for engineering use?"
**Via MCP / API:** Use `figma.variables.getLocalVariables()` to extract all
variable names and run the CSS mapping test programmatically. Check variable
`description` fields for E8 and `codeSyntax` objects for E10.
 
### CSS Mapping Test (apply to every token name)
 
```
Input:   color/interactive/button/background/primary/default
Step 1:  color-interactive-button-background-primary-default
Step 2:  --color-interactive-button-background-primary-default
Result:  ✅ Valid CSS custom property
```
 
```
Input:   btn-BG_main
Step 1:  btn-BG_main          (slash already absent — red flag)
Step 2:  --btn-BG_main
Result:  ❌ Mixed case, underscore — FAIL E1 and E2
```
 
### Scoring (Domain 6)
 
- E1, E2, E4 each worth 10 pts (Critical)
- E3, E5, E6, E9 each worth 7 pts (Major)
- E7, E8, E10 each worth 4 pts (Minor)
- **Domain max: 70 pts → normalized to /20**
---
 
## Scoring & Report Generation
 
### Check-Level Scoring
 
Each individual check is scored based on its status:
 
- **PASS** = full points for that check
- **WARN** = 50% of points for that check
- **FAIL** = 0 points
### Severity Weights (consistent across all domains)
 
| Severity | Points per Check |
|---|---|
| Critical | 10 |
| Major | 7 |
| Minor | 4 |
 
### Normalization Formula
 
Each domain is normalized to a 20-point scale:
 
```
Domain Score = (Points Earned / Domain Max) × 20
```
 
### Overall Score
 
```
Total = Sum of 6 normalized domain scores  (max = 120, displayed as /100)
Final Score = (Total / 120) × 100
```
 
### Grade Bands
 
| Score (inclusive) | Grade | Label |
|---|---|---|
| 90–100 | A | Production Ready |
| 75–89 | B | Solid Foundation, Minor Gaps |
| 55–74 | C | Functional but Needs Work |
| 35–54 | D | Significant Gaps — Refactor Recommended |
| 0–34 | F | Rebuild Recommended |
 
---
 
## Output Format
 
Produce the audit report in the following structure. Be specific — cite actual
token names, collection names, or Style names from the user's file wherever
possible.
 
```
# Figma Token System Audit Report
**Date:** [date]
**System / File:** [name if known]
**Audit Confidence:** High / Medium / Low  (based on inputs provided)
 
---
 
## Executive Summary
[2–3 sentences on overall health and the single most important action.]
 
## Score
| Domain | Score | Grade |
|---|---|---|
| 1. Architecture & Layer Separation | XX/20 | [letter] |
| 2. Naming Convention & Taxonomy    | XX/20 | [letter] |
| 3. Collection Structure & Modes    | XX/20 | [letter] |
| 4. Scoping & Visibility            | XX/20 | [letter] |
| 5. Unsupported Properties Handling | XX/20 | [letter] |
| 6. Engineering Handoff Alignment   | XX/20 | [letter] |
| **OVERALL**                        | **XX/100** | **[letter]** |
 
---
 
## Findings
 
### 🔴 Critical Failures  (fix before any further work)
[For each Critical FAIL:]
- **[Check ID] [Check Name]:** What was found. What it breaks. Exact fix.
 
### 🟡 Major Warnings  (address in next sprint)
[For each Major WARN or FAIL:]
- **[Check ID] [Check Name]:** What was found. Recommended fix.
 
### 🔵 Minor Notes  (backlog / polish)
[For each Minor WARN or FAIL:]
- **[Check ID] [Check Name]:** What was found. Suggested improvement.
 
---
 
## Prioritized Action Plan
 
### Immediate (this week)
1. [Most impactful Critical fix]
2. ...
 
### Short-Term (next sprint)
1. [Major fixes]
2. ...
 
### Long-Term (backlog)
1. [Minor improvements and future-proofing]
2. ...
 
---
 
## What's Working Well ✅
[Acknowledge genuine strengths — collections that ARE correct,
naming segments that follow convention, etc. Be specific.]
```
 
---
 
## Audit Confidence Levels
 
Report confidence based on available inputs:
 
| Confidence | Inputs Available |
|---|---|
| **High** | Figma file access + JSON export + CSS output |
| **Medium** | Screenshots of Variable panel + token name list |
| **Low** | Partial name list only, or verbal description |
 
At Low confidence, flag checks that cannot be verified and note what the user
should share to complete them. Do not guess Pass when evidence is absent —
default to WARN with "Unable to verify — please share [X]."
 
---
 
## Quick Reference: Most Common Failures
 
Use this list to prioritize your inspection order — check these patterns first
as they account for the majority of failures in real-world audits:
 
1. **A4** — Components using Primitive tokens directly (most common failure)
2. **N2** — Abbreviations in token names (`bg`, `btn`, `txt`)
3. **S2/S3** — Color bleeding into spacing pickers (scoping not configured)
4. **C1** — Color and Spacing in the same collection
5. **H2/H5** — Style names not aligned to token taxonomy
6. **E4** — JSON exports resolving aliases to raw values instead of preserving references
7. **N6** — Tokens named after their value (`color/ffffff`, `spacing/8px`)
---
 
## Appendix: Terminology Glossary
 
| Term | Definition |
|---|---|
| Primitive Token | Raw value token (e.g. `color/blue/500 = #0D7377`). No contextual meaning. |
| Semantic Token | Alias token that references a primitive and carries intent (e.g. `color/interactive/primary`). |
| Alias | A variable value that points to another variable rather than a raw value. |
| Mode | A named variant of a collection's values (e.g. Light, Dark, Desktop, Compact). |
| Scope | A Figma configuration that restricts which UI property pickers a variable appears in. |
| W3C Design Tokens (DTCG) | W3C Design Tokens Community Group specification — the standard JSON format for design token interchange (formerly referred to as DTCG). |
| Style Dictionary | Community-maintained open-source tool for transforming design token JSON into platform-specific outputs (originally created by Amazon, now maintained under the tokens-studio org). |
