---
name: web-design-guidelines
description: Use when building, reviewing, or auditing web interfaces, React/Next.js components, dashboards, or HTML artifacts for layout rhythm, typography, accessibility, keyboard navigation, and interaction polish.
---

# Web Design & UI Audit Guidelines

Apply production web interface engineering standards (distilled from Vercel's `web-design-guidelines` and modern design-system practices) when authoring or reviewing frontend code and interactive HTML artifacts.

## When to Use

- Building or restyling web apps, dashboards, landing pages, or standalone HTML widgets.
- Conducting a UI/UX quality and accessibility audit before shipping frontend changes.
- Fixing layout shift (CLS), broken keyboard focus, cramped spacing, or inconsistent dark/light theming.

## 1. Interactions, Keyboard & Focus

- **Full keyboard operability**: Every interactive element must be reachable via `Tab` and operable with `Enter` / `Space` / `Escape`.
- **Visible focus rings**: Never use `outline: none` without providing a high-contrast `:focus-visible` ring (`focus-visible:ring-2 focus-visible:ring-offset-2`).
- **Hit targets**: Keep interactive buttons, tabs, and icon controls at least `36px × 36px` (`44px × 44px` on touch viewports).
- **Optimistic & disabled states**: Disable submit triggers during in-flight mutations, preserve button width to prevent layout jump, and display clear inline error recovery.

## 2. Typography & Numeric Discipline

- **Tabular numerals**: Apply `font-variant-numeric: tabular-nums` (`tabular-nums` in Tailwind) to all counters, timestamps, prices, tables, and live metrics so digits never jitter on update.
- **Readable measure & hierarchy**: Constrain body copy to `60ch–75ch`, pair `text-wrap: balance` on headings with `text-wrap: pretty` on paragraphs, and maintain a strict 3-tier typographic scale (heading, body, muted metadata).
- **Truncation safety**: Every flex/grid child that holds user-supplied or dynamic text needs `min-width: 0` (`min-w-0`) and explicit `truncate` or `line-clamp-*` rules so long strings never blow out container widths.

## 3. Layout, Surfaces & Visual Rhythm

- **Zero layout shift (CLS)**: Reserve explicit aspect ratios (`aspect-video`, `width`/`height`) or skeleton placeholders for asynchronous images, charts, and embeds.
- **Layered surface elevation**: Differentiate background canvas, cards, sticky headers, and modals using subtle border luminance (`1px solid` with low-alpha contrast) rather than heavy drop shadows alone.
- **Scroll & viewport hygiene**: Use `100dvh` (`min-h-dvh`) instead of `100vh` for full-height layouts, and keep sticky toolbars compact with `backdrop-filter: blur(12px)`.

## 4. Motion & Accessibility Compliance

- **Respect `prefers-reduced-motion`**: Wrap non-essential transforms and animations in `@media (prefers-reduced-motion: no-preference)` (`motion-safe:` in Tailwind).
- **Semantic HTML & ARIA**: Use native `<button>`, `<nav>`, `<main>`, `<dialog>`, and `<label for="...">` elements before reaching for custom ARIA roles; icon-only buttons require `aria-label`.
- **Contrast**: Verify ≥ 4.5:1 contrast for body text and ≥ 3:1 for icons, borders, and secondary labels across both light and dark themes.
