# Project Evolution Log

## 2026-04-23 - Bug Fixes: Null-Safety, Async/Await Race Conditions (Issues #16–#19)

- **What**: Fixed 10 null-safety and async race condition bugs across dictionary waterfall, word modal, quiz navigation, and stats filter
- **Why**: GitHub issues #16–#19 documented crashes when API responses were empty, `mAudioBtn` was missing from DOM, quiz navigation fired before async options loaded, and stats accessed `.flash` on words added before perf tracking existed
- **Impact**: Word addition, quiz flow, and stats view no longer crash under edge-case data; quiz options always fully loaded before display
- **Technical Detail**:
  - `addWords()`: guard `data[0]` with `if (!data || !data.length) throw`; guard `usage[0].startsWith` with existence check
  - `_parseFreeDictEntry()`: `(m.definitions || []).forEach()`
  - `_fetchWordnik()`: `if (Array.isArray(rel)) rel.forEach()`
  - `_haikuEnrichWord()`: wrap `JSON.parse(text)` in try-catch
  - `openModal()`: early-return guard after `getElementById('mAudioBtn')`
  - `_showNewQuizArea()`: quiz nav onclick handlers made `async`, `await _renderNqCard()`
  - `restartCurrentQuiz()`: made `async`, `await startNotesQuiz()` / `await startVocabQuizDirect()`
  - `_getFilteredBase()`: add `perf[w.id].flash &&` before accessing `.flash.c`/`.flash.w` in unseen filter
  - `selectNqOption()`: guard `if (!track || track.answered) return`

## 2026-04-23 - Dictionary Waterfall + AI Gap-Fill + Web Speech Audio Fallback

- **What**: Replaced single-tier Free Dictionary API call in `addWords()` with a 4-tier waterfall: (1) Free Dictionary API, (2) Wordnik fallback (optional key), (3) Claude Haiku 4.5 gap-fill for missing IPA/definition/usage, (4) Web Speech API audio fallback at playback time. Audio button in word modal now uses `speechSynthesis` when no audio URL is stored.
- **Why**: Core vocabulary entries and obscure words often returned incomplete data (missing IPA, no audio, vague definition). The waterfall progressively enriches each word at near-zero cost.
- **Impact**: Words added via the vocab input now receive IPA phonetics, audio, and usage examples from the best available source. All existing words gain Web Speech API pronunciation via the modal audio button even without a stored URL.
- **Technical Detail**: `addWords()` refactored with three new helpers: `_parseFreeDictEntry()` (extracts structured data from Free Dict response), `_fetchWordnik()` (definitions + examples + related words via Wordnik API; key in `localStorage('wordnik_key')`), `_haikuEnrichWord()` (targeted Haiku 4.5 call — only requests fields that are still missing; reuses `anthropic_key` pattern). Audio button handler updated: `new Audio(url)` if URL present, else `speechSynthesis.speak()` if available, else hidden.

## 2026-04-23 - Dropdown Clipping Fix & AI Image Word Review UI

- **What**: Fixed the `+` file-attach button dropdown being clipped by parent containers. Upgraded the AI "Scan Page" image flow from a crude `window.confirm()` dialog to a full in-modal word review UI with two toggleable chip sections.
- **Why**: The dropdown was being cut off at the `.add-section-card` boundary due to `overflow: hidden` on that container and `overflow-y: auto` on `.main-scroll-area`, making it unusable. The old confirm dialog gave no control over which words to add. User wanted to accept/reject individual words and also receive AI-suggested hard words beyond just the underlined ones.
- **Impact**: The `+` dropdown now renders fully visible regardless of scroll position. The Scan Page flow now shows underlined words and up to 8 AI-suggested hard words as toggleable chips — users can deselect any word before adding to Lexicon. A "← Scan Another Image" link resets the modal for a second scan.
- **Technical Detail**:
  - `.file-dropdown` CSS changed from `position: absolute` to `position: fixed` with `z-index: 9999`
  - `toggleFileDropdown()` now calls `e.currentTarget.getBoundingClientRect()` and sets `menu.style.top`/`left` dynamically, bypassing all overflow clipping
  - Gemini Vision prompt updated to return `{ "underlined_words": [...], "suggested_words": [...] }` (up to 8 hardest non-underlined words)
  - `processImage()` parses both arrays with backward-compat fallback to legacy `words` key
  - New `#aiReviewSection` HTML block added inside `#aiModal` with two chip group divs, "Add Selected Words" button, and reset link
  - New CSS: `.ai-word-chip` pill (toggle selected/deselected via `.deselected` class), `.ai-review-add-btn`, `.ai-review-reset-link`
  - New functions: `showWordReview(underlined, suggested)`, `confirmWordSelection()`, `resetAiModal()`
  - `CLAUDE.md` updated to add Step 0 (update `evolution.md` + `comprehensive_project_summary.md` before every push)

## 2026-04-22 - M3 Warm/Gamified Design Implementation

- **What**: Applied a complete Material Design 3 (M3) warm/gamified aesthetic inspired by Duolingo and Brainscape across the entire app. Replaced all fonts (Playfair Display / DM Mono / DM Sans) with Plus Jakarta Sans. Converted the dark ChatGPT-inspired sidebar to white with a green-tinted gradient (#F0FDF4), emerald pill-shaped nav with SVG icons, and "L" logo mark. Redesigned cards with 14px radius, color-coded 4px left borders (green for mastered, amber for learning, red for new), circular SVG score rings, and color-coded top borders on flashcards (amber for learning, emerald for mastered). Updated all buttons to pill-shaped with emerald primary and drop-shadow. Changed app background to warm #FFF7ED. Applied M3 rounded aesthetic (18px radius) to the Add panel, flashcards, modals, quiz, and stats views. Login screen now displays as a warm card with emerald CTA buttons.
- **Why**: To create a more engaging, gamified, and visually cohesive user experience that encourages learning through warm colors and playful design language.
- **Impact**: The app now feels modern, cohesive, and premium. The warm palette and M3 design system make vocabulary learning feel more rewarding and less sterile.
- **Technical Detail**: Font changed to Plus Jakarta Sans throughout; palette includes #FFF7ED (warm bg), #10B981 (emerald), #F59E0B (amber), #EF4444 (red), #F0FDF4 (sidebar gradient). All components (sidebar, cards, buttons, modals, flashcards, quiz) updated to M3 specifications.

## 2026-04-21 - Per-User Data Isolation & Sample Decks for All Users

- **What**: Fixed demo deck seeding for logged-in users: `seedDemoData` is now guest-only; new `mergeDemoDataForUser()` runs after `pullFromCloud` resolves to merge demo decks without overwriting real user data (triggered once per account via `lexicon_demo_seeded` flag). Added delete button (🗑️) for every sidebar deck; deletion moves items to the first remaining deck instead of hardcoded General. Replaced Library list view with a 2-column card grid showing content preview and deck-name badge per card.
- **Why**: Users wanted to see example decks even after signing in, and the previous approach (seeding before cloud pull) meant existing users never got the demo content. Also improved library browsing with a visual card-based grid.
- **Impact**: All users (guest and signed-in) see curated example decks with sample content. Any deck can be deleted without data loss.
- **Technical Detail**: Added `mergeDemoDataForUser()` with `lexicon_demo_seeded` guard flag (user-namespaced, per-account). Library grid now renders 2 columns of cards with preview content and deck badge.

## 2026-04-21 - GitHub Push Process & Project Conventions

- **What**: Added `CLAUDE.md` at project root documenting the mandatory 3-step GitHub workflow (commit → create issue → close issue), curl-based GitHub API approach (since gh CLI is not installed), commit message conventions (feat/fix/docs/refactor), and project overview for future Claude Code sessions.
- **Why**: To establish a repeatable, documented process for pushing code and creating issue records without relying on third-party CLI tools.
- **Impact**: Future development sessions have a clear, copy-paste-ready workflow for all GitHub interactions.

## 2026-04-21 - Example Decks with Full Content & Complete Library Rename

- **What**: Seeded 4 example decks for first-time guests: "Atomic Habits", "How to Avoid a Climate Disaster" (Bill Gates), "The Alchemist", and "Class 10 Science". Each deck contains 6–7 notes, 7 flashcards, and 6–7 vocabulary items. Renamed "All Words" aggregate view to "Complete Library" across 5 locations (HTML, sidebar, deck header, quiz selector, JS logic). Added `lexicon_demo_seeded` guard flag to prevent re-seeding on return visits.
- **Why**: To give new users immediate, engaging content to explore and learn from, reducing the blank-slate friction.
- **Impact**: First-time users see a rich, pre-populated library with diverse learning material. The Complete Library branding is more descriptive and premium.
- **Technical Detail**: Example deck data added to seedDemoData(); guard flag prevents re-seeding unless user clears browser storage or signs out.

## 2026-04-21 - Enforce Per-User Data Isolation & Clear Guest State

- **What**: Removed un-namespaced localStorage reads at global init and sign-out; reset all in-memory arrays (ALL_VOCAB, ALL_NOTES, perf, etc.) on sign-out; clear un-namespaced guest localStorage keys on sign-out. `pullFromCloud` now returns true for brand-new users. On first login, migrate any guest-created data into the user's UID namespace, then clear guest keys. Firestore security rules enforce server-side that each user can only read/write their own document.
- **Why**: Previous sessions leaked data between different email accounts on the same browser because `localStorage` was not namespaced and Firestore had no server-side access control.
- **Impact**: Multi-user security is now enforced. Guest data no longer persists between sessions and doesn't leak to logged-in users.
- **Technical Detail**: All localStorage keys now scoped by userId; Firestore rules check uid == request.auth.uid; guest data migrated on first sign-in via `migrateGuestData()`.

## 2026-04-20 - Single-Page Layout Restructure & Notes/Deck Pivot

- **What**: Replaced the tab-based navigation with a single scrollable main page. Renamed "Projects" to "Decks". Added a sidebar Practice section. Introduced a Library grid combining all content types (Vocab, Notes, Flashcards). Added Notes as a first-class content type. Added an Add Section card with type pills, file attachment, and a Generate → AI preview flow.
- **Why**: The user pivoted the app's purpose from a vocabulary-only tool to a full note-taking + flashcards + vocab learning platform. The tab layout was limiting and the single-page approach with a unified Library makes all content types accessible at a glance.
- **Impact**:
    - Main area is now a single scrollable page with: Deck Header → Add Section → AI Preview → Library
    - Flashcards, Quiz, Stats are accessed via a "Practice" section in the sidebar (shown as a full-area overlay with ← Library back button)
    - Library has filter pills: All / Notes / Flashcards / Quizzes / Vocab
    - Add Section toolbar: [+] file attach (Image/PDF/File) | type pills (Vocab/Notes/Flashcard/Quiz) | deck select | Generate → button
    - Generate → shows a per-type AI preview panel with Accept / Edit / Discard per card
    - Deck header updates dynamically (dot + name + item count) when decks are switched from sidebar
- **Technical Detail**:
    - Added `renderLibrary(filter)` replacing `renderDict()` — combines vocab, notes, custom cards in one grid
    - Added `openPracticeView(type)` / `closePracticeOverlay()` for the practice session overlay
    - Added `setLibFilter(filter)`, `toggleTypePill()`, `toggleFileDropdown()`, `handleGenerate()`, `acceptPreview()`, `discardPreview()`
    - `applyProjectFilter()` now calls `renderLibrary()` and updates the deck header count
    - `switchProject()` closes the practice overlay and refreshes the page header
    - Existing flashcard/quiz/stats JS logic is completely unchanged; only their container moved to `#practiceOverlay`
    - `lexicon_notes` data added to cloud sync payload

## 2026-04-20 - Deck System & Notes (Previous Session)

- **What**: Renamed "Projects" → "Decks" throughout the UI. Added a Notes section per deck. Added a deck dashboard view (replaced by the single-page layout in the same session). Added a dual-nav system (all-words vs deck mode), later superseded by the single-page layout.
- **Why**: Preparation for the full pivot to a notes+learning platform.
- **Impact**: Notes stored in `lexicon_notes` localStorage key, synced to Firestore. Note editor modal added.

---

## 2026-04-09 - Performance Optimization & Optimistic UI
- **What**: Implemented parallel fetching for core data and migrated the authentication state handler to an "Optimistic UI" pattern (loading from cache first, syncing in background).
- **Why**: To eliminate long loading screens ("Loading vocabulary...") caused by sequential network requests to Firebase and local storage.
- **Impact**: Application startup is now near-instant. Users see their data immediately from local cache, while cloud synchronization happens silently in the background.
- **Technical Detail**: 
    - Replaced sequential `fetch()` calls with `Promise.all()`.
    - Removed `await` from the cloud pull in `handleAuthState` to prevent network latency from blocking the UI.

## 2026-04-09 - Firebase Google Authentication
- **What**: Integrated Firebase Authentication with Google Sign-In using the script-tag (CDN) approach. Established a functional login/logout flow.
- **Why**: To enable user-specific data persistence and progress tracking across devices, fulfilling a core roadmap objective.
- **Impact**: Transitioned from a local-only application to a cloud-authenticated ecosystem. Users can now securely identify themselves via Google.
- **Challenges & Solutions**:
    - *Localhost/Domain Constraints*: Configured Firebase authorized domains to handle local development environment.
    - *Popup Management*: Resolved issues with authentication popups closing prematurely by properly handling the `signInWithPopup` promise and state transitions.

---

## 2026-03-30 - UI Restructure & Sidebar Navigation
- **What**: Reorganized the application to feature a ChatGPT-inspired dark sidebar for project management. Unified vocabulary loading to support both core files and custom `localStorage` projects. Implemented an "All Words" global view.
- **Why**: To improve navigation and organization. The user requested a more professional, sidebar-based layout where projects are easily accessible on the left, similar to modern AI chat interfaces.
- **Impact**: Enhanced project-based organization and a more premium, scalable user interface. Users can now seamlessly switch between specific books/topics and a holistic view of all their vocabulary.

---

## 2026-03-25 - Project Organization System
- **What**: Introduced `projects.json` for metadata management and scoped `localStorage` keys (e.g., `lexicon_perf_{projectId}`) for project-specific performance tracking.
- **Why**: The user wanted to categorize vocabulary by book or topic (e.g., "BOOK_Name") without mixing results and statistics.
- **Impact**: Enabled multiple isolated study paths within the same application, allowing for focused learning on specific texts or subjects.

---

## 2026-03-17 - Separating Quiz Words
- **What**: Split the flat vocabulary file into `vocabulary.json` (core dictionary) and `quiz_words.json` (additional words for testing). Modified the loading logic to merge them for quizzes but keep the dictionary clean.
- **Why**: To keep the main dictionary focused on primary words while allowing a broader range of vocabulary for practice sessions.
- **Impact**: Cleaner dictionary UI and more diverse quiz content.

---

## 2026-03-14 - Project Inception
- **What**: Initial creation of the Flashcards app with `vocabulary.json` (Source of Truth) and interactive study modes (Flashcards, Quiz).
- **Why**: The user needed a personal tool to track and study new English words they encounter.
- **Impact**: Established a functional, local-first vocabulary learning platform with persistent progress tracking.
