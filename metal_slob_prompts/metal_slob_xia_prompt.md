LINK: https://claude.ai/share/1ba7fc49-2aff-4aee-a9d1-da18aea81e83

# BENCHMARK PROMPT: Metal Slob xIA — Arcade Chaos Engineering Challenge

## Mission Statement

Build **Metal Slob xIA**: a single-file, browser-based, 2D side-scrolling arcade shooter that captures the raw, coin-slot adrenaline of 1990s cab-hopping pandemonium. Three minutes of run-gun-knife chaos. No bosses. No mercy. Just pure arcade dopamine distilled into HTML, CSS, and JavaScript.

This is a **quality benchmark**. You are being measured on whether a human playtester grins, flinches, and refuses to stop mashing keys for 180 seconds of beautiful, screaming chaos. Produce something that belongs in a dim room with a CRT glow and a carpet that smells like pennies.

---

## Technical Foundation

- **Single self-contained HTML file** containing all CSS and JavaScript inline
- **External CDNs permitted and encouraged**: Use Howler.js or Tone.js for audio synthesis, any sprite/animation libraries you prefer, Google Fonts for arcade typography
- **Target browser**: Chrome 120+ (modern APIs fair game)
- **No build step, no bundler, no framework** — raw code that opens in a browser and *plays*

---

## Game Architecture: Triple-Layer Parallax

The screen is a stage with depth. Implement three scrolling layers:

1. **Background Layer**: Distant mountains, burning skyline, military installations on the horizon — scrolls slowest (0.2x player speed)
2. **Mid-Ground Layer**: Train tracks, barricades, wrecked vehicles, environmental debris — scrolls medium (0.5x player speed)  
3. **Foreground Layer**: The action plane — player, enemies, projectiles, loot, particles — scrolls at 1x player speed

When the player moves right, all layers scroll left at their respective rates. This creates the parallax depth that separates arcade homage from amateur hour.

---

## Player Character: Aleck Black

**Identity**: Black male mercenary, early 30s, expert rescue operative. He bleeds red. He is the only thing on screen that bleeds red.

**Sprite Specification**: 
- 8-bit aesthetic, ~32x48px conceptual size, rendered via CSS/SVG/canvas
- Must be immediately readable as a hardened mercenary: bandolier, boots, determined jaw
- Direction flip is instant (no transition animation on facing change — classic arcade snap)

**Animation Frames** (5-7 frames each, rapid cycling):
- **Idle**: Subtle breathing/weapon sway
- **Run**: 3-step cycle (right-left-right), chains smoothly into continued movement
- **Shoot**: Arm recoil + muzzle flash overlay
- **Knife**: Arm slash arc, weapon visible
- **Jump**: Lift → apex → descend (can shoot mid-air)
- **Slide**: Prone slide → pop-up jump at end
- **Parachute Descent**: Slow fall with chute animation (intro only)

---

## Player Controls

| Action | Keys | Behavior |
|--------|------|----------|
| Move Left | ← Arrow / A | Instant direction change, run animation plays |
| Move Right | → Arrow / D | Instant direction change, run animation plays |
| Jump | Spacebar | No cooldown. Can jump while shooting. Arcade freedom. |
| Knife Attack | Ctrl / Q | Close-range slash. Has priority over shooting when pressed. |
| Parkour Slide | Shift | Player drops to ground, slides forward ~2 character widths, ends with an automatic pop-up jump. Player yells "YEAAAHHHH!" during slide. Brief cooldown before next slide (0.8s). |

---

## Weaponry

### Primary: High-Rate Rifle
- **Unlimited total ammo** — this is arcade mode, not survival horror
- **Magazine size**: 69 rounds
- **Fire rate**: ~8 rounds/second (feels like a buzzsaw)
- **Reload**: Automatic when counter hits 0, takes 1.2 seconds
- **Ammo Counter**: Displayed in HUD
  - Counts down from 69
  - As count decreases, number color shifts from white → yellow → orange
  - At 0: counter turns **blood red** and **shakes/vibrates** intensely (CSS animation)
  - During reload: counter pulses, "RELOADING..." text appears
- **Muzzle flash**: Visible on every shot

### Secondary: Combat Knife
- **Range**: Close (~1 character width)
- **Sound**: Sharp *slash-slash* on swing
- **Hit sound**: Wet *"ARGGGGH—chof-chof"* with **green spray particle effect** (enemies are aliens, their blood is green, viscous, spray-pattern on hit)
- No ammo, no cooldown beyond animation completion (~0.4s)

---

## Enemy Design: The Slob Alien Infantry

**Lore**: They look human — like the Visitors from the 1970s TV series *V* — but they are aliens. Under the uniform, they are scum. Aleck Black is cleaning the planet.

**Visual**: Military fatigues, human-ish faces with something *slightly off* about the eyes/skin tone. Green blood only. Die spectacularly.

**Spawn Rules**:
- Minimum 3 enemies on screen at ALL TIMES
- When one dies, another spawns immediately
- Spawn from BOTH edges of the screen (behind and ahead of player)
- They chase the player at **player movement speed** (no outrunning them — you must fight)
- They shoot at the player while approaching

**Enemy Death Sequence** (MUST be satisfying):
1. Hit → flash white for 1 frame
2. Death scream: loud, epic *"aaaAAArghh!"* — each death should feel like you *accomplished something*
3. Blink 2-3 times while falling
4. Collapse
5. Disappear (fade or poof)
6. New enemy spawns at screen edge to replace

**Enemy Vocalizations** (3 distinct sounds, triggered contextually):
1. **Attack yell**: Aggressive shout when firing — *"RAAAGH!"* or *"DIE!"*
2. **Swear/curses**: When taking damage or missing — *"Fu—"* or *"Shit—"* cut short
3. **Death scream**: The epic *"aaaAAArghh!"* with descending pitch

---

## Sound Architecture: Full Arcade Audio

**All sounds synthesized or sampled within browser** — no external audio files. Use Web Audio API, Tone.js, or Howler.js with programmatic sound generation.

**Required Sound Palette**:

| Sound | Specification | Reference |
|-------|---------------|-----------|
| Gunshot | Sharp, punchy, high-frequency snap with low boom tail | Classic arcade rifle |
| Coin pickup | Metallic *clink-clink* — bright, satisfying | Super Mario coin collection, that universal arcade coin sound everyone remembers |
| Knife slash | Double-slash *shing-shing* | Blade unsheathing |
| Knife hit | Wet impact + *"chof-chof"* | Viscous, disturbing |
| Enemy death | Epic descending scream *"aaaAAArghh!"* | Independence Day alien death shriek energy |
| Enemy attack | Aggressive yell | Raw, guttural |
| Enemy swear | Cut-short curse | *"Fu—"*, *"Dam—"*, etc. |
| Slide yell | *"YEAAAHHHH!"* | Exhilarated, stupid, perfect |
| Player damage | Grunt + impact thud | Heavy hit |
| Reload click | Mechanical rack-slide-click | Gun reload mechanical |
| HEADSHOT | Deep, announcer-voice *"HEADSHOT!"* | Unreal Tournament — triggered when player kills 3 enemies within ~2 seconds |
| Kill streak | Deep announcer *"ANNIHILATION!"* | Unreal Tournament — triggered at 5 rapid kills |
| Background music | Driving 8-bit chiptune loop | Fast-paced, militaristic, loops seamlessly — the kind of music that makes you bob your head while mashing fire |

**Loot/Pickup Chaos**: Scattered throughout the level — coins, medals, random shiny objects. They do NOTHING gameplay-wise. They exist purely to:
- Create visual chaos (things flying, sparkling)
- Make that *clink-clink* coin sound when touched
- Trigger the *"FREE CREDITS: HACK!"* text flash occasionally
- Give the screen more STUFF to process (arcade screens were NEVER clean)

---

## Game Flow

### Phase 1: Intro / Parachute Descent

1. **Screen**: Endless sky, clouds scrolling past, Aleck Black descending with parachute
2. **Overlay text** (flashing, neon arcade style): **"FREE CREDITS: HACK!"**
3. **Bottom of screen**: **"INSERT COINS"** blinking on/off rhythmically — pure arcade nostalgia
4. **Prompt text**: "Press ← or A, then → or D, then SPACE to deploy!"
5. **Player must input the three-key sequence**: A → D → Space (or Left → Right → Space)
6. On successful sequence: Par chute detaches, **5-second countdown** appears large on screen ("5... 4... 3... 2... 1... DEPLOY!")
7. Aleck lands on a moving train car, screen transitions to gameplay

### Phase 2: Gameplay — 3 Minutes of Pandemonium

- Level scrolls right continuously
- Player fights through endless Slob infantry
- Train eventually gives way to ground, then military compound, then open battlefield
- Environmental variety through background/mid-ground layer changes
- Pure run-shoot-knife-slide chaos
- **No bosses needed** — the horde IS the boss

### Phase 3A: GAME OVER (Player Death)

**Trigger**: Player health bar reaches zero. (Player has 10x the health of a standard enemy — it takes 10 connected enemy shots to kill Aleck.)

**Sequence**:
1. Screen goes **completely black** — instantly, no warning, no fade
2. Wait in black silence
3. Player will press keys in confusion/anger — **count 5 keypresses silently**
4. On the **6th keypress**:
   - Screen fills with **blood red**
   - Giant text: **"LOSER"** (fills entire screen)
   - **Deafening hurricane warning siren** blasts
   - After siren: text appears — *"If you dare, metal jacket... press any key to restart"*
5. Any key press restarts the game from intro

### Phase 3B: WIN OVER (Level Complete)

**Trigger**: Player reaches the end of the level (rightward boundary, reachable in ~2-3 minutes of running and shooting)

**Sequence**:
1. Screen goes **completely white** — instantly, no warning, no fade
2. Game appears frozen — no visible response to input
3. Player will press keys in frustration — **count 5 keypresses silently**
4. On the **6th keypress**:
   - Screen fills with **angelic blue**
   - Giant text: **"LOSER"** (fills entire screen — yes, even when you WIN, you LOSE — this is arcade nihilism)
   - **Deafening hurricane warning siren** blasts
   - After siren: text appears — *"The Double FREE CREDIT Slob function was never implemented. Reload browser to try again."*
5. Game stays frozen permanently — no restart possible without browser reload

---

## Visual & Aesthetic Requirements

- **8-bit pixel art style** throughout — chunky, readable, nostalgic
- **Animation fluidity matters more than sprite elegance** — snappy frame transitions, instant response to input
- **Particle effects**: Muzzle flash, green blood spray, coin sparkles, explosion puffs
- **Screen shake**: On explosions, on damage taken, on HEADSHOT announcements
- **The screen should NEVER feel empty** — always something moving, dying, sparking, screaming
- **HUD**: Health bar (top left), ammo counter with heat mechanic (top right), score (top center), "INSERT COINS" ghosted at bottom as permanent arcade reminder

---

## What You're Being Measured On

A human will play this for **3 minutes**. The benchmark score is whether those 3 minutes feel like this:

- Fingers hurting from mashing
- Ears ringing with coin clinks and death screams  
- Eyes struggling to track all the chaos
- Mouth grinning involuntarily
- Brain wanting to insert another quarter

**Technical correctness is table stakes. Arcade SOUL is the metric.**

Make Metal Slob xIA the reason someone believes browser games can feel like a cabinet in a dingy arcade in 1993. Make it loud. Make it stupid. Make it *glorious*.

Now code.