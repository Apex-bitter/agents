# Web Design Guidelines Reference Details

## CSS Custom Properties & Surface Token Architecture

Define semantic surface, border, and typography tokens in `:root` and `.dark` so components adapt cleanly across themes without hardcoded hex values:

```css
:root {
  --surface-canvas: #fafafa;
  --surface-card: #ffffff;
  --surface-overlay: rgba(255, 255, 255, 0.85);
  --text-primary: #111827;
  --text-secondary: #4b5563;
  --text-muted: #6b7280;
  --border-subtle: rgba(17, 24, 39, 0.08);
  --focus-ring: #2563eb;
}

.dark {
  --surface-canvas: #090d16;
  --surface-card: #111827;
  --surface-overlay: rgba(17, 24, 39, 0.82);
  --text-primary: #f9fafb;
  --text-secondary: #d1d5db;
  --text-muted: #9ca3af;
  --border-subtle: rgba(255, 255, 255, 0.09);
  --focus-ring: #60a5fa;
}
```

## Automated Accessibility & CLS Audit Checklist

| Category | Rule ID | Target Selector / Property | Pass Criterion |
| :--- | :--- | :--- | :--- |
| Keyboard Focus | `focus-visible-ring` | `button, a, input, [tabindex]` | Visible `:focus-visible` outline (`>= 2px`, `>= 3:1` contrast) |
| Hit Targets | `touch-target-size` | Interactive controls | `>= 36x36px` desktop, `>= 44x44px` mobile |
| Numeric Stability | `tabular-numerals` | Counters, prices, tables, timers | `font-variant-numeric: tabular-nums` applied |
| Flex/Grid Overflow | `min-width-zero` | Flex/grid children with dynamic text | `min-width: 0` + `overflow: hidden` + `text-overflow: ellipsis` |
| Layout Shift | `cls-aspect-ratio` | `img, video, iframe, canvas` | Explicit `width`/`height` or `aspect-ratio` reserved |
| Motion Safety | `reduced-motion` | Keyframe animations & transitions | Wrapped in `@media (prefers-reduced-motion: no-preference)` |
