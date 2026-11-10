---
name: gavana-reference-images
description: Create a new Gavana image from one or more supplied image references without changing the sources. Use when a user asks Codex to make, combine, adapt, or generate an image from local image paths, the macOS clipboard, or existing Gavana assets/nodes and needs the result placed in a Gavana canvas.
---

# Gavana Reference Images

Use `gavana` as the source of truth for canvas state and image jobs.
Create a new output image; never overwrite a supplied reference unless the user explicitly asks to edit that source.

## Workflow

1. Confirm the target is a new image and identify the desired canvas. If the user names a canvas, run `canvas list` and resolve one exact match. Ask when there are zero or multiple matches; do not guess.
2. Run `canvas get` before changing the canvas. Use an existing empty image node when the user names one. Otherwise create an image node with a clear title. When a named Section is provided, place the new node inside that Section's bounds.
3. Accept references in the order supplied. Use `node:<id>` or `asset:<id>` for existing Gavana images, a local PNG/JPEG/WebP/GIF path for a file, or `clipboard` for a macOS clipboard image. Do not ask the user for asset or node IDs.
4. State the role of each reference in the prompt. If the user has not supplied roles, use the neutral default “combine both references” and say that you will create a new image from them. Ask a short question only when product identity versus style direction would materially change the result.
5. Run `image edit` with one `--reference` per source. The CLI privately uploads local and clipboard inputs first and replaces them with stable asset handles.
   To change one existing image instead of combining several, pass that image as `--source` (Gavana sends it to the model first, as Image 1) and keep `--reference` for extra guidance; `--change-region x,y,width,height` (repeatable, 0-1 fractions, needs `--source`) names the only areas allowed to change.
6. Wait for the durable result unless the user asks to queue it only. Report the output asset, target node, preview, and canvas review link returned by Gavana.

```sh
gavana image edit canvas:OWNER_UID:CANVAS_ID \
  --target node:OUTPUT_NODE_ID \
  --reference ~/Desktop/product.png \
  --reference clipboard \
  --prompt "Use reference 1 for the exact product. Use reference 2 for lighting and composition." \
  --output markdown
```

## Guardrails

- Support at most 16 references.
- Treat local paths and clipboard data as private uploads; do not expose raw storage URLs or credentials.
- Keep reference order meaningful and describe it in the prompt.
- If any local reference is invalid, stop before creating an image job.
- Return the result location; do not claim that a job has completed until Gavana returns a durable result.
