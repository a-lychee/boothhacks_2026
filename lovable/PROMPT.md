# Lovable build prompts: Heads-Up pitch deck

**How to use this**
1. Create a new Lovable project.
2. Paste **Prompt 1** into Lovable.
3. Attach `c1-apricot-reference.png`, which is in this same folder. It shows the target look.
4. When the first build looks right, connect GitHub: Lovable project → **GitHub** → **Connect**. Lovable creates a new repo.
5. Send Claude Code the repo name so it can attach it and work in the same codebase.
6. Use Prompts 2 and 3 one at a time, after Prompt 1 works. Small follow-ups get better results from Lovable than one huge prompt.

The demo comes later. Prompt 1 only reserves a `/demo` route for it.

---

## Prompt 1: App shell, C1 theme, presentation navigation

```
Build a web-based slide presentation app for a product pitch called "Heads-Up" (Kilts Product Competition, Chicago Booth, Phase 2). It must feel like presenting in PowerPoint or Keynote, with an Apple-product-page level of visual polish. I've attached a reference image of the exact visual style I want for the slides.

## Routes
- "/" redirects to "/deck".
- "/deck" is the presentation editor/presenter (main work for now).
- "/deck/presenter" is the presenter-view window (see below).
- "/demo" is a placeholder page in the same visual style that says "Live demo coming soon". The product prototype gets built here later. Do not build it now.

## Code structure (important: I also edit this repo with another AI coding tool)
- One file per slide in src/deck/slides/ (e.g. 01-Title.tsx), each exporting a React component plus metadata: { id, title, section, notes }.
- src/deck/slides/index.ts exports the ordered array of slides. Adding a slide = adding a file and one line.
- All theme tokens (colors, fonts, radii, shadows, gradients) in one file: src/deck/theme.ts plus CSS variables in index.css. No hard-coded colors inside slide files.
- Reusable slide building blocks in src/deck/components/: SlideFrame, Kicker, Headline (two-tone), StatPill, GlassCard, CountUp, Reveal.
- Keep TypeScript, React, Tailwind and shadcn/ui. No other heavy dependencies; use CSS transitions or framer-motion for animation.

## Fixed 16:9 stage (non-negotiable)
- Every slide is authored at exactly 1920×1080 px and the whole stage is scaled uniformly (CSS transform: scale) to fit its container, letterboxed if needed. Content must never reflow or change layout at different window sizes. Use px units inside slides.

## Slide visual style ("C1 Apricot")
- Background on every slide: a warm, single-hue gradient with soft grain:
  - radial-gradient(circle at 78% 46%, #f6c3a5 0, transparent 38%),
  - radial-gradient(circle at 8% 0%, #fbeadd 0, transparent 50%),
  - linear-gradient(160deg, #f8ede4 0%, #f3dccd 100%)
  - Overlay a subtle SVG fractal-noise grain (feTurbulence, baseFrequency ~0.9) at ~22% opacity, mix-blend-mode multiply.
- Colors: ink #2a1d17; muted text rgba(42,29,23,0.6); secondary headline text rgba(42,29,23,0.4); single accent #d4502f (used sparingly: small dot in the brand mark, warnings, one highlighted word or number).
- Fonts (Google Fonts): "Bricolage Grotesque" weight 800 for headlines and big numbers (letter-spacing -0.035em, line-height 1.0); "Instrument Sans" 400/500/600 for everything else.
- Headline pattern: two-tone. The first sentence is in ink, the second in the lighter secondary color. Headlines are big (80–96px) and left aligned.
- Slide chrome: top-left "● Heads-Up" (accent dot + ink text, 19px, weight 600); top-right section label like "01 · The problem" in muted 19px. Padding: 60px top, 104px sides, 56px bottom.
- Cards: white at 92% opacity, 32px radius, padding 32px 38px, shadow 0 40px 90px rgba(70,30,60,0.18). A "stacked" variant puts a second card behind it at 55% white, rotated -4deg and offset up and left, so it looks like a buried notification.
- Stat pills: rounded-full, background rgba(255,255,255,0.5), 1px border rgba(255,255,255,0.8), backdrop blur 16px, padding 14px 24px; number in Bricolage 800 34px + label in muted 18px.
- Small source line bottom-right, 14px muted.
- Overall feel: calm, warm, editorial, premium. Generous white space, one idea per slide, never busy.

## Animation
- Each time a slide is shown, its elements fade up in a staggered sequence (opacity 0 → 1, translateY 24px → 0, 0.8s, cubic-bezier(0.16, 1, 0.3, 1)). Every element can set its own delay.
- Numbers count up from 0 to their value (~1.1s, ease-out) when the slide appears.
- Slide-to-slide: smooth 0.5s crossfade.
- Respect prefers-reduced-motion (no motion, final states only).
- Thumbnails and previews always show the final, fully revealed state with no animation.

## Editor view (/deck default), like PowerPoint or Google Slides
- Dark, neutral app chrome around the slides (background #141518, panels #1b1c20, lines #2e3036, text #e7e7ea) so the warm slides stand out.
- Top bar: deck title on the left; on the right a ‹ › pair with a "3 / 12" counter, a "?" shortcuts button, a "Presenter view (P)" button, and a primary white "▶ Present (F)" button.
- Left sidebar (~232px): vertical thumbnails of every slide (real scaled-down renders, 16:9) with slide numbers. Click to jump. White outline on the current slide; auto-scrolls to keep it visible.
- Center: the current slide on the fixed stage with a soft shadow.
- Right rail: vertical progress dots, one per slide. The current dot is elongated, past dots are brighter, and hovering shows a tooltip with the slide number and title. Click to jump.
- Bottom panel: "Speaker notes" for the current slide (read-only).
- The URL hash tracks the slide (/deck#4) so reloads and links keep your place.

## Keyboard and input
- → / Space / PageDown / ↓ = next; ← / PageUp / ↑ = previous; Home / End = first / last.
- Type a number then Enter = jump to that slide.
- F or F5 = start Present mode; Esc = exit it.
- P = open the presenter window.
- B or . = black screen toggle while presenting.
- ? = shortcuts overlay listing all of the above.
- Swipe left/right on touch devices.
- Ignore shortcuts while typing in an input.

## Present mode
- Requests browser fullscreen and shows only the slide on black, filling the screen.
- The progress dots stay visible on the right in a small translucent pill; cursor and dots fade after ~2s of no mouse movement.
- Clicking the slide advances.
- Esc (or leaving browser fullscreen) returns to the editor view.

## Presenter window (/deck/presenter, opened by P in a new window)
- Like PowerPoint presenter view: large current slide on the left; on the right a "Next" slide preview (or "End of deck") and the speaker notes in large readable text (20px).
- Top bar: a countdown timer starting at 8:00 (our hard time limit, demo included) with Start/Pause and Reset buttons. It turns amber under 1:00 and red when over time (shows +0:12). It starts automatically when the main window enters Present mode. Also show the current clock time.
- Bottom bar: ‹ Previous / Next › buttons and "Slide 3 of 12".
- Keeps in sync with the main window both ways using BroadcastChannel: navigating in either window moves both.

## Slides to create now
Build these with the components above, keeping text short (speaker-led deck). Use exactly this copy:

1. Title (section "Kilts Product Competition 2026"): huge headline "Heads-Up" + secondary line "Ansel's record, back when it matters." Sub line: "A Slack card before every recurring meeting: what you own, what's still open, what was decided." Bottom: "Team [Team name] · Chicago Booth · November 13, 2026". Right side: a single white card mock of the Heads-Up Slack card (sender "Ansel APP 1:30 PM", title "Weekly Product Sync · 2:00 PM", three sections YOU OWN "Forecast update (may already be done)" ▶ 14:32 / STILL OPEN "Launch date: Mar 3 or Mar 10?" ▶ 27:05 / DECIDED "Pricing page ships with v2" ▶ 41:18, buttons "Open last meeting", "Mute series", "Not useful").
2. Meet the team (section "00 · Team"): headline "Meet the team." 4 member cards in a row (photo placeholder circle, "[Name]", "[Role on team]", "[Background]"). Make names and roles easy to edit in one array at the top of the file.
3. The problem (section "01 · The problem"): kicker "Where the value lands vs. where it's needed"; headline "Ansel's notes arrive 6 minutes after the meeting." + secondary "Light users need them 7 days later." Right side: the stacked-card visual. Back card (rotated, 55% white): app icon tile with "A" on a dark plum gradient (#2b2340 → #4a3d66), "Ansel · Meeting summary", "Weekly Product Sync · 7 days ago", tag "Archived". Front card: calendar icon "MON 17", "Weekly Product Sync", "Today · 2:00 PM · starts in 30 min"; rows "Last week's notes" → "Unopened" (accent red #c8442a, semibold), "Your action items" → "0 of 4 marked done", "What was decided" → "Buried in email"; then a callout box (bg #fff1ec, text #b43c22) with a slowly blinking dot: "You're walking in cold." Bottom-left stat pills: "34% of summaries opened", "22s spent reading one", "6% of action items done". Source line: "Source: Kilts Center Ansel case (2026), Ex 3, 5 · card is illustrative".
4. Who we're building for (section "01 · The user"): headline "Light users: under 5 meetings a week." + secondary "Half of all seats. Two-thirds of idle ones." Persona card: "Priya, data analyst · 3–4 meetings a week, including one weekly team sync. Owns one piece of each meeting." Quote pills: "If I only have three meetings that week, this is not something I think to open." / "I usually just ask someone who was there." Stat pills: "51% of all seats", "66% of idle seats", "20% active weekly". Source: "Kilts Center Ansel case (2026), Ex 3, 7; team interviews (n=13)".
5. The opportunity (section "01 · Market"): headline "$75M of ARR sits on light seats with a recurring meeting." Show a simple, elegant funnel or nested-bars visual: All light seats 2.09M / $122M → Light seats that attend 1.71M / $100M → With ≥1 recurring meeting a week 1.29M / $75M (highlighted in accent). Source: "One price per seat ($240M ÷ 4.1M = $58.54); 48.5% recurring (Flowtrace)".
6. What they do today (section "01 · Today"): headline "Today they walk in cold." Three white cards in a row, each with a short title and one line: "Rely on memory: forgets once the email arrives (0.4 Ansel sessions a week)", "Ask a colleague: pulls a teammate off their own work", "Reread old notes: 22 seconds on a summary, then it's buried". Source: "Ex 3, 5, 7, 8".
7. Demo (section "02 · Demo"): headline "Let's see it." + secondary "Heads-Up, 30 minutes before the sync." A large button-style card "Open live demo →" linking to /demo. Notes: "Switch to the demo tab."
8. What we learned (section "03 · Learnings"): headline "What we learned building it." Three numbered cards: "[Learning 1]", "[Learning 2]", "[Learning 3]", each with a one-line placeholder. Plus a small "Riskiest assumption" pill: "Enough recurring meetings leave each light user something they own or still open."
9. What we'd test next (section "03 · Next"): headline "What we'd test next." Rows: "Primary metric: Light attendees active weekly 20% → 26%", "Seat retention, light attendees: 34% → 39%", "Action items acted on: 6% → 20%", "Guardrail: no increase in admin privacy escalations". Source: "Ex 2, 3, 5".
10. Thank you (section "Q&A"): huge "Thank you." + secondary "Questions?"
11. Appendix divider: huge "Appendix".
12. Appendix: Governance and cost (section "Appendix"): headline "Built inside today's permissions." Bullets: "Only same-series records the user can already access as an attendee", "No sharing defaults change, so the FY24 rollback concern doesn't arise", "Private card, audit-logged, respects retention", "No new meeting processing: a small selection step on existing summaries".

Give each slide 1–3 sentences of speaker notes I can edit.

Make it production quality: pixel-perfect spacing, no overflow on any slide at 1920×1080, crisp typography, and smooth 60fps animation.
```

---

## Prompt 2 (after Prompt 1 works): polish pass

```
Polish pass on /deck. Keep the existing structure and theme tokens.
1. Check every slide at 1920×1080 for text overflow, overlap or awkward line breaks (no single word alone on a headline's last line). Fix with spacing or copy line breaks, not by shrinking fonts below 18px.
2. Make sure the thumbnails in the sidebar and the Next preview in the presenter window show the final state of each slide (no half-faded elements).
3. Add a subtle hover lift to the cards on the problem slide.
4. Verify that Esc exits Present mode, that the presenter window stays in sync both ways, and that the 8:00 timer turns amber and red correctly.
```

---

## Prompt 3 (later): export

```
Add an "Export PDF" item to the top bar that prints the deck one slide per page at 16:9 (use a print stylesheet that renders every slide in its final state, landscape, no app chrome).
```

---

## Notes for working across Lovable and Claude Code
- Lovable syncs both ways with the connected repo's default branch. Edits from Claude Code reach Lovable once they're pushed to that branch.
- Don't rename or move the GitHub repo after connecting it. That can break Lovable's sync.
- If a Lovable edit and a Claude Code edit touch the same file at the same time, the later push wins. Avoid working in both at once.
