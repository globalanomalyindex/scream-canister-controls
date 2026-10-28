# Scream Canister Controls: Developer Handoff

A button and control set based on the scream canisters from *Monsters, Inc.* and *Monsters University*. Each UI state borrows a real canister mechanic: the meter is a progress bar, overpressure is an error you can retry, lockout is a disabled state that explains itself, and so on.

- **Design source:** the live demo and design story at https://globalanomalyindex.github.io/scream-canister-controls/.
- **Reference CSS:** `reference/kit.css` is the shared kit the demo uses. Treat it as a working reference, not production code. It is generated, so rebuild the components cleanly from this document.
- **Motion data:** `reference/motion-tokens.css` (timing tokens) and `reference/curves.json` (the 12 measured curves).
- **Status:** design final. No production code exists yet.

## 1. Stack

- React + TypeScript on Vite.
- Plain CSS with custom properties. No CSS-in-JS and no animation library.
- Motion is CSS keyframes and transitions only. Animate `transform` and `opacity` only.
- Storybook with one story per state, matching the state matrix in section 4.

## 2. Design tokens

### Color

| Token | Value | Use |
| --- | --- | --- |
| `--yel` | `#E9AF1A` | Capsule body, focus ring on dark grounds |
| `--navy` | `#0D2952` | Bands, cog dial, focus ring on light grounds |
| `--fill` | `#E0342B` | Scream fill (the meter) |
| `--green` | `#72AF38` | Laugh fill, success badge |
| `--hold` | `#DE2821` | Destructive hold |
| `--pen` | `#E8496F` | Red pen notes. Errors only. |
| `--plate` | `#4A5480` | Steel meter plate |
| `--tile` | `#3B3566` | Glyph button rest (dark grounds) |
| `--tile-hover` | `#4A4488` | Glyph button hover |
| `--tile-off` | `#34303F` | Disabled tile |
| `--per` | `#9890E9` | Notation lines (diagrams only) |
| `--paper` | `#FCFBFB` | Lockout tag, slips |
| `--card-face` | `#E6E8F6` | Card mode face |

Laugh mode swaps `--fill` to `--green`. Nothing else changes.

### Type

| Role | Font | Notes |
| --- | --- | --- |
| Display, page titles | Big Shoulders 800 | Uppercase |
| Button labels, section labels, numbers | Barlow Semi Condensed 700 | Uppercase, tracked |
| Body | Barlow 400 to 600 | |
| Red pen notes | Gochi Hand | Errors only |

### Spacing and layout

- Scale: 4, 8, 12, 16, 24, 32, 48, 64.
- Sections are 64 apart, groups 48, gutters 24. Heading to description 8, description to content 24.
- Grid: 12 columns on a 1296 px content width (86 px columns, 24 px gutters).
- No hairline panels. Separate areas with value steps or space.

### Shape

- Every control is a perfect pill: `border-radius: 999px`. Icon buttons are circles.
- No outlines on shapes. Pieces are told apart by color value.
- Canister drawings and the loader set stay flat cut paper (illustrations, not controls).

### Focus

- `outline: 2.5px solid var(--focus-color); outline-offset: 3px;`
- Focus color: `--yel` on dark grounds, `--navy` on the light Paper ground, `#F4E28A` on Chalk.
- Every control shows it, in every browser. Focus never moves or resizes the control.

## 3. Components

### Capsule (primary)

One per screen. A yellow pill with a navy band at the left end, a steel meter plate with 7 cells marked − and +, and a navy cog dial at the right end.

- **Label:** every state word is stacked in one cell, and only one shows at a time. The button never changes width.
- **Fill:** 7 cells in 500 ms, one cell at a time. The cog turns 40° per cell.
- **Check valve:** presses during a fill are absorbed. The cog jams to show it.

### Glyph button (secondary)

Everything else. A violet pill with a small canister glyph and a label. The glyph's own sight glass fills, and its cap twists on seal. Plain and icon-only versions drop the glyph.

### Icon button

A circle with one icon. Same states as the glyph button, without a fill.

### Hold to vent (destructive)

A glyph button the user holds for 1 s. The red fills in 7 steps, then the action commits. Releasing early cancels and drains back.

### Hold canister (irreversible confirm)

A standing canister. Hold for about 1 s: 7 cells fill, then the cap turns three times to lock. Releasing early lets go in 125 ms.

### Segmented control

A pill track with pill tabs. Arrow keys move between tabs and never animate.

### Loader

- **Popup:** a cut-paper card over the dimmed page, with a canister line that clamps, fills, seals and drops a canister. One loop is 1.42 s.
- **Inline:** a 160 px version that sits in place of content.
- **Spinner:** a 24 px glyph inside a button. Loops in 1 s.

Lifecycle:

1. Show only after 400 ms. Faster requests never show it.
2. Enter in 167 ms. Stay at least 600 ms.
3. Exit in 125 ms from any pose.
4. Change the copy once at 8 s.
5. Pause when offscreen or in a hidden tab.
6. Put messages in one `role="status"` region outside the popup. Mark the busy region `aria-busy="true"`. Announce "Door roster loaded." when done.

### Title lettering

Page titles drop in one letter at a time. One letter lands late, and line 2 pops into yellow from right to left. Plays once when the title scrolls into view. The looping version enters, holds 2 s, then exits upward. It stops when it scrolls off screen and stays still under reduced motion.

## 4. State matrix

| State | Trigger | Capsule | Glyph button | Accessibility |
| --- | --- | --- | --- | --- |
| Idle | Default | Meter empty, cog at rest | Glass empty | |
| Hover | Fine pointer only | Pops up in 3 drawings (7.5%, 2.5%, then holds at 4% larger). Cell 1 blinks on and off (530 ms each). The cog ticks round in steps of 7.5°, one every 167 ms (8 s per turn) | Pops up in 3 drawings and holds at 6% larger. Cap lifts off its band | None. Touch and reduced motion skip it. |
| Pressed | `:active` | Squash in 2 drawings (83 ms), cog winds back 15° | Dips to 97% | |
| Focus | `:focus-visible` | Focus ring | Focus ring | Ring on every control |
| Filling | Press | 7 cells in 500 ms | Glass fills in 7 steps | `aria-busy="true"` |
| Sealed | Fill done | Cog twist-locks (320 ms), check in the dial | Cap twists, badge lands | Label changes to "Sealed" / "Topped up" |
| Overpressure | Error | Relief valve blows at cell 5, red pen circles the meter, label "Retry" | Label "Retry" | Error text names the cause |
| Venting | Undo | Meter drains in 300 ms, cog unwinds | n/a | |
| Disabled | Locked | Muted, lockout tag hangs below | Muted | Stays focusable. A press swings the tag, and the tag text says why. `aria-disabled="true"`, not `disabled`. |

Re-arming: after a seal, the next press starts a new canister. Overpressure retries on the same button. No control ever needs a reset button.

## 5. Motion

### The drawing grid

Motion moves in drawings, like hand animation, not smooth tweens. One drawing = 41.667 ms (24 fps). Each move is a few drawings held still, then cut to the next.

Implementation:

```css
.sheet {
  /* fallback first */
  --ez-press: steps(2, jump-start);
  --ed-press: 83.333ms;
}
@supports (animation-timing-function: linear(0, 1)) {
  .sheet { --ez-press: linear(0.661 0% 50%, 1 50% 100%); }
}
.btn { transition: transform var(--ed-press) var(--ez-press); }
```

Each `linear()` plateau is one drawing. The curve sets how far each drawing moves. Copy the full token set from `reference/motion-tokens.css`.

### Timing

| Move | Duration | Drawings |
| --- | --- | --- |
| Press | 83 ms | 2 |
| Release | 125 ms | 3 |
| Fill | 500 ms | 7 cells |
| Seal twist-lock | 320 ms | |
| Vent drain | 300 ms | |
| Hold to vent | 1 s | 7 steps |
| Lockout tag swing | 500 ms | |
| Loader enter / exit | 167 ms / 125 ms | 4 / 3 |
| Hover pop | 125 ms | 3 |
| Hover blink | 1.06 s loop | on / off |
| Hover cog turn | 8 s per turn | 48 steps of 167 ms |

Rules:

- Any one-off UI move finishes within 300 ms, except the few listed above.
- Exits are faster than entries.
- Keyboard and arrow-key moves never animate.
- Use only the named tokens. No `ease`, `ease-in` or other keyword easings.

### The 12 curves

Fitted to 24 moves tracked frame by frame in the film title cards and the *Monsters University* credits.

| Curve | cubic-bezier | Use |
| --- | --- | --- |
| press | `0.42, 0.67, 0.82, 1.0` | Squash into a press |
| release | `0.19, 0.0, 0.59, 0.34` | Spring back after a press |
| snap | `0.22, 1.05, 0.45, 1.0` | One-drawing pops and cuts |
| arrive | `0.55, 0.71, 0.93, 1.0` | Things rolling or sliding to a stop |
| rise | `0.33, 0.65, 0.69, 1.0` | The up half of a hop |
| drop | `0.15, 0.0, 0.56, 0.32` | Falls, drains, leaving |
| glide | `0.22, 0.0, 0.78, 1.0` | Swings. Also the reduced-motion fade. |
| run | `0.33, 0.33, 0.67, 0.67` | Belts and constant travel |
| overshoot | `0.3, 1.78, 0.34, 1.02` | Character-like arrivals |
| overshoot-soft | `0.3, 1.48, 0.34, 1.02` | Overshoot for controls |
| settle | `0.3, 0.96, 0.58, 1.0` | Back to rest after an overshoot |
| whip | `0.6, 0.05, 0.85, 0.65` | Title letters leaving the frame only |

### Reduced motion

Under `prefers-reduced-motion: reduce`:

- Nothing moves. Fill and seal colors still change, because they carry state.
- Hover shows cell 1 lit, with no blink and no cog turn.
- The popup cross-fades in 150 ms (opacity only, `--bz-glide`).
- Title lettering is a still title, and its Pause and Step controls hide.

## 6. Modes

| Mode | Meaning | Look |
| --- | --- | --- |
| Painted | Live: actions commit | Default |
| Chalk | Practice: results show, then the board wipes itself. Nothing is sent. | Chalk lines on a dark board |
| Card | Records: each seal leaves a card you can return to undo | Controls printed on cards |
| Scream / Laugh | Energy theme | Laugh swaps the fill to green |

## 7. Accessibility checklist

- [ ] Real `<button>` elements. No clickable divs.
- [ ] Focus ring visible on every control (section 2).
- [ ] Disabled controls use `aria-disabled="true"`, stay focusable, and explain why.
- [ ] Labels change by stacking words, so the accessible name is always whole words.
- [ ] Hold actions also work from the keyboard (hold Space or Enter).
- [ ] Loader messages go to one `role="status"` region. Looping titles never announce.
- [ ] The title loop has Pause and Step (WCAG 2.2.2) and stops offscreen.
- [ ] Text contrast is at least 4.5:1. Edges that are the only cue are at least 3:1.
- [ ] Everything works with reduced motion on.

## 8. Do and don't

- Do use red only for errors and destructive actions (and the Scream fill inside the meter).
- Do keep one capsule per screen.
- Don't add outlines, shadows, glows or gradients.
- Don't add reset buttons. Demos and controls re-arm themselves.
- Don't use Pixar or Disney logos, characters or wordmarks. This is a fan-inspired concept.

## 9. Open items

- Text contrast was measured before the meter fill changed from amber to red. Re-check the fill against the empty cells.
- Not tested: real Safari (only WebKit in Playwright), real devices, and the 24 fps steps on a 60 Hz screen.
- A few minor visual details are still open: the state diagram's pointer lines sit in a navy casing, the held canister's floor shadow brightens, Chalk's error state loses its pink, a band on the capsule foot stands slightly proud, and a press right after a fresh canister arrives can be silent for up to 292 ms.
