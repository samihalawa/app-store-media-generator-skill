---
name: app-store-media-generator-skill
description: Generate, critique, source, and QA app store media assets for Apple App Store and Google Play from a URL, local screenshots, or existing repo images. Use when the user asks for app screenshots, store listing images, feature graphics, preview stills, or replacement store media across iPhone, iPad, Android phone, and Android tablet sizes.
---

# App Store Media Generator Skill

## Purpose

Use this skill to create a complete, premium, store-ready iOS and Android media library from the current project's best available visual evidence.

This is a prompt-only, tool-agnostic skill. Do not rely on bundled scripts, helper generators, fixed providers, or a specific image model. Use whatever image generation, image editing, browser screenshot, desktop, or terminal tools are available in the current agent environment.

## Core Contract

Start from existing product visuals before generating anything new. Treat real app UI as the locked source of truth. Use AI as a layout, captioning, adaptation, localization, and polishing layer over real UI captures; do not ask AI to invent the app from memory.

Output separate flattened PNG/JPEG image files, not collages, contact sheets, PDFs, combined previews, or App Store Connect upload-page thumbnails.

## Source Truth And Inventory

### 1. Inventory Existing Visuals First

Scan the complete working repo folder recursively, including dot folders and nested output folders, for image assets:

```bash
find . -type f \( -iname '*.png' -o -iname '*.jpg' -o -iname '*.jpeg' -o -iname '*.webp' -o -iname '*.avif' \) -print
```

Also inspect likely external handoff paths if the user mentioned them, such as `Downloads`, `output`, `dist`, `screenshots`, `assets`, `public`, `store-assets`, `metadata`, `fastlane`, and Play/App Store metadata folders.

For each candidate asset set, check:

- Pixel dimensions and platform fit.
- Whether the screenshot is real app UI, generated mockup, old upload thumbnail, splash screen, icon, game asset, or unrelated content.
- Whether the image is readable at store thumbnail size.
- Whether the image is a standalone screenshot or a collage/contact sheet.
- Whether text is clipped, tiny, distorted, duplicated, mistranslated, or garbled.
- Whether the visual style is repetitive across the set.
- Whether the asset would plausibly pass Apple/Google review as representative of actual app experience.

### 2. Capture Live Product Evidence If Needed

If no usable app screenshots exist, or existing ones are stale/weak, capture fresh product evidence from the URL or app. Prefer authenticated real app surfaces when credentials or session cookies are available.

Use the current environment's browser, mobile, desktop, or screenshot tools first. If no browser screenshot tool is available, use Anchor Browser from a terminal. Do not hardcode API keys in repo files or published skills. Read the key from `ANCHOR_API_KEY`:

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

If `ANCHOR_API_KEY` is missing, state that a key is required and continue with any local screenshot or image-generation tools that are available.

## Critique Before Generation

Before creating replacements, write a direct visual critique:

- What worked previously.
- What failed.
- Which images fail at thumbnail size.
- Which text is too small to read on iPhone.
- Which UI is too dense, warped, fake, cropped, hidden, duplicated, or old.
- Which screenshots should be preserved as source material.
- Which concepts should be discarded.

Do not soften the critique. Store screenshots are conversion assets; unreadable images are failed images.

## Generation Strategy

### Best Workflow

Do not batch-generate a full 40-asset library in one prompt. Use the staged workflow below:

1. Analyze the app from URL/screenshots. Identify category, audience, core promise, top benefits, main flows, brand colors, typography feel, tone, and strongest source screens. Do not generate images yet.
2. Build one visual system. Define background style, caption style, device/UI framing, spacing, safe margins, headline length, palette, shadow depth, and rules for adapting across iPhone, iPad, Android phone, and Android tablet.
3. Create the screenshot sequence. Plan up to 10 iOS assets and 6-8 Google Play assets with platform, device class, source screen, benefit, headline, layout notes, and compliance risks.
4. Generate the first 3 hero screenshots only. They must sell the core value instantly because those are the assets most likely to appear first in stores and search contexts.
5. Critique and fix the visual system.
6. Generate the remaining phone screenshots.
7. Generate iPad/tablet variants by adapting the same concept, not by stretching one phone image.
8. Generate localized variants only after the visual system passes.
9. Run final export QA and produce the manifest.

### Tool-Agnostic Image Generation

Use the current agent's best available image generation or image editing tool. If no direct image generation tool exists, use Desktop Commander or the available macOS/bash terminal tool to open or invoke an agent/environment that can generate images, and pass the final prompt there.

Do not write custom Python image generators unless the user explicitly asks for programmatic image composition. The default path is prompting an image-generation or image-editing environment.

### Master Prompt

Use this compact master prompt when handing off to an image generation/editing tool:

```text
Create a complete, premium, store-ready media library for iOS and Android from the supplied URL and/or screenshots.

Treat the actual app UI as the source of truth. Analyze the app category, audience, brand, tone, visual style, core flows, strongest benefits, and best source screens. Do not invent app features, fake screens, fake UI states, fake claims, rankings, awards, testimonials, pricing, discounts, or unsupported results.

Build one cohesive visual system across all assets: consistent typography, spacing, backgrounds, device/UI framing, shadows, lighting, caption placement, brand colors, safe margins, and image rhythm.

Generate iOS App Store screenshots with portrait-first iPhone assets and iPad variants. Generate Google Play screenshots with Android phone assets plus tablet/large-screen variants. Optional landscape variants only when the app genuinely supports landscape.

The first 3 images must explain the app's main value instantly. Each image must communicate one benefit only. Headlines must be short, clear, and benefit-led, max 5-8 words. Use real app UI screenshots, not imagined screens. Make UI readable at small sizes. Use premium composition but avoid clutter. Use platform-appropriate screenshots: iOS UI for App Store, Android UI for Google Play. Adapt the same concept to each platform rather than stretching one image into every size.

Compliance rules: no fake rankings such as "#1", "best", "top", or "most downloaded"; no "download now", "install now", or similar CTAs; no unsupported awards, testimonials, prices, discounts, or time-sensitive claims; no third-party logos, characters, or copyrighted material unless already present in the app with permission; no distorted UI, blurry text, stretched screenshots, transparent backgrounds, black bars, or cut-off captions. Do not modify actual app UI content except cropping, scaling, masking, or placing it inside the layout.

Output flattened PNG/JPEG assets with no transparency, correct platform-specific aspect ratios and export sizes, organized folders by platform/device/orientation/locale, and a QA checklist showing whether each asset passes size, readability, consistency, UI accuracy, and policy checks.
```

### Per-Image Prompt

Use this for each generated asset after the visual system is defined:

```text
Generate screenshot asset [NUMBER] for [PLATFORM] [DEVICE CLASS] at [WIDTH x HEIGHT].

Use the attached/source screen: [SOURCE SCREEN]. Keep the app UI accurate and unchanged. Place it in the approved visual system. Add one large, readable headline: "[HEADLINE]". Communicate one benefit: [BENEFIT].

Use premium spacing, safe margins, clear hierarchy, consistent background, consistent framing, and readable typography. Do not invent UI or features. Do not use tiny dense listing tables as the central message. Do not crop headline or UI. Export as one standalone flattened PNG/JPEG file, no transparency.
```

### Edit/Fix Prompt

Use this when a generated image is close but flawed:

```text
Refine this generated screenshot. Preserve the exact app UI, headline meaning, device class, and platform. Fix only presentation issues: alignment, spacing, safe margins, legibility, background consistency, device framing, shadows, cropping, and visual balance. Remove blur, distortion, artifacts, fake UI, unsupported claims, overlapping text, repeated template feel, and inconsistent styling.
```

### Localization Prompt

Use this after the base visual system passes:

```text
Create localized variants for [LANGUAGE/REGION]. Preserve the exact same layout, source UI accuracy, brand style, spacing, image order, and visual rhythm. Translate only headline/caption text. Keep each localized headline short, natural, and readable. Do not add new claims or change app functionality.
```

## Recommended Screenshot Sequence

### iOS: Up To 10 Images

The first 3 images must carry the product's core value.

1. Main value proposition: "Do [core outcome] faster"
2. Primary workflow: "Everything in one place"
3. Key differentiator: "Built for [audience]"
4. Feature 1: "Track [thing] instantly"
5. Feature 2: "Automate repetitive work"
6. Feature 3: "See progress clearly"
7. Trust/safety/control: "Your data stays organized"
8. Customization/personalization: "Make it fit your workflow"
9. Cross-device/team/advanced use: "Works wherever you do"
10. Summary/closing benefit: "Start with clarity"

### Google Play: 6-8 Images

Use Android-native source screenshots for Play when possible.

1. Main app promise: make the UI clear immediately.
2. Core workflow: show actual Android app experience.
3. Strongest benefit: avoid exaggerated marketing language.
4. Feature depth: keep taglines short.
5. Productivity/result: avoid small text.
6. Personalization: show real screens.
7. Trust/control: avoid unsupported claims.
8. Final summary: no "install now" CTA.

## Platform Sizes And Constraints

Store requirements change. If exact submission compliance matters, verify current official Apple App Store Connect and Google Play Console requirements before final upload. Use these as practical defaults:

- Apple iPhone 6.5 inch: `1242 x 2688`
- Apple iPhone 6.7 inch compatible: `1290 x 2796`
- Apple iPhone 6.9 inch options: `1260 x 2736`, `1290 x 2796`, `1320 x 2868`
- Apple iPad Pro 12.9 inch: `2048 x 2732`
- Apple 13 inch iPad options: `2048 x 2732`, `2064 x 2752`
- Android phone: `1080 x 1920`, `1080 x 2400`, or a valid Play portrait ratio
- Android 7 inch tablet: `1200 x 1920`
- Android 10 inch tablet: `1600 x 2560`
- Google Play feature graphic: `1024 x 500`

General Play constraints to enforce unless current official docs differ: JPEG or 24-bit PNG, no alpha/transparency, minimum dimension 320px, maximum dimension 3840px, and max dimension not more than 2x the minimum dimension.

If the image generator cannot emit exact size, generate the closest clean asset and then use normal image-resizing/cropping only for final canvas scaling. Do not use resizing as a substitute for fixing bad composition.

## Design And Compliance Rules

Must do:

- Use real UI as locked source.
- Use one visual system across all images.
- Put strongest value in the first 3 screenshots.
- Use short benefit headlines, not feature dumps.
- Keep headline and core feature readable at thumbnail size.
- Export separate platform variants.
- Use iOS-looking source UI for App Store and Android-looking source UI for Google Play when available.
- Keep safe margins for notches, rounded corners, captions, and store cropping.
- Use PNG for UI-heavy screenshots when possible.
- QA every image at 100% zoom and at thumbnail scale.

Must avoid:

- Fake screens.
- Fake rankings like "#1", "Best", "Top", or "Most downloaded".
- Fake awards, testimonials, metrics, reviews, or unsupported claims.
- "Download now", "Install now", or similar CTAs.
- Pricing, discounts, or time-sensitive claims that may become outdated.
- Distorted, blurred, stretched, compressed, or redrawn UI.
- Tiny internal app text as the core message.
- Overloaded captions or taglines covering too much of the image.
- Inconsistent device frames.
- iPhone screenshots on Google Play when Android screenshots are available.
- Android screenshots on the App Store when iOS screenshots are available.
- Generic AI-looking gradients that do not match the app brand.
- Third-party logos, characters, or copyrighted material unless present in the actual app with permission.

## Quality Gate

Reject and regenerate any image with:

- Small headline text.
- Small internal phone text that is central to the message.
- Cropped headline or phone UI.
- Misspelled brand name, wrong domain, or garbled text.
- Overlapping text and UI.
- More than one unrelated feature concept.
- Fake UI that contradicts the real product.
- Collage layout.
- Repeated image template across the full set.
- Wrong aspect ratio for the target slot.
- Transparency, black bars, or visible browser/App Store Connect chrome.

Thumbnail rule: shrink the image mentally to store thumbnail size. If the headline and core benefit are not instantly readable, the image fails.

## Output Organization

Write generated assets to a dated folder in the current project when possible:

```text
output/app-store-media-generated-YYYY-MM-DD/
  apple-iphone/
  apple-ipad/
  android-phone/
  android-tablet-7/
  android-tablet-10/
  play-feature-graphic/
  localized/
  manifest.json
```

Keep original generated images in their provider output folder when the tool requires it, and copy renamed versions into the project output folder. Do not delete originals unless explicitly asked.

## Manifest Template

Return a manifest like:

```json
[
  {
    "filename": "ios_iphone_01_main_value_1320x2868.png",
    "platform": "iOS App Store",
    "device_class": "iPhone 6.9",
    "orientation": "portrait",
    "size_px": "1320x2868",
    "headline": "...",
    "source_screen": "...",
    "purpose": "Main value proposition",
    "status": "ready"
  },
  {
    "filename": "android_phone_01_main_value_1080x1920.png",
    "platform": "Google Play",
    "device_class": "Android phone",
    "orientation": "portrait",
    "size_px": "1080x1920",
    "headline": "...",
    "source_screen": "...",
    "purpose": "Main value proposition",
    "status": "needs review"
  }
]
```

## Final Response

Report:

- What existing images were found and why they passed or failed.
- What new files were generated.
- Which sizes are upload-ready.
- Which generated concepts should be regenerated, if any.
- QA checklist result for size, readability, consistency, UI accuracy, transparency, stretching, and policy risk.
- Any blocker, such as missing image-generation tool or missing `ANCHOR_API_KEY`.

If the repo changed and the user expects shipping, commit and push only intentional files. Do not commit generated private API keys, temporary screenshots, or unrelated dirty work.
