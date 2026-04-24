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
  - **Profile Setup Screen**: 2-step flow (Google-prefilled name + goal selection → daily target/age/city/mobile). Shown once after first login.
  - **Welcome Tour Modal**: 4 paginated cards (Decks, Flashcards, Quiz, Notes) with progress dots and navigation controls.
  - **Contextual Hints**: Non-blocking tooltips on first use of Flashcards, Quiz, deck switching, and Notes. Auto-dismiss after 6s.
  - **Help & Guide Sidebar Panel**: 4 tabs (Getting Started quickstart, Features accordion, Tips & Shortcuts with keyboard shortcuts, Replay Tour to reset and restart tour).

---

## 5. AI & Word Acquisition

- **Google Gemini 1.5 Flash Vision**: Extracts vocabulary from book page photos via the "Scan Page" modal. Returns two lists: `underlined_words` (words visually underlined in the image) and `suggested_words` (up to 8 AI-picked hard/advanced words not underlined). Both lists are presented as toggleable chips — user accepts or rejects each word individually before they enter the Lexicon pipeline.
- **Dictionary Waterfall** (4-tier, in `addWords()`):
  1. **Free Dictionary API** — no key, covers ~80% of common words; fetches definition, IPA, audio URL, usage, synonyms, antonyms
  2. **Wordnik** — optional free key (`localStorage('wordnik_key')`); fires when Free Dict fails or returns no definition; multi-source definitions + examples
  3. **Claude Haiku 4.5 gap-fill** — uses `anthropic_key`; fires only when IPA, definition, or usage is still missing after Tiers 1–2; targeted prompt requesting only the missing fields
  4. **Web Speech API audio fallback** — zero cost, zero key, built-in; used at playback time when no audio URL is stored
- **Add Section Generate flow**: Preview-before-save for all content types (layout complete; full AI backend is next roadmap item)
- **Word Review UI**: After image scan, `#aiReviewSection` inside `#aiModal` shows two chip groups ("Underlined Words" / "AI-Suggested Hard Words"). Chips toggle selected/deselected on click. "Add Selected Words" feeds accepted words to the existing `addWords()` → Dictionary Waterfall → Firestore pipeline. "← Scan Another Image" resets the modal.

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
*Last Updated: April 23, 2026* (Onboarding & Help System)
