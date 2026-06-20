# Reflow books into a bilingual web Reader

We render each Book by extracting its text, splitting it into sentences, and reflowing it into a web Reader as Sentence Pairs (English on top, Chinese below) — rather than overlaying Chinese onto the original PDF or regenerating a bilingual PDF. Overlaying annotations between the fixed lines of a PDF is impractical without breaking layout, and regenerated bilingual PDFs make sentence-level alignment and word-learning interactions hard. The trade-off is that the original page layout is lost.

## Consequences

- Images are placed in their original reading-order position (best-effort), not reordered next to the sentence that references them. Reference-driven repositioning is deferred.
