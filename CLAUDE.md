# CLAUDE.md

## Projekt
Mandantenfähiger digitaler Tagesbericht für Baustellen (Machbarkeitsnachweis, kein Produkt). Es werden ausschließlich erfundene Demo-Daten verwendet, keine Echtdaten.

## Stack
- TypeScript überall
- Frontend: React + Vite
- Backend: Node + Fastify
- ORM: Drizzle mit versionierten SQL-Migrationen
- Datenbank: PostgreSQL 16 mit pgvector
- Tests: Vitest

## Namen
- Tabellen und Spalten: snake_case, Singular, kein `tbl`-Präfix
- Primärschlüssel: UUIDv7
- Jede fachliche Tabelle hat `org_id`

## Sicherheit
- Keine Zugangsdaten, Schlüssel oder Passwörter im Code oder in Commits
- Konfiguration nur über Umgebungsvariablen, Vorlage in `.env.example`
- Niemals `--no-verify` verwenden

## Arbeitsregeln
- Immer erst Plan, dann Umsetzung
- Kleine Commits
- Vor jedem Commit laufen Lint und Tests
- Abhängigkeiten mit exakter Version festschreiben, Lockfile einchecken
- Keine neuen Abhängigkeiten ohne Begründung im Plan
- Nichts bauen, was nicht in der Aufgabe steht

## Sprache
- Dokumentation und Commit-Nachrichten auf Deutsch
- Code-Bezeichner auf Englisch
- Datenbanknamen deutsch, wie in `docs/Anforderung.md`
