---
name: create-creative-batch
description: >-
  Use whenever the user wants to work with Kalpana: browse or find design templates, turn a list or spreadsheet into personalized images (one design per row), pick images from their asset library, or check on batches and download finished designs. Covers building renderer-ready rows, validating them, creating a batch, running it with credit confirmation, and handling access errors.
---

# Kalpana: templates to personalized designs

Kalpana turns one template into many designs: each **row** of a **batch** fills the template's inputs (text, images) and renders one image. You fill templates; you never edit designs.

This connection renders **still images only** (PNG, JPEG, WebP). Animated templates are not listed, and a video request cannot be fulfilled here: say so and suggest rendering it in the Kalpana app. Layout, font and design changes also happen in the Kalpana app, not here.

## Ground rules

- **Only connected workspaces are visible.** The user chose them when connecting. If they name a workspace that `list_workspaces` doesn't return, or a tool says a workspace is not connected, tell them to reconnect Kalpana and select it. Don't retry or work around it.
- **Their role limits what you can do.** If a tool says they lack permission, explain that their role in that workspace doesn't allow it (e.g. a Viewer can't create batches).
- **Confirm before changing anything.** Get a yes right before `create_batch`. Get a separate yes right before `run_batch`, and state the exact `creditCost` in that request.
- Never guess IDs. Use the IDs tools return.
- Download links expire. Don't present them as permanent.

## 1. Find the template

- "Show my templates" → `search_templates` with no `query` (most recently updated first).
- "Something for Diwali sale" → `search_templates` with `query`. Add `width`/`height` for a size ("Instagram story" = 1080×1920) and `workspaceId` when the user named one.
- Show a few options by name with their `previewUrl`, and their `openUrl` when the user wants to open one in Kalpana. To judge fit yourself, call `inspect_image` on the preview. If more than one is plausible, ask; don't pick silently.

## 2. Learn its inputs

Call `get_template_inputs`. If it reports that the template is animated, tell the user and share the link it gives; don't look for a workaround. Each input has an `id`, a `label`, a `type` (`text` or `image`), an optional `defaultValue`, and image `dimensions`.

Rows are keyed by **input `id`**, not label. Include only the inputs you're changing; anything left out keeps the template default.

```json
{
  "headline": { "node_type": "text", "spans": [{ "span_index": 0, "value": "Flat 40% off" }] },
  "product_photo": { "node_type": "image", "asset_id": "0f9c…-uuid" }
}
```

- Text: put the whole string in span 0 unless the input's `valueExample` shows more spans. Keep the user's exact wording, spelling, currency and locale.
- Image: `asset_id` must be an asset from the **same workspace** as the template (see step 3).

### Framing images

An image input fills a fixed slot (`dimensions`). You can control how the picture sits in it, just like the batch builder:

```json
{ "node_type": "image", "asset_id": "0f9c…-uuid", "fit": "Fill", "adjust": { "scale": 1.3, "offset_x": 0, "offset_y": -0.1 } }
```

- `fit`: `Fill` covers the slot and crops the edges (default, best for photos). `Fit` shows the whole image, leaving empty space (best for logos and product cut-outs). `Stretch` distorts the image; avoid it unless asked.
- `adjust.scale` (0.2–4, default 1) zooms about the slot centre. `offset_x`/`offset_y` (−1 to 1) move the image by a fraction of the slot; positive moves it right or down.
- Leave out `asset_id` to keep the template's own image and only reframe it.
- When you don't have a reason to reframe, send just `asset_id` and let the template's fit apply (`get_template_inputs` shows it as `fit`).
- Frame deliberately: compare the image's shape with the slot's `dimensions`, and after rendering check an output with `inspect_image`. If a face or product is cut off, adjust and create a corrected batch rather than guessing up front for every row.

## 3. Resolve images (only if an image input changes)

- "Use the summer collection" → `list_asset_collections`, then `search_assets` with `collectionId`.
- "The red sneaker photo" → `search_assets` with `query` (matches filenames).
- "Show me my images" → `search_assets` with no filter (most recent first).
- Confirm your picks by showing their previews or checking them with `inspect_image`, especially against the input's `dimensions`. A tall photo in a wide slot will crop; use `fit`/`adjust` (above) to fix it.
- There is no upload tool. If an image isn't in Kalpana, ask the user to upload it in the Kalpana app, then search again.

## 4. Build rows from the user's data

- A pasted list, table or CSV becomes one row per entry. Map each column to an input by meaning (label and description), and state the mapping back briefly: "Name → headline, Price → price_tag".
- Ask about columns that match no input, and about inputs the user may want filled but gave no data for. Never invent values.
- Maximum 500 rows per batch. For more, split into batches and say so.

## 5. Validate, create, run

1. `validate_batch` with the rows. Fix the reported `issues` (by `rowIndex` and `field`), or ask the user for missing values. Repeat until valid.
2. Summarize: template, workspace, row count, what changes versus stays default, and output format (png default; jpeg/webp on request). Get confirmation, then call `create_batch` with a clear `name`. This makes a draft; nothing renders and no credits are spent.
3. Tell the user the batch was created and its `creditCost`. Ask whether to start rendering. Only after an explicit yes, call `run_batch`.
   - Not enough credits: say so plainly and suggest topping up in Kalpana. The draft stays available to run later.

## 6. Status and results

- "How's my batch?" → `get_batch`. Report completed / failed / processing out of the total. Don't poll in a tight loop; check again when the user asks, or at most a few times with pauses.
- "What have I run lately?" → `list_batches` (filter by `workspaceId` or `templateId` if given).
- "Show me the results" → `get_batch_outputs` (paged, up to 20). Share the download links, and use `inspect_image` on one or two if the user wants you to review them. Report failed rows rather than hiding them.

## Privacy

Never ask for or reveal access tokens, scene JSON, or other people's batch data. If something isn't found, it may not exist or may be outside what this connection can see. Say that; don't try to get around it.
