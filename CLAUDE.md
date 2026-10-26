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

## Copy voice

The copy paints the reader's life *after* Make A Move (the hero is "Meet the one who wants you."). It never explains how the product works: no questions, scores, rounds, game mechanics or form factor. `copy_value.md` holds the rules and the retired-language table.

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

- **The life you want** (`#hardq`, `.future`): one photograph of the future (`family.jpg`, the user's own photo) beside the Page 3 copy ("Life can be tough, *but meeting your soulmate shouldn't be.*", the "Make a move" line and the byline). It replaced the type wall. On phones the photo stacks above the copy.
- **How it works** (`#pitch`, `.pitch`): the Page 4 copy from `prompts.md`: the marquee logo as its `h2`, then a centred lead paragraph and a supporting line. It sits where the old "Other apps let you skip the hard questions" wall was.
- **How we find your match** (`#gamerules`, `.deck` + `.cards`): two numbered cards (revealing questions, gamified compatibility; the 70% threshold card was dropped) and the "Every player is ID-verified." badge, restored from before commit `3b383b2`. It follows `#pitch`.
- **Type walls** (`.pin.wall`; none on the page right now, but the engine stays for reuse):
  - Each wall runs its own `TypeWall(sec)`: words light up as you scroll, lines slide in from alternating sides (`data-dir="±1"`), and `<em>` words glow pink.
  - A new wall needs only markup: `.pin.wall > .stick > .wline`.
  - `.wline.lead` is the smaller setup line, and `.wnote` is quiet supporting text.
- **The swipe scene and studio pull-back** (`#swipe`, `rant()`):
  - The swipe content lives in a "camera" layer (`.cam`). As you scroll, the last card is rejected at centre stage, then the camera scales down onto the studio's big screen (`.st-slot`), revealing a CSS-drawn studio: two podiums facing each other, since the show is one-on-one.
  - The beat timings are listed in the comment above `rant()`. `--S` and `--slot-top` on `.rant .stick` set the screen's size and position for each breakpoint.
  - The big screen airs the logo and the promise ("Meet someone who wants what you want."), and `#hardq` follows it with the family photo.
  - The rhyme's words fade in from fully invisible as you scroll (`light()`), so nothing is readable before it arrives, and the three swipes finish before line one is fully lit, so the copy always shows progress. The twist line ("…they're married!") never shows in the rhyme: it stays screen-reader-only in the `h2` and airs only as `#rCap` under the card, lighting up as the card is stamped and torn.
- **It's a match** (`#verdict`, `verdict()`): LOCKED → MOVE → the match. At ≤900px the section is 200vh (260vh on desktop) and its clock starts while its top is still 0.35 of a screen below the fold (`prog(el, lead)`), so LOCKED arrives with the stage instead of after a screen of empty dark.
- **Reduced motion is deliberately ignored** (the user's decision, because the page was much less readable with it on). Every visitor gets the full show, whatever their device setting. Don't add `prefers-reduced-motion` handling back without asking. Still check every new scene at ≤900px, where type walls switch to wrapped, flush-left lines.
- **Tokens:** use the `:root` tokens (`--move` pink, `--stay`, `--ink*`, `--stage-*`, `--display`/`--body`) and existing components (`.cta`, `.announce`, `.marquee`, `.cards`) rather than inventing new ones.

## Working conventions

- Use the `designing-user-experiences` skill for UI changes. Copy its audit into `.playwright-mcp/` (gitignored here).
- Text mostly sits on gradients, so check contrast against the lightest stop by hand.
- Verify at 390×844 and 1440×900 with screenshots.
- New landing *directions* get their own folder or repo; don't overwrite this page with a different direction.
