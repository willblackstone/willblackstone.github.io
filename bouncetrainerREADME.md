# Bounce Articulation Trainer

A practice tool for trumpet players, and anyone who articulates: watch a ball launch, fly and land, and play your note **the instant it hits the floor**.

It grew out of an older exercise of watching a tennis rally and playing a note when one player hits the ball. That works, but at normal speed and broadcast resolution it's hard to see the exact moment of contact, and you're stuck with whatever camera angle you get. This trainer keeps the idea and removes the noise: a white floor, a black ball, real physics, and an unmistakable moment of impact.

## What it's for (the vibe)

This is **not a reaction game**. It's about connecting sound with sight.

The point is a long, honest telegraph. The player watches the arc, feels gravity pulling the ball down, and *hears the note before they play it*. Then the attack arrives exactly as the ball meets the floor. It's for teaching students to:

- hear (or picture) a tone before playing it
- feel the gravity of an entry, so an articulation lands rather than gets thrown
- commit to a moment they've predicted, rather than flinching at one they didn't see coming

Because of that, the design favours **high arcs and plenty of time**. Low, fast launches exist ("Flat and fast"), but they're labelled as a reaction drill for well-trained players, not the main course. When in doubt, give the player more time to see it coming.

Other principles that have held throughout:

- **Physics must be real.** Gravity, air drag, restitution and bounce friction are simulated at real-world scale (a 6 m wide floor, a football-sized ball). Timings must feel true, never floaty or frantic.
- **Visual clarity beats decoration.** White stage, black ball, one red accent used only for impacts. The floor shadow matters a lot, since it's a key cue for forecasting the landing.
- **Never play the same thing twice.** Randomised launches stop players memorising the exercise rather than reading the flight.
- **Cues you can remove.** The chute, the ball in flight and even the ball entirely (shadow only) can each be taken away as the player improves.
- **Settings out of the way.** The stage and Start button come first. Everything else sits in a collapsed "Settings and presets" panel.

## How it works

One self-contained file: `bounce-trainer.html`. No build step and no dependencies apart from a Google Font (Archivo). Open it in any browser, including an iPad on a music stand.

### Physics (SI units)

- Constants at the top of the script: `G`, ball radius `R`, quadratic drag `K`, horizontal bounce friction `FR`, wall restitution `EW`, the vertical speed below which the ball stops bouncing and rolls (`VSTOP`), and the timestep `DT` (0.5 ms).
- **Chute:** it's hinged low (`PIV`), its length is `CL`, and the ball starts `S0` from the hinge. The ball accelerates steadily up the tube and leaves the mouth at exactly the launch speed. You can see the launch speed before the ball is airborne, and that's deliberate.
- **`simulate(ang, spd, e, px0, free)`** pre-computes the whole flight when a ball launches. It returns sampled positions (`xs`, `ys`), floor impact events (`ev`: time, x, impact speed), the time spent in the chute (`tc`), when the ball came on screen (`tIn`), the apex and its x position, and the end time. Rendering just interpolates along this path by elapsed time, which keeps timing exact and lets note times be scheduled in advance.
- `free = true` is a quick coarse-step probe used by the solvers (no walls or screen bounds, stops at the first landing).
- **Power mapping:** level 1–10 → 3–8.5 m/s (`lvlSpeed`). **Bounciness mapping:** level 1–10 → restitution 0.35–0.89 (`lvlE`).

### Choosing launches

- `pickAngle()` weights random angles towards the top of the range (density rising with the square of the angle) when **Favour high arcs** is on, which is the default.
- With the chute visible, `makeLaunch()` rerolls launches whose peak would leave the top of the screen or whose first landing would fall off it.
- **Hidden chute** (`hiddenLaunch()`): the chute sits just off the left edge. For each launch it picks a landing point (2 m to the right edge) and a peak height (roughly 60–90% of the screen, leaning high). It then uses nested bisection (`solveV` for speed inside a search over angle) to find the launch that achieves both. The ball always comes in from the left while still rising, peaks high, lands anywhere across the floor, and is on screen for about 1.1–1.3 s. In this mode the angle and power sliders are ignored.

### Timing and display

- Simulation time runs on its own clock (`sim`), so ¾, ½ and ¼ speed slow everything consistently.
- **Impact cue:** the ball flashes red, with a red half-ring and a red floor strip. Only the notes the player should play flash. Once they've all happened the ball turns grey so later bounces don't distract.
- **Visibility modes:** *Always*, *Vanishes* (the ball disappears at a set percentage of each flight it lands from, measured from when it leaves the chute or comes on screen) and *Shadow only*. In every mode the ball is visible inside the chute and reappears in red at impact.
- **Launch modes:** *One at a time* (a pause after each ball finishes) or *Stream* (a fixed interval, with overlapping balls allowed). The chute never swings while a ball is inside it.

### Sound (Web Audio, created on the first Start press)

- **Launch:** a soft low thump at release, then a band-passed noise rush that builds up the chute and peaks as the ball leaves the mouth. Stronger launches make it shorter and brighter. On by default.
- **Tock:** an optional woodblock-style sound on each note, for checking sync. It's scheduled ahead with a 150 ms lookahead, so it's sample-accurate rather than tied to frame timing.

### Settings, presets and sharing

- All settings live in one object `S` with defaults in `DEF`, saved to `localStorage` (`bounceTrainer`). Wrap every storage call in try/catch, because the page may run inside a sandboxed frame.
- **Graded presets** are in `BUILTIN`, from "1 · Start here" (¾ speed, varied high arcs) to "9 · Advanced stream". A preset overrides training settings but leaves personal display options alone (`DISPLAY`: tock, trail, launch values, whoosh).
- **User presets** are saved by name to `localStorage` (`bounceTrainerPresets`).
- **Link sharing:** the URL hash carries a preset id (`#start-here`, `#vanish`, `#nowhere`, …) or a compact settings code (`#s1-…`, letters, digits and hyphens only, defined by the `ENC` table). Decoding accepts older, shorter codes, so links already shared keep working. When you add a setting, **append it to the end of `ENC`** and loosen the length check in `decode()`.

### Keys

Space starts and stops, F goes full screen, and clicking the stage also starts and stops. Space is ignored while typing in a text field.

## Working on it

- There's a test hook on `window.__bt` (`simulate`, `makeLaunch`, `hiddenLaunch`, `pickAngle`, `state`). This is handy for headless checks, for example sampling a few hundred launches and checking the spread of landing points, time on screen and peak height.
- Things that have been checked and are worth rechecking after physics changes:
  - the gap between bounces shrinks by the restitution factor
  - random launches stay on screen
  - hidden-chute landings spread across the floor with at least the minimum time on screen
  - there are no console errors in stream mode
- Keep it one file unless there's a strong reason not to. It's meant to be easy to share, open anywhere and drop into lessons.

## Ideas for later

- **Note length and releases:** after its last bounce the ball rolls into a hole, and the player holds the note until it disappears.
- **Articulation types:** a hard ball for a tongued attack, a soft one for a "du"-style legato, and two linked balls for a slur.
- **Dynamics:** the size of the impact flash follows impact speed.
- **Pitch cues:** the ball's colour or label sets the note, so the player changes notes while keeping the articulation clean.
- **Beat mode:** landings line up with a metronome beat, or deliberately just off it.
- **Two chutes:** for duets or two-part rhythms.
- **Video export:** clips for students without a device.
- **Mic scoring:** detect each note's onset and show how early or late it was. This probably belongs in a PyQt6 companion, where audio timing is more reliable than in a browser.
