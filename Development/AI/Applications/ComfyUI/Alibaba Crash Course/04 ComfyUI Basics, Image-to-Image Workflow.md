---
aliases:
  - 04 ComfyUI
tags:
  - learning
  - dev/ai/visualization
date: 2026-05-10
---
**Sources**: [Source]()

**Related:** [[ComfyUI]], [[Artificial Intelligence]]

---

## Details

This lesson introduces how to create and run a ComfyUI image-to-image workflow. The workflow is built by modifying a text-to-image workflow. This workflow adds an image loading node to load the base image, a VAE encoding node to convert latent spaces, and a image scaling node to generate an image in the desired size, such as 720 x 1280 in portrait mode. This lesson also looks at some new parameters that help control the "creativity" of the model. One such parameter is the noise reduction parameter, which can be set to a value between 0 and 1. At 0, the style of the base image is retained, while at 1, the generated image is completely stylized. For best results, a parameter value between 0.65 and 0.8 is recommended, achieving a balance between the base image and the effect given by the prompt (such as "3D CG"). The lesson demonstrates the entire process from creating the workflow and modifying the noise reduction parameter to control the style of the generated image.

### Image-to-Image Workflow

What if you could take a simple portrait - a real person, a real moment - and turn it into a futuristic, 3D-rendered character, without losing the soul of the original?

In this essential lesson, we explore how to do exactly that using an **image-to-image workflow in ComfyUI**, a powerful technique that blends the familiar with the imaginative.

Unlike pure text-to-image generation, image-to-image doesn’t start from nothing. It starts from **something real** - a photo, a sketch, a concept, and uses AI to **_reinterpret_** it. The pose, expression, and composition remain grounded, while style, texture, and form are transformed.

Think of it as **creative evolution** - same subject, new reality.

### How Image-to-Image Works

At its core, this workflow builds on the familiar text-to-image pipeline - but with a crucial twist: instead of starting from random noise, we begin with a real image.

We use three key nodes to make this happen:

- **Load Image:** Upload your base photo (in this case, a portrait of a person).
- **Image Resize:** Adjust the aspect ratio (e.g., from 1:1 to 9:16) and crop for your target format.
- **VAE Encode:** Convert the pixel-based image into latent space - the language your AI model understands.

Once encoded, the image becomes a foundation, not a final product. It’s fed into the model alongside your prompt, guiding the AI to preserve structure while embracing new styles.

The magic happens in the balance, _between input image and text prompt._


### The Power of Denoise Strength

The **denoise strength** parameter is the heart many image and video generation models.

- Set it to `0`, and nothing changes, the output is identical to the input.
- Set it to `1`, and the AI ignores the image entirely, generating freely from the prompt.

It's a tug-of-war of balance, and your job is to find the perfect balance wherethe AI _listens_ to both the image and the prompt.

It needs to keep the subject’s pose, expression, and lighting, while transforming their appearance into a **3D-rendered, CG-style humanoid**, complete with metallic textures, glowing details, and digital depth. And because the structure stays consistent, the result feels intentional, not random.


### Why This Matters in AI Design

Image-to-image is more than a technical trick - it’s a **creative bridge**.

It allows you to:

- Turn real-world references into stylized concepts
- Maintain consistency across character designs
- Rapidly prototype visual directions without starting from scratch
- Preserve emotional authenticity while exploring futuristic aesthetics

Whether you're designing characters for games, concept art for film, or digital avatars for branding, this workflow gives you precision, control, and creative flexibility. And all this happens within an open, customizable environment, which is ComfyUI.

---

## Claude Sessions
