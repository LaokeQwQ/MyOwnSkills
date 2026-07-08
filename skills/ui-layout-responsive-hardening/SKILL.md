---
name: ui-layout-responsive-hardening
description: Harden admin panels, dashboards, and CRUD web UIs against layout and responsiveness regressions. Use when fixing button wrapping, action-bar overflow, table status/action columns, form responsiveness, or perceived jank during panel switching and preview toggles.
---

# UI Layout Responsive Hardening

Use this skill for existing web UIs that already function but feel fragile under different widths, content lengths, or interaction states.

Typical triggers:

- Buttons collapse into vertical text
- Header action bars overflow or push primary actions out of view
- Table status or action columns do not adapt on narrower widths
- Collection/item editors feel janky while switching resources
- Preview panels cause abrupt layout jumps
- Mobile and tablet layouts are technically present but visually brittle

## Core workflow

1. Identify the exact container that owns the broken layout.
2. Fix the container rules first, then the child sizing rules.
3. Only add breakpoints after the base flex/grid behavior is correct.
4. Improve perceived performance separately from real performance.

## Layout rules that prevent common regressions

- For action bars:
  - use `display: flex`
  - use `align-items: center`
  - use `gap`
  - allow `flex-wrap: wrap` when the row can legitimately wrap
  - give long-lived action buttons `white-space: nowrap`
  - set `flex-shrink: 0` on buttons that must not collapse
  - override generic button rules if they force `width: 100%`

- For page-level action buttons:
  - unify height, padding, radius, and alignment across primary and outlined buttons
  - prefer scoped rules like `.page-actions > .btn-primary` instead of global button changes
  - use `width: auto` and `max-width: 100%`
  - set explicit `min-width` for longer labels such as preview or save
  - audit normal, primary, loading, and disabled states separately; loading must keep readable foreground color, and disabled styling must never leak into a clickable primary button
  - avoid broad descendant color rules such as `.card span` or `.panel-heading span`; they can override button-content spans and create clickable blue buttons with muted gray text. Scope muted copy to direct children or named helper classes.

- For search boxes inside action bars:
  - give them flexible width with `flex: 1 1 ...`
  - keep a minimum width so they do not become unusable
  - allow them to wrap below buttons on smaller widths

- For row-level action groups in tables:
  - wrap buttons in a dedicated `.actions` container inside the cell
  - use `display: flex`
  - use `flex-wrap: wrap`
  - use `justify-content: flex-end` for wide layouts
  - switch to `justify-content: flex-start` or stacked mode on narrow layouts

- For status badges:
  - wrap them in a dedicated `.status` container
  - use `display: flex`
  - use `flex-wrap: wrap`
  - remove badge margins that assume a single-row inline flow

## Responsiveness strategy

- First fix the desktop/base layout without media queries.
- Then add a tablet breakpoint for wrapping and width rebalancing.
- Then add a mobile breakpoint for stacking where needed.

Good breakpoint behaviors:

- Tablet:
  - keep actions on one visual block
  - allow controlled wrap
  - avoid full vertical collapse unless necessary

- Mobile:
  - stack action buttons if the horizontal layout stops being readable
  - expand tiny icon-only actions to full-width controls when needed
  - reduce hidden columns before shrinking everything else

## Perceived performance and jank reduction

If switching panels or resources feels slow:

- cache fetched resources by id
- cache preview output by exact content or content hash
- debounce expensive rerenders caused by text input
- avoid rebuilding entire side lists on every keystroke
- add light opacity/translate transitions during content switching

Useful patterns:

- resource cache: `Map<id, payload>`
- preview cache: `{ content, html }`
- rerender throttle: `setTimeout(..., 100-150ms)`
- switching class: `.is-switching { opacity: ...; transform: ... }`

## Decision rules

- If text becomes vertical:
  - check `white-space`
  - check `flex-shrink`
  - check whether the parent is narrower than the child label

- If a primary button escapes the layout:
  - check whether a generic `.btn-primary { width: 100% }` is leaking into header actions
  - add a scoped override in the page action area

- If table action columns look broken:
  - do not rely on raw `td` flow
  - add explicit wrapper containers for status and actions

- If a fix only works at one width:
  - the base container model is still wrong
  - revisit flex/grid ownership before adding more breakpoints

## Admin/dashboard-specific cautions

- Management UIs often mix:
  - long labels
  - icon buttons
  - search inputs
  - badge groups
  - sticky sidebars

- These combinations break easily if width rules are implicit.
- Prefer explicit layout ownership:
  - header action row owns wrap behavior
  - search box owns flexible growth
  - button group owns no-shrink behavior
  - table cell owns action/status wrapping

## Output expectations

When applying this skill:

- fix the concrete regression
- make the surrounding layout more resilient
- avoid broad global CSS changes unless the bug is truly global
- preserve the existing design language
