---
name: web-design-guidelines
description: Use PROACTIVELY when building web interfaces, when reviewing React or Next.js components, when testing keyboard accessibility and layout shift, or for configuring typography and surface tokens; automatically invoke before shipping frontend UI or HTML artifacts.
---

# Web Design & UI Audit Guidelines

Apply production web interface engineering standards (distilled from Vercel's `web-design-guidelines` and modern design-system practices) when building, restyling, or auditing frontend applications, dashboards, and standalone HTML artifacts.

## When to Use & Decision Matrix

Evaluate UI components against this engineering decision matrix to prevent visual defects and compare implementation tradeoffs:

| UI Dimension | Standard Pattern | Anti-Pattern to Avoid | Tradeoff & Comparison |
| :--- | :--- | :--- | :--- |
| **Keyboard Focus** | `:focus-visible` ring (`2px` offset, `>= 3:1` contrast) | Bare `outline: none` without replacement | Preserves clean mouse clicks while guaranteeing full keyboard navigation |
| **Dynamic Numbers** | `font-variant-numeric: tabular-nums` | Proportional default digits in counters/tables | Prevents horizontal jitter during live metric updates or sorting |
| **Flex/Grid Text** | `min-width: 0` + `truncate` / `line-clamp-2` | Unbounded flex children with long user strings | Stops long URLs or titles from blowing out card or table widths |
| **Async Media / Charts** | Explicit `aspect-ratio` or skeleton box | Unsized `<img>` or client-fetched container | Eliminates Cumulative Layout Shift (CLS) on slow connections |
| **Full-Height Shells** | `min-height: 100dvh` (`min-h-dvh`) | Legacy `100vh` on mobile browsers | Adapts cleanly to dynamic mobile browser address bars |
| **Motion & Transitions** | `@media (prefers-reduced-motion: no-preference)` | Unconditional infinite animations or parallax | Protects vestibular accessibility while keeping polish for default users |

## Input Parameters & Output Format

When running a UI design and accessibility review, this workflow accepts component source input parameters and returns a structured audit output format:

- **Input parameters**:
  - `target_files` (`array[string]`, required): Paths to HTML, CSS, Tailwind, JSX, TSX, or Vue/Svelte component files.
  - `viewport_profiles` (`array[string]`, optional): Desktop (`1440x900`) and mobile (`390x844`) profiles to validate.
  - `theme_modes` (`array[string]`, optional): `["light", "dark"]` surface contrast verification targets.
- **Output format**:
  - Returns a structured audit report containing `summary_metrics` (`cls_risk_count`, `a11y_violations`, `typography_issues`) and `findings` (each with `file`, `line`, `rule_id`, `severity`, and a drop-in CSS/JSX remediation snippet).

```yaml
# ui-audit-rules.yaml — Component review thresholds
audit:
  min_contrast_body: 4.5
  min_contrast_ui: 3.0
  min_touch_target_px: 44
  min_desktop_target_px: 36
  max_line_measure_ch: 75
  require_tabular_nums: true
  require_reduced_motion_guard: true
```

## Core Workflow: Four-Layer Interface Verification

### 1. Interactions, Keyboard Operability & Hit Targets

Ensure every interactive control works predictably with keyboard, pointer, and assistive technologies:

1. Use native semantic elements (`<button type="button">`, `<a href="...">`, `<dialog>`, `<label for="...">`) instead of `<div onClick>`.
2. Enforce minimum hit target dimensions (`36px x 36px` on desktop, `44px x 44px` on touch devices) and visible `:focus-visible` outlines.
3. Preserve button width during async mutations (`disabled` + inline spinner) so adjacent controls never jump.

```tsx
// Accessible metric action card with tabular numerals and truncation safety
export function MetricCard({ label, value, delta, onRefresh, isPending }: MetricCardProps) {
  return (
    <section className="min-w-0 rounded-xl border border-neutral-200/80 bg-white p-4 dark:border-white/10 dark:bg-neutral-900">
      <div className="flex items-center justify-between gap-3">
        <h3 className="min-w-0 truncate text-sm font-medium text-neutral-600 dark:text-neutral-400">
          {label}
        </h3>
        <button
          type="button"
          onClick={onRefresh}
          disabled={isPending}
          aria-label={`Refresh ${label}`}
          className="inline-flex h-9 w-9 shrink-0 items-center justify-center rounded-lg border border-neutral-200 text-neutral-700 transition-colors hover:bg-neutral-100 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-blue-600 focus-visible:ring-offset-2 disabled:opacity-50 motion-reduce:transition-none dark:border-white/10 dark:text-neutral-200"
        >
          <span aria-hidden="true">↻</span>
        </button>
      </div>
      <p className="mt-2 font-mono text-2xl font-semibold tracking-tight tabular-nums text-neutral-950 dark:text-white">
        {value}
      </p>
      <p className="mt-1 text-xs tabular-nums text-emerald-600 dark:text-emerald-400">{delta}</p>
    </section>
  );
}
```

### 2. Automated Static Audit Script for Common UI Regressions

Run a quick static check across JSX/TSX/HTML files to catch missing `aria-label` attributes on icon buttons, raw `outline-none` without `focus-visible:ring`, and missing `tabular-nums`:

```python
import pathlib
import re

RULES = {
    "bare_outline_none": re.compile(r"outline-none(?![^\n\"']*focus-visible:ring)"),
    "legacy_100vh": re.compile(r"\b(h-screen|min-h-screen|100vh)\b"),
}

for path in pathlib.Path("src").rglob("*.tsx"):
    content = path.read_text(encoding="utf-8")
    for rule_id, pattern in RULES.items():
        for match in pattern.finditer(content):
            line_no = content[: match.start()].count("\n") + 1
            print(f"{path}:{line_no}: [{rule_id}] {match.group(0)}")
```

### 3. Structured UI Audit Report Output

Produce machine-readable findings so automated PR reviews or frontend implementers can apply fixes directly:

```json
{
  "summary_metrics": {
    "cls_risk_count": 0,
    "a11y_violations": 1,
    "typography_issues": 1
  },
  "findings": [
    {
      "file": "src/components/PriceTable.tsx",
      "line": 42,
      "rule_id": "tabular-numerals",
      "severity": "medium",
      "remediation": "Add 'tabular-nums' to the price <td> className to prevent digit jitter."
    }
  ]
}
```

## Examples

### Example 1: Fixing Flex Overflow and Number Jitter in a Live Dashboard

```bash
# Search for flex children missing min-w-0 or numeric cells missing tabular-nums
rg -n "truncate" src/components/ | grep -v "min-w-0" || true
```

1. Add `min-w-0` to the parent flex item wrapping a `truncate` child label.
2. Add `tabular-nums` to all live latency, currency, and percentage counters.

### Example 2: Eliminating Layout Shift on Asynchronous Chart Cards

1. Replace an unsized `<div id="chart-mount">` with `className="aspect-[16/9] min-h-[240px] w-full rounded-lg bg-neutral-100/60 dark:bg-neutral-800/40"`.
2. Verify that switching between loading skeleton and rendered SVG chart produces `0.000` Cumulative Layout Shift.

## Troubleshooting

- **Symptom: `text-overflow: ellipsis` (`truncate`) has no effect inside a flex row**
  - **Cause**: Flexbox items default to `min-width: auto`, preventing the container from shrinking below its text content width.
  - **Fix**: Add `min-width: 0` (`min-w-0` in Tailwind) to the flex child holding the truncated text.
- **Symptom: Sticky header border disappears over dark cards**
  - **Cause**: Solid hex borders blend into matching card backgrounds during scroll.
  - **Fix**: Use translucent alpha borders (`border-black/10 dark:border-white/10`) paired with `backdrop-blur-md`.
- **Symptom: Mobile bottom action bar is clipped behind the browser toolbar**
  - **Cause**: Container uses `100vh`, which ignores collapsible mobile address chrome.
  - **Fix**: Replace `100vh` (`h-screen`) with `100dvh` (`min-h-dvh`) and add `pb-[env(safe-area-inset-bottom)]`.

## See Also

- CSS custom property token palettes and the full accessibility rule table live in `references/details.md`.
- Companion skills in this plugin include `skills/skill-creator` and `skills/context-mode`.
- Related frontend workflows pair with `web-app-development`, `generative_ui`, and `accessibility-compliance`.
