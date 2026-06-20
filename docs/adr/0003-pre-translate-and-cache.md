# Pre-translate Books and cache, not on-the-fly

A Book is translated once in a background job at import time and the result is persisted; opening a Book afterwards only reads the cache and renders. Translating a several-hundred-page Book on every open would be slow, would re-spend money on each open, and would not work offline. Before a Book is translated, the system estimates token count and cost and lets the user confirm and pick which model to use for that Book.
