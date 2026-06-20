# Tech stack: FastAPI backend, React/Vite frontend, SQLite via SQLAlchemy

The backend is Python + FastAPI and the frontend is a separate React + Vite + TypeScript app. Document parsing and the LLM tooling are strongest in the Python ecosystem, while the Reader's interactions (Lookup, TTS, selection toolbar) fit React; front/back separation also suits open-sourcing later. Persistence uses SQLAlchemy over SQLite now (zero-ops, file-based) and can move to Postgres when multi-user scale demands it without touching business code.
