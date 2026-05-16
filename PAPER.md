# Game-Builder-Free: Evaluating AI Game-Writing Outputs Through Realistic Project Constraints

**Author:** Bobby Zhang

---

## Abstract

Modern game development is an increasingly multi-disciplinary field where coding, narrative design, and artistic production must function as a single unit. While generative AI tools have proliferated for individual tasks, they often operate in isolation, leading to "thematic drift" — a fragmentation that results in a jarring user experience where generated assets such as 2D sprites do not align with the game's narrative or logic.

This research explores the challenge of maintaining thematic cohesion across different AI modalities by developing a unified game generation pipeline hosted on Hugging Face Spaces. The architecture utilizes a "Source of Truth" workflow built around a two-step image generation chain: a short theme-derived seed description is first expanded by `llama-3.3-70b-versatile` — running with the **Z-Image-Engineer V4** system prompt — into a detailed 200-250 word cinematic image prompt; that enhanced prompt is then passed to **FLUX.1-schnell** via Pollinations.AI to generate the actual sprite image. In parallel, `llama-3.1-8b-instant` generates the HTML5 Canvas game code. This prompt-chaining approach — where one LLM enhances the input for a separate image model — is the core mechanism for cross-modal thematic alignment.

A significant "Logic vs. Theme" gap was identified during development. While AI successfully generated functional code, the visual theme remained incomplete or generic unless every detail was explicitly described and injected. To solve the "Image Gap," this project implemented a system to convert generated images into **Base64 Data URIs**, injecting them directly into the HTML to ensure a self-contained, on-theme playable file. This study concludes that successful AI game generation requires a multimodal pipeline where thematic metadata is shared between models to prevent artistic inconsistency.

**Keywords:** Multimodal AI, Game Development, Thematic Cohesion, Prompt Engineering, Cross-Modal Alignment.

---

## 1. Introduction and Research Question

Game development is evolving rapidly; modern production now extends far beyond coding to include asset design, narrative structure, character art, and environmental storytelling. The proliferation of generative AI has provided developers with tools for creating images, audio, code, and video. However, a significant challenge remains: these AI tools typically operate in isolation, requiring developers to integrate outputs manually. This leads to a lack of **thematic cohesion** — where the code, visual assets, and gameplay feel disconnected from each other even when they share the same surface-level theme.

This project investigates this problem directly by building **Game-Builder-Free**, a Hugging Face Space that takes a single user-written theme as input and generates a fully playable HTML5 canvas game complete with AI-generated sprites. Rather than treating code generation and image generation as separate tasks, the system uses a shared theme as a cross-modal anchor — the same text input drives all three outputs simultaneously: game logic, image prompts, and visual sprites.

The core research questions are:

- **Thematic Decision Making:** How do generative AI models decide on a "theme" from a text prompt, and what mechanisms allow them to maintain consistency across code and image outputs?
- **Cross-Modal Communication:** In what ways can different model types — text-to-code and text-to-image — share thematic information so that the final game follows the intended aesthetic and narrative direction of the original input?
- **Practical Evaluation:** Do realistic project-based tests reveal more about AI tool limitations than synthetic benchmark tests do?

For this project, "quality" is measured by developer usefulness: does the generated game provide something worth playing, testing, or iterating on? A generated game does not need to be perfect. It needs to give a developer a functional starting point.

---

## 2. Related Work

Procedural content generation (PCG) has long studied how algorithms can automatically produce game content. Recent work increasingly connects this tradition to large language models (LLMs), raising new questions about quality, consistency, and thematic alignment.

Ashby et al. developed a knowledge-graph-augmented approach to personalized quest and dialogue generation in role-playing games, finding that grounding LLM outputs in structured world knowledge significantly improved narrative consistency [1]. This is directly relevant to Game-Builder-Free's challenge: the system must maintain thematic consistency not across a narrative, but across modalities — code and image must both reflect the same theme.

Vartinen, Hamalainen, and Guckelsberger studied GPT-based RPG quest generation and reported wide variation in output quality across different prompt structures [2]. Their finding that prompt design heavily influences output usefulness aligns with a central challenge encountered in this project: the LLM frequently ignored game-type-specific constraints (such as top-down vs. side-view perspective) unless those constraints were made extremely explicit in the system prompt. This suggests that the brittleness observed in game-code generation is not unique to code tasks — it reflects a broader sensitivity of LLMs to prompt specificity.

Maleki and Zhao's survey of procedural content generation with LLM integration notes that LLMs are best understood as one tool within a larger PCG pipeline, not a replacement for human game design judgment [3]. Game-Builder-Free takes this view: the system uses LLMs to accelerate asset generation but relies on carefully engineered system prompts and post-processing steps (such as background removal via `rembg`) to enforce quality constraints the models cannot reliably self-impose.

Summerville's work on evaluation methodologies for procedural generation warns against relying on cherry-picked examples and advocates for structured rubrics that assess constraint satisfaction rather than surface-level fluency [4]. This shaped the evaluation approach used in this project: games are evaluated on whether they run correctly, whether sprites match the theme and game type, and whether gameplay mechanics function as specified — not on how visually impressive a single screenshot appears.

Together, these sources frame the core challenge: thematic consistency in multimodal AI generation is harder than single-modality generation, and evaluation must focus on practical usefulness rather than surface quality.

---

## 3. Method

Game-Builder-Free is a four-stage pipeline hosted as a public Hugging Face Space. The user provides a single text input — a theme description and game genre — and the system produces a fully playable HTML5 canvas game with embedded AI-generated sprites. The pipeline can be summarized as:

```
User Theme
    ├── [Groq llama-3.1-8b-instant]          → HTML5 game code
    └── [Seed constructor]                   → short seed descriptions (per sprite)
             ↓
        [Groq llama-3.3-70b as Z-Image-Engineer V4]  → enhanced cinematic prompts
             ↓
        [FLUX.1-schnell via Pollinations.AI]  → sprite images
             ↓
        [rembg + Base64 injection]            → sprites embedded in HTML
```

Critically, `llama-3.3-70b-versatile` does **not** generate images. It generates better text prompts for a separate image model. This prompt-chaining step — using one LLM to improve the input to another model — is the core mechanism for thematic alignment in the image generation path.

### 3.0 Step-by-Step Flow

The following describes exactly what happens from the moment a player submits a theme to the moment the game appears on screen:

**Step 1 — Player inputs a theme.**
The player types a theme description (e.g. *"Ancient Egyptian tomb raid with cursed mummies"*) and selects a game genre (Platformer or Top-Down Shooter). This single text input is the only thing the player provides. It becomes the shared anchor for all downstream generation.

**Step 2 — Groq generates the game code.**
The theme is sent to `llama-3.1-8b-instant` via the Groq API alongside a structured system prompt that encodes game-type-specific rules. The model outputs a complete single-file HTML5 canvas game. At this stage the game references placeholder filenames like `sprite_player.png` and `sprite_background.png` — the actual images do not exist yet.

**Step 3 — Theme is used to generate image prompts.**
In parallel, the same theme is used to construct short seed descriptions for each sprite — one for the player character, one for the background, one for the enemy. Each seed is passed to `llama-3.3-70b-versatile` via Groq, running with the Z-Image-Engineer V4 system prompt. This model expands each short seed into a detailed 200-250 word cinematic image prompt describing lighting, perspective, texture, color palette, and composition. The output of this step is text only — no images are generated here.

**Step 4 — Enhanced prompts are sent to the image generator.**
Each cinematic prompt produced in Step 3 is sent to **FLUX.1-schnell** via Pollinations.AI. FLUX receives only the enhanced prompt — it has no knowledge of the original theme text, the game code, or the seed. FLUX generates the actual pixel image for each sprite. Character sprites are then post-processed with `rembg` to remove the background, leaving only the character on a transparent background.

**Step 5 — Images are embedded into the HTML game file.**
Each generated image is converted to a Base64 Data URI string and injected directly into the HTML game code, replacing the placeholder filenames (`sprite_player.png` → `data:image/png;base64,...`). The final output is a single self-contained HTML file where all sprites are baked in — no external files, no server, no broken image links. The game is immediately playable in the browser iframe.

### 3.1 Pipeline Architecture

**Stage 1 — Code Generation:**
`llama-3.1-8b-instant` via the Groq API receives a structured system prompt and the user's theme. The system prompt encodes strict game-type-specific rules (e.g. gravity and platformer physics vs. top-down movement, bullet direction, enemy spawn logic) as explicit code templates. The model outputs a complete single-file HTML5 game.

**Stage 2 — Prompt Seed Enhancement (LLM → LLM):**
For each sprite needed (player, background, enemy — and additionally platform and goal for platformers), a short seed description is constructed from the user's theme and game type. For example: `"top-down overhead game sprite, viewed from directly above, space marine theme, 64x64 pixel style"`. These seeds are then passed to `llama-3.3-70b-versatile` via Groq, running with the **Z-Image-Engineer V4** system prompt. This model acts purely as a **prompt engineer** — it does not generate any images. Its sole output is an expanded 200-250 word cinematic image prompt with explicit lighting, composition, camera angle, texture, and color grading details. This is a deliberate prompt-chaining technique: using one LLM to produce better input for a separate image generation model.

**Stage 3 — Sprite Image Generation (Enhanced Prompt → FLUX):**
Each enhanced prompt produced in Stage 2 is sent as input to **FLUX.1-schnell** via Pollinations.AI — a completely separate text-to-image model. FLUX.1-schnell receives the cinematic prompt and generates the actual pixel image. It has no knowledge of the original user theme, the game code, or the seed description — it only receives the enhanced prompt. This separation is important: the quality of the sprite depends entirely on how well the Stage 2 LLM translated the theme into visual language that FLUX.1-schnell can act on. Non-background sprites are then post-processed with `rembg` to remove backgrounds, resized to 64×64 pixels, converted to Base64 Data URIs, and injected directly into the HTML replacing `sprite_NAME.png` references. The result is a self-contained playable HTML file.

### 3.2 Evaluation Rubric

Each generated game is evaluated against the following criteria:

| Criterion | Question |
|---|---|
| Code correctness | Does the game run without JavaScript errors? |
| Mechanic implementation | Are the specified mechanics present and functional (movement, shooting, collision)? |
| Theme consistency (code) | Do game colors, labels, and structure reflect the input theme? |
| Theme consistency (sprites) | Do sprite images match the game type perspective and theme? |
| Sprite quality | Are character sprites isolated without background artifacts? |
| Restart functionality | Does the game correctly reset after game over? |

Games were tested across two genres (Platformer, Top-Down Shooter) with multiple themes per genre, with each output evaluated by the author against the rubric above.

---

## 4. Findings and Discussion

### 4.1 Code Generation Quality

The LLM consistently produced functional game structures — canvas setup, game loop, player movement, enemy spawning — but showed systematic failures in specific areas:

**Scope errors** were the most frequent critical bug. The model repeatedly declared `canvas` and `ctx` inside function bodies (e.g. inside `startGame()` or `gameLoop()`) rather than at the top level, causing variables to be undefined when accessed by other functions. This resulted in a completely black game screen with no visible elements. The fix required explicitly specifying in the system prompt that `canvas` and `ctx` must be the very first declarations in the script.

**Physics bleed-across genres** was the second most common issue. The model applied platformer physics (gravity `velY += 0.5`, grounded checks) to top-down shooter games, causing the player to immediately fall off screen. Separating the two game types into distinct prompt templates with explicit `FOR TOP-DOWN SHOOTER ONLY` and `FOR PLATFORMER ONLY` sections reduced but did not eliminate this issue.

**Bullet direction inconsistency** occurred when the model used the mouse click position to compute a bullet direction vector, resulting in bullets traveling diagonally based on cursor distance rather than straight upward. Providing the exact code pattern `bullets.push({x: player.x+player.w/2-4, y: player.y-16, vy:-10})` as a template eliminated this.

These findings suggest that for structured, executable outputs like game code, **LLMs require template-level constraints** — not just descriptions of desired behavior. Describing physics in prose is insufficient; providing the exact code pattern is necessary.

### 4.2 Sprite Generation Quality

Sprite generation revealed a persistent **cross-modal alignment problem** in the prompt-chaining pipeline. The two-step chain — LLM seed enhancer → FLUX image generator — succeeded at thematic content (colors, atmosphere, environment) but failed at structural perspective control. Sprites generated by FLUX.1-schnell consistently showed the wrong game-type perspective regardless of how the seed or enhanced prompt was worded. Player and enemy sprites for top-down shooters appeared in front-facing standing poses (matching platformer style) rather than overhead bird's eye views.

The seed descriptions passed to the Z-Image-Engineer LLM were iterated across multiple sessions with increasingly specific perspective wording:
- Session 1 seed: `"top-down aerial view"` → FLUX output: front-facing sprites
- Session 2 seed: `"bird's eye view directly overhead"` → FLUX output: mostly front-facing sprites
- Session 3 seed: `"like Metal Slug or GTA 2 sprite angle, head at top, feet at bottom"` → FLUX output: partial improvement, inconsistent

This reveals a fundamental limitation of the prompt-chaining approach for perspective control: even when the seed correctly specifies the perspective, and even when the Z-Image-Engineer LLM expands it into a detailed cinematic prompt, the downstream image model (FLUX.1-schnell) does not reliably honor the perspective instruction. This suggests the failure is not in the seed or the prompt enhancement step — it is in FLUX.1-schnell's training distribution, which contains far more front-facing character images than overhead game sprites. The cinematic prompt expansion produced by the Z-Image-Engineer may actually compound this problem by adding photorealistic lighting and composition details that further anchor FLUX toward its most common training patterns.

Background images showed strong thematic consistency across all tested themes. The model reliably matched environmental details, color palette, and atmosphere to the user's input theme.

Background removal via `rembg` successfully eliminated solid background colors from character sprites, though semi-transparent edge artifacts remained visible at small sprite sizes (64×64 pixels).

### 4.3 The Theme-Logic Gap

A key finding is the distinction between **surface theming** and **structural theming**. The LLM successfully applied surface theming: color names, labels, and enemy descriptions matched the input theme. However, structural theming — where the game's mechanics, level layout, and challenge design reflect the theme — was not reliably achieved. A "jungle temple" platformer had the same platform positions, enemy behaviors, and win condition as a "cyberpunk rooftop" platformer.

This aligns with Vartinen et al.'s finding that AI-generated game content tends toward genre conventions rather than theme-specific design. Fixing this would require encoding theme as a structural constraint on game design, not merely a surface label.

### 4.4 Real Test Run Results

The following table records actual test runs performed on the live Game-Builder-Free Space during development. Each run used the full pipeline — Groq code generation, Z-Image-Engineer prompt enhancement, FLUX.1-schnell sprite generation, and rembg background removal — with the game rendered live in the Gradio iframe.

| Test | Theme | Mode | Sprites Generated | Game Launched | What Worked | What Failed | What I Learned |
|---|---|---|---|---|---|---|---|
| 1 | Ancient Egyptian tomb raid with cursed mummies | Top-Down Shooter | 3 FLUX sprites (player, background, enemy) | ✅ Yes | Background rendered correctly — Egyptian desert ruins with strong atmosphere. Enemies visible falling from top. Score and health HUD displayed. | Player invisible on screen. Enemies had grey background box around them. Enemy sprites front-facing not overhead view. | Background generation is reliable; character sprite perspective and background removal need more work |
| 2 | Deep space asteroid mining station under alien parasite attack | Top-Down Shooter | 3 FLUX sprites | ✅ Yes | Enemy sprites successfully rendered. Background rendered dark space station floor. | Player not visible. Game over screen appeared at game start due to health initialising at 0. Canvas background not filling full width — SVG tiling visible. `restartHandler` and `onShoot` both active simultaneously, click during gameplay reset the game. | Multiple listener conflicts in generated code; canvas/ctx scope bugs persist even with explicit system prompt rules |
| 3 | Jungle temple with ancient traps | Platformer | Fallback SVG placeholders (FLUX quota exhausted) | ✅ Yes | Game structure correct — platforms, player, enemies, goal all present. Score and lives UI displayed. Player movement working. | All sprites showed as coloured boxes with text labels ("player", "enemy", "backgr"). Physics bug: player fell through platforms intermittently. Keyboard not registering on some inputs. | HF inference quota depletes fast; fallback placeholders confirm code pipeline works independently of image pipeline |
| 4 | Neon cyberpunk city drone invasion | Top-Down Shooter | 3 FLUX sprites | ✅ Yes | Background (dark cyberpunk cityscape with neon) matched theme well. Enemy drones visible falling from top. Bullet firing on click worked. | Player spawned at bottom edge instead of center. Bullets disappeared immediately after firing — `vy` was positive (downward) instead of negative. Too many enemies — spawn rate was 90ms not 90 frames. | Spawn rate unit confusion (ms vs frames) is a recurring model error; bullet direction must be explicitly constrained to straight upward |
| 5 | Neon cyberpunk rooftops | Platformer | 3 FLUX sprites | ✅ Yes | Sprites generated and embedded as base64 correctly. Background showed cyberpunk cityscape. | Platformer physics bug: `grounded=false` set inside platform loop `else` branch — player fell through all platforms instantly. Score displayed twice on canvas. | Grounded logic must be enforced as "set false BEFORE loop, set true only on landing" — describing it in prose is not enough, exact code pattern required |

**Summary:** Across 5 test runs, the code pipeline launched a game in all 5 cases. Sprite generation succeeded in 4 of 5 runs (1 hit quota limit). However, every generated game had at least one functional bug requiring a system prompt fix. The most frequent failures were player spawn position, canvas/ctx variable scope, and bullet direction. Background images showed the strongest thematic consistency; character sprites showed the weakest perspective accuracy.

A key next improvement would be an **automatic validation step** — a lightweight JavaScript linter or headless browser test that checks for common structural errors (canvas declaration position, presence of `Promise.all`, `gameOver` flag usage) before the game is presented to the user. This would catch the majority of recurring code generation bugs without requiring manual inspection.

---

## 5. Limitations

Several limitations constrain the conclusions of this project.

**Model capability ceiling:** `llama-3.1-8b-instant` is a relatively small model optimized for speed. Larger models (e.g. 70B parameter class) would likely produce more reliable code with fewer structural bugs. The choice of a small model was driven by the free-tier constraint of the Groq API, which is a real-world project constraint but not an ideal research condition.

**Image generation perspective control:** Pollinations.AI/FLUX.1-schnell cannot reliably generate overhead-view character sprites through prompt engineering alone. A dedicated game-sprite fine-tuned model, or a model supporting ControlNet-style structural conditioning, would be needed to reliably control perspective. This is a fundamental limitation of the current pipeline, not a prompt engineering failure.

**Single evaluator:** All rubric scores were produced by the author. A stronger study would recruit multiple evaluators — ideally practicing game developers or game design students — and measure inter-rater agreement to validate the rubric's reliability.

**Two game types only:** The current system supports only Platformer and Top-Down Shooter genres. Other genres (puzzle, infinite runner, strategy) were attempted but removed due to higher bug rates. The findings may not generalize to genres with more complex game logic.

**No player testing:** Games were evaluated for technical correctness and thematic consistency but were not tested by players for fun, difficulty balance, or engagement. A complete evaluation would include playtest sessions.

---

## 6. Conclusion

Game-Builder-Free demonstrates that a multimodal AI pipeline can generate playable HTML5 games from a single text input, but also reveals significant limitations in cross-modal thematic consistency. The system succeeds at surface-level theming — matching colors, labels, and environmental imagery to the user's input — but struggles with structural theming, sprite perspective control, and code reliability.

The central finding is that **LLMs generating executable code require template-level constraints, not prose descriptions**. Describing desired behavior in natural language produced inconsistent results across generations; providing exact code patterns as templates produced far more reliable outputs. This has practical implications for any AI-assisted development tool: the more structured and executable the output must be, the more structured and executable the constraint must also be.

The sprite perspective problem — where character sprites consistently appeared in front-facing poses regardless of game type — represents an unresolved cross-modal alignment challenge. The text-to-image model's training distribution appears to dominate over prompt instructions when those instructions conflict with common image conventions. Solving this likely requires fine-tuning or structural conditioning rather than prompt engineering.

This project supports the broader argument from Summerville [4] and Maleki and Zhao [3] that AI game generation tools should be evaluated against realistic project constraints. A tool that produces a visually impressive screenshot but a non-functional or mechanically inconsistent game is not a useful development tool. Realistic evaluation — running the game, checking mechanics, testing edge cases — reveals failure modes that screenshot-based evaluation cannot.

Future work should investigate whether larger code models reduce structural bug rates, whether sprite fine-tuning can solve the perspective alignment problem, and whether sharing a structured thematic representation between the code and image models (rather than just sharing the raw text prompt) can improve structural thematic coherence.

---

## References

[1] Ashby, T., Webb, B. K., Knapp, G., Searle, J., & Fulda, N. (2023). Personalized Quest and Dialogue Generation in Role-Playing Games: A Knowledge Graph- and Language Model-based Approach. *Proceedings of the 2023 CHI Conference on Human Factors in Computing Systems*.

[2] Vartinen, S., Hamalainen, P., & Guckelsberger, C. (2024). Generating Role-Playing Game Quests With GPT Language Models. *IEEE Transactions on Games*.

[3] Maleki, M. F., & Zhao, R. (2024). Procedural Content Generation in Games: A Survey with Insights on Emerging LLM Integration.

[4] Summerville, A. (2018). Expanding Expressive Range: Evaluation Methodologies for Procedural Content Generation.
