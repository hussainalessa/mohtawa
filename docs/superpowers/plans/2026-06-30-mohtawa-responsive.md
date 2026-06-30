# Mohtawa Landing Page — Responsive Layout Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add tablet and desktop responsive layouts to `mohtawa-landing.html` so it re-architects into a full marketing layout on wide screens while the phone design stays byte-for-byte identical.

**Architecture:** All changes live in the single self-contained file `mohtawa-landing.html`. Phone (<768px) styles are the untouched base. Tablet (≥768px) and desktop (≥1024px) layouts are added as a clearly-commented responsive block appended at the end of the existing `<style>`, plus two small markup additions (header nav links; two wrapper divs in the booking render). All JavaScript interaction logic is unchanged.

**Tech Stack:** Plain HTML + CSS (CSS grid/flex, media queries, CSS custom properties) + the existing vanilla JS. No build step, no dependencies. Mohtawa design-system tokens are already inlined in `:root`.

## Global Constraints

- Single self-contained file; **no build step, no new dependencies** (copied verbatim from spec).
- **No change to phone (<768px) rendered output.** Guaranteed by construction: every new rule lives inside `@media (min-width: 768px)` or `@media (min-width: 1024px)`.
- **No copy changes, no new sections.** Only new visible content allowed: desktop header nav links reusing existing anchors (`#services`, `#why`, `#faq`).
- **Preserve all interaction logic:** FAQ accordion, booking picker/validation/confirmation, smooth-scroll, mobile sticky CTA.
- Breakpoints: phone `<768px` (base, no query), tablet `768–1024px` (`@media (min-width: 768px)`), desktop `>1024px` (`@media (min-width: 1024px)`). Container max-widths: tablet **720px**, desktop **1120px** (`--container-lg`).
- Reuse inlined design tokens; invent no new color/type values.
- **No horizontal body scroll at any width.**

## File Structure

- **Modify:** `mohtawa-landing.html` — root source of truth (committed).
- **Sync:** `public/mohtawa-landing.html` — gitignored preview copy. After every edit to the root file, copy it into `public/` before previewing (the dev server roots at `public/`). This copy is never committed.

## Preview / verification setup

The repo has a dev server config `mohtawa-static` (serves `public/` on port 4321).

- Start once: `preview_start { name: "mohtawa-static" }` → returns `serverId`. Reuse that `serverId` for all `preview_eval` / `preview_resize` calls below.
- The page URL is `http://localhost:4321/mohtawa-landing.html`.
- **Sync command** (run after each edit, before previewing):
  ```bash
  cp "mohtawa-landing.html" public/mohtawa-landing.html
  ```
- To load/refresh in the preview: `preview_eval { serverId, expression: "location.href='http://localhost:4321/mohtawa-landing.html'" }` then re-eval checks.
- Widths used: **375** (phone), **820** (tablet), **1280** (desktop) via `preview_resize { serverId, width, height: 900 }`.

---

### Task 1: Responsive foundation — frame, header nav, sticky CTA, smooth scroll

Establishes the responsive block, drops the phone-card framing on ≥768px (full-bleed bands with centered content via a `--pad-x` custom property), turns the header into a nav bar, hides the mobile sticky CTA, and enables anchor smooth-scroll with a sticky-header offset.

**Files:**
- Modify: `mohtawa-landing.html` — header markup (`.site-header__inner`) and the `<style>` block (append responsive section).

**Interfaces:**
- Produces: CSS custom property `--pad-x` (horizontal padding that centers content within the container width); responsive section comment marker `/* ===== RESPONSIVE ===== */` that later tasks append into; header nav element `.site-header__nav`.

- [ ] **Step 1: Add the header nav markup**

In the header, replace this block:

```html
      <div class="site-header__inner">
        <img class="site-header__logo" src="assets/logos/mohtawa-logo-color.png" alt="Mohtawa">
        <button class="btn btn--accent btn--sm btn--pill" data-scroll-book style="white-space:nowrap;">Book a call</button>
      </div>
```

with:

```html
      <div class="site-header__inner">
        <img class="site-header__logo" src="assets/logos/mohtawa-logo-color.png" alt="Mohtawa">
        <nav class="site-header__nav">
          <a href="#services">Services</a>
          <a href="#why">Why us</a>
          <a href="#faq">FAQ</a>
        </nav>
        <button class="btn btn--accent btn--sm btn--pill" data-scroll-book style="white-space:nowrap;">Book a call</button>
      </div>
```

- [ ] **Step 2: Append the responsive foundation CSS**

Immediately **before** the `@media (prefers-reduced-motion: reduce)` rule at the end of the `<style>` block, insert:

```css
  /* ============================================================
     ===== RESPONSIVE =====
     Phone (<768px) is the untouched base above.
     Tablet ≥768px (720px container) · Desktop ≥1024px (1120px container).
     Later tasks append their media-query rules into this section.
     ============================================================ */

  /* anchor smooth-scroll with an offset for the sticky header */
  html { scroll-behavior: smooth; }
  section[id], #book { scroll-margin-top: 80px; }

  /* nav links are desktop/tablet only */
  .site-header__nav { display: none; }

  @media (min-width: 768px) {
    /* centered horizontal padding → full-bleed bands, centered content */
    :root { --pad-x: max(22px, calc((100% - 720px) / 2)); }

    /* drop the phone-card framing */
    .backdrop { display: block; padding: 0; background: var(--surface-card); }
    .phone { max-width: none; width: auto; min-height: 0; background: transparent;
      box-shadow: none; border-radius: 0; overflow: visible; }

    /* header becomes a nav bar; content centered to the container */
    .site-header__inner { max-width: 1120px; margin: 0 auto; }
    .site-header__nav { display: flex; align-items: center; gap: 26px; margin-left: auto; margin-right: 28px; }
    .site-header__nav a { font-size: 15px; font-weight: 600; color: var(--text-muted); }
    .site-header__nav a:hover { color: var(--azure-600); }

    /* center content of the simple bands (hero/testimonials/booking/footer
       are handled in their own tasks) */
    .logos, .services, .why, .faq {
      padding-left: var(--pad-x); padding-right: var(--pad-x);
    }

    /* sticky bottom CTA is a mobile-only pattern */
    .sticky-cta { display: none; }
  }

  @media (min-width: 1024px) {
    :root { --pad-x: max(22px, calc((100% - 1120px) / 2)); }
  }
```

- [ ] **Step 3: Sync to preview and load**

Run:
```bash
cp "mohtawa-landing.html" public/mohtawa-landing.html
```
Then `preview_start { name: "mohtawa-static" }` (note the `serverId`) and
`preview_eval { serverId, expression: "location.href='http://localhost:4321/mohtawa-landing.html'" }`.

- [ ] **Step 4: Verify desktop (1280px)**

`preview_resize { serverId, width: 1280, height: 900 }`, then `preview_eval`:
```js
(() => ({
  navDisplay: getComputedStyle(document.querySelector('.site-header__nav')).display,
  stickyDisplay: getComputedStyle(document.querySelector('.sticky-cta')).display,
  phoneMaxWidth: getComputedStyle(document.querySelector('.phone')).maxWidth,
  servicesLeftPad: parseInt(getComputedStyle(document.querySelector('.services')).paddingLeft, 10) > 22,
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
  navLinks: [...document.querySelectorAll('.site-header__nav a')].map(a => a.textContent),
}))()
```
Expected: `navDisplay:"flex"`, `stickyDisplay:"none"`, `phoneMaxWidth:"none"`, `servicesLeftPad:true`, `noHScroll:true`, `navLinks:["Services","Why us","FAQ"]`.

- [ ] **Step 5: Verify phone unchanged (375px)**

`preview_resize { serverId, width: 375, height: 812 }`, then `preview_eval`:
```js
(() => ({
  navDisplay: getComputedStyle(document.querySelector('.site-header__nav')).display,
  phoneMaxWidth: getComputedStyle(document.querySelector('.phone')).maxWidth,
  stickyTransitionable: !!document.querySelector('.sticky-cta'),
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
}))()
```
Expected: `navDisplay:"none"`, `phoneMaxWidth:"440px"`, `stickyTransitionable:true`, `noHScroll:true`.

- [ ] **Step 6: Verify anchor scroll**

At 1280px: `preview_eval { serverId, expression: "document.querySelector('.site-header__nav a[href=\\'#faq\\']').click(); new Promise(r=>setTimeout(()=>r(Math.round(document.getElementById('faq').getBoundingClientRect().top)),600))" }`
Expected: a small number near 80 (the section top sits just below the sticky header, not at 0 hidden behind it).

- [ ] **Step 7: Commit**

```bash
git add mohtawa-landing.html
git commit -m "Add responsive foundation: bands, header nav, hide mobile sticky CTA"
```

---

### Task 2: Hero & client logos responsive

**Files:**
- Modify: `mohtawa-landing.html` — append CSS into the RESPONSIVE section.

**Interfaces:**
- Consumes: `--pad-x`, the RESPONSIVE section, `.hero`, `.hero__inner`, `.hero__copy`, `.hero__title`, `.hero__lede`, `.hero__photo-wrap`, `.hero__photo`, `.logos__grid` (all defined in the base styles).

- [ ] **Step 1: Append hero + logos CSS**

Inside the `@media (min-width: 768px)` block (after the existing rules), add:

```css
    /* hero — tablet: centered band, larger type, wider photo */
    .hero { padding-left: var(--pad-x); padding-right: var(--pad-x); }
    .hero__copy { padding-left: 0; padding-right: 0; padding-top: 48px; }
    .hero__title { font-size: 46px; }
    .hero__lede { max-width: 560px; }
    .hero__photo-wrap { padding-left: 0; padding-right: 0; }
    .hero__photo { aspect-ratio: 16 / 9; }

    /* client logos — single row of 6 */
    .logos__grid { grid-template-columns: repeat(6, 1fr); gap: 10px; }
```

Then add a **new** `@media (min-width: 1024px)` rule (separate from the foundation one, appended right after it) for the two-column hero:

```css
  @media (min-width: 1024px) {
    /* hero — desktop: two columns, copy left / photo right */
    .hero__inner {
      display: grid; grid-template-columns: 1.05fr 0.95fr;
      gap: 48px; align-items: center; padding: 64px 0;
    }
    .hero__copy { padding-top: 0; }
    .hero__title { font-size: 56px; }
    .hero__lede { max-width: none; }
    .hero__photo-wrap { padding: 0; }
    .hero__photo { aspect-ratio: auto; height: 100%; min-height: 440px; }
  }
```

- [ ] **Step 2: Sync + reload**

```bash
cp "mohtawa-landing.html" public/mohtawa-landing.html
```
`preview_eval { serverId, expression: "location.reload()" }`.

- [ ] **Step 3: Verify desktop (1280px)**

`preview_resize { serverId, width: 1280, height: 900 }`, then `preview_eval`:
```js
(() => ({
  heroDisplay: getComputedStyle(document.querySelector('.hero__inner')).display,
  heroCols: getComputedStyle(document.querySelector('.hero__inner')).gridTemplateColumns.split(' ').length,
  titlePx: getComputedStyle(document.querySelector('.hero__title')).fontSize,
  logosCols: getComputedStyle(document.querySelector('.logos__grid')).gridTemplateColumns.split(' ').length,
  copyAboveOrLeft: document.querySelector('.hero__copy').getBoundingClientRect().right
                   <= document.querySelector('.hero__photo-wrap').getBoundingClientRect().left + 1,
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
}))()
```
Expected: `heroDisplay:"grid"`, `heroCols:2`, `titlePx:"56px"`, `logosCols:6`, `copyAboveOrLeft:true`, `noHScroll:true`.

- [ ] **Step 4: Verify tablet (820px)**

`preview_resize { serverId, width: 820, height: 1000 }`, then `preview_eval`:
```js
(() => ({
  heroDisplay: getComputedStyle(document.querySelector('.hero__inner')).display,
  titlePx: getComputedStyle(document.querySelector('.hero__title')).fontSize,
  logosCols: getComputedStyle(document.querySelector('.logos__grid')).gridTemplateColumns.split(' ').length,
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
}))()
```
Expected: `heroDisplay:"block"` (single-column), `titlePx:"46px"`, `logosCols:6`, `noHScroll:true`.

- [ ] **Step 5: Verify phone unchanged (375px)**

`preview_resize { serverId, width: 375, height: 812 }`, then `preview_eval`:
```js
(() => ({
  heroDisplay: getComputedStyle(document.querySelector('.hero__inner')).display,
  titlePx: getComputedStyle(document.querySelector('.hero__title')).fontSize,
  logosCols: getComputedStyle(document.querySelector('.logos__grid')).gridTemplateColumns.split(' ').length,
}))()
```
Expected: `heroDisplay:"block"`, `titlePx:"38px"`, `logosCols:3`.

- [ ] **Step 6: Commit**

```bash
git add mohtawa-landing.html
git commit -m "Responsive hero (2-col desktop) and 6-up client logos"
```

---

### Task 3: Services & Why us responsive

**Files:**
- Modify: `mohtawa-landing.html` — append CSS into the RESPONSIVE section.

**Interfaces:**
- Consumes: `.card-stack`, `.service-card`, `.why`, `.section-head--left`, `.why__list`, `.why__item` (base styles).

- [ ] **Step 1: Append services + why-us CSS**

Inside the `@media (min-width: 768px)` block, add:

```css
    /* services — responsive grid (2-up tablet, fills available width) */
    .card-stack { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 18px; }
    .service-card { height: 100%; }

    /* why us — 2×2 grid */
    .why__list { display: grid; grid-template-columns: 1fr 1fr; gap: 22px 24px; }
```

Inside the **desktop** `@media (min-width: 1024px)` block (the one created in Task 2), add:

```css
    /* services — exactly 3-up on desktop */
    .card-stack { grid-template-columns: repeat(3, 1fr); gap: 20px; }

    /* why us — heading in its own left column, 2×2 grid on the right */
    .why { display: grid; grid-template-columns: 1fr 2fr; gap: 40px; align-items: start; }
    .why .section-head--left { margin-bottom: 0; }
    .why__list { gap: 26px 28px; }
```

- [ ] **Step 2: Sync + reload**

```bash
cp "mohtawa-landing.html" public/mohtawa-landing.html
```
`preview_eval { serverId, expression: "location.reload()" }`.

- [ ] **Step 3: Verify desktop (1280px)**

`preview_resize { serverId, width: 1280, height: 900 }`, then `preview_eval`:
```js
(() => ({
  servicesCols: getComputedStyle(document.querySelector('.card-stack')).gridTemplateColumns.split(' ').length,
  whyDisplay: getComputedStyle(document.querySelector('.why')).display,
  whyCols: getComputedStyle(document.querySelector('.why')).gridTemplateColumns.split(' ').length,
  whyListCols: getComputedStyle(document.querySelector('.why__list')).gridTemplateColumns.split(' ').length,
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
}))()
```
Expected: `servicesCols:3`, `whyDisplay:"grid"`, `whyCols:2`, `whyListCols:2`, `noHScroll:true`.

- [ ] **Step 4: Verify tablet (820px)**

`preview_resize { serverId, width: 820, height: 1000 }`, then `preview_eval`:
```js
(() => ({
  servicesCols: getComputedStyle(document.querySelector('.card-stack')).gridTemplateColumns.split(' ').length,
  whyListCols: getComputedStyle(document.querySelector('.why__list')).gridTemplateColumns.split(' ').length,
  whyDisplay: getComputedStyle(document.querySelector('.why')).display,
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
}))()
```
Expected: `servicesCols` ≥ 2 (auto-fit; 2 or 3), `whyListCols:2`, `whyDisplay:"block"` (heading stacks above grid on tablet), `noHScroll:true`.

- [ ] **Step 5: Verify phone unchanged (375px)**

`preview_resize { serverId, width: 375, height: 812 }`, then `preview_eval`:
```js
(() => ({
  servicesDisplay: getComputedStyle(document.querySelector('.card-stack')).display,
  whyListDisplay: getComputedStyle(document.querySelector('.why__list')).display,
}))()
```
Expected: `servicesDisplay:"flex"`, `whyListDisplay:"flex"` (both base single-column stacks).

- [ ] **Step 6: Commit**

```bash
git add mohtawa-landing.html
git commit -m "Responsive services (3-up) and why-us (2x2 + side heading)"
```

---

### Task 4: Testimonials & FAQ responsive

**Files:**
- Modify: `mohtawa-landing.html` — append CSS into the RESPONSIVE section.

**Interfaces:**
- Consumes: `.testimonials`, `.testimonials__head`, `.testimonials__rail`, `.testimonial`, `.faq`, `.faq__list` (base styles).

- [ ] **Step 1: Append testimonials + FAQ CSS**

Inside the `@media (min-width: 768px)` block, add:

```css
    /* testimonials — center head + rail to the container; rail becomes a grid */
    .testimonials__head { padding-left: var(--pad-x); padding-right: var(--pad-x); }
    .testimonials__rail {
      padding-left: var(--pad-x); padding-right: var(--pad-x);
      display: grid; grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
      gap: 18px; overflow-x: visible;
    }
    .testimonial { flex: initial; }

    /* faq — center to the container, constrain reading width */
    .faq__list { max-width: 760px; margin-left: auto; margin-right: auto; }
```

Inside the **desktop** `@media (min-width: 1024px)` block, add:

```css
    /* testimonials — exactly 3-up on desktop */
    .testimonials__rail { grid-template-columns: repeat(3, 1fr); }
```

- [ ] **Step 2: Sync + reload**

```bash
cp "mohtawa-landing.html" public/mohtawa-landing.html
```
`preview_eval { serverId, expression: "location.reload()" }`.

- [ ] **Step 3: Verify desktop (1280px)**

`preview_resize { serverId, width: 1280, height: 900 }`, then `preview_eval`:
```js
(() => ({
  railDisplay: getComputedStyle(document.querySelector('.testimonials__rail')).display,
  railCols: getComputedStyle(document.querySelector('.testimonials__rail')).gridTemplateColumns.split(' ').length,
  cardCount: document.querySelectorAll('.testimonial').length,
  faqMaxW: getComputedStyle(document.querySelector('.faq__list')).maxWidth,
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
}))()
```
Expected: `railDisplay:"grid"`, `railCols:3`, `cardCount:3`, `faqMaxW:"760px"`, `noHScroll:true`.

- [ ] **Step 4: Verify tablet (820px)**

`preview_resize { serverId, width: 820, height: 1000 }`, then `preview_eval`:
```js
(() => ({
  railDisplay: getComputedStyle(document.querySelector('.testimonials__rail')).display,
  railCols: getComputedStyle(document.querySelector('.testimonials__rail')).gridTemplateColumns.split(' ').length,
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
}))()
```
Expected: `railDisplay:"grid"`, `railCols` ≥ 2, `noHScroll:true`.

- [ ] **Step 5: Verify phone unchanged (375px)**

`preview_resize { serverId, width: 375, height: 812 }`, then `preview_eval`:
```js
(() => ({
  railDisplay: getComputedStyle(document.querySelector('.testimonials__rail')).display,
  railOverflowX: getComputedStyle(document.querySelector('.testimonials__rail')).overflowX,
  faqMaxW: getComputedStyle(document.querySelector('.faq__list')).maxWidth,
}))()
```
Expected: `railDisplay:"flex"`, `railOverflowX:"auto"`, `faqMaxW:"none"`.

- [ ] **Step 6: Commit**

```bash
git add mohtawa-landing.html
git commit -m "Responsive testimonials (3-up grid) and centered FAQ column"
```

---

### Task 5: Booking two-column & footer multi-column

This task contains the only JS-markup change: wrapping the booking card's two halves so CSS can place them side by side on desktop. The wrapper divs must NOT break the existing event wiring (which uses `root.querySelectorAll('.date-chip' / '.time-slot' / 'input[name=...]' / 'select' / '#confirm-btn')` — all still descendants of `#book`, so selectors keep working).

**Files:**
- Modify: `mohtawa-landing.html` — the `renderForm()` markup inside the booking IIFE, the footer markup, and the RESPONSIVE CSS section.

**Interfaces:**
- Consumes: the booking IIFE's `renderForm()` template literal; `.booking__card`, `.booking__divider`, `.site-footer`, `.site-footer__contact`, `.site-footer__bottom` (base styles).
- Produces: new wrapper classes `.booking__grid`, `.booking__col--when`, `.booking__col--who`; footer wrapper `.site-footer__inner`.

- [ ] **Step 1: Wrap the booking card halves in `renderForm()`**

In `renderForm()`, the card currently holds (in order): the "Select a day" label, `.date-rail`, the "Select a time" label, `.time-grid`, `.booking__divider`, `.booking__fields`, and the submit `<div>`. Restructure the inside of `<div class="booking__card"> … </div>` so those children are grouped:

```html
      <div class="booking__card">
        <div class="booking__grid">
          <div class="booking__col booking__col--when">
            <div class="booking__row-label" style="margin-bottom:12px;">
              ${icon('calendar', { size: 18, color: 'var(--azure-600)' })}
              <span class="t">Select a day</span>
            </div>
            <div class="date-rail no-scrollbar">
              ${dates.map((d, i) => `
                <div class="date-chip" data-date="${i}">
                  <span class="wd">${WD[d.getDay()]}</span>
                  <span class="dd">${d.getDate()}</span>
                  <span class="mo">${MO[d.getMonth()]}</span>
                </div>`).join('')}
            </div>
            <div class="booking__row-label" style="margin:22px 0 12px;">
              ${icon('clock', { size: 18, color: 'var(--azure-600)' })}
              <span class="t">Select a time</span>
              <span class="booking__tz">GMT+3 · Kuwait</span>
            </div>
            <div class="time-grid">
              ${TIMES.map((t) => `<div class="time-slot" data-time="${t}">${fmtTime(t).replace(':00', '')}</div>`).join('')}
            </div>
          </div>
          <div class="booking__col booking__col--who">
            <div class="booking__divider"></div>
            <div class="booking__fields">
              <label class="field">
                <span class="field__label">Full name</span>
                <input class="field__control" type="text" name="name" placeholder="e.g. Hussain Al-Sabah" value="${state.name}">
              </label>
              <label class="field">
                <span class="field__label">Work email</span>
                <input class="field__control" type="email" name="email" placeholder="you@company.com" value="${state.email}">
              </label>
              <label class="field">
                <span class="field__label">Phone / WhatsApp</span>
                <input class="field__control" type="tel" name="phone" placeholder="+965 ..." value="${state.phone}">
              </label>
              <label class="field">
                <span class="field__label">What would you like to discuss?</span>
                <span class="field__select-wrap">
                  <select class="field__control" name="topic">
                    ${TOPICS.map((o) => `<option value="${o}"${o === state.topic ? ' selected' : ''}>${o}</option>`).join('')}
                  </select>
                  <span class="field__chevron"><svg width="18" height="18" viewBox="0 0 24 24" fill="none"><path d="M6 9l6 6 6-6" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg></span>
                </span>
              </label>
            </div>
            <div style="margin-top:22px;">
              <button class="btn btn--accent btn--lg btn--pill btn--block" id="confirm-btn"${isValid() ? '' : ' disabled'}>
                Confirm my consultation ${icon('arrow', { size: 20, color: '#fff' })}
              </button>
              <p class="booking__fineprint">
                ${icon('shield', { size: 14, color: 'var(--text-subtle)' })} No spam. We reply within one business day.
              </p>
            </div>
          </div>
        </div>
      </div>`;
```

(The `booking__head` block above the card and all the event-wiring code below the template are unchanged.)

- [ ] **Step 2: Wrap the footer content in `.site-footer__inner`**

In the footer markup, wrap everything **except** the `.site-footer__pattern` div in an inner container, and split contact into its own column. Replace the footer body:

```html
    <footer class="site-footer">
      <div class="site-footer__pattern" aria-hidden="true"></div>
      <div class="site-footer__inner">
        <div class="site-footer__brand">
          <img class="site-footer__logo" src="assets/logos/mohtawa-logo-white.png" alt="Mohtawa">
          <p class="site-footer__blurb">A digital consultancy for Gulf brands — content, design, and development advice from one accountable team.</p>
        </div>
        <div class="site-footer__contact">
          <a href="mailto:hussain@mohtawacm.com"><i data-icon="mail" data-size="17" data-color="var(--azure-300)"></i> hussain@mohtawacm.com</a>
          <a href="tel:+96566068182"><i data-icon="phone" data-size="17" data-color="var(--azure-300)"></i> +965 6606 8182</a>
          <span><i data-icon="pin" data-size="17" data-color="var(--azure-300)"></i> Kuwait City, Kuwait</span>
        </div>
        <div class="site-footer__bottom">
          <span>© 2026 Mohtawa Marketing</span>
          <span>www.mohtawa.co</span>
        </div>
      </div>
    </footer>
```

Note: `.site-footer > *:not(.site-footer__pattern)` (the base rule that lifts content above the pattern) now matches `.site-footer__inner`, which still works.

- [ ] **Step 3: Append booking + footer CSS**

Inside the `@media (min-width: 768px)` block, add:

```css
    /* booking — center band to the container */
    .booking { padding-left: var(--pad-x); padding-right: var(--pad-x); }
    .booking__card { max-width: 920px; margin-left: auto; margin-right: auto; }

    /* footer — center band; multi-column inner */
    .site-footer { padding-left: var(--pad-x); padding-right: var(--pad-x); }
    .site-footer__inner {
      display: grid; grid-template-columns: 1fr auto; gap: 24px 48px; align-items: start;
    }
    .site-footer__bottom { grid-column: 1 / -1; }
```

Inside the **desktop** `@media (min-width: 1024px)` block, add:

```css
    /* booking — two columns: when | who */
    .booking__grid { display: grid; grid-template-columns: 1fr 1fr; gap: 32px; align-items: start; }
    .booking__col--who .booking__divider { display: none; }
```

- [ ] **Step 4: Sync + reload**

```bash
cp "mohtawa-landing.html" public/mohtawa-landing.html
```
`preview_eval { serverId, expression: "location.href='http://localhost:4321/mohtawa-landing.html'" }`.

- [ ] **Step 5: Verify desktop layout (1280px)**

`preview_resize { serverId, width: 1280, height: 900 }`, then `preview_eval`:
```js
(() => ({
  bookingGridDisplay: getComputedStyle(document.querySelector('.booking__grid')).display,
  bookingCols: getComputedStyle(document.querySelector('.booking__grid')).gridTemplateColumns.split(' ').length,
  dividerHidden: getComputedStyle(document.querySelector('.booking__col--who .booking__divider')).display,
  footerInnerDisplay: getComputedStyle(document.querySelector('.site-footer__inner')).display,
  footerBottomSpan: getComputedStyle(document.querySelector('.site-footer__bottom')).gridColumnStart,
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
}))()
```
Expected: `bookingGridDisplay:"grid"`, `bookingCols:2`, `dividerHidden:"none"`, `footerInnerDisplay:"grid"`, `footerBottomSpan` is `"1"` (spans full width), `noHScroll:true`.

- [ ] **Step 6: Verify booking still functions (1280px)**

`preview_eval`:
```js
(() => {
  const fire = (el, t) => el.dispatchEvent(new Event(t, {bubbles:true}));
  document.querySelector('.date-chip').click();
  document.querySelectorAll('.time-slot')[4].click();
  const n = document.querySelector('input[name="name"]'); n.value='Test'; fire(n,'input');
  const e = document.querySelector('input[name="email"]'); e.value='t@a.com'; fire(e,'input');
  const enabled = !document.querySelector('#confirm-btn').disabled;
  document.querySelector('#confirm-btn').click();
  return { enabled, confirmed: !!document.querySelector('.booked'),
           bookedTitle: (document.querySelector('.booked__title')||{}).textContent };
})()
```
Expected: `enabled:true`, `confirmed:true`, `bookedTitle:"You're booked in."`. Then reload to reset.

- [ ] **Step 7: Verify phone unchanged (375px)**

`preview_eval { serverId, expression: "location.reload()" }`, then `preview_resize { serverId, width: 375, height: 812 }`, then `preview_eval`:
```js
(() => ({
  bookingGridDisplay: getComputedStyle(document.querySelector('.booking__grid')).display,
  dividerShown: getComputedStyle(document.querySelector('.booking__col--who .booking__divider')).display,
  footerInnerDisplay: getComputedStyle(document.querySelector('.site-footer__inner')).display,
  dateChips: document.querySelectorAll('.date-chip').length,
}))()
```
Expected: `bookingGridDisplay:"block"`, `dividerShown:"block"`, `footerInnerDisplay:"block"`, `dateChips:8`.

- [ ] **Step 8: Commit**

```bash
git add mohtawa-landing.html
git commit -m "Responsive booking (2-col desktop) and multi-column footer"
```

---

### Task 6: Cross-device verification sweep

Final guard task: confirm no console errors, no horizontal scroll, all assets load, and the phone layout is unchanged at the exact 375px baseline across the whole page.

**Files:**
- None modified (verification only). May re-sync `public/` if needed.

- [ ] **Step 1: Reload clean**

```bash
cp "mohtawa-landing.html" public/mohtawa-landing.html
```
`preview_eval { serverId, expression: "location.href='http://localhost:4321/mohtawa-landing.html'" }`.

- [ ] **Step 2: Console + assets + no-scroll at all three widths**

For each width — `preview_resize` to `{1280,900}`, `{820,1000}`, `{375,812}` — run `preview_eval`:
```js
(() => ({
  brokenImgs: [...document.images].filter(im => !im.complete || im.naturalWidth === 0).map(im => im.getAttribute('src')),
  noHScroll: document.documentElement.scrollWidth <= window.innerWidth,
  sections: document.querySelectorAll('section, footer').length,
}))()
```
Expected at every width: `brokenImgs:[]`, `noHScroll:true`, `sections:8`.

Then `preview_console_logs { serverId, level: "error" }` → expected: no error logs.

- [ ] **Step 3: Spot-check phone baseline (375px) against committed mobile design**

`preview_resize { serverId, width: 375, height: 812 }`, then `preview_eval`:
```js
(() => ({
  phoneMaxWidth: getComputedStyle(document.querySelector('.phone')).maxWidth,
  heroTitlePx: getComputedStyle(document.querySelector('.hero__title')).fontSize,
  navHidden: getComputedStyle(document.querySelector('.site-header__nav')).display,
  stickyExists: !!document.querySelector('.sticky-cta'),
  servicesStack: getComputedStyle(document.querySelector('.card-stack')).display,
  bookingStack: getComputedStyle(document.querySelector('.booking__grid')).display,
}))()
```
Expected: `phoneMaxWidth:"440px"`, `heroTitlePx:"38px"`, `navHidden:"none"`, `stickyExists:true`, `servicesStack:"flex"`, `bookingStack:"block"` — i.e. the phone design is intact.

- [ ] **Step 4: Optional visual confirmation**

`preview_screenshot { serverId }` at 1280px and at 375px to eyeball the desktop layout and the unchanged phone layout. (If the screenshot tool is unresponsive, the computed-style checks above are sufficient evidence.)

- [ ] **Step 5: Final commit (if any sync/cleanup)**

No code change expected. If the working tree is clean, skip. Otherwise:
```bash
git add mohtawa-landing.html
git commit -m "Verify responsive layout across phone/tablet/desktop"
```

---

## Self-Review

**1. Spec coverage:**
- Approach (one file, CSS-driven) → Tasks 1–5. ✓
- Three breakpoints / container widths (720/1120) → Task 1 (`--pad-x`, media edges). ✓
- Global frame & header (drop phone frame, nav bar, hide sticky CTA) → Task 1. ✓
- Hero two-column desktop / single-column tablet; logos 6-up → Task 2. ✓
- Services 3-up / auto-fit tablet; Why-us 2×2 + side heading → Task 3. ✓
- Testimonials 3-up grid; FAQ centered ~760px → Task 4. ✓
- Booking two-column (+ markup wrap, logic preserved); footer multi-column → Task 5. ✓
- Constraints: no mobile change (media-query-only), no deps, preserve logic, no horizontal scroll → enforced by construction + verified in every task's phone check and the Task 6 sweep. ✓
- Verification at 375/820/1280, console/assets/anchors/booking-flow → Tasks 1–6. ✓

**2. Placeholder scan:** No TBD/TODO; every CSS and markup change is shown in full; every verification step has a concrete expression and expected output. ✓

**3. Type/selector consistency:** New classes introduced in Task 5 (`.booking__grid`, `.booking__col--when`, `.booking__col--who`, `.site-footer__inner`, `.site-footer__brand`) are used consistently in that task's CSS and verification. `--pad-x` defined in Task 1 and consumed in Tasks 2/4/5. Desktop `@media (min-width: 1024px)` block is created in Task 2 and appended to in Tasks 3/4/5 (noted in each). ✓
