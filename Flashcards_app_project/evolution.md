# Project Evolution Log

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
