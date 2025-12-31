# TokyoSentences Learning Series — Project Specs

Light specs for tokyo-01 through tokyo-12. Each will be expanded into a full README when implementation begins.

---

## tokyo-01-sqlite-foundation

**Goal:** Add a local SQLite database and verify it works.

**Builds on:** tokyo-00-hello-electron

**Deliverables:**
- SQLite integration (better-sqlite3)
- Database file created in app directory
- One test table (`test_items` with id, name, created_at)
- Insert a few test records on app startup
- Display query results in the renderer window
- IPC channel for renderer to request data from main process

**Key concepts:**
- Main process vs renderer process (security boundary)
- IPC (Inter-Process Communication) with `ipcMain` and `ipcRenderer`
- Preload scripts and contextBridge
- Why SQLite runs in main process only

**Student will ask Claude Code to:**
- Add better-sqlite3 dependency
- Create a database initialization module
- Set up IPC handlers
- Create a preload script exposing safe database methods
- Query and display data in HTML

**Git checkpoint:** "Add SQLite database with IPC communication"

---

## tokyo-02-episodes-crud

**Goal:** Full create/read/update/delete for Episodes.

**Builds on:** tokyo-01-sqlite-foundation

**Deliverables:**
- Episodes table (id, title, sort_order, notes, created_at, updated_at)
- List view showing all episodes
- Create episode form
- Edit episode form
- Delete with confirmation
- Reorder episodes (drag-and-drop or move up/down buttons)
- Data persists across app restarts

**Key concepts:**
- CRUD operations pattern
- Forms in Electron (HTML forms, collecting input)
- Optimistic UI vs waiting for database confirmation
- Sort order management

**Student will ask Claude Code to:**
- Create the episodes table schema
- Build a two-panel layout (list + detail)
- Implement each CRUD operation
- Add reordering functionality
- Style the UI to look reasonable

**Git checkpoint:** "Full episodes CRUD with reordering"

---

## tokyo-03-sentences

**Goal:** Add Sentences as children of Episodes.

**Builds on:** tokyo-02-episodes-crud

**Deliverables:**
- Sentences table (id, episode_id FK, sort_order, english_source_text, status, notes, created_at, updated_at)
- Episode detail view shows its sentences
- Sentence CRUD within episode context
- Sentence ordering within parent episode
- Status dropdown (Draft, Comparing, Consolidating, Canon Updated, Needs Review, Ready to Export)
- Navigation: episodes list → episode detail → sentence detail

**Key concepts:**
- Foreign key relationships
- Parent-child UI patterns
- Nested navigation / breadcrumbs
- Enum/status fields

**Student will ask Claude Code to:**
- Create sentences table with FK constraint
- Modify episode detail view to list sentences
- Build sentence create/edit forms
- Implement status dropdown
- Add navigation between views

**Git checkpoint:** "Sentences with episode parent-child relationship"

---

## tokyo-04-model-responses

**Goal:** Capture LLM outputs for each sentence.

**Repo transition:** This begins the unified `tokyo-sentences-course` repo. Branch: `project-04-model-responses`

**Builds on:** tokyo-03-sentences (imported or rebuilt)

**Deliverables:**
- ModelResponses table (id, sentence_id FK, source_label, response_text, is_latest, pasted_at)
- Four default paste fields in sentence detail (ChatGPT, Claude, Gemini, Grok)
- Paste preserves formatting (whitespace, Japanese characters)
- Timestamp auto-populated on paste
- "Show history" toggle for viewing previous responses per source

**Key concepts:**
- One-to-many with history pattern
- "is_latest" flag management
- Handling large text fields
- Textarea behavior and paste events

**Student will ask Claude Code to:**
- Create model_responses table
- Add paste fields to sentence detail view
- Implement paste event handling with timestamps
- Build history toggle UI
- Create branch and commit

**Git skills introduced:** Creating branches, switching branches

---

## tokyo-05-consolidation

**Goal:** Add the workspace for synthesis decisions.

**Branch:** `project-05-consolidation`

**Builds on:** tokyo-04-model-responses

**Deliverables:**
- ConsolidationNotes table (sentence_id FK, notes_text, open_questions_text)
- Two textarea fields in sentence detail view
- Auto-save on blur (or explicit save button)
- "Copy to notes" button next to each model response field

**Key concepts:**
- 1:1 relationship (sentence always has exactly one consolidation record)
- Auto-save patterns
- Clipboard API for copy functionality

**Student will ask Claude Code to:**
- Create consolidation_notes table
- Add fields to sentence detail UI
- Implement auto-save or save button
- Add copy-to-clipboard helper buttons

**Git checkpoint:** "Add consolidation workspace"

---

## tokyo-06-canonical-basic

**Goal:** Author the canonical translation fields (without versioning).

**Branch:** `project-06-canonical-basic`

**Builds on:** tokyo-05-consolidation

**Deliverables:**
- CanonicalVersions table (id, sentence_id FK, is_current, japanese_line, furigana_version, english_gloss, grammar_nuance_notes, created_at)
- Form to edit canonical fields
- Display furigana as plain text (bracket notation, no rendering)
- Validation: warn if furigana field has no brackets

**Key concepts:**
- Core content authoring workflow
- Field validation (warnings vs errors)
- Preparing for versioning (is_current flag exists but only one record per sentence for now)

**Student will ask Claude Code to:**
- Create canonical_versions table
- Build canonical pack editing form
- Add bracket notation validation warning
- Display canonical pack in sentence detail

**Git skills introduced:** `git diff` between branches

---

## tokyo-07-versioning

**Goal:** Add version history to canonical packs.

**Branch:** `project-07-versioning`

**Builds on:** tokyo-06-canonical-basic

**Deliverables:**
- Multiple CanonicalVersions per sentence (one marked is_current)
- "Save as new version" action (prompts for change_note)
- Version history list showing timestamp + change_note
- "View this version" to see old content (read-only)
- "Restore this version" copies old version to new current

**Key concepts:**
- Versioning pattern (append-only, flag current)
- History UI patterns
- Restore = create new version with old content (not actual rollback)

**Student will ask Claude Code to:**
- Modify save to create new versions
- Build version history sidebar/panel
- Implement view and restore actions
- Handle is_current flag correctly

**Git skills introduced:** `git log`, reading commit history

---

## tokyo-08-vocabulary

**Goal:** Add structured vocabulary to each canonical version.

**Tag:** `v0.8-vocabulary`

**Builds on:** tokyo-07-versioning

**Deliverables:**
- VocabEntries table (id, canonical_version_id FK, term, reading, meaning, jlpt_level, pronunciation_tips, nuance_notes, example_sentence)
- Add/edit/delete vocab entries within canonical pack
- JLPT dropdown (N5, N4, N3, N2, N1, Unknown)
- Vocab entries display in sentence detail
- Vocab copied when creating new canonical version

**Key concepts:**
- Deeply nested data (Sentence → CanonicalVersion → VocabEntry)
- Dynamic form lists (add/remove entries)
- Data copying on version creation

**Student will ask Claude Code to:**
- Create vocab_entries table
- Build dynamic vocab entry form (add/remove rows)
- Implement JLPT dropdown
- Handle vocab copying when versioning

**Git skills introduced:** Tags, `git tag`, checking out tags

---

## tokyo-09-glossary

**Goal:** Project-wide glossary with optional linking from vocab entries.

**Tag:** `v0.9-glossary`

**Builds on:** tokyo-08-vocabulary

**Deliverables:**
- GlossaryEntries table (id, english_term, preferred_japanese, reading, usage_notes, tags)
- Glossary management view (separate from sentences)
- Glossary CRUD
- Optional glossary_entry_id FK on VocabEntries
- "Link to glossary" dropdown when editing vocab
- Visual indicator when vocab term matches a glossary entry

**Key concepts:**
- Cross-entity relationships
- Optional foreign keys
- Lookup/linking UI patterns
- String matching for suggestions

**Student will ask Claude Code to:**
- Create glossary_entries table
- Build glossary management view
- Add glossary link dropdown to vocab form
- Implement match indicator

**Git checkpoint:** "Add project glossary with vocab linking"

---

## tokyo-10-search

**Goal:** Find sentences by vocabulary, JLPT, status; find incomplete sentences.

**Tag:** `v0.10-search`

**Builds on:** tokyo-09-glossary

**Deliverables:**
- Search view with query input
- Search by Japanese term or reading (across all vocab)
- Filter by JLPT level
- Filter by sentence status
- Filter by episode (or "all episodes")
- Review queue: sentences missing required fields
- Click-through from results to sentence detail

**Key concepts:**
- SQL JOINs across multiple tables
- Filter/facet UI patterns
- Computed views (review queue is a filtered query)

**Student will ask Claude Code to:**
- Build search view with filters
- Write search queries with JOINs
- Implement review queue logic
- Add result click navigation

**Git checkpoint:** "Add search and review queue"

---

## tokyo-11-export

**Goal:** Export data and support user-selected project location.

**Tag:** `v0.11-export`

**Builds on:** tokyo-10-search

**Deliverables:**
- Export single sentence as JSON
- Export episode as JSON
- Export full project as JSON (with glossary)
- User-selectable project folder (file dialog)
- Database moves to selected location
- Lock file mechanism (create on open, warn if exists)
- Auto-backup snapshots (daily, in project folder)

**Key concepts:**
- JSON serialization of relational data
- Electron file dialogs
- File system operations
- Lock files for single-user enforcement
- Scheduled tasks (backup on interval)

**Student will ask Claude Code to:**
- Build export functions for each scope
- Implement project folder selection
- Add lock file logic
- Set up auto-backup

**Git skills introduced:** Release workflow concept

---

## tokyo-12-settings

**Goal:** Configurable settings and UI polish.

**Tag:** `v0.12-settings`

**Builds on:** tokyo-11-export

**Deliverables:**
- Project Settings view
- Custom source labels (add beyond four defaults)
- Hide/archive unused sources
- Help tooltip for furigana convention
- Toast notifications for save confirmations
- Status bar showing current project path
- Keyboard shortcut: Cmd/Ctrl+N for new sentence (stretch)

**Key concepts:**
- Settings persistence
- Dynamic UI based on configuration
- User feedback patterns (toasts, status bar)
- Keyboard shortcuts in Electron

**Student will ask Claude Code to:**
- Build settings view
- Implement custom source labels
- Add toast notification system
- Wire up keyboard shortcuts

**Final git checkpoint:** "TokyoSentences v1.0 complete"

---

## Summary: Repo Structure

```
Phase 1 — Standalone repos:
  tokyo-00-hello-electron/
  tokyo-01-sqlite-foundation/
  tokyo-02-episodes-crud/
  tokyo-03-sentences/

Phase 2 & 3 — Unified repo (tokyo-sentences-course):
  Branches: project-04, project-05, project-06, project-07
  Tags: v0.8, v0.9, v0.10, v0.11, v0.12
  Main: final v1.0 state
```

---

## Summary: Git Skills Progression

| Project | Git Skill |
|---------|-----------|
| 00 | clone (via GitHub Desktop) |
| 01 | — |
| 02 | add, commit (via Claude Code) |
| 03 | — (reinforce) |
| 04 | branches, checkout |
| 05 | — (reinforce) |
| 06 | diff between branches |
| 07 | log, reading history |
| 08 | tags, checkout tag |
| 09 | — (reinforce) |
| 10 | — |
| 11 | releases concept |
| 12 | — |
