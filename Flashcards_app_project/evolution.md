# Nexora — Project Evolution Log

> Reverse-chronological changelog. Each entry covers what changed, why, and the impact.

---

## 2026-04-30 — Bug-finder agent sweep: 7 new bugs fixed

**What:** Two automated Haiku bug-finder agents scanned all remaining sections of app.html and found 7 bugs that were then fixed.

**Why:** Post-issue-sweep automated audit to catch bugs not covered by the existing issue tracker.

**Impact:**
- lsGet() no longer crashes on corrupted localStorage (affects all app data reads)
- Guest-to-user data migration no longer crashes on malformed localStorage during sign-in
- showLoginError() no longer crashes when loginError element is absent from DOM
- showLibraryList() no longer crashes when libListSort element is absent
- _daysAgo() shows empty string instead of "NaN d ago" for invalid date strings
- vfRateCard() (vocab flip practice) no longer crashes when called past end of queue
- Custom card IDs now include random suffix preventing same-millisecond ID collisions

**Technical Detail:**
- `lsGet()`: JSON.parse now in try-catch, returns fallback on SyntaxError
- `handleAuthState()`: introduced `_parseSafe()` helper for guest localStorage migration
- `showLoginError()`: added `if (!el) return;` guard
- `showLibraryList()`: wrapped sortSel access in `if (sortSel) {}`
- `_daysAgo()`: `isNaN(ts)` check before date arithmetic
- `vfRateCard()`: `if (!w) return;` guard added
- Card IDs: `'cc_' + Date.now() + '_' + Math.random().toString(36).substr(2, 6)` in 3 places

---

## 2026-04-30 — Bulk Issue Resolution: 30+ Bug Fixes Across All Sections

**What:** Resolved 30+ open GitHub issues spanning CSS dark mode, JS null-check crashes, async race conditions, UX bugs, and data sync issues.

**Why:** Systematic bug sweep of all open GitHub issues to improve app stability, dark mode fidelity, and data integrity.

**Impact:**
- App no longer crashes on edge cases (out-of-bounds arrays, null word lookups, stale IDs)
- Dark mode now renders correctly across all major UI surfaces (login, flashcards, quiz, stats, help panel)
- "Nexora" branding complete — old "Smritikosha" text removed from profile setup screen
- Escape key now universally closes all modals (detail popup, image scan, custom card)
- Sync failures in notes/custom cards now show error feedback instead of silent stuck state
- Quiz score display no longer shows NaN when zero questions answered
- Search results now resolve item index at click time, preventing stale-index crashes

**Technical Detail:**
- CSS: Replaced `#fff`/hardcoded hex with `var(--surface)`, `var(--emerald-700)`, `var(--red)`, `color-mix()` design tokens in 15+ rules
- JS: Added `if (!w) return` / `if (!item) return` guards in `rateCard`, `openModal`, `_renderNqCard`, `moveWordToProject`
- JS: Wrapped `_addWordsFromText` in try/finally to guarantee `currentProjectId` restore
- JS: Added `AbortController` 10s timeout to `_callAnthropicDistractors` fetch
- JS: `_checkSectionDone` now checks `isConnected` and filters out `.tile-collapsing` tiles
- JS: `renderSearchResults` now uses `findIndex(x => x.id === ...)` at click time
- JS: `syncToCloud` fire-and-forget calls in notes/custom cards now have `.catch(() => {})`
- JS: `nextQuiz()` bounds-check calls `showQuizDone()` instead of overrunning array
- JS: `selectQuizOption()` now null-checks `perf[w.id]` and `w.usage[0]`
- JS: `finishProfileSetup()` validates `_profileGoal` when not skipping
- JS: `handleAddModalGenerate()` closes Add modal before opening image modal to prevent overlap
- JS: `startNotesQuiz()` uses crypto-safe fallback ID instead of `Math.random()`

---

## 2026-04-30 — Context-Aware Definition Selection + Synonyms & Antonyms Enrichment

**What:** Added Tier 1.5 context-aware definition selection via Gemini (`_selectRelevantDefs`), which picks the 2–3 most relevant definitions for a word based on the deck's title and vocabulary sample. Extended Tier 3 Gemini gap-fill to also fetch 2 synonyms and 2 antonyms per word when missing.

**Why:** Words often have many definitions; showing the most context-relevant ones improves study effectiveness. Synonyms and antonyms provide richer vocabulary context and deepen understanding beyond a single definition.

**Impact:** Word detail cards now surface definitions most relevant to the deck being studied. Users see up to 2 synonyms and 2 antonyms on enriched word cards, strengthening vocabulary connections.

**Technical Detail:**
- `_selectRelevantDefs(word, allDefs, deckTitle, deckVocab)` — sends all definitions + deck context to Gemini proxy; parses a JSON array of 1-based indices; returns top 3 matching `allDefs` entries
- Tier 1.5 inserted in `addWords()` pipeline: fires after Free Dict succeeds and `wordData.allDefs.length > 2`
- `_haikuEnrichWord` extended: `needsSynonyms` / `needsAntonyms` flags added; prompt now requests `"synonyms": [string, string]` and `"antonyms": [string, string]`; results stored as `wordData.synonyms` and `wordData.antonyms` (sliced to 2)

---

## 2026-04-30 — App Renamed to Nexora + Code Navigation System

**What:** Renamed app from Smritikosha/Lexicon to Nexora across all project files. Added a code navigation system with @@SECTION markers and CODE_MAP.md index.

**Why:** New brand identity (Nexora). Navigation system introduced to allow agents to jump directly to the right code section without reading the full 10,500-line app.html.

**Impact:** All display names, localStorage keys (`nexora_*`), taglines, and documentation now reflect the Nexora brand. Future code changes are faster and more precise with the section map.

**Technical Detail:** 170+ text replacements across 13 files. 42 `@@SECTION` markers added to app.html. CODE_MAP.md created with 40 section entries.

---

## 2026-04-30 — SM-2 Spaced Repetition, Daily Queue, Search Palette, Keyboard Shortcuts, UX Polish

**What:** Implemented SM-2 spaced repetition algorithm with four pure functions (`defaultSM2`, `applyRating`, `getDueCount`, `updateDailyBadge`) to track word mastery intervals and due dates. Added daily review queue badge in sidebar with breakdown (lapsed → review → new priority order). Built Cmd+K search/command palette modal with real-time filtering. Keyboard shortcuts wired: Space (flip), 1 (wrong), 2 (correct). Context sentences now flow from add modal through preview and display on flashcard back. SM-2 state badge added to word detail modal. Post-rating toast appears below flip actions. Onboarding simplified to 1 profile step + 2-slide tour.

**Why:** SM-2 delivers scientifically-proven spacing algorithm for long-term retention. Daily queue badge gives quick visibility into study load. Search palette improves discoverability. Keyboard shortcuts reduce friction during study. Context sentences anchor memory to real usage.

**Impact:** Users see SM-2 intervals and due dates on every word, enabling intelligent study planning. Daily queue badge encourages consistent review. Cmd+K search makes finding words fast. Keyboard-first study (Space+1/2) speeds up sessions. Simplified onboarding reduces friction.

**Technical Detail:**
- SM-2 state stored per word in localStorage with `{interval, easiness, repetitions, dueDate}`
- `_renderLibListHTML()` rewritten to show SM-2 state pills with "Due in Xd" chips
- `openSearch()`, `closeSearch()`, `runSearch()` manage the search modal; Cmd+K/Escape wired in global keydown handler

---

## 2026-04-30 — Three UX Fixes: Sample Deck Labels, Guest Light Mode, Remove Word-Added Alert

**What:** (1) Sample decks now show a non-bold "(sample)" label at reduced opacity. (2) "Continue without signing in" now starts in light mode regardless of OS preference. (3) Removed browser alert after adding vocab words successfully.

**Why:** Sample deck labels distinguish built-in content from user decks. Guest dark mode was jarring. The alert felt out of place.

**Impact:** Cleaner sidebar, better first-run experience for guests, quieter add-word flow.

---

## 2026-04-30 — Fix Flashcards Section Card Navigating to Practice Instead of List

**What:** Fixed `showDeckSection('flashcards')` which was launching practice mode instead of displaying the flashcard items list.

**Why:** User reported clicking the Flashcards card sent them to practice mode rather than showing the item list like Notes and Vocab.

**Impact:** Clicking Flashcards in the deck home now correctly opens the library list of flashcard items.

---

## 2026-04-29 — Fix Notes/Vocab Section Navigation (Deck-Specific Mode)

**What:** Fixed Notes and Vocab tiles doing nothing when clicked from a specific deck's home, and fixed the Back button restoring the wrong view.

**Why:** `showDeckSection` did not route for 'notes' and 'vocab' in deck-specific mode; the library section was hidden so nothing appeared.

**Impact:** Notes, Flashcards, and Vocab tiles now correctly navigate to filtered list views. Back button correctly restores the deck home.

---

## 2026-04-29 — Add Modal Text-Input Preview/Review Step

**What:** Added a confirm-before-save preview step in the +Add modal for Vocab, Note, and Flashcard types. After generating, a preview card is shown. Buttons change to "← Edit" and "✓ Save". Multi-type selections show all previews stacked.

**Why:** User requested the same accept/decline UX that image scan already provided, applied to typed text input.

**Impact:** Users can review and confirm (or back out and edit) before any typed content is persisted.

---

## 2026-04-28 — Add Modal Multi-Select, PRD/Design Docs, Brand Review

**What:** Upgraded Add Modal from radio-style type pills to multi-select checkboxes. Vocab + Note + Flashcard can be checked simultaneously; Image/PDF exclusive. Added Phase 2 planning documents.

**Why:** Type pills were mutually exclusive. Multi-select lets users create a vocab entry, note, and flashcard from a single text block in one submit.

**Impact:** Content creation is more flexible. The modal is self-contained for all add flows including file scan. Deck Home CSS scaffolding in place for Phase 3.

---

## 2026-04-28 — Phase 2 — Sidebar Redesign, Add Modal (Cmd+N), Header Actions, Scroll Fix

**What:** Narrowed sidebar to 220px; added mastery dots (4-dot quartile indicator) to each deck. Added Daily Queue Badge placeholder. Redesigned deck header: Cmd+K Search ghost pill + "+ Add" emerald button. Built Add Modal with Cmd+N/Escape/Cmd+Enter support. Fixed scroll position bug: `_renderLibListHTML()` now saves/restores scroll position.

**Why:** Inline add section was cluttered and broke scroll position. Modal gives a clean, focused creation flow. Mastery dots give instant deck health visibility.

**Impact:** All content creation flows through a focused modal. Scroll position preserved after adds. Sidebar shows per-deck mastery at a glance.

---

## 2026-04-26 — App Rebranding: Lexicon/Smritikosha → Nexora

**What:** Renamed app to Nexora. Updated page title, login screen, sidebar logo, and all documentation.

**Why:** User requested a fresh, modern name that is easier to pronounce and remember.

**Impact:** All visible branding refreshed. User experience remains unchanged.

---

## 2026-04-28 — Batch AI Tile Preview & Multi-Source Word Enrichment

**What:** Replaced per-word Free Dictionary API calls in image scan with single batch Gemini call. New `_saveWordWithEnrichment` function saves immediately with AI seed, then enriches async (Free Dict → Wordnik → Gemini). Added `dataSources[]` field to track contributing APIs. Removed alert popups on word accept.

**Why:** Old image scan was wasteful—called Free Dictionary once per word for preview, then again on accept. Single batch Gemini call for preview is faster; enrichment happens in background without blocking UI.

**Impact:** Reduced API call count significantly. Faster tile preview and word acceptance. No alert popups. Full data source transparency.

**Technical Detail:** `_saveWordWithEnrichment()` saves with cached AI data, then async enrichment via Free Dict → Wordnik → Gemini gap-fill. `_mergeWordSources()` merges multiple sources with dedup.

---

## 2026-04-27 — Rich Image-Scan Popup with Tiles, Definitions, Difficulty System

**What:** Redesigned image vocabulary scan popup to display extracted words as styled tiles with async-fetched definitions. Added difficulty scoring system (Easy/Medium/Hard/Very Hard). Two labelled sections ("📖 Underlined Words" / "✨ AI-Suggested Words") separated by divider.

**Why:** Previous scan listed words as plain text with no enrichment. Users couldn't see definitions before accepting. No difficulty metadata for prioritization.

**Impact:** Image scanning shows full dictionary definitions before acceptance. Difficulty badges provide visual cues for study planning.

**Technical Detail:** New CSS classes for tiles and difficulty badges. `_fetchTileDefinition()` async helper fetches Free Dict and computes difficulty via `_scoreDifficulty()`.

---

## 2026-04-27 — Docs: Add PRD and Reformat PRD Analysis with Proper Markdown

**What:** Added `PRD.md` (new product requirements document) and reformatted `PRD_analysis.md` with proper markdown structure (headers, tables, lists).

**Why:** `PRD_analysis.md` had plain text tables that didn't render; `PRD.md` was missing.

**Impact:** Project documentation is now complete and readable in any markdown viewer.

---

## 2026-04-26 — Fix: Show Real API Error Instead of Silent "No Words Found" on Image Scan

**What:** Fixed error handling in image processing functions. Previously when API returned error with HTTP 200, app silently fell back to empty arrays and displayed "No words found."

**Why:** User reported image upload always returned "no words" despite the prompt working. Root cause was API hitting free-tier quota; secondary cause was app not checking `data.error`.

**Impact:** Image scanning now shows the actual API error instead of silently failing. Users get actionable feedback.

---

## 2026-04-26 — Feat: Secure Gemini API Key via Cloudflare Worker Proxy

**What:** Removed hardcoded `GEMINI_API_KEY` from client-side HTML. Replaced all 4 direct Gemini API fetch calls with calls to a Cloudflare Worker proxy endpoint. API key now stored securely as Cloudflare Worker secret.

**Why:** Hardcoded API key was being detected and restricted by Google. Key was discoverable in browser DevTools, causing quota abuse flags. Moving to Cloudflare ensures it never reaches the client.

**Impact:** API key is now secured behind server-side proxy. All Gemini-powered features work reliably without quota abuse risk.

---

## 2026-04-25 — Refactor: Replace Anthropic with Gemini Flash, Hardcode API Key

**What:** Replaced all Anthropic/Claude Haiku API calls with Gemini 1.5 Flash across 4 fetch sites (word enrichment, distractor generation, image scanning, image vocab modal enrichment). Removed API key input fields from UI.

**Why:** User requested app work out-of-the-box without manual API key entry. Unified all AI calls under single provider (Gemini) to simplify maintenance.

**Impact:** App fully functional without users entering any API keys. Reduced cognitive load and improved first-time UX.

**Technical Detail:** `_haikuEnrichWord()` replaced with Gemini fetch; function signature unchanged. `_callAnthropicDistractors()` replaced with Gemini fetch; fallback preserved.

---

## 2026-04-25 — Feat: Word Preview Before Saving and Image Vocab Review Modal

**What:** Generate flow now shows AI Preview card with fetched definition before saving. Image + Vocab opens popup modal with all extracted words (underlined + AI-suggested) with per-word Accept/Edit/Decline buttons.

**Why:** Typing a word and clicking Generate added it immediately without showing definition. Attaching image showed generic placeholder instead of actual extracted words.

**Impact:** Users can now review word definitions before accepting them. Image scans fully functional with clean review modal.

---

## 2026-04-25 — Fix: Onboarding Funnel Bug Fixes (5 Issues)

**What:** Fixed five bugs: (1) Step 2 disappeared post-animation (missing `active` class), (2) profile name/goal fields never injected into HTML, (3) swipe gesture fired on diagonal scrolls, (4) hint height hardcoded at 80px, (5) progress label had no clamp on step count.

**Why:** Haiku audit surfaced critical and medium bugs breaking step navigation and silently swallowing profile data.

**Impact:** Step 2 stays visible. Sidebar correctly shows user's name and goal. Tour no longer skips on scroll. Hints position correctly near viewport edges.

---

## 2026-04-24 — Feat: Redesign User Onboarding Experience

**What:** Overhauled all onboarding—Profile Setup (animated progress bar, gradient icons, goal validation with shake, directional slides), Welcome Tour (4 CSS-animated hero illustrations per slide), and Onboarding Completion (serif typewriter ceremony, word-by-word definition stagger, Day 1 streak badge).

**Why:** User requested premium, design-elevated onboarding. Existing flow used basic emoji icons and step dots with no animation.

**Impact:** First-time users experience warm, premium onboarding matching the app's literate aesthetic. Mobile users get bottom-sheet layouts.

**Technical Detail:** New CSS keyframes for animations (slide, flip, cascade, typewriter, etc.). Goal data now surfaces in UI via `applyProfileToUI()`.

---

## 2026-04-23 — Bug Fixes: Null-Safety, Async Race Conditions, Timer Leaks (Issues #16–#29)

**What:** Fixed 19 bugs across three passes: null-safety in dictionary waterfall and word modal; async race conditions in quiz navigation and stats filter; auto-flip timer leaks; undefined values in Wordnik usage array; `w.opposites` null crash; quiz distractors stored at wrong index.

**Why:** Three sequential Haiku 4.5 reviews surfaced crashes when API responses were empty, modal missing from DOM, quiz navigation firing before async loads, stats accessing nonexistent fields, and background timers firing after modal close.

**Impact:** Word addition, quiz flow, and stats view no longer crash under edge-case data. Quiz options always fully loaded before display. Timer state properly managed across sessions.

**Technical Detail:** Guards added throughout: `if (!data || !data.length)` in fetch paths, `(array || [])` null checks, `clearAutoFlipTimer()` called before state resets, captured index snapshot in async blocks to avoid race conditions.

---

## 2026-04-23 — Dictionary Waterfall + AI Gap-Fill + Web Speech Audio Fallback

**What:** Replaced single-tier Free Dictionary API call with 4-tier waterfall: (1) Free Dictionary API, (2) Wordnik fallback, (3) Claude Haiku 4.5 gap-fill for missing IPA/definition/usage, (4) Web Speech API audio fallback at playback time.

**Why:** Core vocabulary and obscure words often returned incomplete data (missing IPA, no audio, vague definition). Waterfall progressively enriches each word at near-zero cost.

**Impact:** Words receive IPA phonetics, audio, and usage examples from best available source. All words gain Web Speech API pronunciation even without stored URL.

**Technical Detail:** `addWords()` refactored with `_parseFreeDictEntry()`, `_fetchWordnik()`, `_haikuEnrichWord()` helpers. Audio button: uses URL if present, else `speechSynthesis.speak()`.

---

## 2026-04-23 — Dropdown Clipping Fix & AI Image Word Review UI

**What:** Fixed `+` file-attach button dropdown being clipped by parent overflow. Upgraded "Scan Page" image flow from `window.confirm()` dialog to full in-modal word review UI with two toggleable chip sections.

**Why:** Dropdown cut off at `.add-section-card` boundary due to `overflow: hidden`. Old confirm dialog gave no control over word selection. User wanted to accept/reject individual words and receive AI-suggested hard words beyond underlined ones.

**Impact:** Dropdown renders fully visible. Scan flow shows underlined and up to 8 AI-suggested hard words as toggleable chips. Users can deselect any word before adding.

**Technical Detail:** `.file-dropdown` changed from `position: absolute` to `position: fixed` with z-index 9999. Gemini prompt updated to return `{underlined_words, suggested_words}`.

---

## 2026-04-22 — M3 Warm/Gamified Design Implementation

**What:** Applied complete Material Design 3 warm/gamified aesthetic (inspired by Duolingo, Brainscape). Replaced all fonts with Plus Jakarta Sans. Converted dark sidebar to white with green-tinted gradient. Added emerald pill-shaped nav with SVG icons. Redesigned cards with 14px radius and color-coded left borders (green/amber/red). Updated buttons to pill-shaped with emerald primary and drop-shadow.

**Why:** To create engaging, gamified, visually cohesive user experience encouraging learning through warm colors and playful design language.

**Impact:** App now feels modern, cohesive, and premium. Warm palette and M3 design make vocabulary learning feel more rewarding.

**Technical Detail:** Palette includes #FFF7ED (warm bg), #10B981 (emerald), #F59E0B (amber), #EF4444 (red). All components updated to M3 specifications.

---

## 2026-04-21 — Per-User Data Isolation & Sample Decks for All Users

**What:** Fixed demo deck seeding for logged-in users: `seedDemoData` now guest-only. New `mergeDemoDataForUser()` merges demo decks after `pullFromCloud` without overwriting real data (triggered once via `nexora_demo_seeded` flag). Added delete button for every sidebar deck; deletion moves items to first remaining deck instead of hardcoded General.

**Why:** Users wanted example decks even after signing in. Previous approach meant existing users never got demo content.

**Impact:** All users (guest and signed-in) see curated example decks. Any deck can be deleted without data loss.

**Technical Detail:** `mergeDemoDataForUser()` with per-account guard flag. Library grid renders 2 columns of cards with preview content and deck badge.

---

## 2026-04-21 — GitHub Push Process & Project Conventions

**What:** Added `CLAUDE.md` documenting mandatory 3-step GitHub workflow (commit → create issue → close issue) and curl-based GitHub API approach (gh CLI not installed).

**Why:** To establish repeatable, documented process for pushing code without relying on third-party CLI tools.

**Impact:** Future development sessions have clear, copy-paste-ready workflow for all GitHub interactions.

---

## 2026-04-21 — Example Decks with Full Content & Complete Library Rename

**What:** Seeded 4 example decks for first-time guests: "Atomic Habits", "How to Avoid a Climate Disaster", "The Alchemist", "Class 10 Science" (each with 6–7 notes, 7 flashcards, 6–7 vocab items). Renamed "All Words" to "Complete Library" across 5 locations.

**Why:** To give new users immediate, engaging content and reduce blank-slate friction.

**Impact:** First-time users see rich, pre-populated library with diverse learning material. Complete Library branding is more descriptive and premium.

---

## 2026-04-21 — Enforce Per-User Data Isolation & Clear Guest State

**What:** Removed un-namespaced localStorage reads at init and sign-out. Reset all in-memory arrays on sign-out. Clear guest keys on sign-out. Migrate guest-created data into user's UID namespace on first login. Firestore rules enforce server-side access control.

**Why:** Previous sessions leaked data between different email accounts on same browser because localStorage was not namespaced.

**Impact:** Multi-user security now enforced. Guest data no longer persists between sessions and doesn't leak to logged-in users.

---

## 2026-04-20 — Single-Page Layout Restructure & Notes/Deck Pivot

**What:** Replaced tab-based navigation with single scrollable main page. Renamed "Projects" to "Decks". Added sidebar Practice section. Introduced Library grid combining all content types (Vocab, Notes, Flashcards). Added Notes as first-class content type. Added Add Section card with type pills, file attachment, and AI preview flow.

**Why:** Tab layout was limiting. Single-page approach with unified Library makes all content types accessible at a glance. User pivoted app's purpose to full note-taking + flashcards + vocab platform.

**Impact:** Main area is single scrollable page. Flashcards, Quiz, Stats accessed via "Practice" overlay. Library has filter pills: All/Notes/Flashcards/Quizzes/Vocab. Generate → shows per-type AI preview.

---

## 2026-04-09 — Performance Optimization & Optimistic UI

**What:** Implemented parallel fetching for core data and migrated auth handler to "Optimistic UI" pattern (cache first, sync in background).

**Why:** To eliminate long loading screens caused by sequential network requests.

**Impact:** Application startup is now near-instant. Users see data immediately from cache while cloud sync happens silently.

---

## 2026-04-09 — Firebase Google Authentication

**What:** Integrated Firebase Authentication with Google Sign-In using script-tag CDN. Established functional login/logout flow.

**Why:** To enable user-specific data persistence and progress tracking across devices.

**Impact:** Transitioned from local-only to cloud-authenticated ecosystem. Users can securely identify via Google.

---

## 2026-03-30 — UI Restructure & Sidebar Navigation

**What:** Reorganized application with ChatGPT-inspired dark sidebar for project management. Unified vocabulary loading to support core files and custom localStorage projects. Implemented "All Words" global view.

**Why:** To improve navigation and organization with professional, sidebar-based layout similar to modern AI chat interfaces.

**Impact:** Enhanced project organization and premium, scalable user interface. Users seamlessly switch between specific decks and holistic "All Words" view.

---

## 2026-03-25 — Project Organization System

**What:** Introduced `projects.json` for metadata and scoped `localStorage` keys (e.g., `nexora_perf_{projectId}`) for project-specific performance tracking.

**Why:** User wanted to categorize vocabulary by book or topic without mixing results and statistics.

**Impact:** Multiple isolated study paths within same application, enabling focused learning on specific texts.

---

## 2026-03-17 — Separating Quiz Words

**What:** Split vocabulary file into `vocabulary.json` (core dictionary) and `quiz_words.json` (additional for testing). Modified loading logic to merge for quizzes but keep dictionary clean.

**Why:** To keep main dictionary focused on primary words while allowing broader vocabulary for practice.

**Impact:** Cleaner dictionary UI and more diverse quiz content.

---

## 2026-03-14 — Project Inception

**What:** Initial creation of Flashcards app with `vocabulary.json` (Source of Truth) and interactive study modes (Flashcards, Quiz).

**Why:** User needed personal tool to track and study new English words.

**Impact:** Established functional, local-first vocabulary learning platform with persistent progress tracking.
