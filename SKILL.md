---
name: app-store-media-generator-skill
description: Generate and critique app store media assets for Apple App Store and Google Play. Use when the user asks for app screenshots, store listing images, app preview stills, Play feature graphics, or replacement store media across iPhone, iPad, Android phone, and Android tablet sizes.
---

# App Store Media Generator Skill

## Purpose

Use this skill to create upload-ready app store media from the current project's best available visual evidence. This is a prompt-only skill: do not rely on bundled scripts, helper generators, or a fixed image provider. Use whatever image generation, image editing, browser screenshot, desktop, or terminal tools are available in the current agent environment.

## Core Rule

Start from existing project visuals before generating anything new. The output should be separate image files, not collages, contact sheets, PDFs, or one combined preview.

## Workflow

### 1. Inventory Existing Visuals First

Scan the complete working repo folder recursively, including dot folders and nested output folders, for existing image assets:

```bash
find . -type f \( -iname '*.png' -o -iname '*.jpg' -o -iname '*.jpeg' -o -iname '*.webp' -o -iname '*.avif' \) -print
```

Also inspect likely external handoff paths if the user mentioned them, such as `Downloads`, `output`, `dist`, `screenshots`, `assets`, `public`, `store-assets`, `metadata`, `fastlane`, and Play/App Store metadata folders.

For each candidate asset set, check:

- Pixel dimensions and platform fit.
- Whether the screenshots are real app UI, generated mockups, or old upload thumbnails.
- Whether the image is readable at app-store thumbnail size.
- Whether the image is a standalone screenshot or a collage.
- Whether text is clipped, tiny, distorted, or duplicated.
- Whether the visual style is repetitive across the set.
- Whether the asset would plausibly pass Apple/Google review as representative of the app.

### 2. If Existing Visuals Are Missing Or Weak, Capture Live Product Evidence

If no usable app screenshots exist, or the existing ones are stale, first try the environment's browser, mobile, desktop, or screenshot tools against the real product. Prefer authenticated real app surfaces when credentials or session cookies are available.

If no browser screenshot tool is available, use Anchor Browser from a terminal. Do not hardcode API keys in repo files or published skills. Read the key from `ANCHOR_API_KEY`:

```bash
curl --request POST \
  --url https://api.anchorbrowser.io/v1/tools/screenshot \
  --header 'Content-Type: application/json' \
  --header "anchor-api-key: ${ANCHOR_API_KEY}" \
  --data '{"url":"https://oulang.ai","capture_full_height":true,"wait":2000}' \
  --max-time 90 \
  --output /tmp/screenshot.png \
  -w "HTTP %{http_code} Size %{size_download}\n" \
  -s && file /tmp/screenshot.png && imgcat /tmp/screenshot.png
```

If `ANCHOR_API_KEY` is missing, state that a key is required and continue with any local image-generation or browser tools that are available.

### 3. Critique Before Generation

Before generating replacements, write a direct critique of the current images. Keep it practical and visual:

- What worked previously.
- What did not work.
- Which images fail at thumbnail size.
- Which text is too small to read on iPhone.
- Which UI is too dense, warped, fake, cropped, or hidden.
- Which concepts should be preserved.
- Which concepts should be discarded.

Do not soften the critique. Store screenshots are conversion assets; unreadable images are failed images.

### 4. Generate With The Current Environment's Image Tools

Use the current agent's best available image generation or image editing tool. The skill is tool-agnostic:

- In Codex/OpenAI environments, use the native image generation tool if available.
- In ChatGPT desktop or browser environments, use that environment's image generation capability.
- In Manus, Claude, or other agent environments, use the local image-generation, browser, or design tool available there.
- If no direct image generation tool exists, use Desktop Commander or the available macOS/bash terminal tool to open or invoke an agent that does have image generation, such as Codex CLI, and pass the prompt there.

Do not write custom Python image generators unless the user explicitly asks for programmatic image composition. The default path is prompting an image-generation model or app.

### 5. Generation Prompt Requirements

Every image-generation prompt must enforce:

- One standalone image file per screenshot.
- No collages, contact sheets, grids, or multi-image preview boards.
- One clear feature concept per image.
- App-store upload intent stated explicitly.
- Target platform and exact pixel size stated explicitly.
- Large readable headline and subheadline.
- Minimal internal app text.
- At most one main device frame, unless the platform specifically benefits from tablet plus phone comparison.
- Mostly upright device angle; no extreme rotation.
- Actual app UI concepts, not generic fake app screens.
- No App Store Connect UI, browser chrome, watermarks, debug overlays, or upload-page thumbnails.
- No tiny listing tables or dense screenshots that become unreadable on iPhone.
- No duplicated phone panes, diagonal split seams, or decorative screen shards.

### 6. Recommended Store Asset Concepts

Generate a focused set before broad variants:

- `01-free-post`: publish rentals, jobs, second-hand, and services.
- `02-rent-search`: search rentals by city, budget, and room type.
- `03-jobs`: find Chinese-speaking jobs in Spain.
- `04-chat-ai`: AI helper or chat-assisted search.
- `05-contact`: direct WhatsApp/phone/contact flow.
- `06-content`: TV, videos, coupons, or local life content.

For each concept, the app screen should contain 3-5 large UI elements maximum. It is better to show two large cards than ten unreadable listings.

### 7. Platform Sizes

Prefer these target sizes unless the user requests different store slots:

- Apple iPhone 6.5 inch: `1242 x 2688`
- Apple iPhone 6.7 inch compatible: `1290 x 2796`
- Apple iPad Pro 12.9 inch: `2048 x 2732`
- Android phone: `1080 x 2400`
- Android 7 inch tablet: `1200 x 1920`
- Android 10 inch tablet: `1600 x 2560`
- Google Play feature graphic: `1024 x 500`

If the image generator cannot emit the exact size, generate the closest clean asset and then use a normal image-resizing tool only for final canvas scaling/cropping. Do not use resizing as a substitute for fixing bad composition.

### 8. Quality Gate

Reject and regenerate any image with:

- Small headline text.
- Small internal phone text that is central to the message.
- Cropped headline or phone UI.
- Misspelled brand name, wrong domain, or garbled Chinese.
- Overlapping text and UI.
- More than one unrelated feature concept.
- Fake UI that contradicts the real product.
- Collage layout.
- Repeated image template across the full set.
- Wrong aspect ratio for the target slot.

Thumbnail rule: shrink the image mentally to App Store Connect thumbnail size. If the headline and core feature are not instantly readable, the image fails.

### 9. Output Organization

Write generated assets to a dated folder in the current project when possible, for example:

```text
output/app-store-media-generated-YYYY-MM-DD/
  apple-iphone65/
  apple-ipad129/
  android-phone/
  android-tablet-7/
  android-tablet-10/
  play-feature-graphic/
  manifest.json
```

Keep original generated images in their provider output folder when the tool requires it, and copy renamed versions into the project output folder. Do not delete originals unless explicitly asked.

### 10. Final Response

Report:

- What existing images were found and why they passed or failed.
- What new files were generated.
- Which sizes are upload-ready.
- Which generated concepts should be regenerated, if any.
- Any blocker, such as missing image-generation tool or missing `ANCHOR_API_KEY`.

If the repo changed and the user expects shipping, commit and push only intentional files. Do not commit generated private API keys, temporary screenshots, or unrelated dirty work.
