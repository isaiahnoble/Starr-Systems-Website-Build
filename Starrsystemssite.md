# Starr Systems — cinematic scroll site upgrade

This repo is an existing Base44 app (React + Vite, `@base44/sdk`) for www.starr-systems.com, synced to Base44 through GitHub. We are replacing the front-end presentation of the landing page with a cinematic scroll experience built around a glowing UFO hero graphic. The UFO's parts and stages stand for building a fully integrated business system.

## Hard rules (never break these)
- Work only on the branch `scroll-upgrade`. Never push to, merge into, or commit on `main`. Merging to `main` publishes to the live site, and only I do that.
- Do not modify `src/entities`, `src/api`, `base44/` config, or any existing `@base44/sdk` usage. Presentation only.
- Do not run any deploy or publish command.
- Never commit secrets, tokens, `.env*` files, raw MP4s, or Higgsfield job output. Add `/raw` and `.env*` to `.gitignore` first.
- Do not delete or rewrite existing pages. Add the new landing page as new components and mount it from `src/pages/Home.jsx`. Keep the old Home content in a backup file (`Home.legacy.jsx`) so I can roll back.
- Never present invented numbers as real client results. All metrics live in `src/config/metrics.config.js` with values clearly flagged `PLACEHOLDER`.

## Stack
- React + Vite (already installed). Add: `gsap` (ScrollTrigger), `lenis`. Add `three` only for the ambient particle field and the X-ray/thermal lens effect.
- The UFO itself is rendered from pre-generated image sequences drawn on pinned `<canvas>` elements. Do not rebuild it as a 3D model.
- Fonts: Inter Tight, Instrument Serif (italic accents), a mono face for readouts.
- Style: pure black background, cool white to electric blue gradient on the product name (taken from the Lotus mark), mono readouts, glass cards.

## Architecture
- One `<SmoothScroll>` wrapper: Lenis feeds `ScrollTrigger.update`; ScrollTrigger is the single source of scroll state.
- Each chapter is its own component in `src/components/landing/` (`ChapterSpin`, `ChapterComponents`, `ChapterStages`, `ChapterInterior`, `ChapterPowerUp`, plus `XrayLens`, `SpinViewer`, `Lineup`, `StartBuildCard`).
- Scrubbing: ScrollTrigger `progress` maps to a frame index; draw with `ctx.drawImage`. Use a shared `useFrameSequence(path, count)` hook that preloads frames (low-res first), and draws from a ref. Never put per-frame values in React state.
- Choreography lives in data, not in components: one config object per chapter describing beats (start/end progress, what moves, which text appears). Changing timing should never require touching component logic.
- Animate refs and canvas, not React state, to avoid re-renders on every scroll tick.
- Clean up every ScrollTrigger, Lenis instance and Three.js resource on unmount (`useGSAP` or `gsap.context`, `dispose()` geometries and materials).

## Asset pipeline
- Generation: Higgsfield MCP. Stills with `gpt_image_2_5` (4k, high quality, 16:9). Films with `kling3_0` (mode 4k, duration 10, sound off, 16:9) and always pass both `start_image` and `end_image`.
- Before generating anything: check my credit balance and estimate the total cost of the plan. If the balance can't cover the full plan, stop and tell me.
- Save raw downloads to `/raw` (gitignored). Trim the frames Kling holds at the start before slicing.
- Output frames to `public/frames/<chapter>/` as WebP. The target is about 200 frames at 1920px, but obey this size budget first: each chapter under about 15 MB, total `public/frames` under about 60 MB. If over budget, lower WebP quality first, then resolution, then frame count, and tell me what you changed. If the total can't fit, stop and ask me about external hosting before committing.
- Generate a manifest (`frames.manifest.json`) with frame counts and dimensions per chapter so the hook doesn't hard-code them.

## Performance and accessibility
- Cap `devicePixelRatio` at 2. Lazy-load chapters that are not near the viewport. Pause drawing when a chapter is off screen.
- Mobile: serve smaller frames and fewer of them, or a static image fallback for heavy chapters.
- Respect `prefers-reduced-motion`: show a static image per chapter with the text and readouts, no scrubbing.
- Sound is off by default. The toggle starts a Web Audio hum and power-up sweep only after a user gesture.
- No layout shift. Reserve space for canvases. Keep text legible against the black.

## Workflow
1. Plan first: write `PLAN.md` (chapter list, file list, asset budget, risks), then proceed without waiting unless a hard rule would be crossed.
2. Build in this order: scaffolding and smooth scroll, stills, films, frame slicing, then chapters one at a time.
3. After each chapter works, run `npm run dev` and verify it mid-scroll (forward and backward, plus a resize), then commit with a clear message: `feat(landing): <chapter>`.
4. Before saying you're done: run a production build (`npm run build`), confirm no console errors, and list anything unfinished or approximated. Do not say it's done if any chapter was not actually checked.
5. If something is ambiguous (brand copy, service names, palette), use clearly marked placeholders and list them at the end instead of guessing.

## Copy placeholders to flag in the final report
Tagline, "[N] systems integrated", "[N] automations live", service names for the four-UFO lineup, spec table rows, discovery-call slots, and the start-your-build card (button must say "Preview only — nothing was sent" until I wire a real form).
