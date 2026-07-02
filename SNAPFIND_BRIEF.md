# SNAPFIND — Project Brief & Build Instructions

> **How to use this file:** Open a Claude Code session on your laptop inside this repo
> and say: *"Read SNAPFIND_BRIEF.md and follow it. Start with Phase 1."*
> This file contains everything decided during research. Do not re-litigate decisions
> here unless something is technically impossible.

---

## 1. What we are building (one paragraph)

**Snapfind** — a desktop app for creators that answers one question: *"where is that
screenshot / recording / download?"* It watches the folders a creator already uses
(Screenshots, Recordings, Downloads, custom folders), reads the text inside every
image with **offline OCR**, and gives one search box. Type "invoice", "wifi password",
an error message, a sponsor's name — the matching file appears instantly, even from
months ago. v1.1 extends this to **video**: OCR sampled frames of screen recordings
and jump to the exact timestamp. 100% offline. No account. No cloud. No API keys.
One-time purchase.

**Tagline direction:** "Ctrl+F for everything on your screen." / "Stop scrolling
through 200 screenshots. Just type what you remember."

## 2. Who pays and why (validation summary)

- Target buyer: **content creators** (YouTubers, streamers, designers, devs who make
  content) + general power users. They accumulate hundreds of screenshots and screen
  recordings and constantly lose things.
- Proof people pay for this problem-family: a popular indie tool (recent-captures
  shelf, built by YouTuber Robert Hee, ~$7 one-time, ~20k users) solves the *recent*
  half. Snapfind solves the *old* half — finding the one from 3 weeks ago. Sibling
  product, NOT a copy.
- Direct competitor: **Screenotate** (Mac+Win, OCRs screenshots, searchable, local).
  Its weakness = it only indexes screenshots taken through ITS OWN hotkey. Snapfind
  indexes **the files you already have, retroactively**, from any capture tool.
  That sentence is the landing-page headline.
- Non-competitors: Snipping Tool "Text actions", PowerToys Text Extractor, Windows
  Photos "Scan text" — all copy text from ONE image, no library search. Google Photos
  needs cloud upload. Microsoft Recall = privacy backlash + Copilot+ hardware only.
  Position AGAINST Recall: "the private version".

## 3. Hard constraints (do not violate)

1. **Zero budget.** No paid services, no servers, no API keys, no telemetry,
   no accounts. Everything runs on the user's machine.
2. **Windows-only launch.** The founder has NO Apple hardware and no Apple Developer
   account ($99/yr). Do NOT ship macOS builds — an unnotarized paid Mac app shows
   "damaged and can't be opened" to customers. Keep code cross-platform-clean
   (path handling, no Windows-only APIs in core logic) so a Mac port is easy later,
   but build/test/ship Windows only. Landing page gets a "macOS coming soon — leave
   your email" waitlist instead.
3. **Timeline: 2 days.** Cut scope, not quality. Anything marked v1.1 slips if needed.
4. **Founder is in India, no Stripe.** Payments via a Merchant of Record:
   **Dodo Payments** (India-founded MoR, ~4–5.5% fees, INR payouts) or **Gumroad**
   (10% flat, simplest, auto file delivery + license keys). Both = $0 upfront.
5. Price: **$7–9 one-time** (launch price; can raise later). 7-day free trial.

## 4. Tech stack (decided)

- **Electron** + electron-builder (NSIS `.exe` installer). Chosen over Tauri because
  the founder's skills are HTML/JS and speed matters more than binary size.
- Renderer: **plain HTML/CSS/JS, no framework.** The founder is designing the UI
  himself in HTML (via claude.ai/design) — when he provides that HTML, use it as the
  renderer UI and wire it up. Until then, build with clean semantic markup that's
  easy to reskin.
- **better-sqlite3** with **FTS5** for the search index (instant prefix search).
- **Tesseract.js** for OCR — bundle the English traineddata so it works offline.
- **chokidar** for folder watching.
- **ffmpeg-static** for video thumbnailing + frame extraction (v1.1 video OCR).
- Offline licensing: **Ed25519-signed license keys**, public key embedded in app,
  verification fully offline. Include a separate Node script (NOT shipped in the app)
  that generates signed keys from the private key, for pasting into Gumroad/Dodo.

## 5. Feature spec

### Phase 1 — core (Day 1, must ship)
1. **First-run setup:** auto-detect `~/Pictures/Screenshots` and other OS defaults;
   user can add any folders (Downloads, OBS recordings folder, etc.).
2. **Background indexer:** scan existing images (png/jpg/jpeg/webp), OCR each, store
   normalized text + path + date + thumbnail in SQLite. Progress UI
   ("312 / 1,204 indexed"). Throttled so the machine stays responsive; resumes if the
   app closes mid-index. New files indexed within seconds via watcher.
3. **Search UI:** one big search box → thumbnail grid, instant-as-you-type, matched
   words highlighted. Searches OCR text AND filenames (filenames means non-image
   downloads are findable too). Filters: date range, folder, file type.
   Empty query shows most recent captures.
4. **Result actions:** copy image, copy OCR'd text, open file, reveal in folder,
   delete (with confirm).
5. **Tray app + global hotkey** (Ctrl+Shift+F) to summon search from anywhere;
   Esc hides. Launch-at-login toggle.
6. **Settings:** folders, re-index, hotkey, theme (follow OS light/dark).

### Phase 2 — money + polish (Day 2)
7. **Trial + license:** 7-day trial, then license key required. Offline Ed25519
   verification. Keygen script for the founder.
8. **Installer:** `npm run dist` → NSIS installer, app icon, sensible defaults.
9. **Landing page:** new page in this repo's existing static site (`apps/snapfind/`,
   matching the repo's vercel.json setup) — hero with the one-line pitch, demo video
   slot, "how it works" (3 steps), privacy section ("your files never leave your
   device"), pricing card ($7 one-time), buy button (Gumroad/Dodo link placeholder),
   **macOS waitlist email form** (use a free formspree-style embed or mailto fallback
   — zero budget), FAQ (vs Screenotate / vs Recall / refunds).

### v1.1 — the creator killer feature (only if Day 2 has slack; else week 2)
10. **Video search:** for mp4/mov/mkv in watched folders, extract 1 frame every 5s
    (ffmpeg-static), OCR frames, index with timestamps. Search result = video +
    thumbnail; clicking shows matching timestamps; "open at time" launches player.
    This is the "I have 270 recordings and can't find the one" feature NO competitor
    has. Indexing is slow — run it as lowest-priority background queue, images first.
11. Later ideas (do NOT build now): duplicate screenshot cleaner, "big old recordings"
    disk-space reclaimer, favorites/tags, semantic image search via local embeddings.

## 6. Quality bar

- Search results in <50ms on a 5,000-item index.
- Idle RAM (window hidden) under ~150MB.
- Handle gracefully: huge images, zero-byte files, moved/deleted files (prune index),
  locked files, non-English text (OCR gets what it gets — normalize to lowercase,
  collapse whitespace before indexing).
- No telemetry, no network calls at all (the app should work with Wi-Fi off — test it).

## 7. Two-day schedule

**Day 1:** Phase 1 complete and running on the founder's own screenshots folder by
tonight. Founder tests with real data, lists rough edges.
**Day 2 AM:** fixes + trial/licensing + installer.
**Day 2 PM:** landing page, founder sets up Gumroad/Dodo account, records the
30-second demo (type "invoice" → the right screenshot pops up), launches:
X/Twitter thread, r/Windows11, r/DataHoarder, r/productivity, Product Hunt later.

## 8. Business context (from research session, July 2026)

- The "zero competition + high pain + solo-buildable" unicorn does not exist; the
  winning shape is *painful problem + wrong-shaped existing solutions* — which this is.
- Realistic goal for launch: **first 10–20 sales ($70–150) + an email list + a public
  build story.** That validates. It is NOT expected to be a living immediately. The
  Mac waitlist size tells us whether the $99 Apple account is worth buying.
- Ideas researched and rejected this session (don't revisit): client proofing tools
  (GoVisually $9/mo exists), invoice chasing (bundled into FreshBooks/Bonsai/Wave),
  salon waitlist auto-fill (Fresha/GlossGenius ship it), AI-visibility/GEO trackers
  (20+ tools, $19–39/mo low end taken), Hazel-for-Windows file organizer (File Arbor
  + Sortio launched), compress-to-target-size tool (free client-side web tools own it,
  e.g. BulkPicTools), small-landlord software (crowded), AI-spam applicant screening
  (good idea but needs API + B2B sales — parked, wrong fit for $0 budget).

## 9. Working style for the local Claude session

- Build on branch `claude/saas-product-idea-xqoglu` (or a branch the founder names).
- Commit in small working increments with clear messages.
- App code lives in a new `snapfind/` directory at repo root (keep the static site in
  `apps/` untouched except adding the landing page).
- When the founder pastes his HTML design, adapt it into the renderer — keep his
  visual design, wire real data into it.
- The founder is learning: explain briefly what each major piece does when you
  introduce it, but don't lecture.
