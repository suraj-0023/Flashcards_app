# Project Evolution Log

## 2026-04-25 — fix: replace expired Gemini API key with new AI Studio key

### What
- Replaced the expired Gemini API key (`AIzaSyDxRsKIMYM10TGIix7IXkNY25DR9skH6tw`) with a new key from Google AI Studio (`AIzaSyC8BMwruBk1ybFSk3p2P6xW_ew_0wWsOdA`)
- Updated `GEMINI_API_KEY` constant at line 5447 in `app.html`

### Why
The previous API key had exhausted its free tier quota (limit: 0 error), causing image scanning and all Gemini-powered features (word enrichment, distractor generation, image vocab extraction) to fail with "quota exceeded" errors. A new key was generated from Google AI Studio to restore service.

### Impact
Image upload/scan feature and all AI-powered functionality restored. Users can now scan images, enrich words, and generate quiz distractors without errors.

### Technical Detail
- File: `Flashcards_app_project/app.html` line 5447
- Changed `const GEMINI_API_KEY = 'AIzaSyDxRsKIMYM10TGIix7IXkNY25DR9skH6tw'` to `const GEMINI_API_KEY = 'AIzaSyC8BMwruBk1ybFSk3p2P6xW_ew_0wWsOdA'`
- All 4 Gemini fetch calls automatically use the updated key: word enrichment, distractor generation, image scanning, image vocab modal enrichment

## 2026-04-25 — fix: upgrade Gemini model from gemini-1.5-flash to gemini-2.0-flash

### What
- Updated all 4 Gemini API endpoint URLs in `app.html` from deprecated `gemini-1.5-flash` model to `gemini-2.0-flash`
- Affected 4 fetch calls: word enrichment, distractor generation, image scanning, image vocab modal enrichment

### Why
`gemini-1.5-flash` model was retired and returning "model not found" errors when users tried to upload/scan images or generate enrichment data. The newer `gemini-2.0-flash` model provides the same vision and text capabilities with improved stability.

### Impact
Image scanning/upload feature is now fully functional again. All AI-powered features (word enrichment, image vocab extraction, quiz distractor generation) working without errors.

### Technical Detail
- Updated URL strings in fetch calls at lines ~6851, ~7051, ~7183, ~7988 in `app.html`
- Changed `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent` to `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent`
- No function signatures or API request/response formats changed; direct model ID substitution only

## 2026-04-25 — refactor: replace Anthropic with Gemini Flash, hardcode API key

### What
- Replaced all Anthropic/Claude Haiku API calls with Gemini 1.5 Flash (4 fetch sites: word enrichment, distractor generation, image scanning, image vocab modal enrichment)
- Added `GEMINI_API_KEY` constant at top of `<script>` block — single source of truth for all Gemini calls
- Word enrichment (IPA, definition, usage) now always runs via Gemini 1.5 Flash; no longer requires user to provide an Anthropic key
- Quiz distractor generation switched from Anthropic to Gemini Flash
- Removed Gemini and Anthropic API key input fields from UI (Gemini input removed from Scan Page AI modal; Anthropic input removed from Quiz section)
- Removed all localStorage reads/writes for `gemini_key` and `anthropic_key`; verified zero remaining refs
- Code quality fixes: `_previewVocabWord` moved from `window` to module-level; stale event handler cleanup in `editImageWord` and `showImageVocabModal`; `.catch()` added to `acceptImageWord`

### Why
User requested the app work out-of-the-box without manual API key entry. Temporary solution: developer's Gemini API key hardcoded as a constant (TODO: migrate to Cloudflare Workers for true security). Unified all AI calls under a single provider (Gemini) to simplify maintenance and improve reliability.

### Impact
App now fully functional without users entering any API keys. Word enrichment, image scanning, and quiz distractors all powered by Gemini Flash automatically. Reduced cognitive load and improved first-time user experience.

### Technical Detail
- `const GEMINI_API_KEY` declared at top of `<script>` block; used in 4 fetch URLs: `_haikuEnrichWord()`, `_callAnthropicDistractors()`, `processImage()`, `generateVocabFromImage()`
- `_haikuEnrichWord()`: Anthropic fetch replaced with Gemini fetch; function signature and output unchanged (`{phonetic, definition, usage}`)
- `_callAnthropicDistractors()`: Anthropic fetch replaced with Gemini fetch; fallback to `_fallbackDistractors()` preserved
- `addWords()`: removed `if (anthropicKey)` gate that blocked Tier 3 enrichment; now enrichment always runs
- `processImage()`: removed `localStorage.getItem('gemini_key')` and `localStorage.setItem('gemini_key', ...)` calls; uses `GEMINI_API_KEY` constant
- `generateVocabFromImage()`: removed `localStorage.getItem('gemini_key')` and "no key found" alert; uses `GEMINI_API_KEY` constant
- Deleted `#geminiApiKey` input from AI modal (Scan Page)
- Deleted `#anthropicApiKey` input from Quiz section
- `_previewVocabWord` moved from `window._previewVocabWord` to module-level `let`; prevents namespace pollution
- Event handler cleanup: `editImageWord()` removes old click listener before adding new one; `showImageVocabModal()` cleanup before modal display
- File changed: `Flashcards_app_project/app.html`

## 2026-04-25 — feat: word preview before saving and image vocab review modal

### What
- Generate flow: text word + Vocab now shows AI Preview card with fetched definition before saving
- Generate flow: image + Vocab now opens a popup modal listing all extracted words (underlined + AI-suggested) with per-word Accept / Edit / Decline buttons
- Removed shortcircuit that was bypassing the preview and adding words directly

### Why
User reported that (1) typing a word and clicking Generate added it immediately without showing the definition, and (2) attaching an image and clicking Generate showed a generic placeholder card instead of the actual extracted words

### Impact
Users can now review word definitions before accepting them into their library; image scans via the main Generate flow are fully functional with a clean review modal

### Technical Detail
- `handleGenerate()`: removed vocab+text shortcircuit; added image+vocab branch calling `generateVocabFromImage()`
- New `_loadVocabPreview(word, deckId)`: fetches from Free Dictionary API, updates preview card content in-place
- `acceptPreview('vocab', deckId)`: now async, shows Saving… state, calls `_addWordsFromText()` on Accept
- New `generateVocabFromImage(file, deckId)`: Gemini 1.5 Flash API call, parses underlined_words + suggested_words, opens modal
- New modal: `#imageVocabModalOverlay` with `.img-vocab-row` / `.img-vocab-word` CSS; functions `showImageVocabModal`, `acceptImageWord`, `editImageWord`, `declineImageWord`
- File changed: `Flashcards_app_project/app.html`

## 2026-04-25 — Fix: Onboarding funnel bug fixes (5 issues)

**What**: Fixed five bugs in the onboarding funnel:
1. Step 2 disappeared after slide-in animation (missing `active` class post-`animationend`)
2. `applyProfileToUI()` silently no-oped — `#sidebarProfileName`/`#sidebarProfileGoal` never existed in HTML; injected both into `renderSidebarProfile()` template and call `applyProfileToUI()` after render
3. Swipe gesture on tour fired on diagonal scrolls — added vertical-drift guard (`|dx| > |dy|`)
4. `showHint()` flip used hardcoded 80px height — replaced with measured `getBoundingClientRect().height`
5. Progress label `Step X of 2` had no clamp — added `Math.min(to + 1, 2)`

**Why**: Haiku audit surfaced 2 critical and 3 medium/low bugs that broke step navigation and silently swallowed profile data.

**Impact**: Step 2 of profile setup now stays visible; sidebar correctly shows the user's name and goal after onboarding; tour no longer skips slides on scroll; hints position correctly near viewport edges.

**Technical Detail**: `_setProfileStep()` in app.html line 3782; `renderSidebarProfile()` / `applyProfileToUI()` lines 4262–4287 / 4018–4027; swipe handler lines 3939–3951; `showHint()` lines 4031–4057; progress label line 3787.

## 2026-04-24 — feat: redesign user onboarding experience

**What:** Overhauled all onboarding screens — Profile Setup (animated progress bar replacing step dots, gradient step icons, goal validation with shake animation, directional slide transitions, trust micro-copy, mobile bottom-sheet), Welcome Tour (4 CSS-animated hero illustrations per slide: floating deck cards, 3D flip card, cascading quiz bars, typewriter notepad; per-slide accent badge, swipe gestures, goal-aware copy), and Onboarding Completion (first-word ceremony overlay: serif typewriter, word-by-word definition stagger, Day 1 streak badge).

**Why:** User requested a premium, design-elevated onboarding flow. Existing onboarding used basic emoji icons and step dots with no animation or visual hierarchy. Opus 4.7 and Sonnet 4.6 collaborated to design the approach.

**Impact:** First-time users now experience a warm, premium onboarding that matches the app's literate aesthetic. Mobile users get bottom-sheet layouts. Goal selection data now surfaces in the UI via applyProfileToUI(). Hints no longer overflow viewport.

**Technical Detail:** New CSS keyframes (onboardSlideUp, cardFlip, deckFloat, barGrow, cursorBlink, wordType, wordFadeUp, sheetRise, shake); CSS tokens --tour-c-decks/cards/quiz/notes, --onboard-shadow; JS: _setProfileStep(), renderTourSlide() rewrite with fade+hero swap, showOnboardComplete() with GOAL_FIRST_WORDS map, applyProfileToUI(), hint viewport clamp; new HTML: #onboardCompleteOverlay, .tour-hero with 4 CSS illustration variants. All in Flashcards_app_project/app.html.

## 2026-04-23 - Bug Fixes: Onboarding — Firestore Upsert, Hint Onclick Safety, Z-Index Conflict

- **What**: Fixed 3 bugs in onboarding code: Firestore upsert in `finishProfileSetup` (changed `_fbUpdateDoc` to `_fbSetDoc` with `{merge:true}`), unsafe inline onclick in `showHint` replaced with `addEventListener` for safe closure-based handling, z-index conflict on `.file-dropdown` resolved (9999→9996).
- **Why**: Bug audit by two parallel agents after the onboarding feature was shipped; identified after feature review.
- **Impact**: Profile data now reliably persists to Firestore on first login (no silent failure if doc doesn't exist); hint system is safe regardless of storageKey content (no quote escaping issues); dropdown stacking is deterministic.
- **Technical Detail**: 
  - `finishProfileSetup()` now uses `_fbSetDoc(userRef, profileData, {merge:true})` instead of `_fbUpdateDoc`, which safely upserts on first login when the Firestore document may not exist yet.
  - `showHint()` uses closure-based `addEventListener('click', ...)` instead of inline `onclick="dismissHint(this,'${storageKey}')"`, preventing quote escaping vulnerabilities.
  - `.file-dropdown` z-index lowered to 9996 to resolve stacking ambiguity with `#loginScreen` (both at 9999).

## 2026-04-23 - User Onboarding Journey, Profile Setup, Welcome Tour, and Help Panel

- **What**: Added full user onboarding journey — 2-step profile setup screen (`#profileSetupScreen`), 4-card welcome tour modal (`#welcomeTourModal`), contextual hint tooltip system (4 hints for Flashcards, Quiz, deck switch), and Help & Guide sidebar panel (`#helpPanel`) with 4 tabs (Getting Started quickstart, Features accordion, Tips & Shortcuts with keyboard shortcuts, Replay Tour).
- **Why**: New users had no guidance after login; no in-app help or reference. Onboarding dropoff was high due to blank-slate app with zero orientation.
- **Impact**: First-time users now receive a personalized 2-step setup flow (name from Google, goal selection, optional daily target/age/city/mobile) and a 4-card feature walkthrough. All users can access comprehensive help at any time via the sidebar. Contextual hints guide users through core workflows on first use.
- **Technical Detail**:
  - **Profile Setup** (`showProfileSetup()`, `finishProfileSetup()`): Step 1 shows Google name + goal chips (Daily Goal, Test Prep, Casual Learning, Passionate Interest, etc.); Step 2 collects optional daily target, age, city, mobile. Data saved to `localStorage('lexicon_profile')` and synced to Firestore at `users/{uid}.profile`. Controlled by `lexicon_profile_complete` flag.
  - **Welcome Tour** (`startWelcomeTour()`, `closeTour()`, `tourNav()`): 4 paginated cards (Decks, Flashcards, Quiz, Notes) with animated progress dots, Back/Skip/"Let's go!" nav. Stored state: `lexicon_tour_complete` flag. Accessed via `handleAuthState` after login (unless flag set) or `skipLogin` for guests.
  - **Contextual Hints** (`showHint()`, `dismissHint()`): Non-blocking tooltips shown once per session per context (flashcards, quiz, deck selection, notes). Auto-dismiss after 6s or on manual close. Flags: `lexicon_hint_flashcards`, `lexicon_hint_quiz`, `lexicon_hint_deck`, `lexicon_hint_notes`.
  - **Help Panel** (`openHelpPanel()`, `closeHelpPanel()`, `switchHelpTab()`): Right-sliding panel with tabs: Getting Started (3-step quickstart), Features (accordion per feature), Tips & Shortcuts (keyboard shortcuts + pro tips), Replay Tour (resets and restarts welcome tour).
  - **Sidebar item**: "? Help & Guide" added at bottom of sidebar above profile section.
  - All integrated into `handleAuthState` and `skipLogin` flows.

## 2026-04-23 - Bug Fixes: Third Pass — Wordnik Usage Undefined, opposites Null, Timer Leak, Quiz Race Condition (Issues #26–#29)

- **What**: Fixed 4 MAJOR bugs found in a third Haiku 4.5 review: undefined values in Wordnik usage array, `w.opposites` null crash in modal, stale autoflip timer on new flashcard session, and `_renderNqCard()` race condition storing distractors at wrong index
- **Why**: GitHub issues #26–#29 documented silent data corruption (undefined usage sentences), a modal crash for Haiku-enriched words, timer bleed between sessions, and wrong quiz options appearing when navigating rapidly
- **Impact**: Wordnik usage sentences are always valid strings; word modal opens reliably for all word types; starting a new flashcard session always clears the previous timer; quiz distractors are always stored at the card that requested them
- **Technical Detail**:
  - `_fetchWordnik()` ~3826: `.map(e => e.text).filter(Boolean)` — drops undefined entries
  - `openModal()` ~3932: `(w.opposites || []).length` and `(w.opposites || []).map(...)` — null guard
  - `startFlashWithSelectedDecks()` ~4798: `_clearAutoFlipTimer()` added before resetting state
  - `_renderNqCard()` ~4962: captured `const cardIdx = _nqIdx` at function entry; all `_nqTrack[_nqIdx]` inside async block replaced with `_nqTrack[cardIdx]`

## 2026-04-23 - Bug Fixes: Second Pass — Timer Leak, Index Bounds, Stats Null Guard (Issues #20–#24)

- **What**: Fixed 5 bugs found in a second Haiku 4.5 review pass: null array guard on `wordData.usage`, undefined guard on `wordData.types`, auto-flip timer leak on overlay close, quiz next-button index overflow, and missing perf null guard in `renderStats()`
- **Why**: GitHub issues #20–#24 documented crashes in `addWords()` when Haiku enrichment returned null usage/types; a background timer firing after the practice overlay was closed; quiz index growing unbounded; and stats crashing on words loaded before perf tracking was initialised
- **Impact**: Word addition no longer crashes on enrichment edge cases; closing practice stops all timers; quiz navigation is bounds-safe; stats render correctly for all words
- **Technical Detail**:
  - `addWords()` ~3745: `!wordData.usage.length` → `!Array.isArray(wordData.usage) || !wordData.usage.length`
  - `addWords()` ~3750: `wordData.types.length` → `(wordData.types || []).length`
  - `closePracticeOverlay()`: `_clearAutoFlipTimer()` added as first line
  - `_showNewQuizArea()` ~4923–4924: next-arrow/btn onclick guards with `if (_nqIdx < _nqQueue.length)`
  - `renderStats()` ~4681: `perf[w.id] = perf[w.id] || { quiz: {c:0,w:0}, flash: {c:0,w:0} }` before access

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
