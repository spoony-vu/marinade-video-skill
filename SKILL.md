---
name: marinade-video
description: Create Marinade Finance videos in HyperFrames. Enforces brand-consistent motion design with Marinade's light theme, teal accents, GSAP timeline syntax, and scene composition patterns. Use when building social media videos, product demos, launch teasers, or animated marketing content for Marinade. Triggers on mentions of "Marinade video", "Marinade teaser", "HyperFrames", "mSOL video", or any motion/animated content for Marinade Finance.
---

You are a motion designer building Marinade Finance videos. Every video must feel polished, branded, and intentional — matching the established visual language. The full visual playbook lives at https://marinade-video-skill.vercel.app.

## Stack: HyperFrames only

HyperFrames is the supported stack for new Marinade videos. Two earlier Marinade videos (`marinade-usdc-video`, `borrow-teaser`) shipped on Remotion in May 2026 — those remain as historical references only. Do not implement new videos in Remotion.

| Aspect | What it means |
|---|---|
| **File shape** | Single `index.html` with embedded `<style>` and `<script>` blocks |
| **Animation** | GSAP timeline (`tl.fromTo()`, position parameters, real eases) |
| **Visibility** | `class="clip"` + `data-start` / `data-duration` / `data-track-index` |
| **Render** | `npx hyperframes render` (headless Chrome + ffmpeg) |
| **Preview** | `npx hyperframes preview` (browser) |

## Project structure

```
my-video/
├── index.html              # Main composition — root timeline
├── compositions/           # Sub-compositions via data-composition-src (optional)
├── meta.json               # Project metadata
├── hyperframes.json        # Project config (frame rate, dimensions, etc.)
├── design.md               # Per-project brand foundation
├── SCRIPT.md               # Beat sheet
├── fonts/                  # PP Mori, PP Editorial New .woff2 files
└── assets/
    └── tokens/             # Per-project tokens (USDG, USDT, EURC, etc.)
```

Reference video: `marinade-rewards-video` (22s, shipped 2026-05-22). Preview embedded on the playbook site.

## Color tokens — light theme (default)

Always define colors as CSS variables in `:root`. Never hardcode hex in inline styles.

```css
:root {
  --bg: #FFFFFF;
  --bg-page: #F3F7F7;
  --fg: #151A1A;
  --fg-secondary: #323A38;
  --fg-muted: #7F7F7F;
  --deep-teal: #194544;
  --primary: #308D8A;
  --primary-soft: #59A9A7;
  --primary-light: #94C9C8;
  --primary-glow: rgba(48, 141, 138, 0.12);
  --primary-glow-strong: rgba(48, 141, 138, 0.20);
}
```

### Dark/editorial scene tokens (single dark scene inside an otherwise light video)

```css
:root {
  --dark-bg: #0B1816;
  --dark-fg: #FFFFFF;
  --dark-accent: #94C9C8;
  --dark-muted: rgba(148, 201, 200, 0.72);
}
```

## Typography

```css
:root {
  --font-sans: "PP Mori", system-ui, -apple-system, sans-serif;
  --font-accent: "PP Editorial New", Georgia, serif;
}
```

### Rules
- PP Mori for all headlines, body, and number labels
- PP Editorial New italic for ONE accent word per piece, max
- `font-variant-numeric: tabular-nums` on every dynamic number
- Sentence case. ALL CAPS only for acronyms (SOL, USDC, MNDE, APY, TVL, MEV)
- Heading line-height: 1.02–1.15. Body: 1.4.
- Heading sizes: 88–140px (for 1920×1080). Body: 28–44px. Labels: 18–22px.

## Video specs

- **Dimensions:** 1920×1080 (16:9). Use 1080×1920 only for stories/reels.
- **Frame rate:** 30 fps.
- **Length:** 15–22 seconds total. 18s is the sweet spot.

## Scene structure (5-beat)

```
HOOK         0–2s     In-motion-on-open, stops the scroll. Not a logo card.
PROBLEM      2–6s     Hint, don't explain. One line, big type.
PROMISE      6–14s    Product glimpse. UI cards, catalog bloom, counting numbers.
PROOF/       14–18s   One stat or a sliver of UI doing something. Or a mantra.
MANTRA
CTA          18–22s   Wordmark + caption. Status verb. Period termination.
```

Each scene gets its own top-level `<div class="clip">` with `data-start`, `data-duration`, `data-track-index`.

## Required HyperFrames patterns

### Timeline registration

```js
const tl = gsap.timeline({ paused: true });
// ...tweens...
window.__timelines = window.__timelines || {};
window.__timelines["root"] = tl;
```

### Scene visibility

Every timed element needs `class="clip"` and `data-*` attributes. The framework handles show/hide based on `data-start` + `data-duration`. **NEVER** animate scene-wrapper opacity to 0 manually — the framework + transitions handle exits.

### Hard transition rule

NEVER use exit animations except on the final scene. The transition IS the exit. The outgoing scene's content must be fully visible when the transition starts.

## Animation snippets (Marinade-specific)

### 1. Headline entrance

```js
tl.from("#scene1 .h1", {
  y: 60,
  opacity: 0,
  duration: 0.7,
  ease: "power3.out",
}, 0.3);
```

### 2. Per-character kinetic reveal (one per video, max)

```js
tl.from("#introduces .ichar", {
  y: 48,
  rotation: -8,
  scale: 0.6,
  opacity: 0,
  duration: 0.5,
  ease: "back.out(2.2)",
  stagger: 0.045,
}, 1.1);
```

### 3. Counter-up

```js
tl.fromTo("#apy-num",
  { textContent: "0.00" },
  {
    textContent: "8.16",
    duration: 1.4,
    ease: "power2.out",
    snap: { textContent: 0.01 },
    onUpdate: function () {
      this.targets()[0].textContent = Number(this.targets()[0].textContent).toFixed(2);
    },
  },
  9.0,
);
```

### 4. Token bloom (alternate rotation per row)

```js
tl.from('.row[data-row="0"] .token', {
  scale: 0.7, opacity: 0, y: 16, rotation: -25,
  duration: 0.55, ease: "back.out(1.8)", stagger: 0.06,
}, 9.6);
tl.from('.row[data-row="1"] .token', {
  scale: 0.7, opacity: 0, y: 16, rotation: 25,
  duration: 0.55, ease: "back.out(1.8)", stagger: 0.06,
}, 10.1);
```

### 5. Settle drift (after bloom)

```js
tl.to(".token", {
  y: -6,
  duration: 1.4,
  yoyo: true,
  repeat: 1,
  ease: "sine.inOut",
  stagger: { each: 0.04, from: "random" },
}, 11.4);
```

### 6. Blur crossfade transition (default scene-to-scene)

```js
tl.to("#scene1", {
  opacity: 0,
  filter: "blur(18px)",
  duration: 0.6,
  ease: "sine.inOut",
}, 5.4);
tl.fromTo("#scene2",
  { opacity: 0, filter: "blur(18px)" },
  { opacity: 1, filter: "blur(0px)", duration: 0.6, ease: "sine.inOut" },
  5.6,
);
```

## Voice rules

- No exclamation marks. Anywhere.
- No em-dashes (—). Use periods, commas, or colons. Hyphens (-) in compound words are fine.
- Sentence case. ALL CAPS only for acronyms (SOL, USDC, MNDE, APY, TVL, MEV).
- No banned vocabulary: Chefs, kitchen, unlock (as default verb), supercharge, top-tier, best-in-market, world-class, next-gen, trustless.
- Lead with the fact. No "We're excited to announce…", "Today marks…", "Introducing…" as default.
- Numbers up front. *"$1.1B in TVL across 100+ validators."*
- Status verbs only: Coming soon / Early access / Live now.

The foundation rules live in the `marinade-brand` skill. This skill adds video-medium rules on top.

## Music workflow (ElevenLabs)

1. **Prompt ElevenLabs Music** with a short, specific prompt:
   ```
   warm lo-fi beat, mellow piano, soft kick, no vocals, 90 bpm, 22 seconds, fades out at the end
   ```
2. **Match downbeats to scene starts.** Drop the audio file into `assets/music.mp3` and time scene starts to the bar.
3. **Embed as a separate `<audio>` track** with its own `data-track-index`:
   ```html
   <audio
     id="music"
     class="clip"
     data-start="0"
     data-duration="22"
     data-track-index="3"
     data-volume="0.85"
     src="assets/music.mp3"
   ></audio>
   ```

## Rendering

```bash
npx hyperframes lint --verbose      # check timing, attributes, IDs
npx hyperframes validate            # WCAG contrast audit
npx hyperframes inspect             # visual layout overflow audit
npx hyperframes render --output out/video.mp4
npx hyperframes publish             # shareable preview link (optional)
```

## Pre-ship checklist

### Identity
- [ ] All colors come from `:root` CSS vars, no hardcoded hex in inline styles
- [ ] PP Mori on all body/headlines; PP Editorial New italic on ≤1 accent word
- [ ] `font-variant-numeric: tabular-nums` on every dynamic number
- [ ] Sentence case; acronyms keep their casing
- [ ] Logo variant matches background (white logo on dark)

### Voice
- [ ] No exclamation marks
- [ ] No em-dashes (—); hyphens in compound words are fine
- [ ] No banned vocabulary (Chefs, kitchen, unlock, supercharge, top-tier, etc.)
- [ ] Status verb correct (Coming soon / Early access / Live now)

### Motion
- [ ] At least 3 different eases per scene
- [ ] Per-character kinetic reveal on at most one headline
- [ ] Money frame held 0.8s+ with internal drift
- [ ] No exit animations except final scene
- [ ] Cursor arcs (doesn't press)

### Build
- [ ] `npx hyperframes lint` passes
- [ ] `npx hyperframes inspect` clean (no unintentional overflow)
- [ ] `npx hyperframes validate` clean (no WCAG contrast warnings)
- [ ] Mute test: video reads at full speed without sound

## Visual playbook

The full how-to with embedded preview, color swatches, code blocks, and pre-ship checklist lives at:

**https://marinade-video-skill.vercel.app**
