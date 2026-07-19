# Pic A Boo — Project Handoff (read me first)

If you are Claude Code picking this project up: this doc is the source of truth. Read it fully before touching anything. The human who owns this has never used a terminal — be patient, explain commands, don't assume knowledge.

---

## WHAT THIS IS

**Pic A Boo** — a two-person online photobooth web app. Two people, in real time, each take 6 selfies. The twist that IS the product: **you don't pick your own photos. Your partner picks which 2 of you appear; you pick which 2 of them appear. Nobody sees themselves until the final "BOO" reveal.** Output is a vertical photobooth strip (4 photos, alternating you/boo) sized for Instagram Stories (9:16).

The whole app is a single file: **`index.html`** (named `index.html` so it deploys straight to a site root, e.g. on Netlify; ~1265 lines, HTML + CSS + JS in one). It works. It has been tested on two real devices and the core loop runs.

---

## SETUP FOR THE HUMAN (never used a terminal)

You need two things installed. Do this once.

1. **Install Node.js**: go to nodejs.org, download the "LTS" version, run the installer, click through it. That's it.
2. **Install Claude Code**: open the Terminal app (Mac: press Cmd+Space, type "Terminal", Enter. Windows: search "PowerShell"). Type this and press Enter:
   ```
   npm install -g @anthropic-ai/claude-code
   ```
3. **Open the project**: in the terminal, type `cd ` (with a space), then drag the pic-n-boo folder onto the terminal window and press Enter. Then type:
   ```
   claude
   ```
4. **To test the app locally** (so the camera works — it needs https or localhost):
   ```
   npx serve
   ```
   Then open the link it gives you (like localhost:3000). To test two people, open it on your phone using your computer's IP, or just deploy (see below).

Ask Claude Code to explain any of this if stuck.

---

## LOCKED RULES — do not violate these, they were hard-won

- **Logo top-left on every screen EXCEPT the start screen**, 130px, clickable → home. Start screen's big logo is its own header.
- **Normal spelling.** No "ur/u" text-speak. (User rejected it hard.)
- **NO em-dashes in copy.** Write like a human types — periods, short sentences.
- **Voice: funny-menace / romantic-playful-scary.** "Sweet with a knife behind it." Anchor line: "Be kind. Or don't." Current taglines are in the file; keep this register.
- **Grid rule: alternating strip.** 4 photos, top to bottom: host, guest, host, guest. (Was a 2×3 grid earlier; that's dead. It's a 1×4 vertical strip now.)
- **Two people only.** Not 1, not 3+. The two-person limit is the design.
- **Design = navy TV.** Deep navy (#24406B), animated TV static overlay, cyan (#4FE3FF) + hot pink (#E24A9B) accents, Anton font for poster text, Plus Jakarta Sans for body. Menu items and buttons are text that turns pink + slides + gains a cyan ▸ arrow on hover.

---

## THE MECHANIC (exact)

1. Host creates room → gets a 4-letter code. Guest joins with code.
2. Both see each other live (WebRTC video, falls back to snapshot slideshow).
3. Host presses Start → synced 3-2-1 countdown, 6 shots each. You watch YOUR BOO's shots pile up during shooting; you NEVER see your own.
4. Picking screen: you pick 2 of THEIR 6 shots. Pick a film filter (shared — both see the same one). Your cells in the preview stay "shut" (hidden).
5. Both hit "Lock it in."
6. Reveal: HOST presses BOO → TV static surges over the strip and clears → all 4 photos appear at once, including how your boo chose to show YOU.
7. Download Photo (JPEG, 1080×1920) or Video (MP4/WebM signal-tune-in animation).

---

## TECH STACK

- **Pure client-side.** One HTML file. No build step. No framework.
- **Supabase** (backend): realtime channels for sync, Postgres for sessions/photos, Storage for images.
  - URL: `https://ogmnssrzxhrqhzlycabe.supabase.co`
  - Anon key is in the file (public, safe to expose).
  - Tables `sessions` + `photos` and a `photos` storage bucket are already set up with permissive anon RLS. Rooms auto-expire ~7 days.
- **WebRTC** for live video (STUN only, no TURN — so ~10-20% of networks fall back to snapshots. This is expected, not a bug.)
- **MediaRecorder** for the video export (outputs MP4 where supported, else WebM).
- **IndexedDB** for the local gallery (per-browser, doesn't sync across devices).

---

## CURRENT STATE — what works, what's shaky

**Works (tested on 2 devices):** create/join, room codes, synced countdown, 6-shot capture, cross-pick, reveal, JPEG download.

**Recently fixed, needs re-testing on 2 devices:**
- Lock-in sync (was: both people lock but nobody advances). Rewrote with explicit `locked` broadcast + re-confirm ping.
- Rematch ("Again?" button) (was: only worked for host / got stuck). Now either person can trigger; host drives.
- Mobile bugs: iOS video `.play()`, audio-unlock-on-tap, `--vh` viewport fix, responsive start screen. **User reported "phone web is buggy" but never gave the specific symptom — chase this down first.**

**Known-fragile (WebRTC-era code, test hard):**
- WebRTC video connection (fails on strict networks → snapshot fallback).
- Reaction burst (#7): camera stays alive through the session to snap the user's face when BOO fires. Timing (650ms) is a guess.
- Video (MP4) export: format depends on browser; renders at 540×960 for speed.

**Features present:** rematch, full gallery (click the polaroid wall on start screen), timer taunt on picking, reaction burst, MP4 video export.

**Custom frame system:** drop a 1080×1920 PNG named `frame-1.png`..`frame-12.png` into `assets/frames/` with transparent photo windows → it auto-appears as a selectable frame. Window coords and a layout guide are in `assets/frames/`. Windows: x173, w734, h413, at y150/575/1000/1425.

---

## FILES

- **`index.html`** — THE APP. This is the only file that matters. Everything else is history.
- `app-snapshot-bak.html` — backup from before WebRTC was added. Proven-working snapshot-only version. Fall back to this if WebRTC breaks everything.
- `assets/logo.png` — the Pic A Boo wordmark (transparent bg).
- `assets/frames/` — custom frame system: `_layout-guide.png`, `README.md`.
- `start.html`, `lobby.html`, `shooting.html`, `picking.html`, `reveal.html`, `frame-none.html`, `frames.html`, `mockup.html` — standalone design prototypes. Superseded by the live app. Keep for reference, don't edit.
- `index2.html`, `index3.html` — old pre-redesign versions (glitch aesthetic, old mechanic). `index3.html` was the backend source the app was built from. Historical only. (Note: an even older `index.html` used to hold this same name before the live app file was renamed to `index.html` for Netlify — if you find a stray old glitch-era `index.html` alongside this one, that's the legacy file, not the app.)
- `GEMINI-FRAME-PROMPT.md` — prompt for generating a blue-glitch frame in Gemini (user wanted this; hadn't finished it).
- `TEST-CHECKLIST.md` — deploy + two-device test steps.

---

## DEPLOY (how the user tests with 2 devices)

Camera needs https or localhost. Fastest: drag the **contents** of the pic-n-boo folder (not the folder itself) to **app.netlify.com/drop**. The app is `index.html`, so the URL you get works as-is, no path suffix needed. Netlify anonymous drops are password-locked until the user claims the site (sign up, turn off password protection in Access & security).

---

## SUGGESTED FIRST MOVES FOR CLAUDE CODE

1. Get the app running locally (`npx serve`) and confirm it loads.
2. Chase the vague "phone is buggy" report — get the user to name the exact symptom, screen, and phone.
3. Re-test lock-in sync and rematch on two devices (recently rewritten, unverified).
4. Consider (with user's OK) splitting index.html into separate files — but only if the user wants it; one file is fine and simpler for a beginner.
5. The user values: honesty, being told when they're wrong, no sugarcoating, concise answers. They use "RETRY" (redo better), "Reply" (draft a reply), "Think" (pros/cons/how/conclusion), "Hold" (save topic) as command words.

---

## THE USER

First-time-ish coder, works a lot on mobile, learns by doing and revising. Strong design instincts (the 1×4 strip and "you never see your own photo" refinements were their calls and were right). Wants to be told the truth, including when an idea won't work. Don't flatter. Explain tradeoffs honestly. When they say "idk," they're handing you the decision — make it and explain why.
