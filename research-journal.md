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



## Session 7 Journal

**Date:** May 2, 2026
**Project:** Text-to-Game Generator (HuggingFace Space)

---

### Overview

This session was a long and iterative journey building a fully working **AI-powered HTML5 game generator** hosted on HuggingFace Spaces. The app takes a theme and game genre, generates playable game code, and generates sprite images — all for free.

---

### What I Built

A HuggingFace Space (`LeafCat79/Game_Builder_Free`) that:
- Takes a **theme** (e.g. "Jungle temple with ancient traps") and **game genre** as input
- Uses an LLM to **generate complete, playable HTML5 canvas games**
- Uses an image AI pipeline to **generate themed sprites** (player, background, enemy)
- **Embeds sprites as base64** directly in the HTML so the game is fully self-contained
- Renders the game live in an **iframe** inside the Gradio UI

---

### Architecture (Final)

| Component | Model | Provider |
|---|---|---|
| Game code generation | `llama-3.1-8b-instant` | Groq API (free) |
| Image prompt enhancement | `llama-3.3-70b-versatile` acting as Z-Image-Engineer V4 | Groq API (free) |
| Sprite image generation | FLUX via Pollinations.AI | Pollinations (free, no key) |

---

### Problems Encountered & Solution

#### 1. Memory Limit Exceeded (OOM — 16GB)
**Problem:** Loading large models locally (Qwen3-Coder-Next ~140GB, ERNIE-Image ~32GB) crashed the Space container.
**Solution:** Switched to API-based inference — models run on external servers, Space uses ~200MB RAM.

#### 2. Build Timeout (Exit Code 143)
**Problem:** `llama-cpp-python` compiles from C++ source during Docker build, taking 20+ minutes and exceeding HF's build timeout limit.
**Solution:** Removed `llama-cpp-python` entirely, switched to API calls which need no compilation.

#### 3. HF Inference API Quota Exhausted (402 Error)
**Problem:** Free monthly HF credits depleted — every image call returned `402 Payment Required`.
**Solution:** Tried multiple providers:
- nscale → didn't host FLUX.1-schnell
- hf-inference → quota depleted
- Google Imagen-3 → needed Google Cloud project setup (unavailable)
- **Pollinations.AI** → completely free, no API key, no signup ✅

#### 4. Game Not Working (Black Screen / Player Falls)
**Problem:** Three bugs in AI-generated game code:
1. Black screen — images not loaded before game loop started
2. Player falls into void — `grounded=false` set inside `else` of platform loop
3. Keyboard not working — `keydown` listener on `canvas` instead of `window`

**Solution:** Added explicit CRITICAL RULES to the system prompt:
- Use `Promise.all()` to wait for all images before calling `gameLoop()`
- Set `grounded=false` **before** the platform loop, not inside `else`
- Add keyboard listeners on `window`, not `canvas`

#### 5. Sprites Showing as Colored Boxes with Text Labels
**Problem:** FLUX image generation was silently failing and falling back to SVG placeholders. The fallback was hiding the real error.
**Solution:** Added visible error reporting to the status message so the actual API error could be diagnosed. Discovered the real issue was the wrong provider (`nscale` doesn't host FLUX) and depleted HF quota.

#### 6. Wrong Provider for Models
**Problem:** Multiple models claimed to be on inference providers but weren't:
- `Qwen/Qwen3-Coder-Next` — local only, no inference provider
- `baidu/ERNIE-Image` — on `fal-ai` provider
- `Tongyi-MAI/Z-Image-Turbo` — ZeroGPU only
- `FLUX.1-schnell` on nscale — not supported

**Solution:** Verified each model's actual provider through the HF API and community reports before coding.

#### 7. ZeroGPU Requires HF PRO
**Problem:** ZeroGPU (free on-demand GPU) requires HuggingFace PRO subscription ($9/month).
**Solution:** Avoided ZeroGPU entirely by using external free APIs (Groq + Pollinations).

#### 8. Duplicate Score/Lives Display
**Problem:** Generated game code was rendering UI elements twice.
**Solution:** Added explicit instruction in system prompt to draw UI only once per frame.

---

### Models Explored (and Why Rejected)

| Model | Reason Rejected |
|---|---|
| `Qwen/Qwen3-Coder-Next` | No inference provider, local only |
| `baidu/ERNIE-Image` | fal-ai provider depleted quota |
| `Tongyi-MAI/Z-Image-Turbo` | ZeroGPU only (needs HF PRO) |
| `unsloth/Z-Image-Turbo-GGUF` | Diffusers GGUF support not merged yet |
| `mradermacher/Janus-Pro-1B-GGUF` | GGUF only works for text, not image gen |
| `MaxedOut/ComfyUI-Starter-Packs` | Component files only, not a standalone model |
| `Phil2Sat/Qwen-Image-Edit-Rapid-AIO-GGUF` | ComfyUI only, needs custom nodes + VAE |
| `FLUX.1-schnell` via hf-inference | Monthly HF quota exhausted |
| `Google Imagen-3` | Requires Google Cloud project (setup issues) |

---

### Final Tech Stack

```
requirements.txt:
  openai       ← Groq API (OpenAI-compatible)
  requests     ← Pollinations.AI HTTP calls
  Pillow       ← Image resizing
  gradio       ← UI
```

**Total build time:** ~30 seconds (no compilation)
**Space RAM usage:** ~200MB (no local models)
**Cost:** $0

---

### Key Lessons Learned

1. **Always verify inference provider availability** before writing code — many HF models have no hosted endpoint
2. **Free quotas are monthly** — HF inference credits run out fast with image generation
3. **No-key APIs exist** — Pollinations.AI proves you don't always need an API key
4. **LLM-generated game code needs strict rules** — physics, keyboard input, and image loading all require explicit instructions in the system prompt or the model makes common bugs
5. **Error visibility matters** — silent fallbacks hide real problems; always surface errors to the user

---

### Current Status

✅ Space is running at `LeafCat79/Game_Builder_Free`
✅ Game code generation working (Groq)
✅ Image prompt enhancement working (Z-Image-Engineer style via Groq 70B)
✅ Sprite generation working (Pollinations.AI/FLUX)
✅ Games render in live iframe window
⚠️ Generated game physics still occasionally needs prompt refinement



