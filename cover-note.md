# Cover Note — Bobby

This packet is a **starter draft** of your paper, built from your `paper-starter.md`. It is not a model answer and it is not the final version. Your job over the next four sessions is to make it yours. Three moves do that.

---

## A note before you read the packet

The parallel-example was written around an imagined "Quest Seed Usability Lab." That's a placeholder. **You have a real Space — `Game_Builder_Free` — that you've been iterating on this week (three commits in 24 hours as of April 24).** That's the actual anchor for your paper.

Read the packet for *structure* — the journal-to-paper move, the way the rubric narrows from "is the writing good?" to "does the output reduce design work?", the cautious finding sentence. Then we swap the imagined Space for your real one. That swap is most of the work.

---

## Anchor

The paper's central claim works for `Game_Builder_Free` with very little adjustment. Both Spaces are about evaluating whether AI tools help with game development, and both can be tested with realistic project briefs instead of generic prompts.

Three steps, doable in one or two sessions:

- Pick **two or three realistic project briefs** for `Game_Builder_Free`. Not "make me a game," but "I'm building a 2D horror game where the player is a delivery driver — generate three quest seeds with one non-combat mechanic each, one map location each, and one object that changes meaning by the end." Use the kind of structured prompt the parallel-example uses, but tuned to what your Space actually does.
- Run them. Save the outputs.
- Replace the example outputs in `PAPER.md` section 4 with two or three of your real ones — the ones that show the *fluent-but-not-design-useful* gap most clearly. Or, if your Space *did* produce design-useful output, the paper's finding can pivot toward that ("here's when this kind of tool actually helps a developer move from blank page to prototype").

The paper title and the imagined Space name (`Quest Seed Usability Lab`) should be replaced throughout with `Game_Builder_Free`. We can do this together — it's a global find-and-replace plus a few paragraph rewrites.

The "developer usefulness rubric" in section 3 (Constraint fit, Playable mechanic, Narrative hook, Specificity, Rewrite burden) is solid. Keep it. It's a real evaluation framework.

## Voice

Three paragraphs to rewrite in our working session:

1. **Section 1 introduction.** Your real story is interesting and not in the draft: you have a game-development background (Brackeys Game Jam, "Chickens-nightmare" project), and you came into this course curious about whether AI tools help with the *edges* of game development — concept art, website copy, early worldbuilding — without replacing the design work. Say that. The introduction should sound like a developer testing tools, not a generic study of AI quest generation.
2. **Section 3 method.** Explain *why* you're using realistic project briefs in your own words. Your earlier journal work already shows this instinct (you tested Qwen3-Coder-WebDev for a real game website concept; you tested HunYuan-Motion1.0 with creative game-relevant prompts). Bring that voice in.
3. **Section 5 limitations.** Your honest limitations: you're the only evaluator, your Space is one tool not a comparison across many, and you spent some weeks not actively iterating before picking it back up. Name what you actually couldn't do.

Tell me each one out loud first, then we rewrite together.

## Session 7 Journal

**Date:** May 2, 2026
**Project:** Text-to-Game Generator (HuggingFace Space)

---

## Overview

This session was a long and iterative journey building a fully working **AI-powered HTML5 game generator** hosted on HuggingFace Spaces. The app takes a theme and game genre, generates playable game code, and generates sprite images — all for free.

---

## What We Built

A HuggingFace Space (`LeafCat79/Game_Builder_Free`) that:
- Takes a **theme** (e.g. "Jungle temple with ancient traps") and **game genre** as input
- Uses an LLM to **generate complete, playable HTML5 canvas games**
- Uses an image AI pipeline to **generate themed sprites** (player, background, enemy)
- **Embeds sprites as base64** directly in the HTML so the game is fully self-contained
- Renders the game live in an **iframe** inside the Gradio UI

---

## Architecture (Final)

| Component | Model | Provider |
|---|---|---|
| Game code generation | `llama-3.1-8b-instant` | Groq API (free) |
| Image prompt enhancement | `llama-3.3-70b-versatile` acting as Z-Image-Engineer V4 | Groq API (free) |
| Sprite image generation | FLUX via Pollinations.AI | Pollinations (free, no key) |

---

## Problems We Encountered & How We Solved Them

### 1. Memory Limit Exceeded (OOM — 16GB)
**Problem:** Loading large models locally (Qwen3-Coder-Next ~140GB, ERNIE-Image ~32GB) crashed the Space container.
**Solution:** Switched to API-based inference — models run on external servers, Space uses ~200MB RAM.

### 2. Build Timeout (Exit Code 143)
**Problem:** `llama-cpp-python` compiles from C++ source during Docker build, taking 20+ minutes and exceeding HF's build timeout limit.
**Solution:** Removed `llama-cpp-python` entirely, switched to API calls which need no compilation.

### 3. HF Inference API Quota Exhausted (402 Error)
**Problem:** Free monthly HF credits depleted — every image call returned `402 Payment Required`.
**Solution:** Tried multiple providers:
- nscale → didn't host FLUX.1-schnell
- hf-inference → quota depleted
- Google Imagen-3 → needed Google Cloud project setup (unavailable)
- **Pollinations.AI** → completely free, no API key, no signup ✅

### 4. Game Not Working (Black Screen / Player Falls)
**Problem:** Three bugs in AI-generated game code:
1. Black screen — images not loaded before game loop started
2. Player falls into void — `grounded=false` set inside `else` of platform loop
3. Keyboard not working — `keydown` listener on `canvas` instead of `window`

**Solution:** Added explicit CRITICAL RULES to the system prompt:
- Use `Promise.all()` to wait for all images before calling `gameLoop()`
- Set `grounded=false` **before** the platform loop, not inside `else`
- Add keyboard listeners on `window`, not `canvas`

### 5. Sprites Showing as Colored Boxes with Text Labels
**Problem:** FLUX image generation was silently failing and falling back to SVG placeholders. The fallback was hiding the real error.
**Solution:** Added visible error reporting to the status message so the actual API error could be diagnosed. Discovered the real issue was the wrong provider (`nscale` doesn't host FLUX) and depleted HF quota.

### 6. Wrong Provider for Models
**Problem:** Multiple models claimed to be on inference providers but weren't:
- `Qwen/Qwen3-Coder-Next` — local only, no inference provider
- `baidu/ERNIE-Image` — on `fal-ai` provider
- `Tongyi-MAI/Z-Image-Turbo` — ZeroGPU only
- `FLUX.1-schnell` on nscale — not supported

**Solution:** Verified each model's actual provider through the HF API and community reports before coding.

### 7. ZeroGPU Requires HF PRO
**Problem:** ZeroGPU (free on-demand GPU) requires HuggingFace PRO subscription ($9/month).
**Solution:** Avoided ZeroGPU entirely by using external free APIs (Groq + Pollinations).

### 8. Duplicate Score/Lives Display
**Problem:** Generated game code was rendering UI elements twice.
**Solution:** Added explicit instruction in system prompt to draw UI only once per frame.

---

## Models Explored (and Why Rejected)

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

## Final Tech Stack

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

## Key Lessons Learned

1. **Always verify inference provider availability** before writing code — many HF models have no hosted endpoint
2. **Free quotas are monthly** — HF inference credits run out fast with image generation
3. **No-key APIs exist** — Pollinations.AI proves you don't always need an API key
4. **LLM-generated game code needs strict rules** — physics, keyboard input, and image loading all require explicit instructions in the system prompt or the model makes common bugs
5. **Error visibility matters** — silent fallbacks hide real problems; always surface errors to the user

---

## Current Status

✅ Space is running at `LeafCat79/Game_Builder_Free`
✅ Game code generation working (Groq)
✅ Image prompt enhancement working (Z-Image-Engineer style via Groq 70B)
✅ Sprite generation working (Pollinations.AI/FLUX)
✅ Games render in live iframe window
⚠️ Generated game physics still occasionally needs prompt refinement

## Stretch

You're a stronger candidate for stretch than I was treating you as before, given the recent activity. One real extension:

You have `Game_Builder_Pro` and `Game_Builder_Free` as two separate Spaces. Run **the same brief through both**. Compare what the paid/pro version produces vs. the free one. That's a real comparison — it answers "does paying for the better model actually help with this kind of game-dev task?" — and it's the kind of finding that reads well in a portfolio. One paragraph in section 4 reporting what you found.

Skip if no time. Anchor + Voice is the priority.

---

## Two housekeeping nudges

**Re-entry note.** You went quiet for several weeks before picking up `Game_Builder_Free` again. For the journal, write a short Week 7 entry that names the gap and what brought you back. That kind of honesty reads well in a portfolio — projects with re-entry stories are real projects. The parallel-example doesn't have this because it's imagined; yours will.

**Source verification.** The candidate references in `PAPER.md` are abstract-checked only, via Consensus. Verify each one: title, authors, venue, what the paper actually claims. The procedural-content-generation literature is real, but the specific papers in the draft are placeholders.

---

AI + Research Level 2 • Paper Phase
