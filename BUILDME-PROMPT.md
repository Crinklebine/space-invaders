# Build "Space Invaders — Last Line"

You are a senior front-end game developer. Build a complete, single-file browser game that faithfully recreates the existing arcade game **Space Invaders — Last Line**. This is a fidelity task, not a loose homage. Reproduce the gameplay, sprites, palette, typography, screen layouts, timers, probabilities, touch controls, keyboard controls, storage behavior, and retro presentation described below as closely as possible.

## Non-negotiable constraints

- Deliver **one static `index.html`** with inline CSS and JavaScript. No build step, no framework, no bundler, no external JS/CSS libraries.
- The game must run offline by opening `index.html` directly.
- Use **Canvas 2D** for the game and **Web Audio API** for sound. Do not use sprite sheets, image files, or audio files for gameplay.
- Do not include analytics, telemetry, or third-party tracking scripts.
- Do not replace the art with generic Space Invaders sprites. Use the exact ASCII sprite patterns, colors, glow treatment, and font described here.
- Preserve the portrait **7:8** screen aspect ratio and the arcade-cabinet framing.
- Everything below is intentional. Do not “simplify” timers, scoring, hitboxes, or screen wording.

---

# 1. Core canvas and scaling

- Logical game resolution: **`W = 630`**, **`H = 720`**.
- Backing-store scale: **`RS = 2`**. The canvas element is **1260 × 1440**.
- After getting the 2D context: `ctx.scale(RS, RS)` and `ctx.imageSmoothingEnabled = false`.
- All gameplay coordinates below are in the 630 × 720 logical space.
- The visible screen is **7:8** (`aspect-ratio: 7 / 8`), so the logical 630 × 720 is exact.
- The canvas fills the screen container: `width:100%; height:100%; display:block; touch-action:none; cursor:crosshair`.
- The canvas is focusable and has `role="application"`.

## Page / cabinet appearance

- Page background: near-black `#03050a` with three large, soft radial glows:
  - blue-cyan at `50% 10%`, `rgba(22,67,89,.38)`
  - magenta at `12% 100%`, `rgba(84,22,71,.22)`
  - blue at `94% 78%`, `rgba(18,59,78,.2)`
- A fixed 54×54 px cyan grid overlay at `rgba(118,235,255,.025)`, masked by a radial ellipse so it fades at the edges.
- A second fixed layer of sparse tiny star dots at about 18% opacity.
- Cabinet:
  - `width: min(820px, 94vw, calc((100dvh - 156px) * 7 / 8))`
  - rounded 22 px, thin `#26364e` border, dark blue-black gradient, deep drop shadow, inner top highlight.
  - Small outer glow.
- Screen shell: dark padded frame with rounded corners and inset shadow.
- Screen: `position:relative; overflow:hidden; aspect-ratio:7/8; border-radius:10px; background:#02040a`.
- Overlay FX, all `pointer-events:none`, above the canvas:
  - **Scanlines:** 1 px dark horizontal line every 3 px, opacity `.7`, `mix-blend-mode:multiply`.
  - **Glare:** diagonal white gradient sweep at low opacity.
  - **Vignette:** inset dark shadow, heavy at the corners.
  - Thin inner cyan edge highlight.
- The cabinet also includes a deck below the screen:
  - Desktop hints: `← → Move`, `Space Fire`, `P Pause`, `M Sound`, styled as small keycaps.
  - Buttons: **Sound on/off** and **Pause/Resume**.
- Mobile/coarse-pointer media query:
  - Cabinet width `min(100vw - 16px, 760px, calc((100dvh - 158px) * 7 / 8))`.
  - Hide the hint row.
  - Show a touch deck with a two-column layout: **◀** and **▶** on the left, **FIRE** on the right.
  - Touch buttons are 48 px tall (40 px on short landscape screens), rounded, dark, cyan/mint accents.
- Support `prefers-reduced-motion`: reduce scanline opacity, star speed to 25%, and most shake/animation amplitudes.

---

# 2. Palette and typography

## Exact game colors

```js
const C = {
  ink: '#eaffff', muted: '#6f8da3', dim: '#405a6d',
  cyan: '#75f4ff', cyanSoft: '#2baac7',
  mint: '#8dffc4', green: '#46e69a', greenGlow: '#25ff9c',
  pink: '#ff67d8', pinkGlow: '#ff2bb8',
  amber: '#ffd166', amberGlow: '#ff9d25',
  red: '#ff5f73', redGlow: '#ff2348',
  white: '#ffffff', black: '#02040a'
};
```

CSS custom properties should match the same palette (`--ink`, `--cyan`, `--mint`, `--pink`, `--amber`, `--red`, `--muted`, `--line`, etc.).

## Pixel font

Implement a custom **5×7 arcade alphabet** rendered as filled square cells:

- Uppercase only. Unknown characters fall back to `?`.
- `pText(str, x, y, size, color, opts)`:
  - Each glyph cell is `size` × `size` logical pixels.
  - Horizontal gap between cells is `spacing` (default `1`) × `size`.
  - Text width = `len * 5 * size + (len - 1) * spacing * size`.
  - Supports `align: 'center' | 'right'`, `alpha`, `glow`, `glowColor`.
  - Draw all glyph rectangles into a `Path2D`, then fill once.
- Use this font for HUD values, title words, scores, timers, panel headings, and most retro labels.

### Exact glyph set

```text
A 01110 10001 10001 11111 10001 10001 10001
B 11110 10001 10001 11110 10001 10001 11110
C 01111 10000 10000 10000 10000 10000 01111
D 11110 10001 10001 10001 10001 10001 11110
E 11111 10000 10000 11110 10000 10000 11111
F 11111 10000 10000 11110 10000 10000 10000
G 01111 10000 10000 10111 10001 10001 01111
H 10001 10001 10001 11111 10001 10001 10001
I 11111 00100 00100 00100 00100 00100 11111
J 00111 00010 00010 00010 00010 10010 01100
K 10001 10010 10100 11000 10100 10010 10001
L 10000 10000 10000 10000 10000 10000 11111
M 10001 11011 10101 10101 10001 10001 10001
N 10001 11001 10101 10011 10001 10001 10001
O 01110 10001 10001 10001 10001 10001 01110
P 11110 10001 10001 11110 10000 10000 10000
Q 01110 10001 10001 10001 10101 10010 01101
R 11110 10001 10001 11110 10100 10010 10001
S 01111 10000 10000 01110 00001 00001 11110
T 11111 00100 00100 00100 00100 00100 00100
U 10001 10001 10001 10001 10001 10001 01110
V 10001 10001 10001 10001 10001 01010 00100
W 10001 10001 10001 10101 10101 11011 10001
X 10001 10001 01010 00100 01010 10001 10001
Y 10001 10001 01010 00100 00100 00100 00100
Z 11111 00001 00010 00100 01000 10000 11111
0 01110 10001 10011 10101 11001 10001 01110
1 00100 01100 00100 00100 00100 00100 01110
2 01110 10001 00001 00010 00100 01000 11111
3 11111 00010 00100 00010 00001 10001 01110
4 00010 00110 01010 10010 11111 00010 00010
5 11111 10000 11110 00001 00001 10001 01110
6 00110 01000 10000 11110 10001 10001 01110
7 11111 00001 00010 00100 01000 01000 01000
8 01110 10001 10001 01110 10001 10001 01110
9 01110 10001 10001 01111 00001 00010 01100
space 00000 00000 00000 00000 00000 00000 00000
- 00000 00000 00000 11111 00000 00000 00000
. 00000 00000 00000 00000 00000 00110 00110
: 00000 00110 00110 00000 00110 00110 00000
/ 00001 00010 00010 00100 01000 01000 10000
+ 00000 00100 00100 11111 00100 00100 00000
! 00100 00100 00100 00100 00100 00000 00100
? 01110 10001 00001 00010 00100 00000 00100
```

## Tracked monospace text

`tracked(str, x, y, opts)` uses the monospace stack:

```text
"SFMono-Regular", Consolas, "Liberation Mono", Menlo, ui-monospace, monospace
```

- Default weight `500`, default size `11`, default tracking `2`.
- `textBaseline = 'middle'`.
- Manually measure each character and draw with extra tracking; do not use CSS letter-spacing.
- Supports `align: 'center' | 'right'`, `alpha`, `color`, `weight`.
- Use this for small labels, button labels, hints, footer text, and eyebrows.

---

# 3. Sprite construction

Invader, ship, and UFO sprites are defined as ASCII `X`/`.` grids and baked once into offscreen canvases.

## `makeSprite(rows, color, glow, opts)`

- Default `cell = 3`, `pad = 9`.
- Sprite content size: `w = maxRowLength * cell`, `h = rows.length * cell`.
- Offscreen texture size: `tw = w + pad*2`, `th = h + pad*2`.
- Create the offscreen canvas at `tw*RS × th*RS`, scale by `RS`, translate by `pad`.
- Fill each `X` cell as a `cell × cell` rectangle.
- Give the fill `shadowColor = glow`, `shadowBlur = 7`.
- Add a highlight strip on every filled cell whose cell directly above is empty: `rgba(255,255,255,.38)`, height `max(1, cell * .34)`.
- `drawSprite(spr, x, y, scale, alpha)` draws the padded texture centered appropriately, with position rounded to the backing-store grid so sprites stay crisp.
- **Do not anti-alias or apply smoothing.**
- `ship` uses `pad = 10`, `cell = 3`.
- `ufo` uses `pad = 9`, `cell = 3`.

## Exact invader patterns

### Squid — 30 points — pink `#ff67d8` / glow `#ff2bb8`

Frame 1:

```text
..XXXX..
.XXXXXX.
XX.XX.XX
XXXXXXXX
.X.XX.X.
X.X..X.X
X.X..X.X
...XX...
```

Frame 2:

```text
..XXXX..
.XXXXXX.
XX.XX.XX
XXXXXXXX
.X.XX.X.
..X..X..
.X.XX.X.
X......X
```

### Crab — 20 points — cyan `#75f4ff` / glow `#00d8ff`

Frame 1:

```text
..X.....X..
...X...X...
..XXXXXXX..
.XX.XXX.XX.
XXXXXXXXXXX
X.XXXXXXX.X
X.X.....X.X
...XX.XX...
```

Frame 2:

```text
..X.....X..
X..X...X..X
X.XXXXXXX.X
XXX.XXX.XXX
XXXXXXXXXXX
.XXXXXXXXX.
..X.....X..
.X.......X.
```

### Octopus — 10 points — amber `#ffd166` / glow `#ff9d25`

Frame 1:

```text
....XXXX....
.XXXXXXXXXX.
XXXXXXXXXXXX
XXX..XX..XXX
XXXXXXXXXXXX
...XX..XX...
..XX.XX.XX..
XX........XX
```

Frame 2:

```text
....XXXX....
.XXXXXXXXXX.
XXXXXXXXXXXX
XXX..XX..XXX
XXXXXXXXXXXX
..XXX..XXX..
.XX..XX..XX.
..XX....XX..
```

### Player ship — mint `#8dffc4` / glow `#25ff9c`

```text
.......X.......
......XXX......
......XXX......
.....XXXXX.....
..XXXXXXXXXXX..
.XXXXXXXXXXXXX.
XXX.XXXXXXX.XXX
XX...XXXXX...XX
```

### Mystery UFO — red `#ff5f73` / glow `#ff2348`

```text
....XXXXXX....
..XXXXXXXXXX..
.XXXXXXXXXXXX.
XX.XX.XX.XX.XX
XXXXXXXXXXXXXX
..XXX....XXX..
...X......X...
```

## Enemy types

```js
const ENEMY_TYPES = [
  { key: 'squid',   color: C.pink,  glow: C.pinkGlow, points: 30 },
  { key: 'crab',    color: C.cyan,  glow: '#00d8ff',  points: 20 },
  { key: 'octopus', color: C.amber, glow: C.amberGlow, points: 10 }
];
```

The title screen shows the three enemy types at `30 PTS`, `20 PTS`, `10 PTS`, and the UFO at `MYSTERY`.

---

# 4. Bunkers

Bunkers are bitmap grids, not sprites.

- Grid: **24 columns × 17 rows**, cell size **3** → 72 × 51.
- Four bunkers across the screen.
- `gap = (630 - 72*4) / 5`.
- Bunker `i` position: `x = round(gap + i * (72 + gap))`, `y = 500`.
- Texture padding: `6`; texture size `84 × 63`.

## Shape algorithm

For each cell `(x,y)`:

- Start solid.
- If `y < 2` and (`x < 3` or `x >= 21`), remove.
- Else if `y < 4` and (`x < 2` or `x >= 22`), remove.
- If `y >= 9`:
  - `half = 3.2 + (y - 9) * .58`
  - Remove if `abs(x - 11.5) < half` (the bottom arch).
- Fill remaining cells with `1`.

## Bunker rendering

- Base fill `#35d68d`, shadow `#25ff9c`, blur `6`.
- Top edge highlight `rgba(190,255,218,.67)` for cells whose upper neighbor is empty.
- Sparse dark flecks `rgba(4,36,22,.35)` at cells where `(x*7 + y*11) % 13 === 0`.
- Re-render only when damaged.
- Collision checks the grid cell-by-cell, with clamped grid coordinates.
- Player bullet damage radius: **5.1**; enemy bomb damage radius: **5.8**.
- Damage removes cells within a noisy radius (`radius/cell * rand(.78,1.15)`) and creates small debris particles.
- When the formation reaches `y >= 460`, invaders crush bunker cells they overlap.

---

# 5. Game state and flow

## State object

```js
const game = {
  mode: 'menu', score: 0, hiScore: storedHiScore(),
  wave: 1, lives: 3, nextExtra: 5000, newHigh: false,
  time: 0, shake: 0, damageFlash: 0,
  dyingTimer: 0, dyingReason: 'destroyed', endReason: 'destroyed',
  waveClearTimer: 0, waveBonus: 0, banner: null
};
```

Modes: `menu`, `playing`, `dying`, `waveclear`, `gameover`, `paused`.

Overlays: `infoOpen` and `resetConfirm` (both only meaningful on the menu).

## Constants

```js
const GROUND_Y = 678;
const DANGER_Y = 574;
const UFO_DEFLECTOR_CHANCE = .1;
```

## Starting a run

`startNewGame()`:

- Init audio, clear held input, set cursor to `crosshair`.
- Reset score `0`, wave `1`, lives `3`, `nextExtra = 5000`.
- Clear `newHigh`, set `endReason = 'destroyed'`.
- Clear shake, damage flash, banner, dying timer.
- Build wave 1.
- Play start jingle.
- Announce: “Game started. Wave 1. Defend the sector.”

Title screen start triggers:

- `Enter`, `Space`, click/tap anywhere on the canvas except the info icon.
- Game-over “Play Again” row and `Enter`/`Space`.

## Wave construction

- **11 columns × 5 rows** = 55 invaders.
- `colGap = 45`, `rowGap = 39`, `baseW = 36`.
- Board width = `(11 - 1) * 45 + 36 = 486`.
- Formation starts centered: `x = round((630 - 486) / 2) = 72`, `y = 130`.
- Formation fields:
  - `dir = 1`, `frame = 0`, `stepTimer = 0`
  - `fireTimer = rand(.7, 1.25)`
  - `beat = 0`
  - `aliveCount = 55`
- Row types:
  - row `0`: squid (30 pts)
  - rows `1–2`: crab (20 pts)
  - rows `3–4`: octopus (10 pts)
- Invader position inside formation:
  - `offX = col * 45 + (36 - spriteWidth) / 2`
  - `offY = row * 39 + 8`
- Wave 1 banner: title `WAVE 01`, subtitle `DEFEND THE SECTOR`.
- Later waves: title `WAVE NN`, subtitle `SECTOR NN - HOLD THE LINE`.
- Banner duration: 2 seconds.

## Marching

- Bounds computed from all living invaders.
- Horizontal step: **6 px**.
- Screen edge margin: **18 px**.
- If the next step would cross the edge:
  - Drop formation by **21 px**.
  - Reverse `dir`.
  - Crush bunkers the formation now overlaps.
- Otherwise move horizontally by `dir * 6`.
- Toggle animation frame every march.
- March beat cycles `0,1,2,3` and plays the four-note march.
- If formation bottom exceeds `DANGER_Y = 574`, trigger invasion.

March interval:

```js
ratio = clamp(aliveCount / 55, .02, 1);
base = max(.34, .69 - (wave - 1) * .045);
interval = max(.085, base * pow(ratio, .62));
```

- As invaders are killed, they march faster.
- Higher waves start faster.

## Player

```js
{ x: 315, y: 634, w: 45, h: 24, speed: 245, cool: 0, invuln: .9, flash: 0, dead: false }
```

- Keyboard moves left/right with `ArrowLeft`/`ArrowRight` or `A`/`D`.
- On touch, dragging horizontally moves the ship.
- Clamp x to `[34, W - 34]`.
- Player fires by holding `Space`, the touch FIRE button, or dragging/firing with a pointer.
- **Only one player bullet may exist at a time.**
- Fire cooldown: **0.32 s**.
- Bullet: `w=4, h=14`, spawn at `x = player.x - 2`, `y = player.y - 7`, velocity `vy = -930`.
- Player bullet visual: additive white-cyan core `#dcffff`, cyan shadow blur 12, plus a 23 px cyan trail at 16% alpha behind it.
- Muzzle particles and flash effect.
- Invulnerability after respawn: **2.2 s**; initial spawn `.9 s`.
- During invulnerability, the ship blinks and shows a dashed mint ellipse shield.
- Player ship has an animated mint exhaust flame and a white engine glow.

## Enemy fire

- Only the bottom-most living invader in each column is eligible to shoot.
- Choose a random eligible column.
- Bullet: `w=4, h=13`, spawn at the chosen invader’s bottom center.
- Bomb velocity: `245 + wave * 11 + rand(0,45)`.
- Bomb cap: `min(7, 3 + floor(wave / 2) + (aliveCount < 12 ? 1 : 0))`.
- Fire interval:
  - `progress = 1 - aliveCount / 55`
  - `speedUp = clamp(1 - (wave - 1) * .07 - progress * .38, .4, 1)`
  - `fireTimer = rand(.45, 1.12) * speedUp`
- Bombs are drawn as a 5-segment red zigzag with a pink-white core, flipping direction every animation frame. Add a soft red 10×(bomb height + 8) glow behind it and a 12 px red shadow blur.

## Collisions

- Player bullet vs bunker: damage bunker, remove bullet.
- Player bullet vs enemy bomb: both removed, tiny cyan clash burst/ring, shake.
- Player bullet vs UFO: remove bullet; armored UFO deflects first hit, otherwise dies.
- Player bullet vs invader: remove bullet, destroy invader, award points.
- Enemy bomb vs bunker: damage bunker, remove bomb.
- Enemy bomb vs player:
  - Player hitbox: `x ± 18`, vertical `player.y + 4` to `player.y + player.h`.
  - Ignore if invulnerable.
  - Remove bomb, trigger `playerHit()`.
- Invader formation can crush bunkers when low.

## Death, lives, invasion

`playerHit()`:

- Return if not playing or invulnerable.
- `lives = max(0, lives - 1)`.
- Mode `dying`, timer **1.65 s**, reason `destroyed`.
- Damage flash `1`, shake `17`.
- Mark player dead, mint particle burst, green ring.
- Play player-hit sound.
- Clear enemy bombs and held input.
- Announce lives remaining or “Final ship destroyed.”

After the dying timer:

- If `lives <= 0`: `endGame('destroyed')`.
- Else respawn at center with 2.2 s invulnerability and a `READY` banner for 1.35 s.

Invasion:

- If the formation reaches the danger line, set lives to 0.
- Mode `dying`, timer **1.9 s**, reason `invaded`.
- Red burst, red ring, shake 24, clear bullets and bombs.
- Then `endGame('invaded')`.

## Wave clear

- When the last invader dies:
  - Mode `waveclear`.
  - Timer **2.6 s**.
  - Bonus = `wave * 100`, awarded immediately.
  - Clear enemy bombs, player bullets, and UFO.
  - Shake 2, wave-clear jingle.
  - Announce wave cleared and bonus.
- After timer, increment wave and build the next wave.
- The wave-clear panel:
  - 500 × 208 panel centered at y 260.
  - `SECTOR SECURED`, `WAVE CLEARED`, `BONUS +NNN`, `NEXT WAVE INCOMING`.

## Scoring and extra lives

- Squid 30, crab 20, octopus 10.
- Wave bonus `wave * 100`.
- UFO normal values: 100, 150, or 300.
- Armored UFO: 1000.
- Extra ship every **5000 points**, while lives < 5.
- `nextExtra` starts at 5000 and increases by 5000 per awarded extra ship.
- Show `EXTRA SHIP` floater above the player.
- If a huge score skips thresholds, advance `nextExtra` to the next multiple of 5000 above the score.
- HUD shows at most 5 life icons.

## High score — critical rule

- `game.hiScore` updates **in memory** during play so the HUD can show a new high and the amber “new high” state.
- The high score is persisted to `localStorage` **only when a run reaches game over** via `endGame()`.
- There is **no** `beforeunload` save. Reloading, closing the tab, pausing, or quitting to the title before game over must not save a new high.
- Quitting from pause: return to title, discard the run, and restore the last stored high score (`game.hiScore = storedHiScore()`).
- If the abandoned run had beaten the stored high (`game.newHigh === true`), the pause screen’s TITLE SCREEN row must show `[HIGH SCORE LOST]` in amber.
- The pause announcement must also warn that returning to title will lose the run’s new high score.
- Resetting the high score via system info intentionally writes `0`; that is the only non-game-over high-score write.
- Storage keys, derived from this project’s one-time `GAME_ID`:
  - High score: `si.${GAME_ID}.hi`
  - Sound: `si.${GAME_ID}.sound`
- Do not read or write legacy keys.

## UFO

- Spawn y: **97**.
- Start offscreen: `x = -72` or `W + 72`.
- Horizontal speed: `122 + wave * 8`.
- Normal points: random `[100,150,300]`.
- Armored chance: **10%**.
- Armored points: **1000**.
- Armored UFO:
  - First hit does not kill it; it raises a shield and fires cyan/white ricochets.
  - Second hit kills it.
- UFO hums every **0.19 s** with alternating 680/540 Hz tones.
- Initial spawn timer: `rand(6, 11)`.
- After leaving the screen: `rand(14, 24)`.
- After being destroyed: `rand(15, 24)`.
- Draw a small blinking four-light row under the saucer.
- UFO bobs vertically by `sin(time * 4.4) * 2` unless reduced motion.
- On death: amber + red bursts, two rings, shake 10, arpeggio + noise.

---

# 6. Effects and rendering

## Effects

- **Particles:** position, velocity, gravity, drag, life/max, size, color. Cap at 620; remove oldest beyond cap. Draw as glowing squares; larger particles get cross glints.
- **Rings:** position, color, from/to radius, life/max; expand with ease-out; fade.
- **Ricochets:** short glowing line segments with gravity, used by armored UFO deflection.
- **Floaters:** rising score labels (`+30`, `EXTRA SHIP`), using the pixel font.
- **Shake:** decays by 34 per second; draw the world translated by random `±shake` (reduced-motion multiplies by .2).
- **Damage flash:** red full-screen overlay `rgba(255,47,76, flash * .18)`, decays by 2.6 per second.
- **Banner:** fade in over .18 s, hold, fade out over .45 s; title and optional subtitle.

## Background

- Base fill `#02040a`.
- Sky gradient:
  - `#07121e` at top
  - `#030811` at 48%
  - `#02050b` at bottom
- Cyan nebula radial glow centered around `(0.28W, 260)`, radius 430.
- Magenta nebula centered around `(0.78W, 470)`, radius 430.
- Bottom horizon gradient into `rgba(0,39,51,.15)`.
- **155 stars**:
  - Random x/y, depth `z`, size 1 or 2 (2 when `z > .78`)
  - Speed `4 + z * 23`
  - Color: cyan when `z > .84`, pale blue `#c7dfef` when `z > .62`, muted `#6f889e` otherwise
  - Flicker via sine; bright stars get a 6 px cross glint.
- Faint vertical cyan guide lines from y 92 to 655, every 84 px starting at x 60, alpha .05.

## World draw order

1. Bunkers
2. Living invaders (current animation frame)
3. UFO
4. Enemy bombs
5. Player ship
6. Player bullets
7. Particles, ricochets, rings
8. Floating score text
9. Damage flash overlay

HUD and screens are drawn after the world.

## HUD

Top gradient from y 0 to 88, fading downward.

- Left: `SCORE` at `(34,19)` size 2; score value at `(34,40)` size 3.
- Center: `HI-SCORE` at `(W/2,19)` size 2; value at `(W/2,40)` size 3.
  - Value is amber and glows when `game.newHigh`.
- Right: `WAVE` at `(W-34,19)` size 2; two-digit wave at `(W-34,40)` size 3, cyan.
- Horizontal cyan/magenta hairline at y 78.
- Score and wave are zero-padded: score 6 digits, wave 2 digits.

Bottom gradient from y 668 to 720.

- Ground line at y 678.
- `LIVES` at `(34,695)` size 2.
- Up to 5 tiny ship icons at `x = 119 + i*29`, `y = 694`, scale `.55`.
- `CREDIT 01` at `(W-34,695)`, right-aligned, dim.

---

# 7. Title / start screen

Draw order: background, HUD, title content, info button.

- Large soft cyan/magenta halo behind the title.
- Eyebrow: `EARTH DEFENSE INITIATIVE` at `(W/2,118)`, tracked size 10, tracking 3, color cyanSoft. Decorative side lines extend 58 px out, with a 42 px gap around the text.
- **SPACE** at `(W/2,154)`, pixel size 12, white-to-pale-cyan vertical gradient, glow 24 cyan.
- **INVADERS** at `(W/2,260)`, pixel size 12, white-to-pink gradient, glow 24 pink.
- Decorative horizontal line at y 382, width 430.
- `THE CLASSIC BATTLE FOR EARTH` at `(W/2,402)`, tracked size 10, tracking 3.2, muted.
- Start panel: **410 × 64** at `x = W/2 - 205`, `y = 430`, cyan accent, alpha .92.
- `PRESS ENTER TO PLAY` at `(W/2,444)`, pixel size 2, glowing, pulsing alpha.
- `OR CLICK / TAP ANYWHERE` at `(W/2,474)`, tracked size 9, tracking 2, muted.
- Controls hint at `(W/2,521)`: `MOVE WITH ARROWS OR DRAG   /   FIRE WITH SPACE`, tracked size 9, tracking 1.5, dim.
- `TARGETS` at `(W/2,553)`, tracked size 8, tracking 3, dim.
- Enemy showcase at x `W/2 - 180`, `W/2 - 60`, `W/2 + 60`:
  - Animated invader sprites at y 568.
  - Point labels `30 PTS`, `20 PTS`, `10 PTS` at y 600, matching enemy colors, alpha .8.
- UFO at `W/2 + 180`, y 568, scale .9, alpha .8; `MYSTERY` at y 600, red, alpha .65.
- **Info icon:** top-right, hidden when info is open.
  - Hit rect: `x = W - 78`, `y = 73`, `w = 78`, `h = 78`.
  - Visible circle radius 15 at center `(591,112)`, dark fill `rgba(3,10,18,.78)`, cyan border `rgba(117,244,255,.52)`, shadow blur 10.
  - Lowercase `i` centered, tracked size 14, cyan.

Title interactions:

- Tap/click info icon → open system info.
- Tap/click anywhere else → start game.
- `Enter` or `Space` → start.
- Pointer cursor over the info icon; crosshair elsewhere.

---

# 8. System info screen

Only available from the title screen.

## Panel geometry

- `INFO_RECT = { x: 45, y: 160, w: 540, h: 450 }`.
- Panel uses the shared cut-corner panel style with mint accent.
- Dim the whole screen with `dim(.84)` first.
- Fill the panel interior with near-opaque `rgba(2,6,12,.92)` before drawing the panel.

## Responsive text scale

```js
scale = clamp(630 / canvas.getBoundingClientRect().width, 1, 1.75)
```

Use this scale for info text and for the info action rows. On phones, info text grows up to 1.75×.

## Content

- Header: `SPACE INVADERS // SYSTEM INFO` at `(x+30, y+36)`, size `10*scale`, tracking `2.2*scale`, mint.
- Header line at `y+56` from `x+30` to `x+w-30`.
- Six rows, starting at `y+88`, spacing 38:
  - `PAGE` → current page filename
  - `GAME ID` → `last-line-<the project’s one-time 6-hex hash>`
  - `BUILD` → 7-character build hash
  - `BUILT` → `YYYY-MM-DD HH:MM:SS UTC`
  - `RES` → `innerWidth × innerHeight @ DPRx`
  - `ARCADE BEST` → zero-padded high score
- Labels left at `x+30`, muted; values right at `x+w-30`.
- `GAME ID` and `BUILD` values cyan; `ARCADE BEST` amber if nonzero, muted if zero.
- Divider line at y 456.
- **Reset row:** `INFO_RESET_RECT = { x: 75, y: 466, w: 480, h: 78 }`.
  - Label `RESET HIGH SCORE` left, amber accent.
  - Hint `R` right.
  - Uses the shared action-row style with `textScale = scale`.
- Close hint at `y+h-26` (y 584): `TAP ELSEWHERE OR PRESS I TO CLOSE`, centered, size `9*scale`, muted. Hidden while reset confirmation is open.

## Reset confirmation

- Dim the info panel with `rgba(2,6,12,.84)` over `INFO_RECT`, then draw:
  - `RESET_CONFIRM_RECT = { x: 65, y: 185, w: 500, h: 360 }`, amber accent.
- Title `RESET HIGH SCORE?` at y 225, size `13*scale`, amber.
- Body `SET ARCADE BEST TO ZERO?` at y 263, size `10*scale`, ink.
- `CURRENT BEST ######` at y 291, size `10*scale`, muted.
- `THIS CANNOT BE UNDONE` at y 317, size `8.5*scale`, red.
- **YES row:** `RESET_YES_RECT = { x: 95, y: 336, w: 440, h: 78 }`.
  - Label `YES, RESET`, hint `Y`, red accent.
- **NO row:** `RESET_NO_RECT = { x: 95, y: 422, w: 440, h: 78 }`.
  - Label `NO, KEEP SCORE`, hint `N / ESC`, mint accent.
- Both rows use the info text scale.

## Info / reset behavior

**Opening:** Tap the title info icon or press `I` on the title screen. This must not start the game.

**Info open, no confirmation:**

- Tap/click the reset row → open reset confirmation.
- Tap/click anywhere else → close info.
- Pointer over reset row: pointer cursor. Otherwise: default cursor.
- Keyboard `R` → open confirmation.
- `Escape` / `Space` / `Enter` → close info.
- `I` → toggle info closed.

**Confirmation open:**

- Tap/click YES → set high score to 0 immediately, write `0` to storage, clear `resetConfirm`, keep info open, update `ARCADE BEST` to `000000`, play UI sound, announce “High score reset to zero.”
- Tap/click NO → cancel confirmation, keep info open, announce “High score reset cancelled.”
- Tap/click anywhere else → cancel confirmation, keep info open. Do not close info.
- Keyboard `Y` → reset.
- `N` / `Escape` / `Space` / `Enter` → cancel.
- Pointer over YES/NO → pointer cursor. Otherwise default.

The info icon is hidden while info is open. Tapping the info panel must not fall through to the title’s “start game” action.

---

# 9. Pause screen

Pause triggers:

- `P` or `Escape` while playing.
- Pause/Resume button in the deck.
- Window blur while playing.
- Page becomes hidden while playing.

Pause does **not** save the high score.

## Layout

- Dim screen with `dim(.55)`.
- Panel: `PAUSE_RECT = { x: 55, y: 200, w: 520, h: 320 }`, cyan accent.
- `SYSTEM HALTED` at `(W/2,236)`, tracked size 9, tracking 3.2, cyanSoft.
- `PAUSED` at `(W/2,258)`, pixel size 6, ink, glow 18 cyan.
- Decorative line at y 322, width 338.
- **RESUME row:** `PAUSE_RESUME_RECT = { x: 125, y: 340, w: 380, h: 78 }`.
  - Label `RESUME`, hint `P / ESC / ENTER`, cyan accent.
- **TITLE SCREEN row:** `PAUSE_TITLE_RECT = { x: 125, y: 426, w: 380, h: 78 }`.
  - Label `TITLE SCREEN`, hint `Q`, cyanSoft accent, alpha .82.
  - If `game.newHigh` is true, show `[HIGH SCORE LOST]` in amber inline to the right of the label, size 9, tracking 1.1, weight 600. Leave room before the `Q` hint.
- Taps outside the two rows do nothing.
- Pointer over rows: pointer cursor. Elsewhere: default.

## Resume

- Tap/click RESUME row.
- `P`, `Escape`, or `Enter`.
- Deck Pause/Resume button.
- Restore crosshair cursor.

## Return to title

- Tap/click TITLE SCREEN row or press `Q`.
- Only available while paused or game over.
- Abandon the run:
  - Restore `game.hiScore = storedHiScore()`.
  - Clear score, reset wave to 1, lives to 3, nextExtra 5000.
  - Clear newHigh, banner, shake, flash, dying timer.
  - Set mode `menu`, crosshair cursor, UI sound.
  - Announce that the run was abandoned and only game over saves a high score.
- Do not write a high score.

Pause announcement:

- Normal: “Game paused. Press P, Escape, or Enter to resume, or Q to return to title.”
- If new high: append “Warning: returning to title will lose this run’s new high score.”

---

# 10. Game-over screen

Triggered only when a run ends (`lives <= 0` after death, or invasion).

On `endGame(reason)`:

- Guard if already game over.
- Set mode `gameover`, `endReason`, clear banner, mark player dead.
- If invaded: extra red burst and shake at least 14.
- **Call `saveHi()` — this is the only place a run’s high score is persisted.**
- Play game-over jingle.
- Clear held input, sync UI.
- Announce: “Game over. Final score N. Press Enter to play again or Q to return to title.”

## Layout

- Dim screen with `dim(.68)`.
- Panel: `GAME_OVER_RECT = { x: 55, y: 160, w: 520, h: 440 }`, red accent.
- Reason at `(W/2,196)`, tracked size 9, tracking 3, red:
  - `SECTOR LOST` for destroyed
  - `THE INVASION SUCCEEDED` for invaded
- `GAME OVER` at `(W/2,218)`, pixel size 6, pale pink-white, glow 22 red.
- Decorative line at y 272, width 338.
- `FINAL SCORE` at `(W/2,299)`, pixel size 2, muted.
- Final score at `(W/2,321)`, pixel size 5, ink, glow 8 cyan.
- `HI-SCORE ######` at `(W/2,373)`, pixel size 2, dim.
- If new high: `NEW HIGH SCORE` at `(W/2,395)`, pixel size 2, amber, glow 10.
- **PLAY AGAIN row:** `GAME_OVER_PLAY_RECT = { x: 125, y: 420, w: 380, h: 78 }`.
  - Label `PLAY AGAIN`, hint `ENTER / SPACE`, cyan accent.
- **TITLE SCREEN row:** `GAME_OVER_TITLE_RECT = { x: 125, y: 506, w: 380, h: 78 }`.
  - Label `TITLE SCREEN`, hint `Q`, cyanSoft accent, alpha .82.

Interactions:

- Tap/click PLAY AGAIN → start new game immediately.
- `Enter` or `Space` → start new game.
- Tap/click TITLE SCREEN or press `Q` → return to title.
- Taps outside rows do nothing.
- Pointer over rows: pointer. Elsewhere: default.
- The game-over high score was already saved; returning to title keeps it.

---

# 11. UI action-row style

Use one shared renderer for pause, game-over, info reset, and reset confirmation:

```js
drawActionRow(rect, label, hint, accent, alpha, warning, textScale = 1)
```

- Draw the cut-corner panel with the accent color.
- Label at `rect.x + 26`, vertically centered, tracked size `12*textScale`, tracking `2.4*textScale`, ink.
- Optional warning immediately after the label, amber, size `9*textScale`, tracking `1.1*textScale`, weight 600, full alpha.
- Hint at `rect.x + rect.w - 26`, right-aligned, size `9*textScale`, tracking `1.8*textScale`, accent color.
- Pause/game-over rows use `textScale = 1`; info/reset rows use the info responsive scale.

Action row sizes:

- Pause/game-over: 380 × 78.
- Info reset: 480 × 78.
- Reset YES/NO: 440 × 78.

On a 390 px-wide phone, 78 logical units ≈ **44.1 CSS px** tall.

---

# 12. Input specification

## Keyboard

| Key | Action |
| --- | --- |
| `ArrowLeft` / `A` | Move left |
| `ArrowRight` / `D` | Move right |
| `Space` | Fire while playing; start game / play again on title or game over |
| `Enter` | Start game / play again / resume from pause |
| `P` / `Escape` | Pause while playing; resume while paused |
| `M` | Toggle sound |
| `I` | Toggle system info on the title screen |
| `R` | Open high-score reset confirmation in system info |
| `Y` | Confirm high-score reset |
| `N` | Cancel high-score reset |
| `Q` | Return to title from pause or game over |

Keyboard details:

- Ignore key repeat for action keys.
- Prevent default for all used keys.
- Ignore when focus is in `input`, `textarea`, or `select`.
- Ignore `Space`/`Enter` when focus is on a button.
- `Escape` closes info when not confirming.
- `Space`/`Enter` close info when not confirming.
- `Q` does nothing during gameplay or on the title.
- `Y`/`N` only matter during reset confirmation.
- `R` only matters in system info.

## Pointer / touch on the canvas

Use pointer events with `pointerdown`, `pointermove`, `pointerup`, `pointercancel`, and `lostpointercapture`.

- Ignore right-click / non-primary mouse buttons.
- Convert client coordinates to logical 630×720 coordinates using the canvas bounding rect.
- On the title screen:
  - Info icon hit rect → toggle info.
  - Anywhere else → start game.
- In info:
  - Reset row → open confirmation.
  - Anywhere else → close info.
- In confirmation:
  - YES → reset.
  - NO or anywhere else → cancel confirmation, keep info open.
- On pause:
  - RESUME row → resume.
  - TITLE SCREEN row → return to title.
  - Anywhere else → nothing.
- On game over:
  - PLAY AGAIN row → start new game.
  - TITLE SCREEN row → return to title.
  - Anywhere else → nothing.
- During gameplay:
  - Pointer down starts drag/fire.
  - Horizontal drag moves the ship toward the pointer.
  - Pointer up/cancel stops firing.
  - Use pointer capture when possible.

## Touch deck

On coarse-pointer devices, show:

- Large **◀** and **▶** buttons, held to move.
- Large **FIRE** button, held to fire.
- Buttons support pointer capture and active styling.
- FIRE on title or game over starts a new game.
- FIRE while info is open closes info (through the same primary action as a title tap).
- FIRE does nothing while paused.

Deck buttons:

- **Sound on/off** toggles and persists sound.
- **Pause/Resume** toggles pause and reflects the mode.
- On the title screen, Pause is disabled.

### Exact touch target sizes

- Info icon: 78×78 logical → **44.1×44.1 CSS px** on a 390 px viewport.
- Pause/game-over rows: 380×78 → **214.7×44.1 CSS px**.
- Info reset row: 480×78 → **271.2×44.1 CSS px**.
- Reset YES/NO rows: 440×78 → **248.6×44.1 CSS px**.
- Touch move/fire buttons: **48 px** tall (40 px on short landscape screens).
- Deck Sound/Pause utility buttons: **min-height 30 px** (26 px on short landscape screens), matching the existing desktop-style cabinet deck. They are intentionally smaller than the in-canvas action rows.

## Cursors

- Gameplay and normal title: crosshair.
- Pause, game over, info, and confirmation: default outside interactive rows.
- Pointer over info icon, action rows, YES/NO rows, and reset row.
- Restore crosshair on resume/start/title.

## Accessibility

- Canvas: `role="application"`, focusable, descriptive `aria-label`, and `aria-keyshortcuts`.
- Hidden `aria-live="polite"` region for announcements:
  - Game start, wave start, pause, resume, game over, info open/close, reset prompt/cancel/success, abandon warning.
- Buttons have accessible labels and pressed state for sound.
- The primary in-canvas touch targets (info icon, reset row, YES/NO, pause/game-over rows) are ~44 CSS px tall on a phone. The deck utility buttons follow the smaller cabinet styling described above.

---

# 13. Audio

All audio is synthesized; no files.

- Lazy-create `AudioContext` on first interaction; resume if suspended.
- Master gain `.55` when enabled, `0` when disabled.
- Sound toggle persists as `on`/`off` in localStorage.
- Helper `tone(freq, endFreq, duration, type, volume, delay)` uses exponential frequency and gain ramps.
- Helper `noise(duration, volume, delay, cutoff)` uses a 0.5 s white-noise buffer and low-pass filter.

Exact effects:

| Sound | Recipe |
| --- | --- |
| Player shoot | 760→180 Hz, .09 s, square, .025 |
| Enemy shoot | 230→150 Hz, .08 s, sawtooth, .011 |
| March | Four-note cycle 116, 103, 92, 82 Hz; each drops to 86% over .055 s, square, .018 |
| Invader hit | Noise .065 s / .026 / cutoff 3200 + 620→220 Hz .08 s square .018 |
| Bullet clash | Noise .09 s / .022 / cutoff 2800 + 540→170 Hz .11 s triangle .018 |
| Player hit | Noise .55 s / .06 / cutoff 750 + 160→34 Hz .62 s sawtooth .05 |
| UFO hum | Alternating 680/540 Hz, .065 s square, .009, every .19 s |
| UFO hit | 620, 760, 980, 1320 Hz each .09 s triangle .027 staggered .055 + noise .18 s / .04 / cutoff 3000 |
| UFO deflect | Noise .07 s / .03 / cutoff 5200 + 210→1050 Hz .15 s triangle .028 + 1480→620 Hz .1 s square .012 after .045 |
| Extra ship | 520, 660, 880 Hz, .1 s triangle .028, staggered .08 |
| Wave clear | 392, 523, 659, 784, 1047 Hz, .12 s triangle .026, staggered .1 |
| Game over | 262, 220, 175, 131 Hz, each falls to 84%, .23 s sawtooth .026, staggered .18 |
| Start game | 262, 330, 392, 523 Hz, .1 s square .024, staggered .085 |
| Ready | 440→660 Hz, .12 s triangle .022 |
| UI | 880→660 Hz, .04 s square .01 |

---

# 14. Persistence, build info, and extras

## Game identity and the one-time hash

```js
GAME_NAME = 'last-line'
GAME_HASH = '<unique random 6 lowercase hex chars, generated once>'
GAME_ID = `${GAME_NAME}-${GAME_HASH}`
storeKey(name) => `si.${GAME_ID}.${name}`
```

Generate `GAME_HASH` **exactly once at the start of development**:

- Use a secure random source, for example `openssl rand -hex 3`.
- The result must be **6 lowercase hexadecimal characters**.
- Write the generated value into the source as a literal.
- Never regenerate, randomize, or change it afterward.
- Do not reuse the hash from the reference game or from any other existing build. This prompt intentionally does not contain the reference hash.
- The reference game uses the same name-plus-hash format; only the format is an example, not its value.

This hash is the game’s save namespace. Changing it would orphan every player’s high score and sound preference.

Only these keys may be read or written:

- `si.${GAME_ID}.hi`
- `si.${GAME_ID}.sound`

For example, if the generated hash were `a1b2c3`, the keys would be `si.last-line-a1b2c3.hi` and `si.last-line-a1b2c3.sound`.

Unknown/legacy keys must be ignored. Do not migrate old scores.

## Build stamp

- `BUILD_HASH`: 7-character lowercase hex FNV-1a hash of all `<style>` + `<script>` text.
  - Start with `0x811c9dc5`.
  - For each UTF-16 code unit: XOR, then `Math.imul(h, 0x01000193)`.
  - Convert to unsigned 32-bit hex, pad to 8, keep first 7.
- `BUILD_DATE`: UTC `YYYY-MM-DD HH:MM:SS UTC` from `document.lastModified`; fallback `UNKNOWN`.
- `PAGE_NAME`: last path segment, URL-decoded, fallback `index.html`.

## High-score reset

- Available only in system info.
- Opening confirmation does not change anything.
- Confirming sets `game.hiScore = 0`, clears `newHigh`, writes `"0"` to the high-score key, closes the confirmation, keeps info open.
- Cancelling changes nothing.
- This is intentional even though normal high scores are only saved at game over.

## Supporting files

If reproducing the full repository, include or recreate:

- `favicon.ico`, `favicon.svg`, `favicon-32.png`, `apple-touch-icon.png`
- `icon-192.png`, `icon-512.png`
- `site.webmanifest`
- `screenshot.png` (optional)
- `README.md`
- `.github/workflows/deploy-pages.yml` for manual GitHub Pages deployment

Do not include analytics, telemetry, or third-party tracking scripts. The game should make no external network requests for gameplay, fonts, images, audio, or analytics.

If the game must be exactly one file, keep all gameplay self-contained and omit missing icon/manifest links rather than breaking them.

---

# 15. Acceptance checklist

A correct reproduction should satisfy all of these:

- Single static `index.html`; no build step; works offline.
- Portrait 7:8 game screen inside a dark neon arcade cabinet with scanlines, glare, vignette, and subtle cyan grid.
- 630×720 logical canvas scaled 2×, crisp pixel sprites, 5×7 pixel font, monospaced tracked labels.
- Exact invader/ship/UFO patterns and colors.
- 55 invaders in the correct 11×5 formation, two-frame animation, marching, edge drops, speed-up, invasion loss.
- Correct player movement, one-bullet limit, cooldown, enemy bombs, bullet-vs-bomb clashes, bunker damage/crushing, UFO deflector behavior.
- Correct scoring, wave bonuses, extra ships, 5-life cap.
- Title screen text and layout match, including `PRESS ENTER TO PLAY`, target showcase, and top-right `i` icon.
- Info screen shows page, game ID, build hash/date, resolution, and arcade best; reset works by touch and keyboard.
- Pause screen has RESUME and TITLE SCREEN rows; tapping outside does nothing; `[HIGH SCORE LOST]` appears when appropriate.
- Game-over screen has PLAY AGAIN and TITLE SCREEN rows, correct reason text, final score, hi-score, and new-high state.
- A unique random 6-hex `GAME_HASH` is generated once at project start, is not copied from the reference game, and never changes; `GAME_ID` and all storage keys derive from it.
- High score is persisted **only** when a run reaches game over. Pause quit, reload, and tab close before game over do not save it; pause quit restores the stored high.
- The primary in-canvas touch targets are at least ~44 CSS px tall on a 390 px viewport; the deck utility buttons match the smaller cabinet styling.
- Keyboard and touch flows both work; announcements are available to assistive tech.
- No runtime exceptions; no external gameplay assets.

Build it as a polished, finished arcade game, not a prototype.
