# Handoff: Heads-Up deck review and cleanup

This note picks up the work from the previous Claude Code session. Read it in full before starting.

## Status as of Nov 12 (read this first)
Run this in a **local** Claude Code session on the user's computer. Their enterprise account's cloud sessions can't reach `registry.npmjs.org` or `fonts.googleapis.com`, so the app couldn't be installed, built, or screenshotted there.

**Done in a cloud session.** Pushed to `claude/festive-turing-ghs5qt` in `a-lychee/heads-up`, not to `main`:
- `355f21f` Typography: Tailwind `sans` is now Instrument Sans, with a new `font-display` (Bricolage Grotesque) replacing every `font-['Bricolage_Grotesque']`. Font weights now load Bricolage 600–800 and Instrument Sans 400–700. `<br />` removed from the headlines in 01, 03 and 05. `.deck-headline` has `text-wrap: balance` and `.deck-slide p` has `text-wrap: pretty`. `6&nbsp;minutes` and `7&nbsp;days` keep numbers with their units. The title's secondary line and "Questions?" are now `block`, which fixes the ignored margin and also a 142px line spacing inherited from the big "Heads-Up" word.
- `8e3c8ba` Removed the legacy slide-maker code (56 files) and the `slide`/`ms`/`canvas`/`comment` Tailwind tokens. Nothing in `src/deck` or `App.tsx` imported any of it.
- `dbf9263` Rewrote `README.md` for Heads-Up.

**None of this has been built, tested, or seen on screen.** Only a TypeScript syntax check was run.

**Still to do:**
1. Run `npm ci`, `npm run build` and `npm test` on the branch, and fix anything that breaks.
2. Take "before" screenshots from `main` (`9c00c28`) and "after" screenshots from the branch: all 12 slides at 1920×1080 with the real fonts loaded. Show the user both.
3. Check these on screen:
   - Title: the secondary line wraps between "Ansel's record," and "back when it matters." (each phrase is `whitespace-nowrap`).
   - Problem: the headline may now take 3 lines in its 960px column. If that looks bad, widen the column or adjust the size.
   - The title card height (fixed 720px) and the Problem back-card offset.
   - Body line-height. `.deck-slide` sets no line-height, so text inherits Tailwind's 1.5.
4. Test navigation: arrows, sidebar thumbnails, progress dots, F to present, Esc to exit, P for the presenter window with the 8:00 timer, B for blackout, number+Enter.

**Open questions for the user:**
- Supabase (`src/integrations/supabase`, `supabase/`, `drizzle/`) is now unused. Remove it? **Ask first.**
- Unused npm dependencies left over from the legacy code (e.g. `@dnd-kit/*`) are still in `package.json`. They can be removed once the build works.
- **Ask before pushing or merging to `main`.** It syncs straight to Lovable.

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
