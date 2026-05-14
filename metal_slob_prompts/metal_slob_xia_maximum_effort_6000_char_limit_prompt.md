METAL SLOB xIA — single-file browser arcade build prompt

GOAL: Complete playable 2D side-scroll run-and-gun as ONE self-contained HTML file — no build step, no external assets, no images, no libraries. 1990s arcade homage, 100% original art/audio (no copyrighted names/sprites/sounds). Canvas 1280x720 pixel-art (imageSmoothingEnabled=false) + HTML/CSS HUD overlays. All graphics procedurally drawn via ctx.fillRect/arc. All audio synthesized via Web Audio (oscillators, noise buffers, biquad filters, gain envelopes). CSS transform scales the stage to viewport, 16:9 preserved.

HERO: "Aleck Black" — Black mercenary, 30s, rescue-expert vibe: bandana, combat vest, cargo pants, boots. Procedural pixel art with run-cycle legs, plus jump/fall/slide/knife/parachute variants.

INTRO (state=INTRO): Title "METAL SLOB xIA", fake "FREE CREDITS: HACK!" teaser, "★ INSERT COINS ★" marquee. Aleck parachutes in a sinusoidal bob against a purple/orange night sky with drifting clouds/stars. Player must enter an exact 3-key sequence to "hack" free credits: LEFT or A → RIGHT or D → SPACE. Each correct step lights an indicator + plays a coin-clink. After the 3rd correct key, start a 5-second countdown (5…4…3…2…1). On 0, cut to PLAYING: Aleck drops onto a moving train with landing thud, dust particles, camera shake.

PLAYING (state = PLAYING, level duration 155 seconds ≈ 2.5 min, no boss required):
SETTING: Top of a speeding train. World auto-scrolls, Aleck stays roughly centered. Triple parallax: (1) far — night-to-sunset sky gradient, stars, mountains, clouds at ~0.05x; (2) mid — dystopian industrial buildings with randomly lit windows + smokestacks at ~0.3x; (3) foreground — train cars with wheels, windows, rail ties, wheel sparks at 1.0x. Rolling smoke from stacks. HUD strip shows a "TRAIN ADVANCE" progress bar with 🚂 icon traveling left→right as level completes.

CONTROLS: Arrows or WASD move; Space = jump (instant, no cooldown, gravity ~1800 px/s²); Ctrl or Q = knife swing (~0.4 s, ~60 px reach, multi-hit set prevents double-tapping same enemy); Shift = slide (0.55 s, 360 px/s, ends in an automatic pop-up jump; 0.8 s cooldown; plays a long YEAAAHHHH yell synthesized via formant bandpass filters sweeping through "Y-E-A-H" vowels).

RIFLE: Auto-fires continuously while not reloading/sliding/knifing. Magazine = EXACTLY 69 rounds. When it empties, auto-reload begins and takes EXACTLY 1.2 seconds (no variation). Reserve ammo is unlimited. Fire cadence ~0.085 s between shots. Each shot: spawns tracer bullet, muzzle flash, sparks, ejected shell casing with gravity, tiny camera bump, rifle crack sound. AMMO UI: big number top-right. Color interpolates from bright yellow toward red as ammo drops; the number physically shakes harder the closer to empty. While reloading it shows "RELOADING" with a blink animation.

KNIFE: Green alien blood spray on hit (particles in #2f2 / #4f4 / #afc / #7e7), heavier camera bump, satisfying meaty hit sound.

HP: Player HP = 1000 (10× an enemy's ~24–32 HP so Aleck is a tank). HP bar top-left, labeled "ALECK BLACK — HP". Invulnerability 0.5 s after taking a hit. At HP ≤ 0 → GAME OVER sequence.

ENEMIES: Disguised alien soldiers — look humanoid-ish but subtly off: pointed ears, one green-tinged eye. Palette-varied per instance. ALWAYS keep at least 3 on screen; spawn from both left and right offscreen at GROUND_Y. Enemy move speed EQUALS player speed (240 px/s) — no cheese. AI: approach to preferred range, then pepper Aleck with shots. Green blood on hit. Voice lines synthesized via formant filters triggered at events: attack-yell on spawn/aggro, an angry swear on taking damage, a death-scream on kill — each pulled from a small pitch/formant variation pool so they sound like different soldiers.

COINS / LOOT: Random pickups (coins, stars, gems) bounce on ground with a metallic clink on pickup + sparkle particles. Purely cosmetic — they pump the combo/arcade feel, no gameplay effect.

COMBO ANNOUNCER: Every 3rd consecutive enemy kill within a short combo window, fire a deep-voiced arcade announcer callout. Voice is synthesized per-syllable via stacked formant bandpass filters (low fundamental ~85–110 Hz). Phrase pool (rotate randomly): ANNIHILATION, DEVASTATION, MAYHEM, OVERKILL, SLOBBERIZED, CARNAGE, TRIPLE SLOB, DECIMATION, HEADSHOT SUPREME, SAVAGERY, BLOODBATH, MERCILESS, METAL RAIN, PANDEMONIUM. Giant center-screen text pop with scale+fade, camera bump. Reset combo counter after the callout.

MUSIC: Continuous driving arcade loop via a 32-step sequencer at 152 BPM in E minor — bass (square/saw), lead riff, kick (filtered noise burst + sine drop), snare (band-passed noise), hi-hats (high-passed noise). Start on first user input (autoplay-policy compliant). Stop on game-over/win.

PRANKS (the payoff — these are THE jokes):
GAME OVER: The moment Aleck dies, IMMEDIATELY cut to a full black screen. No text, no UI, nothing. Silence music. Count the next 6 key presses (any key). On the 6th press, reveal: solid red background, huge white "LOSER" in Impact/condensed display font with wide letter-spacing, bottom subtitle "If you dare, metal jacket… (reload to retry)", and a hurricane-siren sweep (sawtooth sweeping 200→1500 Hz, 4 s). Player must reload browser to restart.
WIN: When the level timer completes, IMMEDIATELY freeze to a full white screen. Silence music. Count 6 key presses. On the 6th reveal: blue background, huge white angelic "LOSER", subtitle "The 'Double FREE CREDIT Slob™' function was never implemented. Reload the browser to start again. You magnificent slob." Same siren. Reload required.

POLISH: CRT scanlines overlay, vignette, subtle CRT glow, camera shake on impacts, particle systems for sparks/smoke/casings/blood/flash/starburst, bullet tracers, muzzle flashes, landing dust, wheel sparks. Everything procedural. Single HTML file, opens locally, no network, no libraries.
