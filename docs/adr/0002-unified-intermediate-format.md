# Convert every input to one intermediate format

All input formats (PDF, Word, Markdown) are parsed into a single structured intermediate format — Markdown plus extracted image files — before sentence-splitting, translation, and rendering. This keeps the translation and Reader logic written once; adding a new input type only means adding a parser. PDF parsing uses a library with built-in OCR (e.g. docling, with pytesseract as fallback) so scanned pages are handled by the same pipeline.
