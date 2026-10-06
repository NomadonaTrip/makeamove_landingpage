# CLAUDE.md — Make A Move landing page

**This is its own repository** (`NomadonaTrip/makeamove_landingpage`), nested inside the app prototype's folder. The parent folder's CLAUDE.md also loads, but it describes the *app* (`../index.html`). For work in this repo, this file takes precedence.

## What this is

The marketing landing page for Make A Move, an online matchmaking game show. Its job is to get the value proposition across and get visitors to sign up. It's a single self-contained `index.html` (inline CSS and vanilla JS, Google Fonts only), with no build step and no tests.

- `index.html`: the landing page. Edit it freely here; it is *not* the app.
- `prompts.md`: source copy for the page (gitignored).
- `guideline.md`: design values (clarity, simplicity, usability).
- `page_ref.JPG`: visual reference.

## Deploys

GitHub Pages serves **branch `feat/landing-page-copy`** at https://nomadonatrip.github.io/makeamove_landingpage/. **Pushing that branch updates the live site**, so never push without explicit approval.

## Links into the app

The app is a separate site (the parent repo's `main`): https://nomadonatrip.github.io/Makeamove_UI_Mockup/. Every link into it must be **absolute**, for example `https://nomadonatrip.github.io/Makeamove_UI_Mockup/index.html#S-A2`:

| Target | Screen |
|---|---|
| `#S-A2` | sign-up (quiz intro) |
| `#S-A7` | sign in |
| `#S-I1` | report a member |

Relative `index.html#…` resolves to this page itself, and `../index.html#…` breaks on Pages.

The page's sign-up buttons ("Find your match", the label used for every CTA) don't go to the app. They jump to the **waitlist form** (`#waitlist`) in the finale. The form has no backend yet: `submitWaitlist(data)` in the script is a placeholder that pretends to succeed, and the real request replaces it. It must resolve on success and reject on failure, because the failure message is already wired up.

## Previewing

Serve **this folder as the site root**, as Pages does, so relative paths behave like production:

```bash
python3 -m http.server 8000   # from landingpage/
```

After editing, load with a cache-busting query (`?v=2`), or the browser may show the old stylesheet.

## Page architecture

Scroll-driven scenes. A `.pin` section is tall and its `.stick` child is `position: sticky`; one scroll engine (`update()`) feeds each scene a 0–1 progress value.

- **Type walls** (`.pin.wall`, e.g. `#hardq` and `#walkon`):
  - Each wall runs its own `TypeWall(sec)`: words light up as you scroll, lines slide in from alternating sides (`data-dir="±1"`), and `<em>` words glow pink.
  - A new wall needs only markup: `.pin.wall > .stick > .wline`.
  - `.wline.lead` is the smaller setup line, and `.wnote` is quiet supporting text.
- **The swipe scene and studio pull-back** (`#swipe`, `rant()`):
  - The swipe content lives in a "camera" layer (`.cam`). As you scroll, the last card is rejected at centre stage, then the camera scales down onto the studio's big screen (`.st-slot`), revealing a CSS-drawn studio: two podiums facing each other, since the show is one-on-one.
  - The beat timings are listed in the comment above `rant()`. `--S` and `--slot-top` on `.rant .stick` set the screen's size and position for each breakpoint.
  - The pitch headline airs on the big screen, so `#hardq` keeps its own `h2` for screen readers only.
- **Reduced motion is deliberately ignored** (the user's decision, because the page was much less readable with it on). Every visitor gets the full show, whatever their device setting. Don't add `prefers-reduced-motion` handling back without asking. Still check every new scene at ≤900px, where type walls switch to wrapped, flush-left lines.
- **Tokens:** use the `:root` tokens (`--move` pink, `--stay`, `--ink*`, `--stage-*`, `--display`/`--body`) and existing components (`.cta`, `.announce`, `.oc`) rather than inventing new ones.

## Working conventions

- Use the `designing-user-experiences` skill for UI changes. Copy its audit into `.playwright-mcp/` (gitignored here).
- Text mostly sits on gradients, so check contrast against the lightest stop by hand.
- Verify at 390×844 and 1440×900 with screenshots.
- New landing *directions* get their own folder or repo; don't overwrite this page with a different direction.
