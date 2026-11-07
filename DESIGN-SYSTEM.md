# מערכת העיצוב — הזמנת שולחן

Design system for `table-booking-he-v3.html`: a dark, warm, RTL-first reservation flow
for a restaurant. Everything below is implemented in that single file; this document is
the contract for extending it.

**Personality:** calm, precise, quietly premium. Motion is feedback, never decoration.
**Platform:** responsive web (baseline: iOS 16 / Chrome 110 / Firefox 110 / Edge 110).
**Direction:** RTL canonical. Every spacing and alignment rule uses logical properties
(`inset-inline`, `margin-inline-start`, `padding-inline`) so an LTR locale is a `dir` swap,
not a rewrite.

---

## 1. Colour tokens

All colours are HSL triplets so they can be composed with alpha:
`hsl(var(--brand) / .18)`. Components never reference a raw colour.

| Token | Value | Role |
|---|---|---|
| `--bg` | `40 6% 7%` | Page background |
| `--s-1` | `40 5% 10%` | Default surface (cards, steps) |
| `--s-2` | `40 5% 13%` | Interactive surface (inputs, chips) |
| `--s-3` | `40 4% 17%` | Hover / pressed surface, tags |
| `--line` | `40 4% 20%` | Hairline borders |
| `--line-2` | `40 4% 28%` | Hover borders |
| `--ink` | `40 25% 94%` | Primary text (≈15.5:1 on `--bg`) |
| `--ink-2` | `40 8% 72%` | Secondary text (≈8.4:1) |
| `--ink-3` | `40 7% 58%` | Tertiary text / disabled (≈6.4:1) |
| `--brand` | `44 86% 57%` | Saffron: action + selection only |
| `--on-brand` | `40 20% 8%` | Text on brand (≈12.6:1) |
| `--ok` | `108 44% 68%` | Confirmed / open |
| `--bad` | `6 76% 70%` | Error / destructive / declined |

**Rules**

- Saffron means *act* or *selected*. Never a passive background, never decoration.
- Status is never colour-only: every state also changes text, weight, or an icon
  (`✓` in completed steps, strikethrough on unavailable times, "חסום" on blocked tables).
- Light mode is a token swap, not a component change: `--bg: 40 30% 97%`,
  `--s-1:#fff`, `--ink: 40 15% 12%`, `--brand: 38 92% 34%` (white ink), `--on-brand: 40 40% 98%`.
  Keeping the shipped product dark-only is a brand decision — the restaurant is an evening venue.

**Contrast:** body text ≥ 4.5:1, large display text ≥ 3:1, borders are decorative
(never the only carrier of meaning). `@media (prefers-contrast: more)` raises the two
muted inks and the border ramp.

---

## 2. Typography

| Token | Stack |
|---|---|
| `--f-sans` | Heebo → system-ui → -apple-system → Segoe UI |
| `--f-serif` | Frank Ruhl Libre → Georgia (headlines, times, step titles) |

| Role | Size | Weight | Line-height |
|---|---|---|---|
| Hero `h1` | `clamp(30px, 8.5vw, 42px)` | 900 serif | 1.08 |
| Section | 30px | 900 serif | 1.15 |
| Step title | 19px | 900 serif | 1.3 |
| Booking time | 24px | 900 serif, tabular | 1.0 |
| Body | 16px | 400 sans | 1.55 |
| Field label | 13px | 500 sans | 1.2 |
| Caption / meta | 13–14px | 400 sans | 1.4 |

Body text is 16px minimum — prevents iOS focus zoom. Numerals in times use
`font-variant-numeric: tabular-nums` so counts and clocks don't jitter as they change.
Maximum three weights in use per screen (400 / 600 / 900).

---

## 3. Spacing, radius, layout

4px base scale: `--sp-1:4 --sp-2:8 --sp-3:12 --sp-4:16 --sp-5:20 --sp-6:24 --sp-8:32`.
Radii: `--r-xs:8 --r-sm:10 --r-md:14 --r-lg:18 --r-pill:999`.

- Container: `max-width: 560px`, `padding-inline: 20px` — a comfortable 45–75 character
  measure for RTL Hebrew at every width.
- Single-column, always. No breakpoint invents a second content column; the table
  timeline is the only horizontally scrolling surface (it is a schedule, not a layout).
- Safe areas: `env(safe-area-inset-bottom)` pads the sticky action bar; the top bar is sticky
  with `backdrop-filter: saturate(180%) blur(18px)`.
- `--tap: 44px` is the floor for every interactive target; inputs are 48px, the primary
  button 52px, steppers 48px.

Breakpoints used (mobile-first, no overrides needed below 480px):

| Range | Behaviour |
|---|---|
| 320–480 | Reference width. 4-up time grid, 2-up area grid, 64px day cards |
| 480–768 | Content stays at 560px, centred; same component sizes |
| 768+ | Centred column; timeline gains horizontal room |

---

## 4. Motion

**Budget:** nothing above 280ms. Press feedback ≤ 110ms. One property family per
interaction. No motion on scroll, no parallax, no looping animation except the spinner.

| Token | Value | Used for |
|---|---|---|
| `--dur-1` | 110ms | Press / tap response |
| `--dur-2` | 170ms | Colour, border, opacity changes |
| `--dur-3` | 220ms | Reveals, chevrons, content fades |
| `--dur-4` | 280ms | Structural: accordion height, action bar |
| `--ease-out` | `cubic-bezier(.22,1,.36,1)` | Entrances, feedback (fast out, soft landing) |
| `--ease-in-out` | `cubic-bezier(.4,0,.2,1)` | Symmetric two-way changes |
| `--ease-emph` | `cubic-bezier(.32,.72,0,1)` | Height/position changes that should feel physical |

### Interaction specification

Each entry: **trigger → rules → feedback → loop/mode**.

**1. Step accordion** — tap a header.
`grid-template-rows: 0fr → 1fr` over 280ms `--ease-emph`; the body content fades and
rises 6px over 220ms with a 40ms delay. The chevron rotates 45°→225° in 220ms, the step
bubble flips to saffron and scales 1.04.
*Feedback:* `aria-expanded`, one section open at a time, locked steps at 45% opacity with
`aria-disabled="true"` and `inert` bodies (so keyboard focus can't enter a collapsed step).
*Modes:* idle → open → done (`✓` replaces the number, title value fades in at 14px muted).
*Keyboard:* only steps ≤ the furthest reached step respond; on keyboard advance, focus moves
to the new step header so screen readers hear where they are.

**2. Day card** — tap.
Selected: background and border cross-fade to saffron in 170ms, labels invert to
`--on-brand` at 72% for the weekday. Unselected days never animate back with a bounce —
just the same 170ms fade. Press: `scale(.96)` in 110ms. Disabled (closed) days sit at 32%
opacity with a "סגור" caption and are skipped by tab order (`disabled`).
The row snaps the selected card into view (`scroll-snap-type: x proximity`).

**3. Guest stepper** — tap − / +.
Count re-renders instantly; the number pops `1.16 → 1` over 200ms `--ease-emph` (Web Animations,
skipped under reduced motion). Buttons are 48px circles, disabled at the min/max, and the
output is `aria-live="polite"` so the new value is announced. Area availability and the time
list re-evaluate silently — invalid selections disappear rather than erroring.

**4. Time chip** — tap.
`aria-pressed` drives the style: saffron fill, dark bold numerals, 170ms. Press `scale(.95)`
in 110ms. Unavailable slots are removed from the list rather than shown struck through;
if nothing is available the step offers up to four alternatives (other area, next days)
as chips, because a dead end is a design failure.

**5. Text field** — focus, error.
Focus: border → saffron plus a 3px `hsl(var(--brand)/.18)` ring, 170ms. Invalid:
border and ring go `--bad`, the field shakes 240ms (iOS-style, 4→-3px, finite), gets
`aria-invalid="true"`, and focus moves to it while the message lands in `role="alert"`.
Typing clears the invalid state immediately — no error that outlives its cause.

**6. Primary button** — hover, press, loading, disabled.
Hover (mouse only, `@media (hover:hover)`): background to 90% brand + soft 22px shadow.
Press: `scale(.975)` in 110ms. Loading: the label fades out in 110ms and a spinner appears
*in place* (absolutely positioned) so the button never changes width; the button is
`disabled` and the double-submit window closes. Disabled: 55% opacity, no shadow, no press.

**7. Bottom action bar** — step 5 reached.
Slides up 280ms `--ease-emph` from `translateY(110%)`; slides out when the step closes.
Visibility flips only after the animation (`visibility 0s linear 280ms`), and the bar is
`inert` while hidden so its button can't be tabbed to off-screen.

**8. Success** — request accepted.
Form unmounts, panel fades/rises 280ms.The check mark draws: circle stroke 280ms then the
 tick 240ms with a 100ms delay (`stroke-dashoffset`). Focus moves to the panel heading
(`tabindex="-1"`) so the result is announced; a polite toast confirms too. The summary is
the receipt — name, date, time, area, party — not a generic "thanks".

**9. Segmented tabs** — back office.
A single thumb slides between four equal segments in 220ms with `--ease-emph`
(`transform: translateX(calc(var(--i) * -100%))`, which is correct in RTL), and the panel
fades/rises 220ms. `role="tablist"` + `aria-selected`.

**10. Booking card action** — destructive.
No native `confirm()`. First tap arms the button: label becomes "לחצו שוב לאישור", a toast
explains, and the armed state expires after 4s. Second tap commits. The pattern is
reversible, keyboard-friendly, and never blocks the page.

**11. Table timeline bar** — hover / press / select.
`brightness(1.08)` on hover, `scale(.97)` on press, a 2px ink outline when selected. The
red "now" line is `pointer-events: none` so it never steals a tap.

**12. Toast** — status.
Drops 8px + fades in 170ms, sits under the sticky bar, auto-dismisses at 3.2s,
`role="status" aria-live="polite"` so it is announced without interrupting.

### Reduced motion

`@media (prefers-reduced-motion: reduce)` collapses every duration and delay to ~0ms,
removes the step-body rise, the success panel entrance and the check draw, keeps opacity
changes instantaneous, and switches programmatic scrolling to `auto`. No information is
carried by motion alone, so nothing is lost.

---

## 5. Components

| Component | States | Notes |
|---|---|---|
| `.acc` step | `lock`, default, `open`, `done` | header is a real `<button>` inside `<h2>` |
| `.day` | default, `:active`, `[aria-pressed=true]`, `[disabled]` | `aria-label` = full date + availability |
| `.chip` (time, settings) | default, `[aria-pressed=true]`, `[disabled]` | min 44px |
| `.area` | default, `[aria-pressed=true]` | 2-up grid, stacks below 360px content width |
| `.stepper` | default, `[disabled]`, `:active` | 48px circle |
| `.btn` | primary, `.ghost`, `.bad`, `.sm`, `[disabled]`, `[data-busy]`, `[data-armed]` | busy state never shifts layout |
| `.field` | default, `:focus`, `[aria-invalid=true]`, `.shake` | label always visible (no placeholder-as-label) |
| `.rc` booking card | pending, confirmed | actions live in a wrapped flex row |
| `.seg` tabs | 4 segments, sliding thumb | `role="tablist"` |
| `.tag` | neutral, `.good`, `.bad` | text, not colour, carries meaning |
| `.toast` | off | single live region, reused |

**Usage rules**

- Do use a token: `background: hsl(var(--s-2))`. Don't paste a hex value into a component.
- Do express state through ARIA (`aria-pressed`, `aria-invalid`, `aria-expanded`) and let CSS
  read that state. Don't add a parallel `.on` class — it drifts from what assistive tech sees.
- Do keep one open step at a time. Don't animate two accordions, or the page jumps twice.
- Do validate on submit and clear on input. Don't validate on every keystroke.
- Don't animate `width`, `height`, `top` or `left` — transform and opacity only.
  (Accordion height via `grid-template-rows` is the one deliberate exception: it is a single
  small container, and the alternative — a hard height jump — is worse.)
- Don't add a decorative animation on scroll, hover-parallax, or a bounce on selection.

---

## 6. Accessibility checklist

- [x] `lang="he" dir="rtl"`; logical CSS properties throughout.
- [x] Skip link to the form.
- [x] Every control is a native `<button>`, `<input>`, `<select>` or `<summary>`.
- [x] Touch targets ≥ 44×44px with ≥ 8px separation.
- [x] Focus-visible ring: 2px saffron, 2px offset, never removed.
- [x] `aria-expanded` + `aria-controls` on steps; `inert` on collapsed bodies.
- [x] `aria-pressed` on all selection controls; `aria-selected` on tabs.
- [x] Errors: `aria-invalid`, `role="alert"`, programmatic focus to the offending field.
- [x] Live regions: guest count, slot message, toast (`polite`, never `assertive` spam).
- [x] Result announced by moving focus to the success heading.
- [x] Status never colour-only; disabled states also remove the element from tab order.
- [x] `prefers-reduced-motion` and `prefers-contrast: more` respected.
- [x] Sound is opt-out, persisted, and never the only confirmation.
- [x] 16px body text (no iOS zoom), 1.55 line height, ~34ch measure.

---

## 7. Performance

- **Composited properties only** for motion: `transform` and `opacity`. Press feedback is a
  transform on the element itself, not a layout change.
- **No animation library.** ~0 KB of JS for motion; only Web Animations for the guest-count pop
  and `stroke-dashoffset` for the check draw.
- **No `will-change`** — the animations are short and infrequent; a permanent layer would cost
  memory for no gain.
- **Event delegation** on the back-office container: one click + one change + one input listener,
  created once, instead of re-binding on every render.
- **`inert` over `pointer-events`** for collapsed content: correct for keyboard and AT, and it
  lets the browser skip hit-testing.
- **Fonts:** two families, `display=swap`, `preconnect` to both Google hosts; system fallbacks
  are metrically close enough that the swap doesn't reflow the hero.
- **First paint is data-free:** settings/bookings render from `localStorage` synchronously, then
  the optional shared DB subscribes. No spinner on load, no layout shift.
- **Long lists:** the day strip is 21 nodes and time slots are ≤ 22 — below any
  virtualization threshold. The timeline is O(tables × bookings) with no DOM churn on scroll.
- **Sound** is a lazily created `AudioContext`, only after a user gesture.

---

## 8. CSS architecture for growth

Single file today, structured so it can be split without rewriting:

```
index.css
├── tokens.css        :root custom properties (this doc, §1–4)
├── base.css          reset, focus-visible, .sr, .skip, typography defaults
├── layout.css        .wrap, .top, .bar, page padding, safe areas
├── components/
│   ├── step.css      .acc .ah .ab .chev
│   ├── day.css       .days .day
│   ├── chip.css      .chip .slots .msg
│   ├── stepper.css   .guests .stepper
│   ├── field.css     .field input textarea select .err
│   ├── button.css    .btn .ic
│   ├── card.css      .rc .tag .dl
│   ├── tabs.css      .seg .tab
│   └── timeline.css  .tlw .tl-row .bar2
├── motion.css        keyframes + prefers-reduced-motion overrides
└── backoffice.css    #back scoped overrides
```

**Naming:** component-first, BEM-shaped for structural children (`.acc`, `.acc .ah`,
`.tl-r .tl-n`), single-class utilities only for layout primitives (`.row`, `.chips`, `.hide`).
No `!important` outside `.hide` and the reduced-motion block.

**Rules of engagement for the team**

1. A new colour needs a token, a role, and a contrast check — not an inline value.
2. A new duration needs to fit the four-step scale; if it doesn't, the interaction is wrong.
3. State lives in ARIA attributes; CSS reads state, never JS-added style classes.
4. Every component ships its `:focus-visible`, `:active`, `[disabled]` and
   reduced-motion behaviour on day one.
5. Back-office overrides stay inside `#back` so the customer flow can never be styled by
   accident.

**Migration path** is mechanical: extract `:root` to `tokens.css`, then move each component
block verbatim (the selectors are already scoped), then replace the `<style>` tag with
`@import`s or a bundle step.

---

## 9. Do / Don't

| Do | Don't |
|---|---|
| 110ms press, 170ms state, 220–280ms structure | 400ms+ hovers or easing `ease` everywhere |
| Motion confirms a change the user caused | Motion introduces content on scroll |
| Saffron for the one next action | Saffron as a decorative wash |
| Announce results in a live region | Rely on the visual check mark alone |
| Collapse errors as soon as the user types | Keep a red field after it's valid |
| Remove unavailable times | Show 20 disabled chips as a wall of dead ends |
| Inline two-step confirm for destructive acts | `window.confirm()` |
