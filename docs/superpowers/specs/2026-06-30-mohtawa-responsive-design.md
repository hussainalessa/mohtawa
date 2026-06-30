# Mohtawa landing page — responsive desktop & tablet design

**Date:** 2026-06-30
**File affected:** `mohtawa-landing.html` (single self-contained file)
**Status:** Approved design — ready for implementation planning

## Goal

The Mohtawa consultation landing page is currently a fixed 440px mobile-first
phone column centered on a gray backdrop. On wide screens this wastes most of
the viewport. Add genuine responsive behavior so the page re-architects into a
full marketing layout on larger devices, while preserving the existing mobile
design exactly.

## Approach

**One file, CSS-driven responsive layout.** Extend `mohtawa-landing.html` with
media queries that re-flow the existing markup. The phone experience stays
byte-for-byte identical; tablet and desktop layouts are layered on via CSS grid
/ flexbox. The JS-rendered sections (testimonials, FAQ, booking) keep their
logic; only the booking render gets a tiny structural tweak (two wrapper divs)
so CSS can place its halves side by side.

Rejected alternatives:
- **Separate desktop file** — duplicates markup/content, drifts out of sync.
- **JS-driven layout switching** — unnecessary; layout is presentational and
  belongs in CSS.

## Breakpoints (three tiers)

| Tier | Range | Container width |
|------|-------|-----------------|
| Phone | `< 768px` | today's 440px phone column (unchanged) |
| Tablet | `768px – 1024px` | centered, ~720px max |
| Desktop | `> 1024px` | centered, ~1120px (`--container-lg`) |

Media query edges: `@media (min-width: 768px)` for tablet-and-up,
`@media (min-width: 1024px)` for desktop. Phone styles remain the base (no
media query), guaranteeing the current mobile design is untouched.

## Global frame & header

- **Phone:** unchanged — 440px column, gray backdrop + azure glow, rounded
  card at ≥480px, sticky bottom CTA.
- **Tablet / Desktop:** drop the phone-card framing and gray backdrop. Sections
  become full-bleed bands (alternating `--surface-card` white and
  `--surface-page`) with their inner content constrained to the centered
  container width. Page reads as the brand's clean white/light-gray surfaces
  edge to edge.
- **Header (tablet/desktop):** real nav bar — logo left; anchor links
  **Services · Why us · FAQ** (existing IDs `#services`, `#why`, `#faq`); accent
  **"Book a call"** pill far right. Keep the sticky blur + bottom border.
- **Sticky bottom CTA:** hidden on tablet/desktop (header CTA replaces it).

## Section-by-section behavior

### Hero & client logos
- **Hero desktop:** two-column. Left = eyebrow chip, headline (~56px), lede,
  accent CTA + trust row. Right = skyline photo with gradient scrim + floating
  "One team, end to end" caption card. Capsule-scatter pattern anchors the band's
  top-right. Vertically centered, generous padding.
- **Hero tablet:** single-column centered (copy above photo); larger type, a
  wider/shorter photo. (Two-column needs more width than a tablet offers.)
- **Hero phone:** unchanged.
- **Client logos tablet/desktop:** single row of 6 names (from 3×2), evenly
  spaced under the overline.

### Services & Why us
- **Services desktop:** 3-up row of equal-height cards. Card internals
  unchanged.
- **Services tablet:** grid via `auto-fit` (2-up, or 3-up if it fits the width).
- **Why us desktop:** heading occupies its own left column (~1/3 width) with the
  four points as a 2×2 grid in the right column (~2/3 width), as a single row.
- **Why us tablet:** 2×2 grid, heading above.
- **Phone:** both unchanged (single-column stacks).

### Testimonials & FAQ
- **Testimonials desktop:** 3-up row, all three cards visible, equal height, no
  horizontal scroll. Avatars/photos unchanged (Khaled keeps the cafe photo).
- **Testimonials tablet:** `auto-fit` grid — 2 (rail still swipeable) or 3 if
  they fit.
- **FAQ desktop:** accordion centered in a constrained ~760px column under the
  centered heading; one-open-at-a-time behavior unchanged.
- **FAQ tablet:** single column, full container width.
- **Phone:** both unchanged (swipe rail + stacked accordion).

### Booking & footer
- **Booking desktop:** card becomes two-column. Left = date rail + time grid.
  Right = form fields + "Confirm my consultation" button + fineprint. Heading/
  lede above, centered.
  - **Only markup change in the effort:** wrap the two groups in `<div>`s inside
    the booking render so CSS grid can place them side by side. All picker,
    validation, and confirmation logic stays identical.
  - **Confirmation ("You're booked in.") state:** stays a single centered card.
- **Booking tablet:** single-column, wider and centered.
- **Booking phone:** unchanged.
- **Footer tablet/desktop:** multi-column row — logo + blurb left; contact list
  (email/phone/location) as its own column right; `© 2026 Mohtawa Marketing` /
  `www.mohtawa.co` line spans full width along the bottom. Azure capsule pattern
  stays in the corner.
- **Footer phone:** unchanged.

## Constraints & non-goals

- **No change to mobile (<768px) output.** Verifiable by diffing rendered
  phone-width DOM/appearance before and after.
- **No new dependencies, no build step** — stays a single self-contained file.
- **No copy changes, no new sections** — purely layout/responsiveness plus the
  desktop header nav links (which reuse existing anchors).
- **Preserve all interaction logic** — FAQ accordion, booking picker/validation/
  confirmation, smooth-scroll, and (mobile) sticky CTA all behave as today.
- Reuse design-system tokens already inlined (`--container-lg`, spacing, radii,
  shadows); no new color/type values invented.

## Verification

- Preview at three widths: **375px** (phone — must match current design),
  **820px** (tablet), **1280px** (desktop).
- Confirm: no console errors; no horizontal body scroll at any width; all images
  load; nav anchors scroll to the right sections; booking two-column wiring still
  selects date/time, validates, and reaches the confirmation state on desktop.
- Compare phone-width rendering against the current committed version to confirm
  it is unchanged.
