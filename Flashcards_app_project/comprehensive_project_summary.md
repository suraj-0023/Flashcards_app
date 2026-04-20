# Comprehensive Lexicon App Development Summary

This document provides a unified overview of the development journey, features, and future roadmap of the Lexicon application.

---

## 1. App Purpose & Pivot

Originally a vocabulary flashcards tool. As of April 2026, pivoted to a full **note-taking + flashcards + vocabulary learning platform** organised around "Decks" (formerly "Projects").

---

## 2. Current Layout & UX

### Sidebar (dark, ChatGPT-inspired)
- **New Deck** button → creates a named deck
- **Deck list** → clicking a deck filters the main page to that deck's content
- **Practice section** → links to Flashcards, Quiz, Stats (open as a full-area overlay)
- **Profile** → auth state, sign in/out, sync badge

### Main Area — Single Scrollable Page
1. **Deck Header** (sticky) — dot + deck name + total item count; updates on deck switch
2. **Add Section** (compact card)
   - Textarea for input
   - Toolbar: `[+]` file attach (Image/PDF/File) → dismissible chips | type pills `[Vocab][Notes][Flashcard][Quiz]` | deck `<select>` | `Generate →`
   - Vocab pill + text → fetches from dictionary API immediately
   - Other types → shows AI Preview section
3. **AI Preview** (hidden by default, appears after Generate →)
   - One card per selected type with `[Accept]` `[Edit]` `[Discard]`
   - Accept saves the item; dims card and shows ✓ Saved
4. **Library** (filter pills: All / Notes / Flashcards / Quizzes / Vocab)
   - Unified grid of all content types for the active deck
   - Vocab cards: existing dict-card design with accuracy badge
   - Note cards: title + preview + date
   - Flashcard cards: front + back preview

### Practice Overlay (Flashcards / Quiz / Stats)
- Accessed via sidebar Practice links
- Full-area overlay with `← Library` back button
- Contains existing flip-card flashcard session, multi-level quiz, and stats view — all unchanged

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

All keys namespaced by `userId` when signed in. Full Firestore sync on every mutation.

---

## 4. Core Learning System (unchanged)

- **Flashcards**: Self-rated (Got It / Need Practice), 3 modes (Word→Def, Def→Word, Usage→Word), includes custom cards
- **Quiz**: 3-tier difficulty (Easy/Medium/Hard), intelligent distractor logic, state-persistent sessions (Back/Next)
- **Stats**: Per-word accuracy breakdown across quiz and flashcard modes

---

## 5. AI & Word Acquisition

- **Google Gemini 1.5 Flash Vision**: Extracts underlined words from book page photos
- **Free Dictionary API**: Auto-fetches definition, phonetics, audio, usage for typed words
- **Add Section Generate flow**: Preview-before-save for all content types (layout complete; full AI backend is next roadmap item)

---

## 6. Tech Stack

- Single HTML file (`vocab_vscode.html`) — ~3300 lines
- Firebase Auth (Google + email) via CDN
- Firestore for cloud sync (single merged doc per user)
- localStorage as primary store (works offline)
- Fonts: Playfair Display, DM Sans, DM Mono
- No build step, no framework

---

## 7. Roadmap

1. **AI Generation Backend** — Wire Generate → button to actual AI (Gemini/Claude) for auto-creating notes, flashcards, quiz questions from pasted text
2. **Quiz Card Type** — Add `lexicon_quiz_cards` data model; display in Library and generate via AI
3. **Mobile / PWA** — Responsive layout, service worker, iOS/Android
4. **Branding & Deployment** — App name, hosting, app stores
5. **Cross-device sync** — Already functional via Firestore; needs edge-case testing

---
*Last Updated: April 20, 2026*
