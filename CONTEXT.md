# Bilingual Reader

A personal tool that turns English books (PDF, Word, Markdown) into a sentence-by-sentence bilingual web reader — English on top, Chinese below — to support reading and vocabulary learning.

## Language

**Book**:
A source document the user imports to read bilingually, regardless of original format (PDF, Word, Markdown).
_Avoid_: file, document, PDF

**Source**:
The original English text of a Book.
_Avoid_: original, English side

**Translation**:
The Chinese rendering of a piece of Source text.
_Avoid_: target, Chinese side

**Sentence Pair**:
One Source sentence aligned one-to-one with its Translation. The atomic unit displayed in the Reader (English on top, Chinese below).
_Avoid_: line, segment, row

**Reader**:
The web view that renders a Book as a stream of Sentence Pairs with images in original reading order.
_Avoid_: viewer, page

**Term List**:
A per-Book table of names and terms with their agreed Translation, used to keep terminology consistent across the whole Book.
_Avoid_: glossary, dictionary, terminology list

**Lookup**:
An on-demand result for a single Source word — definition, phonetic, and spoken pronunciation — shown when the user selects or hovers a word.
_Avoid_: definition popup, translation

**Vocabulary**:
The collection of words the user saves while reading for later review.
_Avoid_: word list, saved words, flashcards

**Selection Action**:
An action triggered on a user text selection in the Reader (Lookup, save to Vocabulary, speak aloud, and — later — Explanation). All such actions share one selection toolbar.
_Avoid_: context menu, popup

**Explanation** _(future)_:
An LLM answer about a selected passage that uses the whole Book as context to relate concepts and give analogies (NotebookLM-style).
_Avoid_: chat, summary, note
