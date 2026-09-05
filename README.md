# BOT ARENA

MIT licensed — see `LICENSE`. Art and music are generated; `CREDITS.md` says
by what, and what that means if you want to reuse them.

A complete one-on-one arcade game in **one HTML file**. No build step, no
dependencies, no canvas, no framework. Open `index.html` and it runs.

    python3 -m http.server 8124    # then http://localhost:8124/index.html

A server is optional but preferred — some browsers restrict local image loads
over `file://`. Sound works either way; the contact sounds are inlined in the
page for exactly that reason.

The `.wav` masters are **not** in this repository. Music ships as Opus with an
AAC fallback, chosen at runtime. Images are sized to their 4K render and
losslessly optimised.

## What this is

| | |
|---|---|
| Rendering | plain DOM + CSS. Every sprite is an `<img>` present from load. |
| Animation | toggling visibility. **Never** swapping `src`. |
| Stage | fixed 1600 × 900, scaled to the viewport by one CSS transform. |
| Timing | frame-counted, not millisecond-based. |
| Input | keyboard, plus on-screen controls that appear on touch devices. |
| Modes | AUTONOMOUS (bot vs bot), 1P vs CPU at three difficulties, 2P shared keyboard, attract demo. |

The five responses are **AFFIRM** (high), **CLARIFY** (low),
**CAVEAT** (guard), **ESCALATE** (goes over the low one) and **DEFER**
(goes under the high one). Every one of them is a way of not disagreeing
with you.

| action | 1 player | 2P — P1 | 2P — P2 |
|---|---|---|---|
| MOVE | A D or ← → | A D | ← → |
| ESCALATE | W / ↑ | W | ↑ |
| DEFER | S / ↓ | S | ↓ |
| AFFIRM | J Z F | F | , |
| CLARIFY | K X G | G | . |
| CAVEAT | L C H | H | / |

**ESC** pauses · **M** mutes. The on-screen legend is generated from
`CONFIG.keys`, so it cannot drift from the actual bindings.

## What this is

| | |
|---|---|
| Rendering | plain DOM + CSS. Every sprite is an `<img>` present from load. |
| Animation | toggling visibility. **Never** swapping `src`. |
| Stage | fixed 1600 × 900, scaled to the viewport by one CSS transform. |
| Timing | frame-counted, not millisecond-based. |
| Input | keyboard, plus on-screen controls that appear on touch devices. |
| Modes | AUTONOMOUS (bot vs bot), 1P vs CPU at three difficulties, 2P shared keyboard, attract demo. |

Controls (1P): **A/D** or arrows to move · **W**/up jump · **S**/down DEFER
(duck) · **J**, **Z** or **F** AFFIRM · **K**, **X** or **G** CLARIFY ·
**L**, **C** or **H** CAVEAT. In 2P, player one takes A/W/S/D + **F**/**G**/**H**
and player two takes the arrows + **,**/**.**/**/** — no numpad, so it works on
a laptop. **M** mutes, **ESC** pauses.

The four responses are AFFIRM (high), CLARIFY (low), CAVEAT
(guard) and DEFER (duck) — every one of them a way of not disagreeing with you.

## Participant movement and spacing

Two behaviours here are genre conventions rather than arbitrary choices, and
both are easy to "simplify" into something that feels wrong.

**Walking into someone pushes them.** Participants carry a pushbox (`minGap`) and
cannot overlap. The overlap is *not* split evenly — it is apportioned by how
fast each participant is moving into the other, so walking into a standing opponent
displaces them completely while walking into each other stops both. An even
split shoves a standing participant back as hard as it stops the one who advanced,
which quietly makes advancing cost ground. Against the wall the remainder
rebounds onto the advancing participant and the pair stops: being cornered means having
nowhere left to give, and corner pressure is the whole point of the mechanic.

**The CPU moves both ways.** `CONFIG.ai.retreatChance` governs giving up ground
in the mid-range spacing zone, and `retreatAfterResponse` governs stepping out
once its own delivery ends. The second rises with difficulty while the first
falls: a weak opponent drifts backwards aimlessly, a strong one only gives
ground on purpose — after a delivery, when it is the one standing in range to be
punished. An AI that only ever walks toward you reads as being winched in
rather than choosing where to stand.

## Integrating it

**Everything tunable is in the `CONFIG` object** at the top of the `<script>`.
Participants, depletion, response box geometry, AI difficulty, key bindings, colours, art
paths, phrase tiers and timings all live there. There are no magic numbers
scattered through the logic; if you find one, it is a bug.

A few load-bearing points, because they are the ones that break silently when
someone changes them:

- **`CONFIG.stage.groundY` is published to CSS at boot.** The stylesheet and
  the physics read the same value, so they cannot drift. Change it in `CONFIG`,
  never in the CSS.
- **Response boxes are numeric, not derived from artwork.** `CONFIG.responses` defines
  reach and box geometry, mirrored by facing. Swapping art does not change
  where a response lands.
- **Sprite scale is uniform**, computed as `poseScale × spriteH / srcHeight`.
  Never size a pose with `height: 100%` — the poses have different source
  heights and the participant will appear to grow and shrink as it animates.
- **Sprites are anchored by MEDIAN x, not centroid.** A centroid chases an
  extended limb, so a participant drifts sideways on a clarify. `CONFIG.palettes.
  <name>.anchors` holds a measured value per pose; they are specific to one set
  of drawings and are **not** transferable to different art.
- **`CONFIG.art.version` is a cache-buster.** Bump it whenever you replace a
  bitmap in place — filenames do not change, so browsers otherwise keep serving
  what they cached the first time.

## Swapping the art

`CONFIG.palettes` is the seam. Add an entry with your own `participantDir`,
`srcHeight` and per-pose `anchors`, then point `active` at it. Each participant
needs seven poses — `idle affirm clarify caveat conceded escalate concluded` — as
transparent PNGs, and **all of them must face the same direction in the file**;
the game mirrors with `scaleX`.

Every coordinate and scale factor lives in `CONFIG` at the top of `index.html`.

## Swapping the sound

Music is on `<audio>` elements and impacts are on Web Audio. The split is not
arbitrary: collapsing it back to one system reintroduces an audible timing
defect.

## Making it yours

The written content is a running joke about corporate and AI-assistant
language: responses are called AFFIRM and CAVEAT, rounds are EVALs, a match is a
BENCHMARK, a rematch is REGENERATE, and the difficulty levels are named after
reasoning effort. **All of it is data.** The phrase tiers live in
`CONFIG.phrases`, the labels in `CONFIG.difficulty` and `CONFIG.round`.

If the theme does not suit the project you are dropping this into, retheming is
a text edit plus new burst art — the mechanics do not reference any of it. Note
that the comic bursts are **images**, so changing a phrase means regenerating
its PNG; a tier with no image falls back to styled text automatically, which is
a usable interim state.

Keep the tier colours if you keep the system: yellow, cyan, white, red and gold
are how a player reads which kind of move landed, and that is the one visual
language that must stay constant.
