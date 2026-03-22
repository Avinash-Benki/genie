# Keyboard animation landing page

## Image / frame generation prompts (WpDev keyboard + fog background)

### 1) Hero shot (single frame / key visual)

Cinematic product shot of a tangerine-orange mechanical keyboard with off-white and light gray keycaps, “WpDev”, floating on soft white clouds, light gray studio fog background, premium diffused lighting, 4k, photorealistic.

### 2) Ultra-premium product photography (assembled keyboard)

Ultra-premium product photography of a sleek tangerine-orange / amber anodized aluminum mechanical keyboard called “WpDev” resting within soft white clouds / volumetric fog, minimalistic editorial studio photoshoot. Background is light gray-to-white mist with gentle gradient falloff, airy and clean. Soft diffused key light with subtle rim highlights outlining the keyboard silhouette, keycap edges, and the rotary knob on the top-right. Controlled reflections on the orange metal case; crisp detail on off-white and light gray keycaps with a few matching orange accent keys. Shallow depth of field, sharp focus on the keyboard, cinematic but bright, luxury modern hardware aesthetic. No clutter, no text overlays, no logos emphasized. Shot with a professional DSLR, 85mm lens, f/2.0, ultra-high resolution, photorealistic, premium editorial product photography.

### 3) Layered “opening” engineering view (for the scroll sequence look)

Exploded layer-separated engineering view of the same “WpDev” keyboard with a tangerine-orange anodized aluminum chassis, every layer precisely separated and floating in perfect alignment, suspended in mid-air within soft white/gray studio fog and cloud haze. Show the keyboard opening into clean layers: off-white + light gray keycaps, switches, top plate, PCB, stabilizers, foam layers, controller, USB-C daughterboard, bottom case, screws, and the rotary knob assembly—all centered, evenly spaced, perfectly aligned. Lighting matches the hero: soft diffused illumination with gentle rim highlights, controlled reflections on orange metal and matte plastics. Editorial engineering aesthetic, ultra-sharp focus, photorealistic, ultra-high resolution. No labels, no annotations, no text.

### 4) Motion / sequence direction (what the 120 frames should do)

Create a 120-frame sequence where the keyboard performs a scroll-friendly mechanical “expand → open → reassemble” action:

| Frames | Description |
|--------|-------------|
| 0–20 | Keyboard fully assembled, centered, stable. |
| 21–55 | Keyboard subtly expands outward (precision widening / spacing cues), still mostly assembled. |
| 56–85 | Keyboard opens into layers (keycaps/switches/plate/PCB/case visible), all aligned, clean separations. |
| 86–119 | Layers glide back together and keyboard returns fully assembled. |

Motion feels magnetic, engineered, controlled, with gentle easing. No jitter. No dramatic spins. Keep camera angle consistent across frames. Keep fog/cloud background consistent and soft.

---

## Google Antigravity — master website prompt (scrollytelling)

### Act as

A world-class Creative Developer (Awwwards-level) specializing in Next.js, Framer Motion, and high-performance scrollytelling.

### The task

Build a high-end “Scrollytelling” landing page for a fictional keyboard brand called “WpDev”. The core mechanic is a scroll-linked animation that plays an image sequence of the WpDev keyboard expanding and opening into layers, then closing/reassembling as the user reaches the end of the scroll.

### Tech stack

| Area | Choice |
|------|--------|
| Framework | Next.js 14 (App Router) |
| Styling | Tailwind CSS |
| Animation | Framer Motion |
| Rendering | HTML5 Canvas (for performance) |

### Visual direction (fog / editorial light theme)

- **Seamless blending:** The website background MUST match the fog/mist background of the image sequence exactly so the edges of frames are invisible.
- **Background color:** Use an eyedropped fog color (example: `#ECECEC` / `#E6E6E6`). The exact value should match the frames.
- **Typography:** Inter or San Francisco. Minimal, tracking-tight.
- **Text color (light page):**
  - Headings: `text-black/90`
  - Body: `text-black/60`
- **Aesthetic:** Premium editorial hardware. Clean, quiet, high-end. No noisy gradients. No unnecessary UI clutter.

### Implementation details

#### 1) The sticky canvas

Create a component: `components/KeyboardScroll.tsx`

- **Outer container:** `h-[400vh]` to create a long scroll.
- **Inside:** a `<canvas>` that is:
  - `sticky top-0 h-screen w-full`
  - perfectly centered
- Canvas should render frames sharply (handle `devicePixelRatio`).

#### 2) Image sequence loading

- Load a sequence of 120 images (0 → 119) exported from ezgif-split.
- **Naming convention:** `frame_[i]_delay-0.04s.webp`
- Preload all images before playing to avoid flicker.
- Show a loading spinner/progress indicator until enough frames are ready (ideally all frames).

#### 3) Scroll → frame mapping (correct + smooth)

- Use `useScroll` from Framer Motion to map scroll progress (0 → 1) to frame index (0 → 119).
- **On scroll:**
  - compute current frame index
  - draw the image to canvas using `requestAnimationFrame`
- Clamp frame index and avoid redundant draws to reduce jank.
- **Canvas draw behavior:**
  - Use a “contain” fit so the keyboard stays fully visible across screen sizes.
  - Keep consistent positioning so the keyboard doesn’t jump between frames.
  - Recalculate canvas size on resize.

#### 4) Story text overlays (timed fade in/out)

Overlay text sections that fade in/out as the keyboard expands/opens. Use these exact beats:

| Scroll | Alignment | Heading | Subtext |
|--------|-----------|---------|---------|
| 0% | Centered | WpDev Keyboard. | Engineered clarity. |
| 25% | Left | Built for Precision. | Every detail, measured. |
| 60% | Right | Layered Engineering. | See what’s inside. |
| 90% | Centered CTA | Assembled. Ready. | Scroll back to replay. |

**Text styling rules:**

- Keep type large, minimal, and spaced.
- Use subtle fade + slight translate (like `y: 10px → 0px`).
- Never block the keyboard center; keep overlays out of the main product area.

#### 5) Polish requirements

- **Loading state:** centered spinner + “Loading WpDev sequence…” (subtle).
- **Smoothness:**
  - avoid stutter
  - avoid flashing backgrounds
  - avoid layout shift
- **Mobile:**
  - Canvas should scale and keep keyboard fully visible (contain fit).
  - Text overlays should reposition gracefully (no overlap).

### Output

Generate the full code for:

- `app/page.tsx`
- `components/KeyboardScroll.tsx`
- `app/globals.css`

Use nano banana to generate any UI components if needed, but keep the UI minimal.

# Dyson landing page

## How I Built a Dyson Website Revamp Using AI - Complete Beginner's Guide

*No coding experience required. Just curiosity and patience.*

---

## What We Built

A premium product website revamp for Dyson featuring:

- A cinematic scroll-driven hero animation
- A hair straightener disassembly sequence controlled by scrolling
- Dyson's exact design language replicated from scratch
- Live on the internet for free

---

## Tools Used

| Tool | Purpose | Cost |
| --- | --- | --- |
| VS Code | Code editor | Free |
| Claude Code | AI that writes all the code | Paid subscription |
| Google Whisk | Generate product images with AI | Free |
| Google Flow | Convert images into animation video | Free |
| ezgif.com | Convert video to image sequence | Free |
| GitHub | Store and version your project | Free |
| Vercel | Host your website live | Free |

---

## Phase 1 — Set Up Your Workspace

### Install VS Code

1. Go to `code.visualstudio.com`
2. Download for Mac or Windows
3. Install and open it

### Install Claude Code

Open VS Code → go to **View → Terminal** → type:

```
npm install -g @anthropic-ai/claude-code
```

Then type:

```
claude
```

Follow the login steps to connect your Anthropic account.

### Create Your Project Folder

In Terminal type these one by one:

```
mkdir ~/Desktop/dyson-revamp
cd ~/Desktop/dyson-revamp
mkdir -p assets/frames
mkdir -p assets/frames-hero
mkdir -p assets/images
git init
```

---

## Phase 2 — Gather Your Design Reference

### Get the Real Brand Design System

1. Go to the brand's official website
2. Press `Cmd + U` to open page source
3. Press `Cmd + A` → `Cmd + C` to copy everything
4. In VS Code create a new file called `brand-reference.html`
5. Paste and save

This gives Claude Code the exact fonts, colours, spacing and component styles the brand uses — so your revamp feels authentic.

### Download the Brand Logo

1. Go to `brandfetch.com` and search the brand name
2. Download the SVG logo
3. Save as `assets/logo.svg`

---

## Phase 3 — Create the Hero Video Animation

This is the most creative part. You are generating AI images and turning them into a scroll-driven animation.

### Step 1 — Generate the Assembled Product Image

1. Go to `labs.google/fx/tools/whisk`
2. In **Subject**: upload a reference photo of the product
3. In **Scene**: type your description of the assembled product on black background
4. In **Style**: type `"Official CGI product render, photorealistic, pure black background, studio lighting"`
5. Generate and download the best result as PNG
6. Save as `assets/images/product-assembled.png`

### Step 2 — Generate the Disassembled Product Image

Using the same assembled image in Whisk:

1. Keep the same Subject image
2. Change **Scene** to:

```
Exploded technical view, all internal components
separated and floating apart vertically, each part
clearly visible and hovering in space, pure black
background, same dramatic angle as original,
premium studio rim lighting
```

1. Generate and download
2. Save as `assets/images/product-disassembled.png`

### Step 3 — Resize Both Images to Match

Before animating, both images need the same canvas size so nothing gets cropped:

1. Go to `canva.com` → New design → **1920 x 1080px**
2. Upload assembled image → center it → scale to **60% of canvas**
3. Set background to pure black `#000000`
4. Download as PNG → save as assembled
5. Repeat exactly for the disassembled image at the same scale and position

### Step 4 — Create the Animation Video in Google Flow

1. Go to `labs.google/fx/tools/flow`
2. Upload assembled image as **start frame**
3. Upload disassembled image as **end frame**
4. Use this prompt:

```
Slow cinematic product disassembly, the product
smoothly and gradually separates into its individual
components, each layer lifts apart with precise
controlled movement, all parts drift apart evenly
and stay fully within the frame throughout the
entire animation, nothing exits the border,
smooth eased movement with gentle deceleration
at the end, pure black background, premium
commercial quality, 8 seconds
```

1. Generate and download the MP4

### Step 5 — Convert Video to Image Sequence

1. Go to `ezgif.com/video-to-jpg`
2. Upload your MP4
3. Set frame rate to **15 fps**
4. Click Convert → Download ZIP
5. Extract the ZIP
6. Copy all frames into `assets/frames-hero/`
7. Count the total number of frames — write this number down

---

## Phase 4 — Build the Website with Claude Code

### Launch Claude Code

```
cd ~/Desktop/dyson-revamp
claude
```

### The Master Prompt

This is the most important part. Copy and paste this entire prompt into Claude Code after the `>` symbol appears. Replace the values in square brackets with your own details:

```
Build a premium product website revamp.

Read brand-reference.html first and extract the
exact design system — fonts, colours, nav structure,
button styles, spacing patterns. Use this as the
design bible for the entire build.

Project structure:
- assets/logo.svg — brand logo
- assets/frames-hero/ — hero disassembly frames
- assets/frames/ — lifestyle scroll animation frames
- assets/images/ — product images

═══════════════════════════════════
HERO SECTION
═══════════════════════════════════

DEFAULT STATE — no scroll:
Full screen, black background #000000.
Content centered on screen:
- Small label above headline matching brand style
- Large bold headline: "[YOUR HEADLINE LINE 1]"
- Second headline line in brand accent colour
- Subtext paragraph below
- Two buttons: primary filled + secondary outlined
All centered, no product image visible yet.

ON SCROLL — split screen:
Sticky section height: 900vh

When user scrolls:
Step 1 (0% to 15%):
All text slides from center to left side smoothly.
translateX from 0 to -30vw, ease cubic-bezier.

Step 2 (10% to 20%):
Canvas appears on right side, fades in.
Position fixed, right 0, width 60vw, height 100vh.

Step 3 (20% to 90%):
Frame sequence plays in canvas as user scrolls.
Frames: assets/frames-hero/
Naming: ezgif-frame-001.jpg onwards
Total frames: [YOUR FRAME COUNT]
Forward only, never reverse.

Canvas drawing — contain mode, no cropping:
const scale = Math.min(
  canvas.width / img.naturalWidth,
  canvas.height / img.naturalHeight
);
const x = (canvas.width - img.naturalWidth*scale)/2;
const y = (canvas.height - img.naturalHeight*scale)/2;
ctx.clearRect(0, 0, canvas.width, canvas.height);
ctx.drawImage(img, x, y,
  img.naturalWidth*scale,
  img.naturalHeight*scale);

Add black rectangle bottom right of canvas
to cover any watermarks:
width 140px, height 50px, background #000000.

Left text changes at scroll milestones:
At 40%: headline changes to "Engineered to perfection."
At 70%: headline changes to "Every component. Perfected."
Each change crossfades over 0.4s.

Step 4 (90% to 100%):
Both panels fade out. Next section slides up.

═══════════════════════════════════
SCROLL ANIMATION SECTION
═══════════════════════════════════

Full screen sticky section below hero.
Height: 900vh
Background: #000000

Dual canvas system:
Foreground canvas: full screen, contain mode
Ambient canvas: same frame, blur(40px) saturate(1.2)

Frames: assets/frames/
Naming: ezgif-frame-001.jpg onwards
Total frames: [YOUR FRAME COUNT]
Forward only, never reverse.

Scroll direction logic:
let lastScrollY = window.scrollY;
let scrollingDown = true;
window.addEventListener('scroll', () => {
  scrollingDown = window.scrollY >= lastScrollY;
  lastScrollY = window.scrollY;
  if (window.scrollY === 0) currentFrame = 0;
}, { passive: true });

Text overlay bottom left:
Fades in at 30% scroll progress.
Fades out at 85% scroll progress.
Headline: "[YOUR SCROLL SECTION HEADLINE]"
Subtext below it.

═══════════════════════════════════
NAVIGATION
═══════════════════════════════════

Match exact nav structure from brand-reference.html.
Fixed top, full width.
Logo: use assets/logo.svg, height 32px.
Nav links matching brand reference exactly.
On scroll: background rgba(0,0,0,0.8)
with backdrop-filter blur(20px).

═══════════════════════════════════
LOADING
═══════════════════════════════════

Preload all frames before starting.
Show thin horizontal line at bottom of screen
filling left to right while loading.
Height: 2px, colour: rgba(255,255,255,0.4).
Hero default state visible immediately while
frames load in background.

═══════════════════════════════════
TECHNICAL
═══════════════════════════════════

CDN imports in this order:
1. Lenis: https://cdn.jsdelivr.net/npm/lenis@1.1.13/dist/lenis.min.js
2. GSAP: https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js
3. ScrollTrigger: https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js

Lenis smooth scroll:
const lenis = new Lenis({ duration: 1.2 });
function raf(time) { lenis.raf(time); requestAnimationFrame(raf); }
requestAnimationFrame(raf);
lenis.on('scroll', ScrollTrigger.update);

All sections below hero:
position: relative, z-index: 10,
explicit background-color set.

Generate: index.html, styles.css, script.js

After building confirm:
- Where totalFrames variables are in script.js
- How to change headline text in each phase
- How to adjust scroll speed
```

**Important — before pasting:**
Replace `[YOUR HEADLINE LINE 1]` with your actual headline
Replace `[YOUR FRAME COUNT]` with the number of frames you counted in Step 5 of Phase 3
Replace `[YOUR SCROLL SECTION HEADLINE]` with your lifestyle section headline

---

## Phase 5 — Preview and Refine

### Always Preview Like This

Never open index.html directly. Always use:

```
npx serve .
```

Then open `http://localhost:3000` in your browser.

### Fix Issues with Plain English

After the first build things will not be perfect. Just describe what looks wrong to Claude Code:

- *"The image is getting cropped on top and bottom"*
- *"The watermark is still showing in the bottom right"*
- *"The text transition is too slow"*
- *"The background colour of the animation doesn't match the page"*
- *"The transition to the next section is too abrupt"*

Claude Code fixes each issue. Bundle multiple fixes into one prompt for efficiency.

---

## Phase 6 — Push to GitHub

### Create a GitHub Repository

1. Go to `github.com`
2. Click **+** → **New repository**
3. Name it `dyson-revamp` or your project name
4. Leave README unticked
5. Click **Create repository**

### Push Your Code

```
git add .
git commit -m "Website revamp complete"
git remote add origin https://YOUR_TOKEN@github.com/YOUR_USERNAME/your-repo-name.git
git branch -M main
git push -u origin main
```

**Where to get YOUR_TOKEN:**
GitHub → Settings → Developer Settings → Personal Access Tokens → Generate new token → copy the `ghp_...` token

---

## Phase 7 — Deploy Live on Vercel

1. Go to `vercel.com`
2. Sign up with GitHub
3. Click **Add New → Project**
4. Find your repository → click **Import**
5. Click **Deploy**

Your site goes live in 30 seconds at a free URL.

### Update Your Live Site Anytime

```
git add .
git commit -m "describe your change"
git push
```

Vercel automatically redeploys within 30 seconds after every push.

---

## You've Got This

Everything breaks on the first try. That is not failure — that is just how building things works. Every fix teaches you something. Every iteration gets closer.

If you build something — drop it in the comments. I genuinely want to see what you create.

And if you're completely stuck, just DM me. Happy to help.

**Happy building.** 🚀