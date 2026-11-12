# Handoff: Heads-Up deck review and cleanup

This note picks up the work from the previous Claude Code session. Read it in full before starting.

## Context
- Booth Hackathon / Kilts Product Competition, Phase 2. The team presents **Fri Nov 13, 2026 at 3:00 PM**. The deck and demo link are due to the Kilts Center at **2:30 PM**. The time limit is 8 minutes, demo included.
- Product: **Heads-Up**, a Slack card sent ~30 min before a light user's recurring meeting that shows what they own, what's still open, and what was decided. See `a-lychee/boothhacks_2026` for case PDFs (if uploaded) and `lovable/PROMPT.md` for the full deck spec.
- Chosen visual style: **"C1 Apricot"**. Reference image: `boothhacks_2026/lovable/c1-apricot-reference.png`. Original HTML prototype: `boothhacks_2026/deck/index.html` (the `data-variant="apricot"` slide).
- The real app lives in **`a-lychee/heads-up`**, a Lovable project (React + Vite + TS + Tailwind + shadcn). Lovable syncs both ways with its `main` branch, so anything pushed to `main` appears in Lovable. **Ask the user before pushing to `main`.**

## The user's request
1. **Review what Lovable built.** Run the app (`npm ci && npm run dev`), open `/deck`, screenshot every slide at 1920×1080 using the real fonts, and check against the C1 reference and the spec in `lovable/PROMPT.md`. The user says **"the text formatting looks weird"**: check spacing, fonts, and line breaks carefully. Also test the navigation: arrows, sidebar thumbnails, progress dots, F → present, Esc exits, P → presenter window with 8:00 timer, B blackout, number+Enter.
2. **Clean up** leftover code from an older slide-maker project and rewrite the README for Heads-Up.

## Issues already found (by reading the code; not yet visually confirmed)
1. **Hard-coded `<br />` in headlines**, copied from a screenshot, force bad breaks such as a lone "6" or "7" at the end of a line:
   - `src/deck/slides/01-Title.tsx:27`, `03-Problem.tsx:23-24`, `05-Opportunity.tsx:24`.
   - Fix: remove them, then use `text-wrap: balance` on `.deck-headline` (and `text-wrap: pretty` on body copy). Size the headline containers so the lines break naturally.
2. **Wrong default font**: `tailwind.config.ts` `fontFamily.sans` is still `"IBM Plex Sans"`, which is never loaded. Tailwind's preflight applies it to `html`, so anything not under `.deck-slide` or `body` falls back to a system font. Fix: set `sans` to Instrument Sans and add a `display` family for Bricolage Grotesque. Replace the scattered `font-['Bricolage_Grotesque']` arbitrary classes with a `font-display` utility.
3. **Font weights**: `src/index.css` loads Bricolage Grotesque at weight 800 only. Load the weights actually used (e.g. `opsz,wght@12..96,600..800`) and Instrument Sans 400–700.
4. `01-Title.tsx`: `mt-[22px]` on an inline `<span>` inside the secondary headline has no effect. Make it a block.
5. Other things to check visually: the Title slide card height (720px fixed), the Problem slide back-card offset, and whether Instrument Sans body text has sensible line-heights (the `.deck-slide` base sets font-size 24px but no line-height).

## Leftovers to remove (verify that nothing in `src/deck` or `src/App.tsx` imports them first)
- `src/slides/`, most of `src/components/slides/` (`MSSlideLayout`, `SlideLayout`, `DarkSlide`, `SlideCanvas`, `SlideGrid`, `SlideOverviewGrid`, `TemplatePicker`, `PresenterView`, `PresentationMode`, `ScaledSlide`, `PresenterNotesPanel`, `SlideThumbnail`, …), `src/components/layout/`, and the legacy `slide`/`ms` color tokens in `tailwind.config.ts`.
- Check whether Supabase (`src/integrations`, `supabase/`, `drizzle/`) is used by the deck. If not, ask the user before removing it.
- Rewrite `README.md` for Heads-Up: what it is, routes (`/deck`, `/deck/presenter`, `/demo`), keyboard shortcuts, where slides live (`src/deck/slides`), how to add a slide, and theme tokens (`src/deck/theme.ts` + `src/index.css`).

## Working method
- After any change, run `npm run build` and the tests (`npm test`), and re-screenshot every slide. Show the user before/after screenshots.
- Commit in small, clear steps.
