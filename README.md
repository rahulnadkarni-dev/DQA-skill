# design-audit

A skill for Claude that produces structured Markdown audit reports comparing a frontend page's implementation against its Figma design reference and the project's active design system.

---

## What it does

Given a Figma link, page routes, and source files, the skill:

1. Fetches design context from Figma (layout, typography, spacing, color tokens, component names)
2. Analyses the codebase for design system usage, hardcoded constants, legacy imports, and component mapping
3. Outputs a structured `.md` audit report with severitised findings, migration status, and concrete remediation steps

The report is always saved as `design-audit-[page-name]-[YYYY-MM-DD].md`.

---

## When Claude triggers this skill

- "Run a design audit on this page"
- "Check how closely our implementation matches the Figma"
- "Design accuracy check for the checkout page"
- "Figma vs code comparison"
- "Migration audit" (e.g. old design system → new)
- Any request pairing a Figma link with page routes or file paths and asking for a written report

---

## Inputs required

| Input | Description |
|---|---|
| **Figma link** | File URL and ideally the specific frame/node link |
| **Page route(s) or component path(s)** | The routes or file paths to audit |
| **Source files** | Paste contents or use your editor's "Add to chat" feature |
| **Design system package names** | e.g. `@acme/ui-core`, `@acme/tokens` |

Optional but improves accuracy:
- Screenshots of the rendered page (desktop + mobile)
- Known migration status (e.g. "migrating from styled-components to Tailwind")
- Names of deprecated libraries to flag explicitly

Claude will ask for any missing required inputs before starting.

---

## Report structure

```
# [Page Name] Design Accuracy Audit
├── Scope
├── Figma MCP Status
├── Overall Assessment
├── Design System Usage Summary   ← table with ✅ / ⚠️ / ❌ / 🔲 status per section
├── Migration Status by Area
│   ├── Clearly migrated
│   ├── Partially migrated / still bespoke
│   └── Not migrated / still on legacy stack
├── Findings                      ← numbered, ordered High → Medium → Low
├── Concrete Flags
│   ├── High confidence issues
│   └── Not currently true        ← explicit list of false assumptions the audit disproved
├── Where the Page Still Needs Work
└── Bottom Line
```

---

## Severity levels

| Level | Meaning |
|---|---|
| **High** | Visible design deviation or deprecated library usage — would be caught in design review |
| **Medium** | Correct library but bespoke overrides that create drift or maintenance risk |
| **Low** | Cleanup opportunity: arbitrary constants, minor token inconsistency, no visible impact |

---

## What the audit covers

**Always audited (no Figma access needed):**
- Design system library imports — current vs legacy vs bespoke
- Hardcoded constants vs token usage (arbitrary widths, gaps, heights, z-indices)
- Component mapping per section
- Mobile vs desktop container separation
- Override and wrapper composition patterns

**Audited when Figma data is available:**
- Spacing, padding, and gap value parity
- Typography (size, weight, line height) parity
- Color token accuracy
- Responsive breakpoint alignment

---

## What it will NOT flag

- Correct usage — that goes in "Migration Status → Clearly migrated", not findings
- Code style issues unrelated to design (naming, tests, performance)
- Components outside the audited route (global nav, footer, other pages)
- Speculation without specific file + pattern evidence

---

## Figma MCP tools used

```
Figma:get_design_context   ← primary
Figma:get_screenshot        ← visual reference
Figma:get_variable_defs     ← token/variable definitions
Figma:get_metadata          ← fallback
```

If all Figma calls fail, the audit continues with code-only analysis. The report notes exactly what was and wasn't available and flags any findings that couldn't be cross-referenced against the design.

---

## Example finding (strong vs weak)

**Weak** — do not write findings like this:
> The layout appears to use custom spacing values rather than design tokens.

No file path, no specific value, "appears to" is not audit language.

**Strong** — findings must look like this:
> **3. Section layout still relies on repeated arbitrary values**
> **Severity:** Medium | **Area:** Page shell
> **Files:** `src/containers/desktop/productListPage/index.tsx`
>
> The page shell defines its own width constants instead of shared layout tokens. Figma specifies max-content-width of 1200px; code uses `[75rem]` and `[62.5rem]` as separate arbitrary values.
>
> ```tsx
> className="max-w-[75rem] min-w-[62.5rem] gap-[28px]"
> ```
> **Fix:** Extract into a named layout token or shared constant.

Every finding must point to a specific file path and a specific code pattern. If it can't, it's not a finding.
