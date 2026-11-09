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

The copy started out painting the reader's life *after* Make A Move, never explaining how the product works. As of 2026-11-04 the user has deliberately brought some mechanics back (the 70% threshold card, "Game Ends. Date Begins.", "Safety in the Spotlight", "A New Kind of Matchmaking"), so follow `prompts_2.md` over the old rule when they conflict. `copy_value.md` holds the rules and the retired-language table.

## Links into the app

The app is a separate site (the parent repo's `main`): https://nomadonatrip.github.io/Makeamove_UI_Mockup/. Every link into it must be **absolute**, for example `https://nomadonatrip.github.io/Makeamove_UI_Mockup/index.html#S-A2`:

| Target | Screen |
|---|---|
| `#S-A2` | sign-up (quiz intro) |
| `#S-A7` | sign in |
| `#S-I1` | report a member |

Relative `index.html#…` resolves to this page itself, and `../index.html#…` breaks on Pages.

The page's sign-up buttons ("Find your match", the label used for every CTA) don't go to the app. They jump to the **waitlist form** (`#waitlist`) in the finale. The form is backed by a Google Form ("MakeaMove Sign-up"): `submitWaitlist(data)` POSTs to its `formResponse` endpoint as `no-cors`, mapping name/email/gender to the form's `entry.*` IDs (`WL_ENTRY`) and `woman`/`man` to its `Female`/`Male` choices (`WL_GENDER`). The opaque reply means it rejects only on network failure, and a mismatched entry ID or choice fails silently on Google's side, so re-check those constants whenever the Google Form is edited.

## Previewing

Serve **this folder as the site root**, as Pages does, so relative paths behave like production:

```bash
python3 -m http.server 8000   # from landingpage/
```

After editing, load with a cache-busting query (`?v=2`), or the browser may show the old stylesheet.

## Page architecture

Scroll-driven scenes. A `.pin` section is tall and its `.stick` child is `position: sticky`; one scroll engine (`update()`) feeds each scene a 0–1 progress value.

- **How it works** (`#pitch`, `.pitch`): the Page 4 copy from `prompts.md`: the marquee logo as its `h2`, then a centred lead paragraph and a supporting line. Its supporting line is set at the lead's size. It follows the swipe scene directly ("Life can be tough…" / `#hardq` was removed on 2026-11-04).
- **How we find your match** (`#gamerules`, `.deck` + `.cards`): three numbered cards (revealing questions, gamified compatibility, the 70% threshold) and the "Every player is ID-verified." badge. It follows `#pitch`.
- **Game Ends. Date Begins.** (`#gameover`, `.deck`) and **Safety in the Spotlight / A New Kind of Matchmaking** (`#rules`, `.back`): restored from commit `408e3f9`, in that order, after `#gamerules`.
- **Type walls** (`.pin.wall`; none on the page right now, but the engine stays for reuse):
  - Each wall runs its own `TypeWall(sec)`: words light up as you scroll, lines slide in from alternating sides (`data-dir="±1"`), and `<em>` words glow pink.
  - A new wall needs only markup: `.pin.wall > .stick > .wline`.
  - `.wline.lead` is the smaller setup line, and `.wnote` is quiet supporting text.
- **The swipe scene and studio pull-back** (`#swipe`, `rant()`):
  - The swipe content lives in a "camera" layer (`.cam`). As you scroll, the last card is rejected at centre stage, then the camera scales down onto the studio's big screen (`.st-slot`), revealing a CSS-drawn studio: two podiums facing each other, since the show is one-on-one.
  - The beat timings are listed in the comment above `rant()`. `--S` and `--slot-top` on `.rant .stick` set the screen's size and position for each breakpoint.
  - After the twist line (there is no "Married" stamp any more), a **question-mark beat** plays: the timeline holds at .61 for `BEAT` (1.1) screens of scroll (`rantProg()`, which is why the section is 550vh, or 510vh at ≤900px). Dayo never tears. A big "?" slams in above his head, his card becomes a heartbroken woman (`.scard.lady`: `heartbreak.jpg`, a 720px head-to-knees JPEG crop of the user's "Tears Beneath the Autumn Trees.png"), and *her* photo tears in two (`tear()`) while `#rNext` ("Avoid heartbreaks, dump the dating debacle…") lights up under the card, in the twist line's place. As Dayo turns into her, the show's spotlight and pink podium ring fade out (`--stage` on `.sstack`) and a cold, foggy backdrop fades in (`.gloom`). The beat ends there: the halves falling and the line's words dropping away run on the pull-back's clock (.61–.68), so the sign's bulbs come on while they're still falling, with no empty moment. Swipe-card selectors must exclude `.lady`.
  - There is no switch-off or blackout any more: as the camera pulls back, the big screen's sign arrives like **headlights in the mist** (the user's brief): the cold fog lingers and a veil of mist (`.tvmist`) drifts in front of the sign; the white bulbs (`--lights`) and the white words "MAKE A MOVE" (`--tx`, three spans with gaps) fade in slowly, blurred and haloed, while the torn photo falls; then the gaps close into the MakeAMove logotype (`--join`) as the mist clears (`--mist`), the box and backdrop come in (`--bg`), "Meet the one meant for you." rises (`--line`), and the colour arrives last (`--hue`, via `color-mix`); all set on `#onTv` by `rant()`, and the house goes gold at .84. `#pitch` follows.
  - The rhyme's words fade in from fully invisible as you scroll (`light()`), so nothing is readable before it arrives, and the three swipes finish before line one is fully lit, so the copy always shows progress. The twist line ("…they're married!") never shows in the rhyme: it stays screen-reader-only in the `h2` and airs only as `#rCap` under the card, lighting up as the card is stamped.
- **It's a match** (`#verdict`, `verdict()`): LOCKED → MOVE → the match. At ≤900px the section is 200vh (260vh on desktop) and its clock starts while its top is still 0.35 of a screen below the fold (`prog(el, lead)`), so LOCKED arrives with the stage instead of after a screen of empty dark.
- **Site footer** (`footer.site-foot`, after `</main>`): the lit marquee sign (links to `#top`), link columns (The show, Players), and a Contact column whose email, phone, address, social handle and company line are **placeholders** (marked with an HTML comment) to replace before launch. A gold bulb run divides it from the finale.
- **Reduced motion is deliberately ignored** (the user's decision, because the page was much less readable with it on). Every visitor gets the full show, whatever their device setting. Don't add `prefers-reduced-motion` handling back without asking. Still check every new scene at ≤900px, where type walls switch to wrapped, flush-left lines.
- **Tokens:** use the `:root` tokens (`--move` pink, `--stay`, `--ink*`, `--stage-*`, `--display`/`--body`) and existing components (`.cta`, `.announce`, `.marquee`, `.cards`) rather than inventing new ones.

## Working conventions

- Use the `designing-user-experiences` skill for UI changes. Copy its audit into `.playwright-mcp/` (gitignored here).
- Text mostly sits on gradients, so check contrast against the lightest stop by hand.
- Verify at 390×844 and 1440×900 with screenshots.
- New landing *directions* get their own folder or repo; don't overwrite this page with a different direction.
