# Comprehensive Lexicon App Development Summary

This document provides a unified overview of the development journey, features, and future roadmap of the Lexicon application.

---

## 1. App Purpose & Pivot

Originally a vocabulary flashcards tool. As of April 2026, evolved into a full **note-taking + flashcards + vocabulary learning platform** organized around "Decks" (formerly "Projects"). The app now combines personal notes, custom flashcards, and curated vocabulary learning in a single, gamified experience inspired by Duolingo and Brainscape.

---

## 2. Current Layout & UX (M3 Warm/Gamified Design)

### Sidebar (M3 White + Green-Tinted Gradient)
- **Logo**: "L" mark in emerald (#10B981)
- **New Deck** button → pill-shaped, emerald, creates a named deck
- **Deck list** → emerald pill active state, white/muted default; clicking filters the main page to that deck's content
- **Practice section** → SVG icons (flashcard, vocab, chart); links to Flashcards, Vocab, Stats (open as full-area overlay)
- **Profile** → auth state, sign in/out, sync badge
- **Background**: White with subtle #F0FDF4 green-tinted gradient; 14px border-radius on deck pills

### Main Area — Single Scrollable Page
1. **Deck Header** (sticky) — dot + deck name + total item count; updates on deck switch
2. **Add Section** (M3 rounded card with 18px radius)
   - Textarea for input
   - Toolbar: `[+]` file attach (Image/PDF/File) → dismissible chips | type pills `[Vocab][Notes][Flashcard]` | deck `<select>` | `Generate →`
   - Vocab pill + text → fetches from Free Dictionary API immediately
   - Other types → shows AI Preview section
   - Buttons: pill-shaped, emerald primary, drop-shadow
3. **AI Preview** (hidden by default, appears after Generate →)
   - One card per selected type with `[Accept]` `[Edit]` `[Discard]`
   - Accept saves the item; dims card and shows ✓ Saved
4. **Library** (2-column card grid)
   - Unified grid of all content types for the active deck
   - **Vocab cards**: M3 aesthetic, 14px radius, color-coded 4px left border (green for mastered, amber for learning, red for new), circular SVG score ring badge, accuracy label
   - **Note cards**: 14px radius, title + content preview + creation date, color-coded left border
   - **Flashcard cards**: 14px radius, front + back preview, color-coded left border
   - Each card shows a small deck-name badge in the top corner
5. **Background**: Warm #FFF7ED, M3 rounded aesthetic throughout

### Practice Overlay (Flashcards / Vocab / Stats)
- Accessed via sidebar Practice links
- Full-area overlay with `← Library` back button
- **Flashcards**: 18px radius cards with amber (learning) or emerald (mastered) top border; flip-card animation; self-rated (Got It / Need Practice); 3 modes (Word→Def, Def→Word, Usage→Word)
- **Vocab**: Landing screen with two options (Vocab Flashcards, Quiz); Quiz shows multi-tier difficulty (Easy/Medium/Hard) with intelligent distractor logic
- **Stats**: Displays per-word accuracy breakdown across quiz and flashcard modes
- All rounded, M3-compliant design

### Login Screen
- Warm card design (#FFF7ED-ish background)
- Emerald CTA buttons (Google Sign-In, Email Sign-In)
- M3 rounded corners (18px), drop-shadow
- Responsive, centered layout

---

## 3. Data Layer (localStorage + Firestore)

| Key | Contents |
|-----|----------|
| `lexicon_projects` | Deck list `[{id, name}]` |
| `custom_vocab` | User-added vocabulary entries |
| `deleted_vocab` | Blacklisted core word IDs |
| `lexicon_custom_cards` | Front/back custom flashcards `{id, front, back, projectId}` |
| `lexicon_notes` | Notes `{id, deckId, title, content, createdAt, updatedAt}` |
| `lexicon_perf_*` | Per-word performance stats scoped by deck |
| `lexicon_demo_seeded` | Flag (user-namespaced) indicating demo decks have been merged for this account |

All keys namespaced by `userId` when signed in. Full Firestore sync on every mutation. Security enforced server-side: each user can only read/write their own document. Guest data is un-namespaced and cleared on sign-out or new guest session.

---

## 4. Core Learning System

- **Flashcards**: Self-rated (Got It / Need Practice), 3 modes (Word→Def, Def→Word, Usage→Word), includes custom cards. Deck selection screen before practice; state-persistent sessions (Back/Next).
- **Vocab Practice**: Landing screen with two options:
  1. Vocab Flashcards — flip-card review of all vocab in selected decks
  2. Quiz — 3-tier difficulty (Easy/Medium/Hard), intelligent distractor selection, multi-level progression
- **Stats**: Per-word accuracy breakdown across quiz and flashcard modes; visual accuracy indicators on library cards
- **Onboarding & Help System** (Apr 2026):
  - **Profile Setup Screen**: 2-step flow (Google-prefilled name + goal selection → daily target/age/city/mobile). Redesigned Apr 24 with animated progress bar, gradient step icons, goal validation with shake animation, directional slide transitions, and mobile bottom-sheet layout. Shown once after first login.
  - **Welcome Tour Modal**: 4 paginated cards (Decks, Flashcards, Quiz, Notes) redesigned Apr 24 with CSS-animated hero illustrations per slide (floating deck cards, 3D flip card, cascading quiz bars, typewriter notepad), per-slide accent badges, swipe gestures, and goal-aware copy. Navigation with progress dots and controls.
  - **Onboarding Completion**: New first-word ceremony overlay (Apr 24) with serif typewriter effect, word-by-word definition stagger animation, and Day 1 streak badge celebration.
  - **Contextual Hints**: Non-blocking tooltips on first use of Flashcards, Quiz, deck switching, and Notes. Auto-dismiss after 6s. Apr 24 fix: hint viewport clamp prevents overflow.
  - **Help & Guide Sidebar Panel**: 4 tabs (Getting Started quickstart, Features accordion, Tips & Shortcuts with keyboard shortcuts, Replay Tour to reset and restart tour).

---

## 5. AI & Word Acquisition

- **Google Gemini 2.0 Flash Vision**: Extracts vocabulary from book page photos via two flows (no API key input required):
  1. **Scan Page Modal** (dedicated UI): Returns underlined_words and suggested_words as toggleable chips; user accepts/rejects each before adding to Lexicon
  2. **Generate Flow** (main Add Section): Attaching an image + selecting Vocab opens a popup word review modal with all extracted words (underlined + AI-suggested); per-word Accept / Edit / Decline buttons allow fine-grained control before saving to library
- **Dictionary Waterfall** (4-tier, in `addWords()`):
  1. **Free Dictionary API** — no key, covers ~80% of common words; fetches definition, IPA, audio URL, usage, synonyms, antonyms, all definitions grouped by part of speech
  2. **Wordnik** — optional free key (`localStorage('wordnik_key')`); fires when Free Dict fails or returns no definition; multi-source definitions + examples
  3. **Gemini 2.0 Flash gap-fill** — routed through Cloudflare Worker proxy for security; fires automatically when IPA, definition, or usage is still missing after Tiers 1–2; targeted prompt requesting only the missing fields
  4. **Web Speech API audio fallback** — zero cost, zero key, built-in; used at playback time when no audio URL is stored
- **Difficulty Scoring System** (Apr 27, 2026):
  - `_scoreDifficulty(wordData)` computes 1–4 score (Easy→Very Hard) based on audio presence, synonym count, and word length
  - Used in `addWords()` for all words; stored in `word.difficulty` field
  - Difficulty badges shown in word tiles (image scan popup), word detail modals (as coloured pill), and library cards (as small coloured dot bottom-right)
  - Colour scheme: green=easy, blue=medium, orange=hard, red=very hard
- **Add Section Generate flow**: Preview-before-save for all content types:
  - Text + Vocab: Shows AI Preview card with fetched definition from Free Dictionary API before saving
  - Image + Vocab: Opens popup modal with two labelled sections ("📖 Underlined Words" / "✨ AI-Suggested Words") displaying extracted words as styled tiles with async-fetched definitions; per-word Accept / Edit / Decline buttons
  - Other types (Notes/Flashcards): Generic inline preview with Accept / Edit / Discard
- **Image Scan Popup UI** (Apr 27, 2026 redesign):
  - `showImageVocabModal(underlined, suggested)` renders two distinct sections separated by a visual divider
  - Each word displayed as a styled `.img-vocab-tile` with phonetic, part-of-speech, and definition fetched async from Free Dictionary API on modal open
  - Difficulty badge (coloured pill) displayed next to word; computed by `_scoreDifficulty(wordData)`
  - Per-word Accept / Edit / Decline buttons for fine-grained control before saving
- **Word Detail Modal Enhancement** (Apr 27, 2026):
  - Difficulty badge (coloured pill) now shown next to word name
  - "All Definitions" section displays all definitions grouped by part of speech; visible only if more than 1 definition
  - "Synonyms" section (renamed from "Related Words") shows related words from API
- **Library Cards Difficulty Indicator** (Apr 27, 2026):
  - Small coloured dot (4px) positioned bottom-right on each vocab card
  - Colour indicates difficulty: green=easy, blue=medium, orange=hard, red=very hard
- **Quiz Distractor Generation**: Powered by Gemini 2.0 Flash (hardcoded API key); automatically generates 3 plausible wrong answers for each quiz question; no user API key input required.

---

## 6. Tech Stack

- Single HTML file (`app.html`) — ~5800 lines
- Firebase Auth (Google + email) via CDN
- Firestore for cloud sync (single merged doc per user)
- localStorage as primary store (works offline); namespaced by userId
- **Font**: Plus Jakarta Sans (replacing Playfair Display, DM Mono, DM Sans)
- **Design System**: Material Design 3 (M3) — warm palette, pill buttons, rounded corners, color-coded accents
- **Colors**:
  - Warm background: #FFF7ED
  - Primary (emerald): #10B981
  - Learning (amber): #F59E0B
  - New (red): #EF4444
  - Sidebar gradient: white → #F0FDF4
- No build step, no framework
- **Gemini API Security**: API key stored securely as a Cloudflare Worker secret; all fetch calls routed through proxy endpoint (`https://gemini-proxy.suraj-kunuku23.workers.dev`) to prevent client-side exposure

---

## 7. Guest Mode & Sample Decks

- **Guest Users**: See 4 pre-seeded example decks on first visit:
  1. Atomic Habits
  2. How to Avoid a Climate Disaster (Bill Gates)
  3. The Alchemist
  4. Class 10 Science
- Each deck contains 6–7 notes, 7 flashcards, and 6–7 vocabulary items for immediate exploration.
- Guest data is un-namespaced in localStorage and cleared on sign-out.
- **Logged-In Users**: See the same example decks on first login (merged via `mergeDemoDataForUser()` after `pullFromCloud` resolves); skips seeding if user already has custom data. The General deck is always preserved.

---

## 8. Roadmap

1. **AI Generation Backend** — Wire Generate → button to actual AI (Gemini/Claude) for auto-creating notes, flashcards, quiz questions from pasted text or uploaded images
2. **Quiz Card Type** — Add `lexicon_quiz_cards` data model; display in Library and generate via AI
3. **Mobile / PWA** — Responsive layout, service worker, iOS/Android
4. **Branding & Deployment** — Custom domain, app stores (iOS/Android), GitHub Pages
5. **Cross-device sync** — Already functional via Firestore; needs edge-case testing
6. **Advanced Stats** — Spaced repetition scheduling, learning curves, time-to-mastery estimates
7. **Social Features** — Deck sharing, collaborative learning, leaderboards

---
*Last Updated: April 27, 2026* (Rich image-scan popup with difficulty system, tile layouts, async definitions)
