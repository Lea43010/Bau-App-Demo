# Entwicklungstagebuch

| Datum | Aufgabe | Vorschlag des Agenten | Meine Änderung oder Ablehnung |
|-------|---------|-----------------------|-------------------------------|
| 2026-11-04 | Fundament: CLAUDE.md, Devcontainer, Compose, .env.example, .gitignore, Doku | Devcontainer über Docker Compose (Services `app` mit Node 20 und `db` mit `pgvector/pgvector:pg16`). Lint und Tests gibt es noch nicht. | _offen_ |
| | | | |
## Block 1 – Fundament (04.11.2026)
- Aufgabe: Devcontainer mit Node 20 und Postgres 16 (pgvector), Agenten-Regeln, Env-Vorlage.
- Vorschlag des Agenten: Compose-basierter Devcontainer mit Dienst `db`, `.env` bleibt außerhalb von Git.
- Meine Änderung: Anforderung v1.1 als docs/Anforderung.md abgelegt; `.env` im neuen Codespace von Hand angelegt, danach Rebuild, `db:5432` erreichbar.