# Nexora — Code Map

> Auto-generated section index for app.html. Use `grep -n "@@SECTION: Name"` to jump to any section.

## How to use
1. Find the section name below for the feature you need to change.
2. Run: `grep -n "@@SECTION: SectionName" app.html` to get the line number.
3. Read only that section (typically 100–300 lines) rather than the full file.

## Sections

| Section | Marker | Line | Description |
|---|---|---|---|
| **HTML/CSS** | | | |
| Styles (global CSS) | `StylesGlobal` | 66 | All CSS for the app (colors, layout, animations, media queries, mobile responsiveness) |
| Login Screen | `LoginScreen` | 5628 | Login/signup UI and auth forms |
| Profile Setup | `ProfileSetup` | 5679 | Profile creation wizard (name, goal, target, demographics) |
| Welcome Tour Modal | `WelcomeTour` | 5758 | Interactive onboarding carousel (decks, cards) |
| Onboarding Complete Overlay | `OnboardCompleteOverlay` | 5792 | Celebration screen after profile setup completion |
| Help Panel | `HelpPanel` | 5807 | Help guide with tabs (Getting Started, Features, Tips, Tour replay) |
| Main App Container | `MainAppContainer` | 5828 | Root app div (sidebar + main, mobile overlay backdrop, hamburger btn) |
| Practice Overlay | `PracticeOverlay` | 5925 | Flashcard/quiz/stats practice UI (decks, cards, scoring, notes) |
| Main Page | `MainPage` | 6241 | Main library view (deck browser, search, add section, bottom nav) |
| Search Modal | `SearchModal` | 12262 | Cmd+K command palette (search vocab, notes, flashcards) |
| Add Modal | `AddModal` | 12275 | Modal for adding vocab/notes/flashcards to decks |
| **JavaScript – Core** | | | |
| Main Script Setup | `MainScript` | 6871 | Script block opener, Gemini proxy URL config |
| State Variables | `JSState` | 6876 | Global state (VOCAB, QUIZ_VOCAB, projects, currentUser, etc.) |
| Auth Helpers | `JSAuthHelpers` | 6898 | Helper functions (getPerfKey, loadPerf, shuffle, etc.) |
| Analytics | `JSAnalytics` | 6922 | Firebase analytics tracking (trackEvent) |
| Storage Layer | `JSStorage` | 6936 | localStorage and Firestore sync (lsGet, lsSet, syncToCloud, etc.) |
| **JavaScript – Onboarding** | | | |
| Onboarding | `JSOnboarding` | 7039 | Profile setup steps, tour navigation, completion ceremony |
| Help Panel Logic | `JSHelpPanel` | 7363 | Help tab content and interactions |
| Auth State Handler | `JSAuthStateHandler` | 7502 | Firebase auth state listener and user login flow |
| Login Functions | `JSLoginFunctions` | 7610 | Sign-in, sign-up, password reset UI and validation |
| **JavaScript – Data** | | | |
| Demo Data Seed | `JSDemoData` | 7779 | Seed data for new users (vocab, notes, custom cards) |
| Load from JSON | `JSLoadJSON` | 7812 | Load vocabulary from JSON files (vocabulary.json, quiz_words.json) |
| **JavaScript – UI & Modals** | | | |
| Dictionary | `JSDictionary` | 8575 | Render vocabulary grid with scores (dict view) |
| Modal Dialog | `JSModalDialog` | 8603 | Word detail popup (definition, usage, SM-2 badge, edit, delete) |
| AI Modal | `JSAIModal` | 8801 | Gemini AI modal for generating vocab from text/images |
| Vocab Preview | `JSVocabPreview` | 8922 | Preview definition before saving vocab word |
| Image Vocab Modal | `JSImageVocab` | 8948 | OCR/Gemini image scanning, word extraction and grid UI |
| **JavaScript – Cards & Practice** | | | |
| Custom Flashcards | `JSCustomFlashcards` | 9229 | Create and manage custom front/back flashcards |
| Flashcards | `JSFlashcards` | 9280 | Flashcard study mode (SM-2 rating, flip, keyboard shortcuts, toast) |
| Quiz | `JSQuiz` | 9582 | Quiz mode with multiple choice and difficulty levels |
| Stats | `JSStats` | 9817 | Performance stats (total, correct, accuracy per word) |
| **JavaScript – Navigation & Views** | | | |
| Flashcard Deck Selection | `JSFlashcardDeckSelection` | 9873 | Deck picker for practice (checkboxes, start practice) |
| Practice Overlay | `JSPracticeOverlay` | 9968 | Practice view switcher (flashcards/quiz/stats) |
| Notes Review | `JSNotesReview` | 10016 | Notes flip-card review mode (deck picker, navigation) |
| New Quiz System | `JSNewQuizSystem` | 10132 | Multiple choice quiz engine with wrong answer review |
| Vocab Practice (Flip) | `JSVocabPractice` | 10448 | Flip card vocab review mode (vocabulary.json words) |
| Library | `JSLibrary` | 10526 | Main library render (decks, vocab, notes, cards sections) |
| Library List View | `JSLibraryListView` | 10594 | Filtered list display (vocab, notes, flashcards) with SM-2 state pills |
| Detail Popup | `JSDetailPopup` | 10758 | Item detail view in list (flip for cards, edit, delete) |
| **JavaScript – Add Section & Management** | | | |
| Add Section | `JSAddSection` | 10854 | Add vocab/notes UI (text textarea, file uploads, type pills) |
| Deck Dashboard | `JSDeckDashboard` | 11077 | Deck home page (stats, section tiles, actions) |
| Notes | `JSNotes` | 11199 | Notes editor and list (create, edit, delete, search) |
| Add Modal | `JSAddModal` | 11280 | Advanced add modal (multi-select types, dual-group pills, review wizard) |
| Mobile Menu | `JSMobileMenu` | 12109 | Mobile sidebar drawer, bottom nav, responsive toggles |
| **JavaScript – Init** | | | |
| Init & Setup | `JSInit` | 12146 | App initialization, Firebase/auth integration, event listeners |

## Common Tasks

### Add a new vocab word to the collection
→ Look in **JSLoadJSON** (7567) to see how vocabulary is loaded and initialized, then check **JSStorage** (6576) to understand how to persist it.

### Change flashcard study mode behavior
→ Start with **JSFlashcards** (9038) for card flipping/SM-2 logic, **JSFlashcardDeckSelection** (9628) for deck picking, and **JSStats** (9572) for scoring.

### Add a new help section or onboarding step
→ Update **JSHelpPanel** (7003) for help tab content or **JSOnboarding** (6679) for profile/tour steps. Styling lives in **StylesGlobal** (65).

### Modify the modal for adding content
→ Check **JSAddModal** (10912) for the +Add modal flow (dual-group pill selection, review wizard), and **JSAddSection** (10487) for the simpler add section on the main page.

### Fix an image or AI feature
→ **JSImageVocab** (8706) handles OCR and Gemini image scanning. **JSAIModal** (8559) is the general AI modal. **JSVocabPreview** (8680) is for definition lookups.

### Change Firebase sync or auth behavior
→ **JSStorage** (6576) handles all cloud sync. **JSAuthStateHandler** (7142) handles login flows. **JSDemoData** (7419) seeds data for new users.

### Modify the search/command palette
→ **JSInit** (11532) wires the Cmd+K shortcut. The palette HTML is at **SearchModal** (11648). Search logic (openSearch, runSearch, renderSearchResults) lives in **JSAddModal** (10912).

---

## File Structure Overview

```
app.html
├── Head
│   ├── Firebase SDK (module)
│   ├── Theme init (inline script)
│   └── StylesGlobal (CSS)
├── Body
│   ├── LoginScreen
│   ├── ProfileSetup
│   ├── WelcomeTour
│   ├── OnboardCompleteOverlay
│   ├── HelpPanel
│   ├── MainAppContainer (sidebar + main)
│   │   ├── Sidebar (profile, nav, theme, mastery dots)
│   │   └── Main Content
│   │       ├── PracticeOverlay (flashcards/quiz/stats)
│   │       └── MainPage (library home)
│   ├── SearchModal (Cmd+K command palette)
│   └── AddModal (+ Add dialog)
└── MainScript (all JavaScript)
    ├── State & Config
    │   ├── JSState
    │   ├── JSAuthHelpers
    │   ├── JSAnalytics
    │   └── JSStorage
    ├── Onboarding & Auth
    │   ├── JSOnboarding
    │   ├── JSHelpPanel
    │   ├── JSAuthStateHandler
    │   └── JSLoginFunctions
    ├── Data Loading
    │   ├── JSDemoData
    │   └── JSLoadJSON
    ├── UI & Modals
    │   ├── JSDictionary
    │   ├── JSModalDialog
    │   ├── JSAIModal
    │   ├── JSVocabPreview
    │   └── JSImageVocab
    ├── Study Features
    │   ├── JSCustomFlashcards
    │   ├── JSFlashcards
    │   ├── JSQuiz
    │   ├── JSStats
    │   ├── JSFlashcardDeckSelection
    │   └── JSVocabPractice
    ├── Navigation & Library
    │   ├── JSPracticeOverlay
    │   ├── JSNewQuizSystem
    │   ├── JSLibrary
    │   ├── JSLibraryListView
    │   └── JSDetailPopup
    ├── Content Management
    │   ├── JSAddSection
    │   ├── JSDeckDashboard
    │   ├── JSNotes
    │   └── JSAddModal
    └── Initialization
        └── JSInit
```

---

**Last updated:** May 7, 2026  
**File size:** ~12,400+ lines  
**Sections:** 44 major code regions

---

**Latest changes (May 7, 2026):** AI UX overhaul — fun loading screen with rotating emoji messages, AI toggle button (on by default), success toast after saving, improved LLM prompts for topic-focused notes and flashcards, vocab-only input hides textarea, onboarding Step 2 shows AI-generated suggestions with mini review before saving; nexora-onboarding.js and nexora-onboarding.css updated with loading/review/success panels and keyframe animations; all line numbers updated.
