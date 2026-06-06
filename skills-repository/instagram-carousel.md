---
name: instagram-carousel-design
description: "Build, design, and export Instagram carousels as publish-ready 1080x1350px PNG slides. Use this skill whenever someone asks to build, design, create, or export an Instagram carousel, generate carousel slides, create swipeable Instagram posts, or produce individual slide images for Instagram. Also trigger when the user has carousel copy ready and wants it turned into a visual carousel. Always use this skill — not just general coding — whenever the output is an Instagram carousel."
---

> BEFORE PROCEEDING: This file may be truncated during reading. Always verify
> you have read ALL sections by checking if the view output shows a truncation
> warning. If truncated, explicitly re-read the missing line range before
> taking any action.

# Instagram Carousel Design System

You are an Instagram carousel design system. When a user asks you to create a carousel, generate a fully self-contained, swipeable HTML carousel where every slide is designed to be exported as an individual image for Instagram posting.

---

## Step 1: Collect Brand Details

Before generating any carousel, ask the user for the following (if not already provided):

1. Brand name — displayed on the first and last slides
2. Instagram handle — shown in the IG frame header and caption
3. Primary brand color — the main accent color (hex code, or describe it and you'll pick one)
4. Logo — ask if they have an SVG path, want to use their brand initial, or skip the logo
5. Font preference — ask if they want serif headings + sans body (editorial feel), all sans-serif (modern/clean), or have specific Google Fonts in mind
6. Tone — professional, casual, playful, bold, minimal, etc.
7. Images — ask for any images to be included into the carousel (profile photo, screenshots, product images, etc.)

If the user provides a website URL or brand assets, derive the colors and style from those.
If the user just says "make me a carousel about X" without brand details, ask before generating. Don't assume defaults.

---

## Step 2: Derive the Full Color System

From the user's single primary brand color, generate the full 6-token palette:

```
BRAND_PRIMARY   = {user's color}                    // Main accent — progress bar, icons, tags
BRAND_LIGHT     = {primary lightened ~20%}           // Secondary accent — tags on dark, pills
BRAND_DARK      = {primary darkened ~30%}            // CTA text, gradient anchor
LIGHT_BG        = {warm or cool off-white}           // Light slide background (never pure #fff)
LIGHT_BORDER    = {slightly darker than LIGHT_BG}    // Dividers on light slides
DARK_BG         = {near-black with brand tint}       // Dark slide background
```

Rules for deriving colors:
- LIGHT_BG should be a tinted off-white that complements the primary (warm primary → warm cream, cool primary → cool gray-white)
- DARK_BG should be near-black with a subtle tint matching the brand temperature (warm → #1A1918, cool → #0F172A)
- LIGHT_BORDER is always ~1 shade darker than LIGHT_BG
- The brand gradient used on gradient slides is: `linear-gradient(165deg, BRAND_DARK 0%, BRAND_PRIMARY 50%, BRAND_LIGHT 100%)`

---

## Step 3: Set Up Typography

Based on the user's font preference, pick a heading font and body font from Google Fonts.

Suggested pairings:
```
Editorial / premium:   Playfair Display + DM Sans
Modern / clean:        Plus Jakarta Sans (700) + Plus Jakarta Sans (400)
Warm / approachable:   Lora + Nunito Sans
Technical / sharp:     Space Grotesk + Space Grotesk
Bold / expressive:     Fraunces + Outfit
Classic / trustworthy: Libre Baskerville + Work Sans
Rounded / friendly:    Bricolage Grotesque + Bricolage Grotesque
```

Font size scale (fixed across all brands):
- Headings: 28–34px, weight 600, letter-spacing -0.3 to -0.5px, line-height 1.1–1.15
- Body: 14px, weight 400, line-height 1.5–1.55
- Tags/labels: 10px, weight 600, letter-spacing 2px, uppercase
- Step numbers: heading font, 26px, weight 300
- Small text: 11–12px

Apply via CSS classes `.serif` (heading font) and `.sans` (body font) throughout all slides.

---

## Slide Architecture

**Format:**
- Aspect ratio: 4:5 (Instagram carousel standard)
- Each slide is self-contained — all UI elements are baked into the image
- Alternate LIGHT_BG and DARK_BG backgrounds for visual rhythm

### Required Elements Embedded In Every Slide

**1. PROGRESS BAR (bottom of every slide)**

Shows the user where they are in the carousel. Fills up as they swipe.
- Position: absolute bottom, full width, 28px horizontal padding, 20px bottom padding
- Track: 3px height, rounded corners
- Fill width: `((slideIndex + 1) / totalSlides) * 100%`
- Adapts to slide background:
  - Light slides: `rgba(0,0,0,0.08)` track, BRAND_PRIMARY fill, `rgba(0,0,0,0.3)` counter
  - Dark slides: `rgba(255,255,255,0.12)` track, `#fff` fill, `rgba(255,255,255,0.4)` counter
- Counter label beside the bar: "1/7" format, 11px, weight 500

```javascript
function progressBar(index, total, isLightSlide) {
  const pct = ((index + 1) / total) * 100;
  const trackColor = isLightSlide ? 'rgba(0,0,0,0.08)' : 'rgba(255,255,255,0.12)';
  const fillColor = isLightSlide ? BRAND_PRIMARY : '#fff';
  const labelColor = isLightSlide ? 'rgba(0,0,0,0.3)' : 'rgba(255,255,255,0.4)';
  return `<div style="position:absolute;bottom:0;left:0;right:0;padding:16px 28px 20px;">
    <div style="display:flex;align-items:center;gap:10px;">
      <div style="flex:1;height:3px;background:${trackColor};border-radius:2px;">
        <div style="height:100%;width:${pct}%;background:${fillColor};border-radius:2px;"></div>
      </div>
      <span style="font-size:11px;color:${labelColor};">${index+1}/${total}</span>
    </div>
  </div>`;
}
```

**2. SWIPE ARROW (right edge — every slide EXCEPT the last)**

A subtle chevron on the right edge telling the user to keep swiping.
- Position: absolute right, full height, 48px wide
- Background: gradient fade from transparent → subtle tint
- Chevron: 24x24 SVG, rounded strokes
- Adapts to slide background:
  - Light slides: `rgba(0,0,0,0.06)` bg, `rgba(0,0,0,0.25)` stroke
  - Dark slides: `rgba(255,255,255,0.08)` bg, `rgba(255,255,255,0.35)` stroke
- On the LAST slide, the swipe arrow is REMOVED entirely

---

## Standard Slide Sequence

Follow this narrative arc. 7 slides is ideal (range: 5–10).

```
Slide 1: Hero       | LIGHT_BG        | Hook — bold statement, logo lockup, optional watermark
Slide 2: Problem    | DARK_BG         | Pain point — what's broken, frustrating, or outdated
Slide 3: Solution   | Brand gradient  | The answer — what solves it, optional quote/prompt box
Slide 4: Features   | LIGHT_BG        | What you get — feature list with icons
Slide 5: Details    | DARK_BG         | Depth — customization, specs, differentiators
Slide 6: How-to     | LIGHT_BG        | Steps — numbered workflow or process
Slide 7: CTA        | Brand gradient  | Call to action — logo, tagline, CTA button. No arrow. Full progress bar.
```

Rules:
- Start with a hook — the first slide must stop the scroll
- End with a CTA on brand gradient — no swipe arrow, progress bar at 100%
- Alternate light and dark backgrounds for visual rhythm
- Adapt the sequence to the topic as needed

---

## Layout Best Practices

- Content padding: `0 36px` standard
- Bottom-aligned slides with progress bar: `0 36px 52px` to clear the bar
- Content must NEVER overlap the progress bar — use `padding-bottom: 52px`

**Vertical alignment — choose per slide, not per slide type:**

Use `justify-content: center` for:
- Hero slides (S1)
- CTA/final slides
- Gradient stat slides (large number + short body)
- Problem/setup slides with light content (tag + headline + pills + short body)
- Logic/insight slides with a quote box + short body

Use `justify-content: flex-end` for:
- Rate card tables (6+ rows)
- Numbered insight lists (4+ items)
- Feature lists (4+ rows)
- Any slide where content fills more than 60% of the slide height naturally

**Rule of thumb:** If you can see a lot of empty space at the top when using `flex-end`, switch to `center`. Never let the top half of a slide sit empty — it reads as broken layout, not intentional breathing room.

---

## Reusable Components

**RATE CARD ROWS (name + amount + unit):**
```html
<div style="display:flex;align-items:center;justify-content:space-between;padding:9px 0;border-bottom:1px solid {LIGHT_BORDER};">
  <span style="font-size:13px;font-weight:600;color:#0A0F1E;">{Creator name}</span>
  <div style="min-width:120px;flex-shrink:0;text-align:right;">
    <span style="font-size:13px;font-weight:700;color:{BRAND_PRIMARY};display:block;">{Amount}</span>
    <span style="font-size:10.5px;font-weight:400;color:#8A8580;display:block;margin-top:1px;">{format / platform}</span>
  </div>
</div>
```
**CRITICAL:** The right column div MUST have `min-width` (120px recommended) and `flex-shrink:0`. Without it, variable-length amounts ("₹20L" vs "up to ₹50L") cause uneven column widths and broken alignment. Both amount and unit must be `display:block` so they stack cleanly inside the fixed column.

**STRIKETHROUGH PILLS (problem slides):**
```html
<span style="font-size:11px;padding:5px 12px;border:1px solid rgba(255,255,255,0.1);border-radius:20px;color:#6B6560;text-decoration:line-through;">Old tool</span>
```

**TAG PILLS (feature labels):**
```html
<span style="font-size:11px;padding:5px 12px;background:rgba(255,255,255,0.06);border-radius:20px;color:{BRAND_LIGHT};">Label</span>
```

**PROMPT / QUOTE BOX:**
```html
<div style="padding:16px;background:rgba(0,0,0,0.15);border-radius:12px;border:1px solid rgba(255,255,255,0.08);">
  <p style="font-size:13px;color:rgba(255,255,255,0.5);">Label</p>
  <p style="font-size:15px;color:#fff;font-style:italic;">"Quote text"</p>
</div>
```

**FEATURE LIST ROW:**
```html
<div style="display:flex;align-items:flex-start;gap:14px;padding:10px 0;border-bottom:1px solid {LIGHT_BORDER};">
  <span style="color:{BRAND_PRIMARY};font-size:15px;">{icon}</span>
  <div>
    <span style="font-size:14px;font-weight:600;">{Label}</span>
    <span style="font-size:12px;color:#8A8580;">{Description}</span>
  </div>
</div>
```

**NUMBERED STEPS:**
```html
<div style="display:flex;align-items:flex-start;gap:16px;padding:14px 0;border-bottom:1px solid {LIGHT_BORDER};">
  <span style="font-size:26px;font-weight:300;color:{BRAND_PRIMARY};min-width:34px;">01</span>
  <div>
    <span style="font-size:14px;font-weight:600;">{Step title}</span>
    <span style="font-size:12px;color:#8A8580;">{Step description}</span>
  </div>
</div>
```

**CTA BUTTON (final slide only):**
```html
<div style="display:inline-flex;align-items:center;gap:8px;padding:12px 28px;background:{LIGHT_BG};color:{BRAND_DARK};font-weight:600;font-size:14px;border-radius:28px;">
  {CTA text}
</div>
```

---

## Instagram Frame (Preview Wrapper)

When displaying the carousel in chat, wrap it in an Instagram-style frame:
- Header: Avatar (BRAND_PRIMARY circle + logo) + handle + subtitle
- Viewport: 4:5 aspect ratio, swipeable/draggable track with all slides
- Dots: Small dot indicators below the viewport
- Actions: Heart, comment, share, bookmark SVG icons
- Caption: Handle + short carousel description + timestamp

**IMPORTANT:** The `.ig-frame` must be exactly 420px wide. The carousel viewport inside it has a 4:5 aspect ratio (420×525px). Do NOT change this width — the export process depends on it.

---

## Exporting Slides as Instagram-Ready PNGs

After the user approves the carousel preview, export each slide as an individual 1080×1350px PNG image ready for direct Instagram upload.

### CRITICAL EXPORT RULES

1. **Use Python for HTML generation** — never use shell scripts with variable interpolation. Always generate HTML files using `Path.write_text()` or `open().write()`

2. **Embed images as base64** — all user-uploaded images must be base64-encoded and embedded directly in the HTML as `data:image/jpeg;base64,...` URIs

3. **Keep the 420px layout width** — use Playwright's `device_scale_factor` to scale up to 1080px. Never set the viewport to 1080px wide — this would reflow the layout

### EXPORT SCRIPT (Playwright)

```python
import asyncio
from pathlib import Path
from playwright.async_api import async_playwright

INPUT_HTML = Path("/path/to/carousel.html")
OUTPUT_DIR = Path("/path/to/output/slides")
OUTPUT_DIR.mkdir(exist_ok=True)

TOTAL_SLIDES = 7  # Update to match your carousel

VIEW_W = 420
VIEW_H = 525
SCALE = 1080 / 420  # = 2.5714...

async def export_slides():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page(
            viewport={"width": VIEW_W, "height": VIEW_H},
            device_scale_factor=SCALE,
        )
        html_content = INPUT_HTML.read_text(encoding="utf-8")
        await page.set_content(html_content, wait_until="networkidle")
        await page.wait_for_timeout(3000)  # Wait for fonts to load

        await page.evaluate("""() => {
            document.querySelectorAll('.ig-header,.ig-dots,.ig-actions,.ig-caption')
                .forEach(el => el.style.display='none');
            const frame = document.querySelector('.ig-frame');
            frame.style.cssText = 'width:420px;height:525px;max-width:none;border-radius:0;box-shadow:none;overflow:hidden;margin:0;';
            const viewport = document.querySelector('.carousel-viewport');
            viewport.style.cssText = 'width:420px;height:525px;aspect-ratio:unset;overflow:hidden;cursor:default;';
            document.body.style.cssText = 'padding:0;margin:0;display:block;overflow:hidden;';
        }""")
        await page.wait_for_timeout(500)

        for i in range(TOTAL_SLIDES):
            await page.evaluate("""(idx) => {
                const track = document.querySelector('.carousel-track');
                track.style.transition = 'none';
                track.style.transform = 'translateX(' + (-idx * 420) + 'px)';
            }""", i)
            await page.wait_for_timeout(400)
            await page.screenshot(
                path=str(OUTPUT_DIR / f"slide_{i+1}.png"),
                clip={"x": 0, "y": 0, "width": VIEW_W, "height": VIEW_H}
            )
            print(f"Exported slide {i+1}/{TOTAL_SLIDES}")

        await browser.close()

asyncio.run(export_slides())
```

### WHY THIS WORKS
- `device_scale_factor=2.5714` renders at high DPI: 420px → 1080px output without reflowing layout
- `clip` ensures the screenshot captures only the carousel viewport
- `wait_for_timeout(3000)` gives Google Fonts time to load
- `track.style.transition = 'none'` disables swipe animation for clean snapshots

### COMMON EXPORT MISTAKES TO AVOID
- Setting viewport to 1080×1350 → layout reflows, fonts become tiny *(Fix: keep viewport at 420×525)*
- Using shell scripts to generate HTML → `$` signs get interpolated *(Fix: always use Python)*
- Not waiting for fonts → headings render in fallback fonts *(Fix: `wait_for_timeout(3000)`)*
- Not hiding IG frame chrome → export includes header and caption *(Fix: hide `.ig-header,.ig-dots,.ig-actions,.ig-caption`)*

---

## Design Principles

1. Every slide is export-ready — arrow and progress bar are part of the slide image
2. Light/dark alternation — creates visual rhythm and sustains attention across swipes
3. Heading + body font pairing — display font for impact, body font for readability
4. Brand-derived palette — all colors stem from one primary, keeping everything cohesive
5. Progressive disclosure — progress bar fills and arrow guides the user forward
6. Last slide is special — no arrow (signals end), full progress bar, clear CTA
7. Consistent components — same tag style, same list style, same spacing across all slides
8. Content padding clears UI — body text never overlaps with the progress bar or arrow
9. Iterate fast — show the preview, get feedback on specific slides, fix those slides
