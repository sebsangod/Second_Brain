---
aliases:
  - 03 ComfyUI
tags:
  - learning
  - dev/ai/visualization
date: 2026-05-10
---
**Sources**: [Source]()

**Related:** [[ComfyUI]], [[Artificial Intelligence]]

---

## Details

This session walks you through setting up an image upscaling workflow in ComfyUI. You’ll start with a basic text-to-image pipeline and add five key nodes: the SD upscale node needs to connect to the VAE decoder’s image output, the checkpoint model, the CLIP text input, and the upscaling model. Key settings include CFG=1 and denoise at 0.2—keep the sampler (e.g., heun) and scheduler (e.g., simple) consistent with your original generation. Aim for an output size around 1024x1024. The comparison node lets you side-by-side the original and upscaled results, and in practice, you’ll see major detail improvements (like sharper eyes). Once the workflow’s built, remember to fine-tune positive prompts for better results and use negative prompts to block unwanted artifacts. The whole process is straightforward—just wire the nodes correctly and tweak the settings for pro-level image enhancement.


### Upscale Images

In the world of AI-generated imagery, the first draft is rarely the final masterpiece.

You’ve crafted a stunning concept, your prompt is precise, and the image looks great — until you zoom in. Suddenly, details blur, textures soften, and the magic fades. That’s where **upscaling** comes in.

In this lesson, we explore how to transform good images into **gallery-ready visuals** using a powerful yet simple workflow in **ComfyUI** - **Upscaling**.

It’s not just about making an image bigger or improving its resolution. It’s about making it _better_. Sharper edges, richer textures, lifelike details. Upscaling breathes new life into AI-generated content, turning low-resolution outputs into high-fidelity assets ready for presentation, print, or production.


### Why Upscaling is Essential

AI image generation often starts in a compressed space - models create images efficiently at smaller sizes (like 512x512), then scale up. But simply stretching a small image leads to blur and artifacts.

True upscaling is **intelligent refinement**. It doesn’t just interpolate pixels - it _reimagines_ them. Using dedicated models and contextual understanding, it adds fine details like skin texture, fabric weave, or eye reflections - details that weren’t there before.

In ComfyUI, this process is fully customizable. You’re not relying on a one-click button. You’re guiding the AI with **precise prompts, models, and parameters** - ensuring the upscaled image stays true to your vision.


### Building the Upscaling Workflow

The workflow we’re using is compact but powerful. It begins with the **text-to-image workflow we created in the last lesson**, with a dedicated **upscaling stage**.

At the heart is the **SD High-Definition Upscaling** node, a specialized module designed to enhance resolution while preserving (or even improving) realism. It takes your base image and rebuilds it at a higher resolution (in our case, `1024x1024`).

For upscaling, your prompts shift focus.

Instead of describing _what_ the image should show, you describe _how it should look_.

✅ **Positive prompts** emphasize clarity:

"ultra-detailed, sharp focus, high-resolution, intricate textures, lifelike skin"

❌ **Negative prompts** filter out flaws:

"blurry, low-res, artifacts, distortion, oversaturated"

You can even label your nodes — like “Positive Prompt” or “Negative Prompt” — to keep your workspace clean and intuitive.

We also match the **sampler** (e.g., _Heun_) and **scheduler** (_Simple_) to the original workflow for consistency, and set the **denoise strength** to 0.2 — a sweet spot between refinement and faithfulness.

Too high, and the image changes too much. Too low, and nothing improves.


### See the Difference

Once the workflow runs, we use a simple but powerful tool: the **AB Images** node, which helps us make a side-by-side comparator.

Zoom in on details of the image. You'll notice the upscaling effects immediately. The upscaled version isn’t just larger - it’s **more _alive_, more detailed.**

This isn’t just plain scaling that we're used to. It’s **_enhancement_**.

It’s AI not just remembering what was there, but imagining what _should_ be.

___

## Resources

1. [Upscale checkpoint on Huggingface](https://huggingface.co/stablediffusionapi/perfectdeliberate-v5?spm=a3c0i.29909304.0.0.20e622c4x9KZVK)

---

## Claude Sessions
