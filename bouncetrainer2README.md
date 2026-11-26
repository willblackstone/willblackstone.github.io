# Bounce Articulation Trainer 2

`bounce-trainer-2.html`: the same idea as the original (`bouncetrainerREADME.md`), still in one file. Play your note **the instant the ball hits the floor**. Everything in that README about the vibe and principles still holds. It's not a reaction game. Physics is real, the stage stays clear, launches never repeat, and every cue can be removed.

v2 adds a path for beginners and builds most of the original's "Ideas for later". Every addition was chosen to serve the original aim: hear the note before you play it, and let the attack land with the ball.

## What's new

### For someone picking it up for the first time
- **Welcome card** on the first visit. It explains the idea in three lines and starts on level 1. If the person arrived through a shared link, the link's settings are kept.
- **Levels in the transport bar.** ‹ and › (or the arrow keys) step through 14 graded levels, and each one adds one thing. Under the bar, each level gives a one-line description and something to **notice**. The notice lines observe, never instruct, and never point at the body or the technique.
- **Level 1 · First steps:** half speed, the same high arc every time, and a grey **landing mark** under the floor (a removable aid, under *On the stage*).
- **How it works** dialog (also opened with `?`).
- Session counter (notes played and time) and a **screen wake lock** while running, so an iPad on a stand doesn't dim.
- The stage shows the current level and its notice prompt while stopped, so full screen still makes sense.

### New ways to practise
- **Land and hold** (notes mode). The ball lands dead, rolls, and drops through a hole in the floor. Hold the note from landing until the ball disappears. The hole appears when the ball is loaded, so the *release* is telegraphed as well as the attack. A red line runs along the floor while the note sounds. `withHole()` takes a dead-ball flight and places the hole where the roll will have lasted a random hold within the *Hold from/To* range. It rejects holes off screen, holes too close to the landing, and holes the ball would reach after it has nearly stopped. (Measured: always finds a hole; holds land within the range plus about 0.19 s for the drop.)
- **On a pulse** (launch mode). A metronome runs, and each launch is timed *backwards* from the beat it should land on (`queuePulse`). The first landing falls on the last beat of every N beats, on the "and", or randomly either. Beat dots at the top right count towards the landing beat, which is ringed. The pulse keeps its real tempo in slow motion, and the flights stretch to fit. (Measured landing error against the beat grid: 0.00 ms.)
- **Hear it first.** A pitch is chosen for each launch (one note, or changing notes from a set such as *C D E F G* or *Open notes*). It sounds softly while the chute aims, then the flight is silent and you carry the note in your ear. The name shows on the ball and large at the top left. Names can be B♭ written or concert pitch.
- **Image words** (optional). A word or image such as *a bell* or *velvet* is shown before each launch, as an intention for the attack's character.
- **Dynamics.** Impact flashes scale with impact speed, so a bounce run visibly dies away.

### Listen (experimental)
- An AudioWorklet (`WK`, loaded from a Blob) detects note onsets sample-accurately from the microphone: a high-pass filter, a fast envelope against a slow noise floor, and a rise ratio over 25 ms. Each onset is matched to the nearest expected note within ±200 ms.
- Results appear in two places: "12 ms late" written on the ground under the landing, and a strip under the bar with one dot per note plus a plain summary ("typically 12 ms late, give or take 9 ms"). There's no score, by design.
- **Calibrate** plays 8 tocks through the speakers and measures how late the mic hears them. That single number covers both input and output latency on that device. It's stored in `bounceTrainer2Listen`.
- Offline test with synthetic brass-like notes: every onset was found, including a quiet one and fast tongued repeats, at a consistent +3 to +12 ms.
- The tock is muted while listening. With the pulse on, use headphones.

### Timing and audio internals
- All sounds are scheduled 250 ms ahead on the audio clock. `ctxAt()` uses `getOutputTimestamp()` to map screen time to the audio time that is *heard* at that moment, so tocks and the metronome line up with the flashes even on devices with output latency. Scheduled sounds are cut on Stop.
- A master volume control is under *Sounds*.
- **Sound delay** (*Sounds*, or `[` `]` while playing, with a readout on the stage): some setups play audio late, so every sound is scheduled that many ms early, and *Check the sync* turns on the tock and starts. Saved per device in `bounceTrainerSync`, shared with v1. Listen needs no change: expected note times already include the shift. (Measured: at +300 ms the tock is set to play 300 ms before the red flash appears; at 0 ms it's within about 6 ms.)

### Fixes over v1
- **Bounce runs** used to fly off the right edge after the first landing (measured: one note per ball). The level now uses steeper, softer launches (about 5 bounces on screen), and in *Every bounce* mode `makeLaunch()` rerolls launches that leave the screen before 3 bounces.
- Fullscreen falls back to the `webkit` API (older iPad Safari), and the button hides where fullscreen isn't possible (iPhone).
- The run stops if the tab is hidden, so audio and visuals never drift apart.
- On narrow screens the stage is 4:3, which gives taller arcs on a phone.

## Compatibility
- v1 settings and saved presets are copied over on the first visit (`bounceTrainer` → `bounceTrainer2`, `bounceTrainerPresets` → `bounceTrainer2Presets`).
- Link codes are now `s2-…`. The `ENC` table has new entries **appended**, and `s1-` codes from v1 still decode. v1 preset hash ids (`#start-here`, `#vanish`, `#nowhere`, …) still work. `#beginner` now opens level 1.

## Working on it
- Test hook: `window.__bt` has the v1 functions plus `withHole`, `start`, `stop`, `update(dt)`, `draw`, `applySettings`, `encode`/`decode`, `WK`, `_set(o)` and `_balls()`. Setting `window.__btFreeze=true` stops the animation loop advancing time, for screenshots.
- Worth rechecking after changes: every level runs for 40 s with no errors, pulse landings sit on the beat grid, holes are always found, landings stay on screen, and codes round-trip.

## Still for later
- Slurs (two linked balls), a duet with two chutes, video export.
- Release timing from the microphone (Listen only judges attacks for now).
