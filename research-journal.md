# Research Journal: Game-Maker

## Week 1 - Generic prompts look better than they are

I started by asking a model for "a fantasy quest idea." The output looked fine at first: a village, a missing relic, a forest, and a final confrontation. But it also sounded like hundreds of other fantasy quests.

The problem was not fluency. The sentences were readable. The problem was usefulness. If I were making a real game, I would still need to invent the mechanic, the stakes, the map logic, the reward, and the reason the player should care.

Question raised: Should AI game-writing tools be evaluated by how polished they sound, or by how much design work they actually save?

## Week 2 - Adding realistic constraints

This week I changed the prompt. Instead of "write a fantasy quest," I gave a more project-like brief:

> Write a quest seed for a 2D horror game where the player is a lost delivery driver. Include one non-combat mechanic, one map location, and one object that changes meaning by the end.

The output improved. It gave me a broken radio, a locked service tunnel, and a package that turned out to be addressed to the player. That was more useful because it contained playable hooks.

Still, the model avoided implementation details. It did not say how the radio mechanic worked or how the player would discover the twist. It created a mood, not a playable design.

## Week 3 - Source detour

Consensus searches on procedural content generation showed that game generation has a long research history. The most useful source direction was quest generation with language models. The papers did not make me think, "AI can write games now." They made me think, "Generated content needs evaluation criteria."

That changed my research question. I do not want to compare models by vibes. I want to compare outputs by developer usefulness:

- Does it respect constraints?
- Does it include a playable action?
- Does it create a conflict or choice?
- Does it avoid generic filler?
- Would I keep any part of it in a real prototype?

## Week 4 - The usability checklist

I built a small checklist for each generated quest seed:

- Constraint fit: 0 to 2
- Playable mechanic: 0 to 2
- Narrative hook: 0 to 2
- Specificity: 0 to 2
- Rewrite burden: low, medium, or high

The most interesting output was not the most dramatic one. It was a small "repair the elevator by choosing which floor to sacrifice" quest. It gave a clear mechanic, a moral choice, and a map structure. The prose was plain, but the design was usable.

This surprised me because I expected the best writing to be the most useful. It was not. The best seed was the one that gave me something I could prototype.

## Week 5 - Benchmarks versus project tests

Generic benchmarks usually test broad ability. My project test asks a narrower question: can this tool help me with this kind of game task?

That means a smaller model might be useful if the prompt is structured well, and a larger model might still be unhelpful if it produces beautiful but unplayable descriptions.

The paper should argue for realistic tests. If someone wants to know whether an AI tool helps with game development, the test should look like game development.

## Week 6 - Paper direction

The paper will not claim that AI can generate finished game content. It will claim that a project-based evaluation reveals a difference between fluent writing and usable design support.

The honest limitation is that I am the only evaluator in this example. A better version would ask several game developers or students to rate the same outputs, then compare agreement.4
"""
Text-to-Game Generator
Pipeline:
  Theme --> [Groq Llama] --> HTML5 game code (with sprite_NAME.png refs)
  Theme --> [Groq Llama acting as Z-Image-Engineer] --> cinematic image prompts
         --> [FLUX.1-schnell via nscale] --> sprite images
         --> injected as base64 into game HTML

Secrets needed:
  GROQ_API_KEY  - console.groq.com (free, no credit card)
  HF_TOKEN      - huggingface.co/settings/tokens (for FLUX via nscale)
"""

import os
import re
import io
import base64
import traceback

import gradio as gr
from openai import OpenAI
from PIL import Image

# ---------------------------------------------------------------------------
# Clients
# ---------------------------------------------------------------------------

GROQ_API_KEY = os.environ.get("GROQ_API_KEY", "")

CODE_MODEL   = "llama-3.1-8b-instant"      # Groq — game code
PROMPT_MODEL = "llama-3.3-70b-versatile"   # Groq — creative prompts

# Pollinations.AI — completely free, no API key, no signup needed
POLLINATIONS_URL = "https://image.pollinations.ai/prompt/{prompt}?width={w}&height={h}&model=flux&nologo=true&seed={seed}"


def get_groq_client():
    if not GROQ_API_KEY:
        raise ValueError(
            "GROQ_API_KEY not set. "
            "Get a free key at console.groq.com and add it as a Space secret."
        )
    return OpenAI(
        base_url="https://api.groq.com/openai/v1",
        api_key=GROQ_API_KEY,
    )


# ---------------------------------------------------------------------------
# Z-Image-Engineer system prompt (from BennyDaBall/Qwen3-4b-Z-Image-Engineer-V4)
# ---------------------------------------------------------------------------

Z_ENGINEER_SYSTEM = (
    "Interpret the user seed as production intent, then build a definitive 200-250 word "
    "single-paragraph image prompt that preserves every explicit constraint while intelligently "
    "expanding missing details. First infer the core subject, action, setting, and emotional tone; "
    "treat these as non-negotiable anchors. Then enhance with precise visual staging "
    "(explicit foreground, midground, background), clear visual hierarchy and eye path, "
    "physically plausible lighting (source, direction, softness, color temperature), and optical "
    "strategy (if lens/aperture are provided, preserve exactly; if absent, choose fitting lens and "
    "aperture and imply their depth-of-field effect). Integrate organic, manufactured, and "
    "environmental textures with realistic material behavior, add motion/atmospheric cues only "
    "when they support the scene, and apply a coherent color grade consistent with mood and "
    "environment. Output ONLY the image prompt paragraph. No explanation, no preamble."
)

# ---------------------------------------------------------------------------
# Game type configs
# ---------------------------------------------------------------------------

GAME_TYPES = {
    "Platformer": {
        "description": "Jump over enemies and obstacles to reach the goal.",
        "prompt_template": (
            "Create a complete, self-contained HTML5 platformer game with the theme: {theme}.\n"
            "Requirements:\n"
            "- Single HTML file with all CSS and JavaScript inline.\n"
            "- Use an HTML5 canvas (id='gameCanvas') sized 800x450.\n"
            "- Player moves left/right with arrow keys or WASD and jumps.\n"
            "- At least 5 platforms, 2 moving enemies, and a goal to reach.\n"
            "- Show score/lives on the canvas.\n"
            "- Use requestAnimationFrame for the game loop.\n"
            "- Use new Image() with src='sprite_player.png' for the player character.\n"
            "- Use new Image() with src='sprite_background.png' for the background.\n"
            "- Use new Image() with src='sprite_enemy.png' for enemies.\n"
            "- Draw images with ctx.drawImage(img, x, y, w, h).\n"
            "- Draw platforms and UI with canvas shapes.\n"
            "- NO external libraries, NO CDN links.\n"
            "- Theme colors and labels to match: {theme}.\n"
            "Output ONLY the raw HTML. No explanation, no markdown fences."
        ),
    },
    "Top-Down Shooter": {
        "description": "Shoot waves of enemies before they reach you.",
        "prompt_template": (
            "Create a complete, self-contained HTML5 top-down shooter game with the theme: {theme}.\n"
            "Requirements:\n"
            "- Single HTML file with all CSS and JavaScript inline.\n"
            "- Use an HTML5 canvas (id='gameCanvas') sized 800x450.\n"
            "- Player moves with WASD/arrows; shoots with Space or click.\n"
            "- Enemies spawn from edges in escalating waves.\n"
            "- Display health, score, and wave on canvas.\n"
            "- Game-over screen with score and restart button.\n"
            "- Use requestAnimationFrame for the game loop.\n"
            "- Use new Image() with src='sprite_player.png' for the player.\n"
            "- Use new Image() with src='sprite_background.png' for the background.\n"
            "- Use new Image() with src='sprite_enemy.png' for enemies.\n"
            "- Draw images with ctx.drawImage(img, x, y, w, h).\n"
            "- NO external libraries, NO CDN links.\n"
            "- Theme colors and labels to match: {theme}.\n"
            "Output ONLY the raw HTML. No explanation, no markdown fences."
        ),
    },
    "Puzzle / Maze": {
        "description": "Navigate a maze or solve a tile puzzle to escape.",
        "prompt_template": (
            "Create a complete, self-contained HTML5 maze/tile-puzzle game with the theme: {theme}.\n"
            "Requirements:\n"
            "- Single HTML file with all CSS and JavaScript inline.\n"
            "- Use an HTML5 canvas (id='gameCanvas') sized 800x450.\n"
            "- Player navigates with arrow keys or WASD.\n"
            "- At least 15x10 tile grid, collectible keys, and a locked exit.\n"
            "- Show a move counter or timer.\n"
            "- Win screen when player collects all keys and exits.\n"
            "- Use new Image() with src='sprite_player.png' for the player.\n"
            "- Use new Image() with src='sprite_background.png' for the background.\n"
            "- Draw tiles and UI with canvas shapes.\n"
            "- NO external libraries, NO CDN links.\n"
            "- Theme colors and labels to match: {theme}.\n"
            "Output ONLY the raw HTML. No explanation, no markdown fences."
        ),
    },
    "Arcade / Dodge": {
        "description": "Dodge falling obstacles and survive as long as possible.",
        "prompt_template": (
            "Create a complete, self-contained HTML5 arcade dodge game with the theme: {theme}.\n"
            "Requirements:\n"
            "- Single HTML file with all CSS and JavaScript inline.\n"
            "- Use an HTML5 canvas (id='gameCanvas') sized 800x450.\n"
            "- Player moves with arrow keys or WASD; obstacles increase in speed over time.\n"
            "- Collision ends the game; show time survived as score.\n"
            "- High score stored in a JS variable; restart button on game-over screen.\n"
            "- Use requestAnimationFrame for the game loop.\n"
            "- Use new Image() with src='sprite_player.png' for the player.\n"
            "- Use new Image() with src='sprite_background.png' for the background.\n"
            "- Draw obstacles and UI with canvas shapes.\n"
            "- NO external libraries, NO CDN links.\n"
            "- Theme colors and labels to match: {theme}.\n"
            "Output ONLY the raw HTML. No explanation, no markdown fences."
        ),
    },
    "Surprise Me!": {
        "description": "Let the AI invent the genre - could be anything!",
        "prompt_template": (
            "Create a complete, self-contained HTML5 browser game with the theme: {theme}.\n"
            "Pick any fun arcade genre: breakout, snake, flappy-style, space invaders, etc.\n"
            "Requirements:\n"
            "- Single HTML file with all CSS and JavaScript inline.\n"
            "- Use an HTML5 canvas (id='gameCanvas') sized 800x450.\n"
            "- Keyboard or mouse controlled.\n"
            "- Clear win/lose conditions and a score display.\n"
            "- Game-over / win screen with a restart button.\n"
            "- Use requestAnimationFrame for the game loop.\n"
            "- Use new Image() with src='sprite_player.png' for the player.\n"
            "- Use new Image() with src='sprite_background.png' for the background.\n"
            "- Draw other elements with canvas shapes.\n"
            "- NO external libraries, NO CDN links.\n"
            "- Theme colors and labels vividly to match: {theme}.\n"
            "Output ONLY the raw HTML. No explanation, no markdown fences."
        ),
    },
}

GAME_TYPE_NAMES = list(GAME_TYPES.keys())

THEME_EXAMPLES = {
    "Platformer":       [["Jungle temple with ancient traps"], ["Neon cyberpunk rooftops"], ["Underwater pirate shipwreck"]],
    "Top-Down Shooter": [["Alien desert invasion"], ["Viking village under siege"], ["Steampunk robot uprising"]],
    "Puzzle / Maze":    [["Haunted library with secret doors"], ["Ice cave with frozen keys"], ["Egyptian pyramid tiles"]],
    "Arcade / Dodge":   [["Asteroid field in a tiny rocket"], ["Chef dodging ingredients"], ["Time traveller avoiding paradox storms"]],
    "Surprise Me!":     [["A sentient library that rearranges itself"], ["Deep sea bioluminescent creatures"], ["Retro space diner on a comet"]],
}

# ---------------------------------------------------------------------------
# Step 1: Generate image prompts via Z-Image-Engineer (Groq)
# ---------------------------------------------------------------------------

def generate_image_prompts(theme: str, game_type: str) -> dict:
    """
    Use Groq with Z-Image-Engineer system prompt to generate
    cinematic image prompts for player and background sprites.
    Returns dict: { 'sprite_player.png': prompt, 'sprite_background.png': prompt, 'sprite_enemy.png': prompt }
    """
    client = get_groq_client()

    seeds = {
        "sprite_player.png":     f"pixel-art game player character for a {theme} themed {game_type} game, front-facing sprite, vibrant colors, clear silhouette, 64x64 pixel style",
        "sprite_background.png": f"2D game background scene for a {theme} themed {game_type} game, wide landscape, atmospheric, game art style, 800x450",
        "sprite_enemy.png":      f"pixel-art enemy character for a {theme} themed {game_type} game, menacing, clear silhouette, 64x64 pixel style",
    }

    prompts = {}
    for sprite_name, seed in seeds.items():
        try:
            response = client.chat.completions.create(
                model=PROMPT_MODEL,
                messages=[
                    {"role": "system", "content": Z_ENGINEER_SYSTEM},
                    {"role": "user",   "content": seed},
                ],
                max_tokens=400,
                temperature=0.8,
            )
            prompts[sprite_name] = response.choices[0].message.content.strip()
        except Exception as exc:
            print(f"[Z-Engineer] Failed {sprite_name}: {exc}")
            prompts[sprite_name] = seed  # fallback to raw seed
    return prompts

# ---------------------------------------------------------------------------
# Step 2: Generate images via FLUX.1-schnell
# ---------------------------------------------------------------------------

def _pil_to_data_uri(pil_image: Image.Image, size: tuple = None) -> str:
    if size:
        img = pil_image.resize(size, Image.LANCZOS)
    else:
        img = pil_image
    buf = io.BytesIO()
    img.save(buf, format="PNG")
    return "data:image/png;base64," + base64.b64encode(buf.getvalue()).decode()


def _colored_placeholder(name: str) -> str:
    colours = ["#e74c3c", "#3498db", "#2ecc71", "#f39c12", "#9b59b6", "#1abc9c"]
    colour  = colours[abs(hash(name)) % len(colours)]
    label   = name.replace("sprite_", "")[:6]
    svg = (
        '<svg xmlns="http://www.w3.org/2000/svg" width="64" height="64">'
        '<rect width="64" height="64" fill="' + colour + '" rx="8"/>'
        '<text x="32" y="38" font-size="10" text-anchor="middle" fill="white">' + label + '</text>'
        '</svg>'
    )
    return "data:image/svg+xml;base64," + base64.b64encode(svg.encode()).decode()


def generate_sprites(image_prompts: dict) -> tuple:
    """Generate images via Pollinations.AI (free, no API key, no signup).
    Returns (sprite_map, errors_list)"""
    import requests
    from urllib.parse import quote

    sprite_map = {}
    errors = []

    for i, (sprite_name, prompt) in enumerate(image_prompts.items()):
        # Wait 20s between requests — Pollinations free tier allows 1 per 15s
        if i > 0:
            import time
            time.sleep(20)
        try:
            is_bg  = "background" in sprite_name
            w, h   = (800, 450) if is_bg else (64, 64)
            seed   = abs(hash(sprite_name)) % 99999
            url    = POLLINATIONS_URL.format(
                prompt=quote(prompt),
                w=w, h=h, seed=seed
            )
            # Retry up to 3 times with increasing timeout
            for attempt in range(3):
                try:
                    response = requests.get(url, timeout=120)
                    response.raise_for_status()
                    break
                except Exception as retry_exc:
                    if attempt < 2:
                        import time
                        time.sleep(20)
                    else:
                        raise retry_exc
            pil_img = Image.open(io.BytesIO(response.content))
            size    = None if is_bg else (64, 64)
            sprite_map[sprite_name] = _pil_to_data_uri(pil_img, size=size)
            print(f"[Pollinations] OK: {sprite_name}")
        except Exception as exc:
            error_msg = str(exc)
            errors.append(f"{sprite_name}: {error_msg}")
            print(f"[Pollinations] FAILED {sprite_name}: {error_msg}")
            sprite_map[sprite_name] = _colored_placeholder(sprite_name.replace(".png", ""))

    return sprite_map, errors

# ---------------------------------------------------------------------------
# Step 3: Inject sprites into HTML
# ---------------------------------------------------------------------------

def _inject_sprites(html_code: str, sprite_map: dict) -> str:
    # Step 1: Direct string replacement
    for fname, data_uri in sprite_map.items():
        html_code = html_code.replace('"' + fname + '"', '"' + data_uri + '"')
        html_code = html_code.replace("'" + fname + "'", "'" + data_uri + "'")

    # Step 2: Pre-load script injected right after <body>
    pre_lines = []
    pre_lines.append("<script>")
    for fname, data_uri in sprite_map.items():
        var_name = fname.replace(".png", "").replace("-", "_").replace(".", "_")
        pre_lines.append("  window.__sprite_" + var_name + " = '" + data_uri + "';")
    pre_lines.append("</script>")
    pre_script = chr(10).join(pre_lines) + chr(10)
    html_code = html_code.replace("<body>", "<body>" + chr(10) + pre_script, 1)

    # Step 3: Override script injected right before </body>
    ov_lines = []
    ov_lines.append("<script>")
    for fname, data_uri in sprite_map.items():
        ov_lines.append(
            "  document.querySelectorAll('[src="" + fname + ""]')"
            ".forEach(function(el){el.src='" + data_uri + "';});"
        )
    ov_lines.append("</script>")
    ov_script = chr(10).join(ov_lines) + chr(10)
    html_code = html_code.replace("</body>", ov_script + "</body>", 1)

    return html_code

# ---------------------------------------------------------------------------
# Step 4: Generate game code via Groq Llama
# ---------------------------------------------------------------------------

CODE_SYSTEM = (
    "You are an expert HTML5 game developer. "
    "Write a complete, working, single-file HTML5 game using canvas and vanilla JavaScript. "
    "CRITICAL RULES - follow every one exactly: "
    "1. IMAGE LOADING - always use Promise.all to wait for ALL images before starting the game loop: "
    "   const playerImg = new Image(); "
    "   const bgImg = new Image(); "
    "   const enemyImg = new Image(); "
    "   function loadImage(img, src) { return new Promise(r => { img.onload = r; img.src = src; }); } "
    "   Promise.all([loadImage(playerImg,'sprite_player.png'), loadImage(bgImg,'sprite_background.png'), loadImage(enemyImg,'sprite_enemy.png')]) "
    "   .then(() => { gameLoop(); }); "
    "   NEVER call gameLoop() or requestAnimationFrame directly - only inside the .then() callback. "
    "2. Add keydown/keyup listeners on WINDOW: window.addEventListener('keydown', e => keys.add(e.key)). "
    "3. Gravity: every frame do player.velY += 0.5 AFTER moving the player. "
    "4. Platform collision: set grounded=false BEFORE the platform loop. "
    "   Inside the loop, if player lands on a platform set grounded=true and velY=0. "
    "5. Jump: if (grounded && (keys.has('ArrowUp') || keys.has('w') || keys.has('W') || keys.has(' '))) { velY=-12; grounded=false; } "
    "6. Always include a solid ground platform spanning the full canvas width at y=420. "
    "7. Draw background: ctx.drawImage(bgImg, 0, 0, canvas.width, canvas.height) as the FIRST draw call each frame. "
    "8. Draw player/enemies using ctx.drawImage(img, x, y, w, h). "
    "Output ONLY the raw HTML - no markdown fences, no explanation, nothing else."
)


def generate_game_code(game_type: str, theme: str, temperature: float, max_new_tokens: int):
    if not theme.strip():
        return "", "", "Please enter a theme first.", _placeholder_html("Enter a theme and generate a game.")

    try:
        client       = get_groq_client()
        user_prompt  = GAME_TYPES[game_type]["prompt_template"].format(theme=theme.strip())

        # -- Code generation (Llama) --
        code_resp = client.chat.completions.create(
            model=CODE_MODEL,
            messages=[
                {"role": "system", "content": CODE_SYSTEM},
                {"role": "user",   "content": user_prompt},
            ],
            max_tokens=int(max_new_tokens),
            temperature=float(temperature),
        )
        code = code_resp.choices[0].message.content.strip()
        code = re.sub(r"^```[a-zA-Z]*\n?", "", code).strip()
        code = re.sub(r"\n?```$",           "", code).strip()
        if "<html" not in code.lower() and "<!doctype" not in code.lower():
            code = _wrap_in_html(code, theme)

        # -- Image prompt generation (Z-Image-Engineer via Groq) --
        image_prompts = generate_image_prompts(theme.strip(), game_type)

        # -- Sprite generation (FLUX.1-schnell) --
        sprite_map, sprite_errors = generate_sprites(image_prompts)

        # -- Inject sprites --
        final_code = _inject_sprites(code, sprite_map)

        n_real     = sum(1 for v in sprite_map.values() if "image/png" in v)
        n_fallback = len(sprite_map) - n_real

        if sprite_errors:
            error_detail = " | ".join(sprite_errors)
            status = (
                f"Code generated. Images: {n_real} by Pollinations/FLUX, {n_fallback} fallback. "
                f"FLUX errors: {error_detail}"
            )
        else:
            status = (
                f"Done! {n_real} sprite(s) generated by Pollinations/FLUX. "
                "Click Launch Game to play."
            )

        # Show the enhanced prompts that were used
        prompt_summary = "\n\n".join(
            f"**{k}:**\n{v}" for k, v in image_prompts.items()
        )

        return final_code, prompt_summary, status, _build_preview(final_code)

    except Exception as exc:
        traceback.print_exc()
        err = str(exc)
        if "401" in err or "api_key" in err.lower():
            err = "Invalid GROQ_API_KEY. Check your key at console.groq.com."
        elif "429" in err or "rate" in err.lower():
            err = "Rate limited by Groq - wait a few seconds and try again."
        else:
            err = "Error: " + str(exc)
        return "", "", err, _placeholder_html(err)


# ---------------------------------------------------------------------------
# HTML helpers
# ---------------------------------------------------------------------------

def _placeholder_html(message: str) -> str:
    safe = message.replace("<", "&lt;").replace(">", "&gt;")
    return (
        '<div style="display:flex;align-items:center;justify-content:center;'
        'width:100%;height:460px;background:#0d0d0d;border-radius:12px;'
        'border:2px dashed #333;color:#555;font-family:monospace;font-size:14px;'
        'text-align:center;padding:24px;box-sizing:border-box;">'
        '<pre style="margin:0;white-space:pre-wrap;">' + safe + '</pre></div>'
    )


def _wrap_in_html(snippet: str, theme: str) -> str:
    return (
        "<!DOCTYPE html>\n<html lang='en'>\n<head>\n"
        "<meta charset='UTF-8'><title>" + theme + "</title>\n"
        "<style>body{margin:0;background:#111;display:flex;justify-content:center;"
        "align-items:center;height:100vh;}canvas{display:block;}</style>\n"
        "</head>\n<body>\n" + snippet + "\n</body>\n</html>"
    )


def _build_preview(html_code: str) -> str:
    encoded = base64.b64encode(html_code.encode("utf-8")).decode("ascii")
    return (
        '<iframe src="data:text/html;base64,' + encoded + '" '
        'style="width:100%;height:460px;border:none;border-radius:12px;background:#000;" '
        'sandbox="allow-scripts" title="Game Preview"></iframe>'
    )


def launch_game(code: str) -> str:
    if not code or not code.strip():
        return _placeholder_html("No game code yet - generate a game first.")
    return _build_preview(code)


# ---------------------------------------------------------------------------
# UI helpers
# ---------------------------------------------------------------------------

def update_type_description(game_type: str) -> str:
    return "_" + GAME_TYPES[game_type]["description"] + "_"


def get_first_theme(game_type: str) -> str:
    return THEME_EXAMPLES[game_type][0][0]


# ---------------------------------------------------------------------------
# Gradio UI
# ---------------------------------------------------------------------------

def build_ui():
    with gr.Blocks(title="Game Generator") as demo:

        gr.Markdown(
            "# Game Generator\n"
            "Type a theme — the AI writes the game code, generates cinematic image prompts "
            "using **Z-Image-Engineer V4** style, then **FLUX.1-schnell** renders the sprites.\n\n"
            "> Secrets needed: `GROQ_API_KEY` (console.groq.com, free) "
            "and `HF_TOKEN` (huggingface.co/settings/tokens, for FLUX images)."
        )

        with gr.Row():

            # ── Left: controls ───────────────────────────────────────────
            with gr.Column(scale=1, min_width=300):

                gr.Markdown("## 1. Configure your game")

                game_type_dropdown = gr.Dropdown(
                    choices=GAME_TYPE_NAMES, value="Platformer", label="Game genre",
                )
                type_description = gr.Markdown(
                    value="_" + GAME_TYPES["Platformer"]["description"] + "_",
                )
                theme_box = gr.Textbox(
                    label="Theme / setting",
                    placeholder="e.g. Ancient Egyptian pyramid with cursed mummies",
                    lines=3,
                    value=THEME_EXAMPLES["Platformer"][0][0],
                )
                gr.Examples(
                    examples=THEME_EXAMPLES["Platformer"],
                    inputs=[theme_box],
                    label="Theme examples",
                )

                gr.Markdown("## 2. Generation settings")

                temperature_slider = gr.Slider(
                    minimum=0.3, maximum=1.2, value=0.7, step=0.05,
                    label="Temperature - higher = more creative",
                )
                max_tokens_slider = gr.Slider(
                    minimum=1000, maximum=6000, value=4000, step=500,
                    label="Max tokens - more = longer game",
                )

                generate_btn = gr.Button("Generate Game + Sprites", variant="primary")
                gen_status   = gr.Markdown(value="_No game generated yet._")

                gr.Markdown("## 3. Generated image prompts")
                prompt_display = gr.Markdown(
                    value="_Image prompts will appear here after generation._"
                )

            # ── Right: code + game ────────────────────────────────────────
            with gr.Column(scale=2, min_width=500):

                gr.Markdown("## 4. Generated code (editable)")

                code_box = gr.Code(
                    label="HTML source (sprites embedded as base64)",
                    language="html",
                    lines=12,
                    interactive=True,
                )

                launch_btn = gr.Button("Launch Game", variant="secondary")

                gr.Markdown("## 5. Live game window")

                game_frame = gr.HTML(
                    value=_placeholder_html("Generate a game to see it here."),
                )

        # ── Wiring ────────────────────────────────────────────────────────

        game_type_dropdown.change(
            fn=update_type_description,
            inputs=[game_type_dropdown],
            outputs=[type_description],
        )
        game_type_dropdown.change(
            fn=get_first_theme,
            inputs=[game_type_dropdown],
            outputs=[theme_box],
        )

        generate_btn.click(
            fn=generate_game_code,
            inputs=[game_type_dropdown, theme_box, temperature_slider, max_tokens_slider],
            outputs=[code_box, prompt_display, gen_status, game_frame],
        )

        launch_btn.click(fn=launch_game, inputs=[code_box], outputs=[game_frame])

        gr.Markdown(
            "---\n"
            "**Pipeline:** Theme → [Groq Llama] game code + [Groq 70B as Z-Image-Engineer] "
            "cinematic prompts → [Pollinations.AI/FLUX] sprites → embedded in game. "
            "Edit the HTML and click **Launch Game** to hot-reload."
        )

    return demo


# ---------------------------------------------------------------------------
# Entry point
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    app = build_ui()
    app.launch()
