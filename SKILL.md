---
name: comfyui-zimage
description: "Generate manga and anime illustrations via local ComfyUI API using Z-Image model with Hojo Tsukasa (80s manga) and Satoshi Urushihara (90s anime) LoRA styles. Supports character-consistent Riko portraits, text-to-image generation, and image upscaling. Use when the user asks to generate manga art, anime illustrations, character portraits, comic-style drawings, or artwork in Hojo/Satoshi style, or mentions Riko, ComfyUI, or Z-Image."
---

# Z-Image Image Generation System

Run all generation scripts directly with Bash. Do not use ToolSearch — all scripts are local.

> **Riko routing rule:** Any request mentioning "Riko" **must** use `generate_riko_*.py` (not `generate_manga_*.py`). Riko's fixed appearance and outfit definitions only exist in the Riko scripts. Style LoRAs can be layered on top via `--lora` and `--style_trigger` flags. See the decision table below.

## Step 1: Identify Route

| Priority | Keyword | Route | Script prefix |
|----------|---------|-------|---------------|
| **1 (Highest)** | Riko (even with a style) | **riko** | `generate_riko_*` |
| 2 | Hojo, Tsukasa Hojo | manga | `generate_manga_*` |
| 3 | Satoshi, Urushihara | manga | `generate_manga_*` |

### Resolution keyword guide

| User says | Meaning | Script suffix |
|-----------|---------|---------------|
| high quality, masterpiece, professional | Good prompt quality | `*_quick.py` |
| high-res, upscale, 4K, large image | High resolution output | `*_highres.py` or `upscale_*.py` |
| default (no specification) | Quick preview | `*_quick.py` |

### Prompt extraction for Riko requests

When extracting a prompt from a reference image for Riko generation, pass only scene-level elements to `--scene`:

- **Include:** actions, poses, emotions, expressions, scene/background, lighting/atmosphere, composition/angle
- **Discard:** hair color/style, skin tone, body type, facial features, clothing (unless user specifies `maid` instead of `school_uniform`) — Riko has fixed definitions for these in `data/riko_character.py`

## Step 2: Execute Generation

```
DIR=/path/to/your/comfyui-zimage-skill
```

### Riko route

```bash
# Pure Riko (realistic style)
python3 $DIR/generate_riko_quick.py \
  --outfit_type school_uniform \
  --scene "[scene description]"

# Riko + Hojo style (black and white manga-style Riko)
python3 $DIR/generate_riko_quick.py \
  --outfit_type school_uniform \
  --scene "[scene description]" \
  --lora "Z-image\\z-image-hojo.safetensors" \
  --style_trigger "Zanshou_kin_Hojo, monochrome, greyscale"

# Riko + Satoshi style (90s anime-style Riko)
python3 $DIR/generate_riko_quick.py \
  --outfit_type school_uniform \
  --scene "[scene description]" \
  --lora "Z-image\\z-image-satoshi.safetensors" \
  --style_trigger "sato, 90s anime style" \
  --style_tags "shiny skin, sparkle"
```

**Outfit options:** `school_uniform` (JK uniform) | `maid` (maid outfit)
**View options:** `--view_type full` (full body) | `--view_type upper` (upper body)

### Manga route (no Riko)

```bash
# Hojo style
python3 $DIR/generate_manga_quick.py \
  --lora "Z-image\\z-image-hojo.safetensors" \
  --prompt "[full prompt]"

# Satoshi style
python3 $DIR/generate_manga_quick.py \
  --lora "Z-image\\z-image-satoshi.safetensors" \
  --prompt "[full prompt]"
```

### High-resolution version

Replace `*_quick.py` with `*_highres.py`. Set `timeout=300000` for high-res generation.

## Step 3: Validate and Display Results

After running a generation script, check the exit code before displaying:

```bash
python3 $DIR/generate_riko_quick.py --outfit_type school_uniform --scene "standing by window"
if [ $? -ne 0 ]; then
  echo "Generation failed — check that ComfyUI is running and the model/LoRA paths are correct."
else
  # Display the generated image
  echo '<a href="[image_url]"><img src="[image_url]" width="350"></a>'
fi
```

Save the returned `image_url` for later upscaling.

## Step 4: Upscale Existing Images

When the user says "upscale" for an already generated image:

```bash
# Riko image upscale (no LoRA needed)
python3 $DIR/upscale_riko.py --image_url "[image_url]"

# Manga image upscale (needs original LoRA)
python3 $DIR/upscale_manga.py --image_url "[image_url]" --lora "[original LoRA]"
```

## Quick Decision Table

| User Request | Route | Script | LoRA |
|--------------|-------|--------|------|
| Draw Riko | riko | `generate_riko_quick.py` | None |
| Riko in Hojo style | riko | `generate_riko_quick.py` | hojo + style_trigger |
| Riko in Satoshi style | riko | `generate_riko_quick.py` | satoshi + style_trigger |
| Mecha girl in Hojo style | manga | `generate_manga_quick.py` | hojo |
| Cyberpunk in Satoshi style | manga | `generate_manga_quick.py` | satoshi |

## Notes

- **LoRA path uses double backslash** `\\` in the `--lora` argument
- **High-res and upscale require** `timeout=300000`
- **Style guides in `data/`** (`style_hojo.md`, `style_satoshi.md`) — consult for prompt crafting, do not delete
