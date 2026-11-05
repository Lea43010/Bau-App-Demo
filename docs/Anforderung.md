Bau-Structura — Anforderung Machbarkeitsnachweis „Digitaler Tagesbericht"

Version 1.1 · Stand: 04.11.2026 · Bauvorgabe für den Machbarkeitsnachweis Grundlage ist Version 1.0 vom 02.08.2026. Diese Fassung ist für das öffentliche Repo bereinigt: ohne Kosten-, Rechts- und Vertriebsplanung.

0. Zweck und Abweichungen gegenüber v1.0

Das Projekt ist ein Machbarkeitsnachweis, kein Produkt und kein Angebot. Es zeigt, wie ein mandantenfähiger digitaler Tagesbericht mit Berechtigungen, KI-Suche und manipulationssicherer Dokumentation aufgebaut werden kann. Es gibt keine Preise, keine Testphase, keine Registrierung für Fremde und ausschließlich erfundene Demo-Daten.

Was sich gegenüber v1.0 ändert:

Thema	v1.0	v1.1
Reihenfolge	Offline-Kern früh (AP-04)	Mandanten und Tagesbericht → KI-Suche → Hash-Kette. Offline zuletzt und optional
KI	Anti-Scope	KI-Suche (RAG) ist im Umfang (F-15). Weitere KI-Funktionen bleiben ausgeschlossen
Zurückgestellt	—	Fotos (F-06), Unterschrift (F-07), PDF (F-08), Exporte (F-10, F-14)
Entfällt	—	Testphase, Plan, Abrechnung (F-13), Betreiber-Oberfläche, Landingpage, Marketing
Hosting	EU-Server mit Docker Compose	Lokal und im Codespace mit Docker Compose. KI-Dienste auf Azure in der EU-Datenzone
Arbeitspakete	AP-01 bis AP-12	AP-A bis AP-G (Abschnitt 8)

Unverändert gelten: Mandantentrennung, Row Level Security, Datenmodell, Statuslogik und Hash-Kette.

Sprachregel: Die Hash-Kette wird als „Manipulationserkennung" beschrieben. Aussagen wie „gerichtsfest", „rechtssicher" oder Compliance-Siegel kommen im Repo und in der Demo nicht vor.

1. Produktentscheidung

Der Machbarkeitsnachweis ist ein digitaler Tagesbericht / Regiebericht, keine Projektmanagement-Plattform.

Ein Tagesbericht ist:

Ein Mitarbeiter erfasst am Ende des Arbeitstages, was geleistet wurde: Stunden pro Person, eingesetzte Geräte, verbautes Material, Wetter, Vorkommnisse. Nach der Freigabe wird der Bericht eingefroren und ist nicht mehr änderbar.

Warum genau das: Der Tagesbericht ist ein append-only Dokument, das dem Erfasser gehört. Er bietet klare Mandanten- und Rollenregeln, eine natürliche Grundlage für Manipulationserkennung (Hash-Kette) und gut durchsuchbare Texte (Vorkommnisse, Bemerkungen) für die KI-Suche.

2. Nicht im Umfang
Nicht gebaut	Begründung
Google Maps, Geo-Fencing, Routenplanung	Lizenz und Aufwand, nicht Kern des Nachweises
KI-Funktionen außer der RAG-Suche (Beschreibungstexte, Risikobewertung, Oberflächenanalyse, Spracherkennung)	Nur eine KI-Funktion, dafür sauber umgesetzt
Sprachaufnahme, Videoaufnahme	Nicht Kern des Nachweises
Datenqualitäts-Dashboard mit Score	Pflichtfeldprüfung im Formular genügt
GPS-Check-in/-out, automatische Standortverfolgung	Mitbestimmungspflichtig, nicht Kern
Support-Ticket-System, E-Mail-Postfach-Verwaltung	Nicht Kern
Native iOS-/Android-Apps	PWA genügt
Social Login, Zahlungsanbieter, Testphase	Nichts Kommerzielles
Nachtragsmanagement, GAEB, Aufmaß, LV	Außerhalb des Nachweises
Fotos, Unterschrift, PDF, Exporte	Zurückgestellt, Neubewertung am 16.01.2027
3. Nutzerrollen

Zwei Rollen, Rolle je Organisation. Eine Betreiberrolle gibt es im Machbarkeitsnachweis nicht.

Rolle	Rechte
Mitarbeiter	Eigene Tagesberichte anlegen und bis zur Freigabe bearbeiten. Sieht nur eigene Berichte und die ihm zugewiesenen Projekte
Bauleitung	Alles vom Mitarbeiter, alle Berichte der Organisation lesen, Projekte und Stammdaten pflegen, Nutzer einladen, Berichte stornieren (nicht löschen)

R-3.1 (MUSS): Jede Berechtigung wird auf Datenbankebene über Row Level Security erzwungen, nicht nur in der Anwendungslogik. Ein Fehler im Frontend darf niemals fremde Mandantendaten freigeben.

4. Mandantenfähigkeit — nicht verhandelbar

R-4.1 (MUSS): Jede fachliche Tabelle trägt eine Spalte org_id als Fremdschlüssel auf organisation. Ohne Ausnahme.

R-4.2 (MUSS): Row Level Security ist auf allen fachlichen Tabellen aktiv. Die Anwendung setzt pro Request die Session-Variable app.current_org. Kein Query ohne diesen Kontext.

R-4.3 (MUSS): Ein automatisierter Test versucht, mit dem Token von Organisation A einen Datensatz von Organisation B zu lesen, zu ändern und zu löschen, für jede API-Route. Der Test läuft bei jedem Commit in der CI.

R-4.4 (MUSS, sobald Fotos gebaut werden): Dateien im Objektspeicher liegen unter org/{org_id}/… und sind nur über zeitlich begrenzte signierte URLs erreichbar.

R-4.5 (MUSS): Die KI-Suche läuft unter derselben Row Level Security wie die übrige Anwendung. Treffer aus fremden Organisationen sind ausgeschlossen, und das ist per Test belegt.

5. Datenmodell (Zielbild)

Namenskonvention: snake_case, Singular, kein tbl-Präfix. Primärschlüssel sind UUIDv7.

organisation        id, name, anschrift, created_at
benutzer            id, org_id, email, name, rolle, aktiv, letzter_login
projekt             id, org_id, nummer, name, auftraggeber_id, adresse,
                    start_datum, end_datum, status, archiviert_am
auftraggeber        id, org_id, name, anschrift, kontakt_name, kontakt_email
mitarbeiter         id, org_id, personalnummer, name, aktiv
geraet              id, org_id, kennung, bezeichnung, aktiv
tagesbericht        id, org_id, projekt_id, berichtsdatum, erfasst_von,
                    wetter_text, temperatur_c, bemerkung, vorkommnisse,
                    status, erstellt_am, freigegeben_am,
                    hash_self, hash_prev
bericht_stunde      id, org_id, tagesbericht_id, mitarbeiter_id, taetigkeit,
                    stunden, zuschlag_art
bericht_geraet      id, org_id, tagesbericht_id, geraet_id, stunden
bericht_material    id, org_id, tagesbericht_id, bezeichnung, menge, einheit
bericht_embedding   id, org_id, tagesbericht_id, textabschnitt, embedding (vector)
audit_eintrag       id, org_id, entitaet, entitaet_id, aktion, benutzer_id,
                    zeitpunkt, nutzdaten_json
einladung           id, org_id, email, rolle, token_hash, gueltig_bis, eingeloest_am

Zurückgestellt (Fotos und Unterschrift): bericht_foto, unterschrift.

R-5.1 (MUSS): tagesbericht.status kennt genau entwurf → freigegeben → storniert. Ein Wechsel zurück aus freigegeben ist unmöglich. Korrekturen erfolgen über einen neuen Bericht mit Referenz auf den stornierten.

R-5.2 (MUSS): Beim Übergang nach freigegeben wird der vollständige Bericht kanonisch serialisiert und mit SHA-256 gehasht. hash_prev verweist auf den Hash des vorherigen freigegebenen Berichts derselben Organisation. Eine nachträgliche Änderung invalidiert alle folgenden Berichte.

R-5.3 (MUSS): Eine Prüffunktion verifiziert die Hash-Kette einer Organisation vollständig und benennt den ersten abweichenden Bericht. Das Ergebnis wird in der App angezeigt; ein PDF-Protokoll ist zurückgestellt.

R-5.5 (MUSS): Nichts wird physisch gelöscht. Archivieren statt löschen.

6. Offline-Architektur (optional, AP-G)

Der Offline-Betrieb wird erst nach den übrigen Paketen und nur auf ausdrückliche Entscheidung gebaut. Zielbild unverändert: Local-First mit lokaler IndexedDB (Dexie), Outbox mit Idempotenzschlüssel (clientseitige UUID), sichtbarer Übertragungsstatus, Server gewinnt bei Konflikten, Stammdaten werden auf das Gerät gespiegelt.

Bis dahin gelten: Primärschlüssel werden so vergeben, dass sie clientseitig erzeugt werden können, und Schreibzugriffe sind idempotent.

7. Funktionale Anforderungen mit Abnahmekriterien

Priorität: M = Muss, S = Soll, Z = zurückgestellt.

F-01 · Organisation und Anmeldung — M

Organisationen und Benutzer für die Demo werden per Skript angelegt. Anmeldung mit E-Mail und Passwort über eine bewährte Bibliothek. Abnahme: Zwei Demo-Organisationen mit je einer Bauleitung und mehreren Mitarbeitern sind per Befehl erzeugbar.

F-02 · Nutzer einladen — M

Bauleitung lädt ein; Einladung ist 7 Tage gültig und einmalig einlösbar. Abnahme: Ein zweites Einlösen desselben Tokens schlägt fehl, ebenso ein abgelaufenes. Der Eingeladene gehört zur selben org_id.

F-03 · Projekt anlegen und verwalten — M

Nummer, Name, Auftraggeber, Adresse, Zeitraum. Archivieren statt löschen. Abnahme: Archivierte Projekte erscheinen nicht in der Auswahl für neue Berichte, ihre Altberichte bleiben abrufbar.

F-04 · Stammdaten Mitarbeiter und Geräte — M

Listen mit Anlegen und Deaktivieren. Abnahme: Deaktivierte Mitarbeiter bleiben in bestehenden Berichten sichtbar, sind aber nicht mehr auswählbar.

F-05 · Tagesbericht erfassen — M

Formular: Projekt, Datum, Wetter, Stundenzeilen, Gerätezeilen, Materialzeilen, Bemerkung, Vorkommnisse. Freigabe durch den Erfasser. Abnahme: Ein Bericht mit 5 Stundenzeilen ist in unter 90 Sekunden erfasst. Nach der Freigabe liefert jeder Änderungsversuch über die API HTTP 409.

F-06 · Fotodokumentation — Z
F-07 · Unterschrift vor Ort — Z
F-08 · PDF-Erzeugung — Z
F-09 · Berichtsübersicht und Filter — M

Liste mit Filter nach Projekt, Zeitraum, Mitarbeiter, Status. Abnahme: Filterkombination liefert bei 5.000 Berichten Ergebnisse in unter 500 ms.

F-10 · Export für Lohn und Abrechnung — Z
F-11 · Stornieren und Korrigieren — S

Bauleitung storniert einen freigegebenen Bericht mit Pflichtbegründung und legt einen Korrekturbericht mit Verweis an. Abnahme: Der stornierte Bericht bleibt abrufbar und ist gekennzeichnet. Die Hash-Kette bleibt gültig.

F-12 · Integritätsprüfung — M

Bauleitung startet die Prüfung der Hash-Kette. Abnahme: Eine direkt in der Datenbank manipulierte Zeile wird erkannt und benannt.

F-13 · Konto und Abrechnung — entfällt
F-14 · Datenexport durch den Kunden — Z
F-15 · KI-Suche über Tagesberichte (RAG) — M

Frage in natürlicher Sprache, zum Beispiel „Welche Behinderungen gab es im Oktober auf Baustelle X?". Zeitraum und Baustelle aus der Frage werden als Filter genutzt. Die Antwort nennt ihre Belege (Bericht, Datum, Baustelle). Embeddings liegen in PostgreSQL (pgvector); das Sprachmodell läuft als Azure OpenAI in der EU-Datenzone mit Anmeldung über Microsoft Entra ID, ohne API-Schlüssel im Code. Abnahme:

Die Beispielfrage liefert eine korrekte Antwort mit Belegen.
Ein Nutzer aus Organisation A erhält nie Treffer aus Organisation B (automatischer Test).
Ein Evaluationsset mit 20 Fragen findet in mindestens 17 Fällen den richtigen Bericht.
Nach der Hash-Kette (AP-E) kennzeichnet die Antwort, ob ein Beleg geprüft ist.
8. Nicht-funktionale Anforderungen
Nr.	Anforderung	Geltung
NF-1	Bedienbar ohne Netz	Optional, nur mit AP-G
NF-3	Ladezeit: Largest Contentful Paint < 2,5 s auf Mittelklasse-Android über gedrosseltes 3G	Soll
NF-4	Bedienelemente mindestens 48 × 48 px, Kontrast AA	Soll
NF-5	Bestätigte Eingaben gehen nicht verloren	Muss
NF-8	Daten und KI-Verarbeitung ausschließlich in der EU	Muss
NF-9	Passwörter mit Argon2id, TLS bei jeder Übertragung	Muss
NF-10	Jede Änderung an einem Bericht im audit_eintrag	Muss
Sicherheit	Keine Zugangsdaten im Code, Secret-Scan in CI und Pre-Commit	Muss
9. Technologie
Baustein	Entscheidung
Frontend	React + TypeScript + Vite
Lokale Datenhaltung (optional)	IndexedDB über Dexie
Backend	Node + TypeScript, Fastify
ORM/Migrationen	Drizzle mit versionierten SQL-Migrationen
Datenbank	PostgreSQL 16 mit Row Level Security und pgvector
KI	Azure OpenAI über Microsoft Foundry, Bereitstellungstyp Data Zone Standard (EU), Anmeldung über Entra ID
Tests	Vitest
Entwicklung	GitHub Codespaces, Docker Compose
CI	GitHub Actions: Lint, Tests, gitleaks

Sollte Drizzle Probleme bereiten, sind Kysely oder Prisma Alternativen; Datenmodell und RLS-Regeln bleiben als SQL gleich.

10. Arbeitspakete

Details und Termine stehen im Zeitplan. Reihenfolge ist bindend.

AP	Inhalt	Abnahme
AP-A	Fundament, Repo, Devcontainer, CI, Secret-Scan	App startet, CI grün, Pre-Commit-Hook blockiert erfundene Schlüssel
AP-B	Organisation, Benutzer, Einladung, Rollen, Row Level Security	Isolationstest je Route grün (R-4.3)
AP-C	Stammdaten, Tagesbericht, Filter, Demo-Daten	F-03 bis F-05, F-09
AP-D	KI-Suche (RAG)	F-15
AP-E	Hash-Kette und Prüffunktion	F-12
AP-F	Demo-Paket: README, Architektur, Screencast, Demo-Skript	Fremde Person startet die Demo in unter 15 Minuten
AP-G	Offline-Fähigkeit (optional)	Flugmodus-Test grün

Offene Fragen und Erweiterungen (zum Beispiel Fotos, Unterschrift, PDF) werden am 16.01.2027 neu bewertet. Neuer Umfang kommt nur gegen gleich großen gestrichenen Umfang hinein.
