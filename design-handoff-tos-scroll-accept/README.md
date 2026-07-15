# Handoff: Scroll-to-Accept Terms of Service (Member Checkout)

## Overview
A scroll-to-accept Terms of Service flow for the Switchyards member checkout page. The agreement checkbox on the checkout page is **locked** until the member opens the Terms modal and scrolls through the full document. Reaching the bottom unlocks acceptance; accepting checks the box on the underlying page and enables the **Pay Now** button. The intent is reassuring and low-friction — legal protection without a "gotcha" feel.

## About the Design Files
The file in this bundle (`Checkout ToS Modal.dc.html`) is a **design reference created in HTML** — a working prototype showing the intended look and behavior. It is **not production code to copy directly**. The task is to **recreate this design in the target codebase's existing environment** (React, Vue, etc.) using its established components, styling patterns, and libraries. If no front-end environment exists yet, choose the most appropriate framework for the project and implement it there.

Note: the prototype is authored as a "Design Component" (custom `<x-dc>` runtime with a `renderVals()` logic class). Treat that structure as a spec, not an API — reimplement the state and markup natively.

## Fidelity
**High-fidelity (hifi).** Final colors, typography, spacing, copy, and interactions are all specified below and should be recreated faithfully using the codebase's existing libraries and patterns. Match the existing checkout page's visual language (serif headers, monospace labels, navy-on-cream).

## Screens / Views

### 1. Checkout page — agreement row (two states)
The existing Member Checkout page (two-column: order summary left, payment method right). This design only changes the **agreement row** that sits between the Google Pay row and the Pay Now button.

**Locked state (default, before reading):**
- Row has a subtle tinted background `rgba(30,42,94,.035)` and 1px border `rgba(30,42,94,.12)`, radius 10px, padding 16px.
- Checkbox: 20×20px, radius 5px, border `1.5px rgba(30,42,94,.28)`, fill `#e7e3d8` (greyed), contains a 🔒 lock glyph (color `#8a8578`). Cursor `not-allowed`; clicking does nothing.
- Label (IBM Plex Mono, 12.5px, color `#8a8578`): "Please read " + link "Switchyards Terms of Service" + " to continue". The link opens the modal.
- Sub-hint below (IBM Plex Mono, 10.5px, `#8a8578`): "○ Tap the link above to open and read the terms."

**Enabled state (after accepting):**
- Row background/border fade to transparent.
- Checkbox border `#1e2a5e`; when checked, fill `#1e2a5e` with a `#f4f1e9` ✓ glyph. Cursor `pointer` (toggleable).
- Label (color `#1e2a5e`): the whole clickable phrase "I agree to the Switchyards Terms of Service" (clicking reopens the modal).
- Sub-hint: "✓ Thanks for reading — you're good to go."

**Pay Now button:** full width, radius 999px, padding 16px, IBM Plex Mono 14px, letter-spacing 2px. Disabled = background `rgba(30,42,94,.28)`, cursor `not-allowed`. Enabled (read **and** checked) = background `#1e2a5e`, text `#f4f1e9`, cursor pointer.

### 2. Terms of Service modal — "still scrolling" state
- **Desktop:** centered sheet, 560px wide, `max-height: 82vh`, radius 16px, background `#f8f6ef`, shadow `0 24px 70px rgba(20,26,52,.32)`. Backdrop `rgba(20,26,52,.42)` + `blur(2px)`; the checkout page behind is dimmed (`blur(1.5px) brightness(.92)`). Entry animation: fade + rise (`translateY(14px) scale(.985)` → none), 0.34s `cubic-bezier(.2,.8,.2,1)`. Clicking the backdrop closes it.
- **Mobile (< 720px):** full-screen takeover, radius 0, slides up from bottom (`translateY(100%)` → 0). Backdrop click does **not** dismiss.
- **Header (fixed):** monospace overline "MEMBERSHIP AGREEMENT" (11px, letter-spacing 1.5px, uppercase, `#8a8578`); serif H2 "Switchyards Terms of Service" (Spectral 500, 24px); subcopy (13px, opacity .7): "A quick read before you join — this protects both you and your club. Scroll through and you're all set."; round ✕ close button top-right (34px circle, white, border `rgba(30,42,94,.18)`). **Progress bar** under header: 3px track `rgba(30,42,94,.1)`, fill `#1e2a5e`, width = scroll %, transition `width .12s linear`.
- **Body (scrollable):** the full Terms text (see Content below). 14px / line-height 1.72, color `#3a3f57`. Section titles in Spectral 600 (`#1e2a5e`), two major group headers ("Site and Services Contents", "General") at 18px with a top border. One highlighted callout block (all-caps "YOU MAY NOT USE THIS SITE…") with left accent bar `#1e2a5e` and tinted bg.
- **Scroll affordance:** a bottom fade (96px, gradient from `#f8f6ef` to transparent) over the body, containing a centered, gently bouncing cue "Scroll to continue ↓" (IBM Plex Mono 11px, `#1e2a5e`, keyframe nudges Y + opacity, 1.6s infinite). Shown whenever the body is **not** scrolled to the bottom; it resets and reappears each time the modal is reopened.
- **Footer (fixed):** full-width button, radius 999px, IBM Plex Mono 13.5px. In this state: background `#1e2a5e`, label "Keep Reading" — clicking scrolls the body down ~85% of its height (smooth).

### 3. Terms of Service modal — "scrolled to bottom" state
- Progress bar fills to 100%; fade + scroll cue disappear.
- Footer button turns green `#1e7a4f`, label "I accept", preceded by a ✓ badge that pops in (scale 0.4→1.12→1, 0.4s).
- Clicking "I accept": closes the modal, **checks the agreement box**, and (with reading done) enables Pay Now. Clicking ✕ or the backdrop (desktop) also closes but leaves the box for the user to tick manually.

## Interactions & Behavior
- **Open modal:** click the ToS link in the agreement row (either state). Resets scroll to top, clears `atBottom`/progress.
- **Scroll tracking:** on body scroll, compute `scrollPct = scrollTop / (scrollHeight - clientHeight)`; `atBottom = scrollTop + clientHeight >= scrollHeight - 28`.
- **First reach bottom:** set sticky `hasRead = true` (unlocks acceptance permanently for the session).
- **"Keep Reading":** smooth-scroll body by `clientHeight * 0.85`.
- **"I accept":** `modalOpen=false`, `agreed=true`.
- **Checkbox click:** only toggles `agreed` if `hasRead` is true; otherwise no-op (locked).
- **Close (✕ / desktop backdrop):** `modalOpen=false` only.
- **Pay Now enabled** iff `hasRead && agreed`.
- **Responsive:** `isMobile = window.innerWidth < 720`, updated on resize; switches modal between centered and full-screen takeover.
- Animations: overlay fade-in 0.25s; sheet in 0.34s `cubic-bezier(.2,.8,.2,1)`; row bg/border transitions .3s; checkbox .2s; progress bar `width .12s linear`; ✓ badge pop 0.4s.

## State Management
- `modalOpen: boolean` — modal visibility.
- `hasRead: boolean` — sticky; true once bottom reached. Gates checkbox and, with `agreed`, Pay Now.
- `agreed: boolean` — checkbox checked. Set true by "I accept"; also user-toggleable once `hasRead`.
- `atBottom: boolean` — live; drives the scroll cue/fade (reset on open).
- `scrollPct: string` (e.g. "42.0%") — progress bar width.
- `isMobile: boolean` — viewport < 720px.
No data fetching. Terms copy is static content.

## Design Tokens
**Colors**
- Navy (primary text / accents): `#1e2a5e`
- Cream page background: `#f4f1e9`
- Modal sheet background: `#f8f6ef`
- Card/input background: `#fdfcf8` / `#fff`
- Body text in modal: `#3a3f57`
- Muted / label grey: `#8a8578`
- Locked checkbox fill: `#e7e3d8`
- Success green (I accept): `#1e7a4f`
- Backdrop: `rgba(20,26,52,.42)`
- Hairlines/borders: `rgba(30,42,94,.12)` – `rgba(30,42,94,.28)`; dotted summary rules `rgba(30,42,94,.35)`
- Link default `#1e2a5e`, hover `#34459a`

**Typography**
- Serif (headers): "Spectral", weights 400/500/600 (Google Fonts)
- Monospace (labels, buttons, summary values): "IBM Plex Mono", 400/500
- Sans (body/form): "Instrument Sans", 400/500/600
- Scale: overline 11px; small labels 12.5px; body 13–14px; H2 (page/modal) 24–27px; H1 30px.

**Radius**
- Inputs/cards 7–10px; modal sheet 16px; buttons & checkbox row pill 999px; checkbox 5px.

**Spacing:** 8px-ish rhythm; card padding 16–18px; modal header 24×28px; body padding 26×28px.

**Shadows:** modal `0 24px 70px rgba(20,26,52,.32)`.

**Animation easing:** `cubic-bezier(.2,.8,.2,1)` for sheet/badge; linear for progress; ease for fades.

## Assets
No image assets. Logo is a simple navy circle with a serif "S". Card-brand marks and Google Pay are represented as simple placeholders in the prototype — swap for the codebase's real payment assets. Fonts load from Google Fonts (Spectral, IBM Plex Mono, Instrument Sans).

## Tweakable prop
- `agreeLabel` (string, default "I agree to the Switchyards Terms of Service") — the checked-state agreement label text.

## Files
- `Checkout ToS Modal.dc.html` — the full prototype (checkout page + modal + all four states and interactions).
