---
name: design-audit
description: >
  Performs a structured design accuracy audit for one or more pages in a frontend codebase,
  comparing the live implementation against a Figma design reference and the project's active
  design system. Use this skill whenever the user asks for a "design audit", "design accuracy
  check", "Figma vs code comparison", "migration audit", or mentions reviewing how closely a
  page or component matches its design. Also trigger when the user provides a Figma link
  alongside page routes or component paths and wants a written report. The output is always a
  structured Markdown (.md) audit report saved as a file.
---

# Design Accuracy Audit Skill

Produces a structured Markdown audit report comparing a page's implementation against its
Figma design reference and the project's current design system.

---

## Prerequisites — Gather Before Starting

Do NOT begin the audit until you have all of the following. If any are missing, ask explicitly.

| Input | What to ask if missing |
|---|---|
| **Figma link** | "Please share the Figma file URL and, if possible, the specific frame/node link." |
| **Page route(s) or component path(s)** | "Which page routes or file paths should I audit?" |
| **Codebase files** | "Please share the relevant source files — paste contents or use your editor's 'Add to chat' feature." |
| **Design system package names** | "What are the names of your design system packages (e.g. @company/eevee, @company/espeon)?" |

Optional but useful:
- Screenshots of the rendered page (desktop + mobile)
- Known migration status ("we're migrating from X to Y")
- Names of deprecated libraries to flag (e.g. `styled-components`, `@headout/aer`)

---

## Step 1 — Fetch Figma Design Context

Use the Figma MCP tools in this order. Try each; note failures explicitly in the report.

```
1. Figma:get_design_context  — primary: layout, spacing, typography, color tokens
2. Figma:get_screenshot       — visual reference for the frame
3. Figma:get_variable_defs    — token/variable definitions used in the design
4. Figma:get_metadata         — fallback: node name and type tree
```

**If ALL Figma MCP calls fail or time out:**
- Note this under "Figma MCP Status" in the report
- Continue the audit using: user-provided screenshots, Figma URL metadata (file ID, node ID),
  and code analysis
- Do NOT abort — a partial audit with honest caveats is more useful than nothing

**What to extract from Figma if accessible:**
- Layout: width, max-width, min-width, padding, gap, margin values
- Typography: font size, weight, line height, letter spacing
- Color: hex or token values for backgrounds, text, borders
- Component names: Figma component names that should map to design-system components
- Spacing tokens: Figma variable names that correspond to design tokens
- Breakpoints: responsive variants in the design

---

## Step 2 — Analyse the Codebase

Read every file shared. For each file, extract:

### Design system usage
Flag every import from a design system package. Categorise as:
- ✅ **Current stack** — imports from the new/target design system packages
- ⚠️ **Partial / override** — uses a current package but wraps it in heavy local overrides
- ❌ **Legacy / deprecated** — imports from old libraries that should have been replaced

### Layout constants
Find and list every hardcoded layout value:
- Arbitrary widths/heights: `[75rem]`, `[600px]`, `570px` etc.
- Hardcoded gaps, paddings, margins not from a token
- z-index magic numbers
- Placeholder/skeleton heights

### Token usage
- Which spacing/color/typography tokens are used correctly (e.g. `space.16`, `color.brand.primary`)
- Which values are raw CSS instead of tokens

### Component mapping
For each UI section visible in the design, identify:
- Which design-system component is used in code (if any)
- Whether that component is the correct/current one per the design system
- Whether a bespoke wrapper or override exists around it

### Responsive behaviour
- Is there a separate mobile container? (e.g. separate dweb/mweb files)
- Are mobile breakpoints handled via design-system primitives or bespoke CSS?

---

## Step 3 — Write the Audit Report

Follow the report schema below exactly, in order. Do not skip sections.
Use "N/A — [reason]" if a section genuinely does not apply.

**Output file naming:** `design-audit-[page-name]-[YYYY-MM-DD].md`

---

## Report Schema

```markdown
# [Page Name] Design Accuracy Audit

**Date:** YYYY-MM-DD
**Design reference:** Figma file `[File Name]`, [desktop/mobile] node `[node-id]`
**Frontend route audited:** `[route/path]`

---

## Scope

[2–4 sentences: which routes, which containers, desktop vs mobile, which sections are in scope.
What is explicitly NOT in scope.]

What is in scope:
- [bullet list of specific areas checked]

What is not fully in scope:
- [bullet list — especially variant routes, global nav, etc.]

---

## Figma MCP Status

[Describe what Figma data was successfully retrieved, what failed, and what was used instead.
Be specific: "get_design_context timed out; get_screenshot succeeded" is more useful than
"Figma was partially available".]

---

## Overall Assessment

[2–4 sentence executive summary. State migration/alignment status clearly and directly.
Do not hedge with language like "seems mostly fine". Make a call.]

---

## Design System Usage Summary

| Area | Status | Notes |
|---|---|---|
| [Section/component name] | ✅ Current stack / ⚠️ Partial / ❌ Legacy / 🔲 Bespoke | [one-line note] |

Status definitions:
- ✅ Current stack — uses the current design-system library with minimal local overrides
- ⚠️ Partial — uses current library but has non-trivial override styles beyond the system's intent
- ❌ Legacy — imports from a deprecated library that should have been replaced
- 🔲 Bespoke — entirely custom; no design-system component used where one should exist

---

## Migration Status by Area

### Clearly migrated
[Bullet list with file paths. Each bullet = one component or section confirmed on current stack.]

### Partially migrated / still bespoke
[Bullet list with file paths. State specifically what is bespoke — wrappers, overrides,
spacing constants, etc.]

### Not migrated / still on legacy stack
[Bullet list with file paths and the deprecated import found.
If empty, say: "No legacy library usage found in audited files."]

---

## Findings

[Numbered. Order: High severity first, then Medium, then Low.
Within the same severity, above-the-fold before below-the-fold.]

### [N]. [Finding title — specific, not vague]

**Severity:** High / Medium / Low
**Area:** [which section or component]
**Files:**
- `path/to/file.tsx`
- `path/to/styles.ts`

[2–5 sentences: what was found, why it matters, what the design specifies (if Figma data was
available), what the code does instead.]

**Evidence:**
[Paste the specific code pattern, value, or import that is the problem]

**Recommended fix:** [1–2 sentence concrete remediation direction]

---

## Concrete Flags

### High confidence issues
[Bullet list — issues you are certain about from code + design comparison]

### Not currently true
[Bullet list — assumptions that might be made about this page that the audit found to be WRONG.
This section is required. State them as quoted claims followed by why they are incorrect.
Example: "The page still uses styled-components" — no import of styled-components was found
in any audited file.]

---

## Where the Page Still Needs Work

[Ordered list of specific, actionable remediation steps. Each item must be specific enough
that an engineer can act on it without further clarification.]

1. [Action item]
2. [Action item]

---

## Bottom Line

[3–5 sentences. Restate migration status. Identify the most impactful remaining gap.
End with a clear recommendation: ship as-is / fix before launch / needs design-system work.]
```

---

## Severity Definitions

| Severity | Definition |
|---|---|
| **High** | Visible design deviation or use of a deprecated/removed library that would be caught in a design review. Users notice it. |
| **Medium** | Implementation uses the correct library but has bespoke overrides that make maintenance harder or create drift risk. Users probably don't notice it. |
| **Low** | Cleanup opportunity: arbitrary constants, minor token inconsistency, or code smell. No visible impact. |

---

## What Makes a Good Finding

A finding is only worth writing if it answers: **"What specific thing in the code does not match
the design or the design system standard, and where exactly is it?"**

If you can't point to a file path and a specific pattern, it's not a finding — it's a vague concern.

### ❌ Weak finding

> **Section layout uses non-standard spacing**
> The layout appears to use custom spacing values rather than design tokens.

Bad because: no file path, no specific value, "appears to" is not audit language.

### ✅ Strong finding

> **3. Section layout still relies on repeated arbitrary values**
>
> **Severity:** Medium
> **Area:** Page shell / section wrappers
> **Files:**
> - `src/containers/desktop/collectionsPage/index.tsx`
> - `src/containers/common/collectionsPage/components/nearbyCitiesSection/styles.ts`
>
> The page shell and multiple section wrappers define their own width, gap, and height constants
> instead of shared layout tokens. The Figma frame specifies a max-content-width of 1200px —
> the code uses `[75rem]` and `[62.5rem]` as separate arbitrary values with no shared constant.
>
> **Evidence:**
> ```tsx
> className="max-w-[75rem] min-w-[62.5rem] gap-[28px]"
> const PLACEHOLDER_HEIGHT = '570px';
> ```
>
> **Recommended fix:** Extract page-shell width and gap into a named layout token or shared
> constant. Replace placeholder heights with skeleton component defaults from the design system.

---

## What NOT to Write Findings For

1. **Things that are correct** — correct usage belongs in "Migration Status → Clearly migrated"
2. **Code style unrelated to design** — naming, tests, performance are out of scope
3. **Things outside the audited route** — global nav, site footer, other pages
4. **Speculation without evidence** — "this might cause problems if..." with no specific pattern

---

## How to Handle Missing Figma Data

**What you can still audit without Figma access:**
- Design system library usage (from imports)
- Legacy library detection
- Hardcoded constants vs token usage
- Mobile vs desktop container separation
- Override / wrapper composition patterns
- Component mapping per section

**What you cannot audit without Figma access:**
- Exact spacing value parity
- Typography size parity
- Color token accuracy
- Icon or asset pixel-match

When a finding is code-only due to missing Figma data, say so:
> **Note:** Figma `get_design_context` timed out. This finding is based on code analysis only.
> The value `[28px]` could not be cross-referenced against the Figma frame — a designer should verify.

---

## Quality Rules

1. Never claim a page "uses the old design system" unless you found an actual import from the
   deprecated library. Visual impression is not evidence.

2. Distinguish migration issues (wrong library) from cleanup issues (right library, messy usage).
   They have different remediation paths.

3. Always note when Figma data was unavailable and what you used instead.

4. Per-section findings must cite the actual file path and the specific pattern causing the issue.

5. "Not currently true" claims are as important as findings. Explicitly state what the audit
   did NOT find — this prevents false assumptions from spreading.

6. Variant routes (entertainment, promo, etc.) are separate design surfaces. Flag as out-of-scope
   unless explicitly asked.

7. Mobile and desktop are separate audit surfaces. If both containers exist, audit both.
