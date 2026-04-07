# Project Evolution Log

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
- **What**: Initial creation of the Flashcards app with `vocabulary.json` (Source of Truth) and interactive study modes (Flashcards, Quiz, Fill in the Blanks).
- **Why**: The user needed a personal tool to track and study new English words they encounter.
- **Impact**: Established a functional, local-first vocabulary learning platform with persistent progress tracking.
