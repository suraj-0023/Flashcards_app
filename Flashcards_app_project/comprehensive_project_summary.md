# Comprehensive Nexora App Development Summary

This document provides a unified overview of the development journey, features, and future roadmap of the Nexora application.

**Last Updated**: 2026-04-30 (Context-aware definition selection via Gemini, synonyms & antonyms enrichment added)

---

## 1. App Purpose & Pivot

Originally a vocabulary flashcards tool. As of April 2026, evolved into a full **note-taking + flashcards + vocabulary learning platform** organized around "Decks" (formerly "Projects"). The app combines personal notes, custom flashcards, and curated vocabulary learning in a single, gamified experience inspired by Duolingo and Brainscape.

---

## 2. Current Layout & UX (M3 Warm/Gamified Design)

### Sidebar (M3 White + Green-Tinted Gradient, 220px width)
- **Logo**: "N" mark in emerald (#10B981) (Nexora)
- **New Deck** button → pill-shaped, emerald, creates a named deck
- **Deck list** → emerald pill active state, white/muted default; each deck shows 4-dot mastery quartile indicator (based on word accuracy ≥75% across ≥3 attempts)
- **Daily Queue Badge** → placeholder (hidden, reserved for Phase 4 SM-2 spaced repetition engine)
- **Practice section** → SVG icons (flashcard, vocab, chart); links to Flashcards, Vocab, Stats (open as full-area overlay)
- **Profile** → auth state, sign in/out, sync badge
- **Background**: White with subtle #F0FDF4 green-tinted gradient; 14px border-radius on deck pills

### Main Area — Single Scrollable Page
1. **Deck Header** (sticky) — dot + deck name + total item count; updates on deck switch; includes ⌘K Search ghost pill and "+ Add" button (Cmd+N shortcut)
2. **Daily Queue Badge** (sidebar) — Shows total items due today; expandable breakdown showing count of new, review, and lapsed items; dismissible with ✕ button (Apr 30)
3. **Add Modal** (Cmd+N to open, Escape to close)
   - **Multi-select content type checkboxes**: Vocab, Note, Flashcard (combinable), and Image/PDF (exclusive)
   - When 2+ types selected: single textarea input → first line becomes vocab word + flashcard front; full text becomes note body; "Save All →" saves to all selected types in one action
   - When Image/PDF selected: file picker (5 MB limit) replaces text input; "Scan File →" runs vision AI extraction
   - **Context Sentence** field for Vocab (optional) — displayed on flashcard back and in word detail modal (Apr 30)
   - Word/title input field (hidden in multi-type and image/PDF modes)
   - Deck selector dropdown
   - Type-specific fields (title+body for Note, front+back for Flashcard)
   - Preview step before save: vocab shows definition preview; note/flashcard show content preview; multi-type shows all previews stacked
   - Cmd+Enter or button click to submit; scroll position preserved after add
   - Smart routing: Vocab → `_addWordsFromText()` for enrichment, Note/Flashcard → direct object creation, Image/PDF → `generateVocabFromImage()`
4. **Library** (2-column card grid, scroll position preserved)
   - Unified grid of all content types for the active deck
   - **Vocab cards**: M3 aesthetic, 14px radius, color-coded 4px left border (green for mastered, amber for learning, red for new), SM-2 state pill + "Due in Xd" chip (Apr 30), circular SVG score ring badge, accuracy label
   - **Note cards**: 14px radius, title + italic content preview + creation date, color-coded left border, "edited Xd ago" timestamp (Apr 30)
   - **Flashcard cards**: 14px radius, front + back preview, SM-2 state pill (Apr 30), color-coded left border
   - Each card shows a small deck-name badge in the top corner
5. **Background**: Warm #FFF7ED, M3 rounded aesthetic throughout

### Practice Overlay (Flashcards / Vocab / Stats)
- Accessed via sidebar Practice links
- Full-area overlay with `← Library` back button
- **Flashcards**: 18px radius cards with amber (learning) or emerald (mastered) top border; flip-card animation; self-rated (Got It / Need Practice); 3 modes (Word→Def, Def→Word, Usage→Word)
  - **Keyboard support** (Apr 30): Space to flip, 1 to mark wrong, 2 to mark correct; visual hint shows available shortcuts
  - **Context sentence** displays on card back when available (Apr 30)
  - **Post-rating toast** provides feedback after rating a card (Apr 30)
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
| `nexora_projects` | Deck list `[{id, name}]` |
| `custom_vocab` | User-added vocabulary entries with SM-2 tracking `{id, word, definition, sm2, context, dataSources[], synonyms[], antonyms[], ...}` (Apr 30) |
| `deleted_vocab` | Blacklisted core word IDs |
| `nexora_custom_cards` | Front/back custom flashcards `{id, front, back, projectId, sm2, ...}` (Apr 30 SM-2) |
| `nexora_notes` | Notes `{id, deckId, title, content, createdAt, updatedAt}` |
| `nexora_perf_*` | Per-word performance stats scoped by deck |
| `nexora_demo_seeded` | Flag (user-namespaced) indicating demo decks have been merged for this account |

**SM-2 Schema** (Apr 30): Each vocab word and custom card tracks spaced repetition state:
```
sm2: {
  interval: number,        // days until next review (starts at 1, grows geometrically)
  easiness: number,        // SM-2 difficulty factor (2.5 default, 1.3–2.5 range)
  repetitions: number,     // successful review count
  dueDate: string          // ISO date, when this item is next due for review
}
```

All keys namespaced by `userId` when signed in. Full Firestore sync on every mutation. Security enforced server-side: each user can only read/write their own document. Guest data is un-namespaced and cleared on sign-out or new guest session.

---

## 4. Core Learning System

- **Flashcards**: Self-rated (Got It / Need Practice), 3 modes (Word→Def, Def→Word, Usage→Word), includes custom cards. Deck selection screen before practice; state-persistent sessions (Back/Next).
- **SM-2 Spaced Repetition** (Apr 30, 2026):
  - Pure functions `defaultSM2()` and `applyRating(sm2, correct)` implement the SM-2 algorithm for interval scheduling
  - Every vocab word and custom card tracks `sm2: {interval, easiness, repetitions, dueDate}`
  - `getDueCount()` computes daily review workload across three categories: new items, items in review cycle, and lapsed items
  - `updateDailyBadge()` displays sidebar badge with breakdown; dismissible with ✕ button
  - Library cards show SM-2 state pill (New/Learning/Mastered) and "Due in Xd" chip
  - Word detail modal shows colored state badge + due date
  - `startSM2Review()` builds sorted daily queue (lapsed → review → new priority) and opens practice view
- **Vocab Practice**: Landing screen with two options:
  1. Vocab Flashcards — flip-card review of all vocab in selected decks
  2. Quiz — 3-tier difficulty (Easy/Medium/Hard), intelligent distractor selection, multi-level progression
- **Stats**: Per-word accuracy breakdown across quiz and flashcard modes; visual accuracy indicators on library cards
- **Search & Command Palette** (Apr 30, 2026):
  - Cmd+K opens full-screen search modal with real-time results filtering
  - Search across all content types (vocab, notes, flashcards) with deck-aware filtering
  - Navigate results with arrow keys; open with Enter
  - Escape closes the modal
- **Keyboard Shortcuts** (Apr 30, 2026):
  - Space: flip flashcard during study
  - 1: mark flashcard as wrong / need practice
  - 2: mark flashcard as correct / mastered
  - Cmd+N: open Add modal
  - Cmd+K: open search palette
  - Visual hint element displays available shortcuts
- **Onboarding & Help System** (Apr 2026):
  - **Profile Setup Screen**: 2-step flow (Google-prefilled name + goal selection → daily target/age/city/mobile). Redesigned Apr 24 with animated progress bar, gradient step icons, goal validation with shake animation, directional slide transitions, and mobile bottom-sheet layout. Shown once after first login.
  - **Welcome Tour Modal**: 4 paginated cards (Decks, Flashcards, Quiz, Notes) redesigned Apr 24 with CSS-animated hero illustrations per slide (floating deck cards, 3D flip card, cascading quiz bars, typewriter notepad), per-slide accent badges, swipe gestures, and goal-aware copy. Navigation with progress dots and controls.
  - **Onboarding Completion**: New first-word ceremony overlay (Apr 24) with serif typewriter effect, word-by-word definition stagger animation, and Day 1 streak badge celebration.
  - **Contextual Hints**: Non-blocking tooltips on first use of Flashcards, Quiz, deck switching, and Notes. Auto-dismiss after 6s. Apr 24 fix: hint viewport clamp prevents overflow.
  - **Help & Guide Sidebar Panel**: 4 tabs (Getting Started quickstart, Features accordion, Tips & Shortcuts with keyboard shortcuts, Replay Tour to reset and restart tour).

---

## 5. AI & Word Acquisition

- **Google Gemini 2.0 Flash Vision**: Extracts vocabulary from book page photos via two flows (no API key input required):
  1. **Scan Page Modal** (dedicated UI): Returns underlined_words and suggested_words as toggleable chips; user accepts/rejects each before adding to Nexora
  2. **Generate Flow** (main Add Section): Attaching an image + selecting Vocab opens a popup word review modal with all extracted words (underlined + AI-suggested); per-word Accept / Edit / Decline buttons allow fine-grained control before saving to library Nexora
- **Batch AI Tile Preview** (Apr 28, 2026):
  - `_batchFetchTileDefinitions(words)` calls Gemini once for all image-scan words together instead of calling Free Dictionary API per-word
  - AI responses cached in `window._tileWordData` for reuse when user accepts a word
  - Significantly reduces API calls during tile preview (single batch call vs. N sequential calls)
- **Smart Word Enrichment Save** (Apr 28, 2026):
  - `_saveWordWithEnrichment(word)` saves immediately with cached AI seed data (card appears instantly in library), then enriches asynchronously
  - Async enrichment pipeline: Free Dict → Wordnik → Gemini gap-fill (only for missing fields)
  - Modal auto-refreshes in real-time as background enrichment completes (via `_openModalWordId` tracker)
  - Eliminates duplicate API calls and no-word alert popups
- **Multi-Source Data Merge** (Apr 28, 2026):
  - `_mergeWordSources(freeDict, wordnik, aiData)` merges definitions from multiple sources with smart dedup logic
  - Picks best non-circular definition; merges up to 5 synonyms + 5 antonyms; dedupes usage examples
  - New `dataSources[]` field on saved words tracks which APIs contributed (e.g., `["Free Dictionary", "AI"]`)
  - Word card modal displays source attribution at bottom ("Sources: Free Dictionary, AI")
- **Dictionary Waterfall** (5-tier, in `_enrichWordAsync()` called after save):
  1. **Free Dictionary API** — no key, covers ~80% of common words; fetches definition, IPA, audio URL, usage, synonyms, antonyms, all definitions grouped by part of speech
  2. **Tier 1.5: Context-Aware Definition Selection via Gemini** — fires after Free Dict when multiple definitions exist (>2); sends word + all definitions + deck title + vocab sample to Gemini proxy; returns 1-based indices of 2–3 most contextually relevant definitions; user sees only the definitions relevant to their deck
  3. **Wordnik** — optional free key (`localStorage('wordnik_key')`); fires when Free Dict fails or returns no definition; multi-source definitions + examples
  4. **Gemini 2.0 Flash gap-fill** — routed through Cloudflare Worker proxy for security; fires automatically when IPA, definition, usage, synonyms, or antonyms are still missing after previous tiers; targeted prompt requesting only missing fields; now also fetches up to 2 synonyms and 2 antonyms per word
  5. **Web Speech API audio fallback** — zero cost, zero key, built-in; used at playback time when no audio URL is stored
- **Difficulty Scoring System** (Apr 27, 2026):
  - `_scoreDifficulty(wordData)` computes 1–4 score (Easy→Very Hard) based on audio presence, synonym count, and word length
  - Used in `addWords()` for all words; stored in `word.difficulty` field
  - Difficulty badges shown in word tiles (image scan popup), word detail modals (as coloured pill), and library cards (as small coloured dot bottom-right)
  - Colour scheme: green=easy, blue=medium, orange=hard, red=very hard
- **Add Section Generate flow**: Preview-before-save for all content types:
  - Text + Vocab: Shows preview card with fetched definition from Free Dictionary API (phonetic, meaning, example); buttons change to "← Edit" / "✓ Save" before committing
  - Text + Note: Shows title and content preview cards
  - Text + Flashcard: Shows Front and Back labels clearly in preview
  - Multi-type selections (Vocab + Note + Flashcard): All selected type previews stacked together
  - Image + Vocab: Opens popup modal with two labelled sections ("📖 Underlined Words" / "✨ AI-Suggested Words") displaying extracted words as styled tiles with async-fetched definitions; per-word Accept / Edit / Decline buttons
  - Image/PDF flow unchanged (no preview step, goes straight to word review)
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

## 6. Code Navigation System

The app.html file contains 42 `@@SECTION` markers that divide the code into logical, named regions (~100–300 lines each). This allows future development agents to quickly jump to relevant sections without reading the entire 11,180-line file.

**How to use:**
1. Find the section name in `Flashcards_app_project/CODE_MAP.md`
2. Run: `grep -n "@@SECTION: SectionName" app.html` to get the line number
3. Read only that section in the editor

**Key sections by feature:**
- **Onboarding**: JSOnboarding (6539), JSHelpPanel (6859), JSAuthStateHandler (6998)
- **Data & Storage**: JSStorage (6437), JSDemoData (7273), JSLoadJSON (7421)
- **Learning Features**: JSFlashcards (8845), JSQuiz (9142), JSStats (9373), JSVocabPractice (9773)
- **UI & Modals**: JSModalDialog (8172), JSAddModal (10561), JSImageVocab (8515)
- **Navigation**: JSLibrary (9850), JSLibraryListView (9884), JSPracticeOverlay (9524)

See `CODE_MAP.md` for the complete index.

---

## 7. Tech Stack

- Single HTML file (`app.html`) — ~11,180 lines
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

## 8. Guest Mode & Sample Decks

- **Guest Users**: See 4 pre-seeded example decks on first visit:
  1. Atomic Habits
  2. How to Avoid a Climate Disaster (Bill Gates)
  3. The Alchemist
  4. Class 10 Science
- Each deck contains 6–7 notes, 7 flashcards, and 6–7 vocabulary items for immediate exploration.
- Guest data is un-namespaced in localStorage and cleared on sign-out.
- **Logged-In Users**: See the same example decks on first login (merged via `mergeDemoDataForUser()` after `pullFromCloud` resolves); skips seeding if user already has custom data. The General deck is always preserved.

---

## 9. Roadmap

1. **AI Generation Backend** — Wire Generate → button to actual AI (Gemini/Claude) for auto-creating notes, flashcards, quiz questions from pasted text or uploaded images
2. **Quiz Card Type** — Add `nexora_quiz_cards` data model; display in Library and generate via AI
3. **Mobile / PWA** — Responsive layout, service worker, iOS/Android
4. **Branding & Deployment** — Custom domain, app stores (iOS/Android), GitHub Pages
5. **Cross-device sync** — Already functional via Firestore; needs edge-case testing
6. **Advanced Stats** — Spaced repetition scheduling, learning curves, time-to-mastery estimates
7. **Social Features** — Deck sharing, collaborative learning, leaderboards

---
*Last Updated: April 30, 2026* (App renamed to Nexora, @@SECTION markers and CODE_MAP.md code navigation system added)
