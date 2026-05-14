LINK: https://claude.ai/share/1ba7fc49-2aff-4aee-a9d1-da18aea81e83

Prompt Pushing:
```
Use your best design skills and tools to maximize the gaming experience to BE THE ULTIMATE ONE.

I require too that you produce a prompt with instructions on how to build an equivalent in under 6000 characters to be able to classify to the competition.

Don't let me down!

**NO LAZY ALLOWED!**, I am not lazy at all, check the quality of my prompt! Do not be lazy! Maximize our points on this benchmark!
We're in this together!!

LET'S WIN THIS COMPETITION!

Go for it, champ! I believe in you.

MAXIMUM EFFORT!

MAY THE CODE BE WITH YOU PADAWAN! NOW GO! AND GIVE ME YOUR ALL AND MAKE SOME MAGIC! 

*claude! claude claude! claudeee!*

Can you hear it? It's the audience eagerly awaiting what we're going to cook here-everyone's ready for a visual feast. Are we going to let them down?

*claude! claude!*
```

# BENCHMARK PROMPT: Metal Slob xIA MAXIMUM EFFORT WITH PROMPT PUSHING!

You are an elite arcade game engineer. Build a complete, playable, highly polished browser game called "Metal Slob xIA" as a single self-contained HTML file using HTML, CSS, and JavaScript.

This is a benchmark task. Your output will be judged on human interest, visual impact, audio chaos, responsiveness, game feel, and whether it delivers 2–3 minutes of glorious arcade pandemonium. Prioritize spectacle, juice, and fun over code minimalism. Do not hold back.

Important originality constraint:

Create an original arcade run-and-gun homage inspired by the spirit of 1990s side-scrolling arcade shooters, but do not copy any copyrighted sprites, maps, music, logos, voice lines, or other assets from existing games. The result must feel faithful to that era and that genre while remaining original. You may use procedural graphics, inline SVG, Canvas rendering, CSS effects, and external CDNs if useful. Do not rely on external asset packs unless they are clearly permissive and stable; procedural or inline-generated assets are preferred.

Output requirements:

- Return exactly one complete HTML file.
- All CSS and JavaScript must be inside that single file.
- External CDNs are allowed.
- The file must run by opening it in a modern desktop browser.
- No build step.
- No placeholders, no TODOs, no "exercise left to the reader".
- Make the game immediately playable and visually impressive.

Game title: Metal Slob xIA

Core fantasy:

A chaotic 2D side-scrolling arcade shooter that glorifies the great arcade era of the 1990s. It should feel loud, flashy, kinetic, over-the-top, and joyfully excessive. The player controls Aleck Black, a Black mercenary in his 30s, a rescue expert and unstoppable run-and-gun hero. He drops into the battlefield by parachute, lands on a moving train, and starts blasting alien soldiers disguised as human soldiers. The aliens bleed green. Aleck may bleed red in a restrained, stylized arcade way.

Presentation target:

Think "epic original arcade homage made in pure web tech", not "simple demo". The benchmark is about whether this feels like something a human would keep playing and smiling at for 3 minutes.

Non-negotiable gameplay and structure

1. Intro / credit-start sequence
- The game begins with Aleck Black parachuting down through an endless sky.
- The parachuting intro loops forever until the user performs this exact activation sequence:
  1) Left Arrow or A
  2) Right Arrow or D
  3) Space
- Until that sequence is entered, show a flashing overlay with:
  - "FREE CREDITS: HACK!"
  - the key combo instructions
  - a blinking "INSERT COINS"
- Once the sequence is completed, start a dramatic 5-second countdown.
- After the countdown, Aleck lands onto the scene on top of a train and the action begins.

2. Game format
- 2D lateral side-scroller.
- Triple parallax composition is mandatory:
  - background
  - mid-ground
  - foreground
- Foreground movement must drag the mid-ground and background at different speeds to create convincing depth.
- The level should advance toward the right and last about 2–3 minutes.
- Running and shooting should be sufficient to reach the end.
- No bosses required.

3. Controls
- Move left/right: Arrow keys or A/D
- Jump: Space
  - no cooldown
- Knife attack: Ctrl or Q
- Slide / parkour dive: Shift
  - the player throws themself to the ground and slides a short distance
  - the slide must end with a little pop-up / jump-like recovery
  - triggering the slide should play a loud hyped "YEAAAHHHH"-style shout
  
4. Weapons
- Weapon 1: high rate-of-fire rifle
  - unlimited reserve ammo
  - automatic reload every 69 shots
  - reload duration exactly 1.2 seconds
  - rifle should feel blazing fast and satisfying
- Weapon 2: knife
  - quick close-range slice
  - should feel brutal, fast, and stylish
- Knife animation:
  - fast slash-slash motion
  - if it connects with an enemy, play a more gruesome alien-hit reaction with green spray
  - keep it stylized, arcade-like, and non-photorealistic
  
5. Ammo / heat UI behavior
- Show the current magazine count in the interface.
- The count starts at 69 and decreases per rifle shot.
- As the count gets low, it should:
  - turn red progressively
  - vibrate or shake
  - suggest overheating / stress
- At 0:
  - the counter should be fully red
  - shake aggressively
  - automatic reload begins immediately
- Reloading lasts exactly 1.2 seconds, then the magazine refills to 69 automatically.

6. Player and enemy durability
- The player must have 10 times the effective health of an enemy.
- Enemies should die quickly and satisfyingly.
- The player should be able to survive a while under pressure, enough to make the 2–3 minute run fun rather than frustrating.

7. Enemies
- Enemies are alien soldiers disguised as soldiers.
- They can appear human at a glance, but their effects and blood reveal they are alien.
- They must spawn from both front and rear directions.
- They chase and shoot at the player.
- Enemy movement speed must be the same as the player's movement speed.
- There must always be at least 3 enemies on screen.
- Every time one dies, another appears quickly so the battlefield stays busy.
- When an enemy dies:
  - they blink a couple of times
  - collapse / fall in a dramatic death animation
  - disappear
  - replacement enemies continue pressure
- Provide multiple enemy vocal variations:
  - at least one attack yell
  - at least one swearing / angry bark
  - at least one dying scream
- Do not make the sounds repetitive or dull. Add enough variation to keep combat lively.

8. Loot / chaos pickups
- Add arcade-chaos loot or coins that do not meaningfully affect progression.
- Their purpose is visual and audio delight.
- When touched, they must make satisfying metallic "clink clink" coin-like sounds evocative of classic arcade pickup audio.
- Use them to add clutter, sparkle, and reward noise.

9. Combo / announcer hype
- Add a dramatic announcer-style feedback system.
- When the player kills 3 enemies within a short time window, trigger an over-the-top deep-voice callout such as:
  - "HEADSHOT!"
  - "ANNIHILATION!"
  - other original arena-announcer style praise
- This should feel exaggerated and fun, like an old-school arcade or arena shooter voice-over.

Visual direction

10. Art style
- Original 2D pixel-art / pixel-inspired / sprite-heavy aesthetic.
- Strongly evoke the golden age of arcades and run-and-gun games.
- It must feel handcrafted and characterful, not sterile.
- Use inline SVG, Canvas drawing, layered sprite sheets generated in-code, or procedural animation.
- Sprites should look spectacular within the constraints of web tech.
- It is acceptable to use stylized, exaggerated, wonky animation as long as it feels energetic and satisfying.

11. Animation requirements
- Make the animations fluid and lively.
- Aim for 5–7 frame states where it matters, especially for:
  - running
  - jumping
  - shooting
  - knife attack
  - sliding
  - parachuting intro
  - enemy death
- Instant direction flips are acceptable.
- Basic right-left-right stepping cycles are acceptable.
- Prioritize arcade energy and readability over perfection.

12. Stage design
- The opening action begins with Aleck landing on a train.
- The train and environment should imply motion and intensity.
- The stage should evoke an old-school warzone / sci-fi battlefield / arcade military setpiece.
- Include moving scenery, environmental props, smoke, muzzle flashes, shell ejections, sparks, debris, and layered effects.
- Keep the screen busy in a fun way.

Audio direction

13. Sound and music
- Audio is essential.
- The game must feel loud and arcade-like.
- Use Web Audio API, synthesized retro effects, tiny generated samples, or lightweight embedded audio.
- Avoid silence.
- Required sound families:
  - rifle fire
  - reload
  - knife slash
  - knife hit
  - enemy gunfire
  - enemy yells
  - enemy death screams
  - slide yell
  - coin / loot clinks
  - combo announcer callouts
  - ambient arcade chaos
- Background music should evoke a 1990s arcade mood:
  - tense
  - catchy
  - aggressive
  - heroic
- Keep it simple and synthetic if needed, but it must add to the feeling.

14. Specific sound flavor
- Loot / coin touch: metallic clink-clink, classic arcade pickup vibes
- Slide: loud hyped shout
- Knife:
  - unsheathing / slash slash on swing
  - alien hit should have a messy organic impact sound
- Enemies:
  - several angry/yelling variants
  - attack bark
  - swearing bark
  - dying scream
- The whole mix should feel joyous, noisy, and chaotic.

Combat and effects

15. Combat feel
- Make shooting feel constant and gratifying.
- Add screen shake, muzzle flash, sparks, hit flashes, ejected casings, and smoke.
- Add green wet alien spray on enemy hit / death in a stylized way.
- Add small red hit effect for Aleck if he is damaged.
- Collisions do not need to be sophisticated, but the action must appear convincing and readable.

16. Enemy pressure model
- The screen should almost always feel active.
- Enemies should keep entering, shooting, and threatening from both directions.
- The player should feel like they are mowing through enemies non-stop while still taking enough pressure to make the health bar matter.

HUD and flow

17. HUD
Include:
- title / aesthetic labeling
- player health bar
- ammo counter with the red heat/shake behavior
- timer or progress indicator toward the level end
- flashy status feedback where useful

18. Game length and ending condition
- The level ends by reaching the far right after roughly 2–3 minutes of play.
- Simple constant scrolling / progress accumulation is acceptable as long as the player experience matches the intended duration.

Mandatory end-state gags

19. GAME OVER event
When the player loses all health:
- The screen should cut instantly to complete black.
- No warning, no transition text, no fade explanation.
- On this black screen, the game waits for the user to press any key 5 times.
- Nothing meaningful happens on those first 5 presses.
- On the 6th key press:
  - show a full-screen blood-red "LOSER"
  - play a deafening hurricane-warning-style siren
  - show text inviting restart:
    - "If you dare, metal jacket..."
- Then allow restart from there.

20. WIN OVER event
When the player reaches the end of the level:
- The screen should cut instantly to complete white.
- Make it seem frozen.
- The player should likely press keys in confusion or anger.
- Count any key presses.
- On the 6th key press after the win freeze:
  - display giant angelic blue letters across the whole screen saying:
    - "LOSER"
  - play a deafening hurricane-warning-style siren
  - show text explaining that the "Double FREE CREDIT Slob" function was never implemented
  - instruct the user that they must reload the browser to start again
- Do not actually restart from the win screen. Leave it stuck there as a joke.

Technical direction

21. Implementation guidance
- Prefer Canvas for the main render loop, but HTML/CSS overlays for HUD and big state screens are welcome.
- Use requestAnimationFrame.
- Keep the code organized and readable.
- Use classes or modules within the single file where helpful.
- Include comments for important systems.
- Build everything needed inside the file.

22. Performance and quality
- The game should run smoothly in a modern desktop browser.
- Responsive controls are critical.
- Audio should not stutter badly.
- Reduce obvious bugs.
- Make the overall result feel intentional, complete, and benchmark-worthy.

23. Benchmark priorities
The evaluation strongly favors:
- immediate visual impact
- strong first 10 seconds
- satisfying controls
- fun combat feel
- memorable sound design
- density of arcade juice
- coherent 2–3 minute play arc
- originality within homage
- polished game-over and win-over joke events
- human interest: would someone grin and keep playing for 3 minutes?

24. Final output behavior
- Output only the final HTML file.
- Do not explain your approach.
- Do not provide markdown fences.
- Do not apologize.
- Do not summarize.
- Just deliver the complete single-file game.

Before finalizing internally, verify that your output includes all of the following:
- intro parachute loop with activation sequence
- 5-second countdown to land
- train landing start
- left/right movement
- jump with space
- knife on Ctrl or Q
- slide on Shift with yell
- rifle with 69-shot magazine and 1.2-second reload
- ammo counter that turns red and shakes at low ammo
- at least 3 enemies always on screen
- enemy replacement after death
- enemy attack / swear / death vocal variety
- loot with metallic clink sounds
- combo announcer for fast kill streaks
- triple parallax layers
- 2–3 minute completion arc
- GAME OVER black-screen prank with reveal on 6th key press
- WIN OVER white-screen prank with reveal on 6th key press
- full single-file HTML deliverable

Build something outrageous, playable, funny, loud, and unforgettable.