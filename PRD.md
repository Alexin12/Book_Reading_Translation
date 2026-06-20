# PRD: Bilingual Reader

> Vocabulary in this document follows [CONTEXT.md](./CONTEXT.md). Architectural decisions are recorded in [docs/adr/](./docs/adr/). Terms like **Book**, **Source**, **Translation**, **Sentence Pair**, **Reader**, **Term List**, **Lookup**, **Vocabulary**, **Selection Action**, and **Explanation** are used in their canonical meaning.

## Problem Statement

Many books the user wants to read exist only in English. Reading them straight is slow, and looking up unfamiliar words breaks the flow. The user wants to both read these books comfortably and pick up unknown English words while doing so, without manually translating anything or fighting with raw PDFs.

## Solution

A self-hosted web app that imports an English Book (PDF, Word, or Markdown), translates it once in the background, and renders it as a **Reader**: a stream of **Sentence Pairs** with the English **Source** on top and the Chinese **Translation** directly below, images kept in their original reading order. While reading, the user can select any word for a **Lookup** (definition, phonetic, spoken pronunciation) and save words to a **Vocabulary** for later review.

## User Stories

1. As a reader, I want to import a PDF Book, so that I can read it bilingually.
2. As a reader, I want to import a Word Book, so that I am not limited to PDFs.
3. As a reader, I want to import a Markdown Book, so that plain-text sources work too.
4. As a reader, I want scanned (image-only) PDF pages to be OCR'd automatically, so that older or scanned books still work.
5. As a reader, I want each Book translated once and cached, so that re-opening it is instant and free.
6. As a reader, I want to open a previously imported Book from a library, so that I can resume reading across devices.
7. As a reader, I want the Reader to show one Source sentence on top and its Translation below, so that I can read and learn simultaneously.
8. As a reader, I want images to appear in their original reading-order position, so that figures stay near their text.
9. As a learner, I want to select or hover an English word and get a Lookup with definition and phonetic, so that I understand unfamiliar words without leaving the page.
10. As a learner, I want to click a word and hear it pronounced in English, so that I learn its pronunciation.
11. As a learner, I want to save a word to my Vocabulary, so that I can review it later.
12. As a learner, I want to view and export my Vocabulary, so that I can study it elsewhere.
13. As a cost-conscious user, I want a token and cost estimate before a Book is translated, so that I am not surprised by the bill.
14. As a cost-conscious user, I want to choose which model translates a given Book, so that I can use a cheap model for casual books and a strong one for important ones.
15. As a user, I want to configure my own model provider and API key, so that I can use OpenAI, DeepSeek, Gemini, or Claude.
16. As a reader, I want recurring names and terms translated consistently across the whole Book via a Term List, so that the translation does not contradict itself.
17. As a reader, I want to correct an entry in a Book's Term List, so that I can fix a wrong canonical translation.
18. As a user, I want to log in with a single password, so that my server and API keys are not open to the public.
19. As a reader, I want to see translation progress while a Book is being processed, so that I know when it will be ready.
20. As a reader, I want to keep reading on phone and laptop, so that the same library is available everywhere.
21. (Future) As a reader, I want to select a passage and ask the model to explain it using the whole Book as context, so that I can understand difficult ideas, their connections, and get analogies.
22. (Future) As a reader, I want to correct a bad Translation inline, so that the cache improves as I read.

## Implementation Decisions

**Architecture (see ADRs):**
- Any input → parsed into one **intermediate format** (Markdown + extracted image files) → sentence-split → translated once in a background job → persisted → rendered by the Reader (ADR-0002, ADR-0001).
- Translation is **pre-computed and cached** per Book; opening a Book reads the cache only (ADR-0003).
- All LLM calls go through a **provider-agnostic model layer** (e.g. LiteLLM). Model selection and API keys are stored **per user** (ADR-0004).
- Per-Book **Term List** plus context-carrying chunking enforce terminology consistency (ADR-0005).
- Deployed self-hosted; protected by a **single-password gate** in swappable auth middleware; all data hangs off a single default **user** so multi-user is additive later (ADR-0006).
- Backend Python + FastAPI; frontend React + Vite + TypeScript; persistence via SQLAlchemy over SQLite, Postgres-ready (ADR-0007).

**Modules to build:**
- `Importer` / parser layer — one parser per input format, all emitting the intermediate format; PDF parser includes OCR fallback for scanned pages.
- `Translation pipeline` — sentence-splitting, chunking with context, Term List extraction/application, producing Sentence Pairs; emits progress and a pre-run cost estimate.
- `Model layer` — provider-agnostic wrapper; reads per-user provider/model/key; reused by the future Explanation feature.
- `Book store` — persisted Books, their Sentence Pairs, images, Term Lists, and status.
- `Reader API` — serves a translated Book as ordered Sentence Pairs + images to the frontend.
- `Reader UI` — renders Sentence Pairs; hosts the **Selection Action** toolbar (Lookup, save-to-Vocabulary, speak; later: Explanation).
- `Lookup service` — single-word definition + phonetic (free dictionary API); pronunciation via browser Web Speech API.
- `Vocabulary store` — saved words per user, with export.
- `Auth middleware` — single-password check today; swappable for multi-user later.

**Reserved seams (not built in MVP, see ADR-0008):**
- Generic Selection Action toolbar so "explain this" becomes one more action.
- Document-context store (the persisted structured Book) later indexed for retrieval.
- `explain(book_id, selection, question)` endpoint shape; whole-Book Q&A will use RAG (chunk + vector retrieval), not full-Book context stuffing.

**Phasing:**
- **MVP:** digital-text PDF → bilingual Reader (Sentence Pairs) + Lookup + TTS, single hard-coded model. No OCR, Term List, cost estimate, multi-format, or login yet.
- **Thicken in order:** ① cost estimate + model selection → ② Term List consistency → ③ Word/Markdown input → ④ OCR for scanned pages → ⑤ single-password login → ⑥ Vocabulary → ⑦ inline Translation correction → ⑧ (future) whole-Book Explanation.

## Testing Decisions

Tests assert **external behavior at the highest seam**, never implementation details. The model layer is stubbed with a fake provider so no test makes a real API call or incurs cost.

- **Translation pipeline seam** (highest-value): given a small fixture input document, assert the produced ordered Sentence Pairs, that images are preserved in reading order, and that a Term List entry is applied consistently across chunks. Run with a stub provider returning deterministic Translations.
- **Importer seam:** given fixture PDF/Word/Markdown inputs (including one scanned-image PDF), assert each produces the same intermediate-format shape.
- **Reader API seam:** given a persisted translated Book, assert the endpoint returns ordered Sentence Pairs + image references for the frontend.
- **Auth middleware seam:** assert protected endpoints reject requests without the password and accept them with it.
- **Cost estimate:** assert the estimate is produced from token counts before any translation job starts.

No prior art exists yet (greenfield); these seams establish the testing conventions for the project.

## Out of Scope

- Preserving the original PDF page layout, or overlaying Translation onto the original PDF / regenerating a bilingual PDF (ADR-0001).
- Reordering images next to the sentence that references them (reading-order placement only).
- Multi-user accounts, registration, and billing (architecture is reserved for it; not built).
- Whole-Book Explanation / Q&A and its RAG vector index (seams reserved; ADR-0008).
- Inline Translation correction.
- Automatic highlighting of likely-unknown vocabulary across the text.

## Further Notes

- Pronunciation uses the browser's Web Speech API (zero backend cost); Lookup definitions use a free dictionary API such as dictionaryapi.dev.
- This PRD could not be published to an issue tracker (no git repo / tracker configured). Run `/setup-matt-pocock-skills` and re-publish if an issue-based workflow is wanted.
