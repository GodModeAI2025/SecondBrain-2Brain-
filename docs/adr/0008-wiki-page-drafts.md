# ADR 0008 — Kategorie-basierte Wiki-Seitenanlage

- **Status:** Accepted
- **Datum:** 2026-04-29

## Kontext

Wiki-Seiten koennen aus verschiedenen Kontexten heraus angelegt werden: manuell, aus offenen Wikilinks (Pending Stubs), oder als Synthese aus Takeaway-Diskussionen. Bisher gab es keine einheitliche Logik fuer Pfad-Bestimmung, Frontmatter-Initialisierung und Kategorie-Zuordnung — jeder Erstellungspfad hatte eigene Konventionen.

## Entscheidung

`src/main/core/wiki-page-draft.ts` — zentrale Factory `createWikiPageDraft(input)` fuer alle Wege der Seitenanlage:

- **Kategorie-Routing:** Sechs Kategorien (`sources`, `entities`, `concepts`, `syntheses`, `sops`, `decisions`) bestimmen den Unterordner. Default: `concepts/`. Aus der Kategorie leitet sich der `type`-Frontmatter-Wert ab (z.B. `concepts` → `concept`).
- **Slug-Generierung:** Titel wird zu URL-sicherem Slug normalisiert (via `slugify()`). Doppelte Brackets, Pipe-Aliase und Pfad-Fragmente werden bereinigt.
- **Seed-Frontmatter:** Neue Seiten starten immer mit `status: seed`, `confidence: low`, `reviewed: false`, `tags: [stub]`. Das stellt sicher, dass sie automatisch im Review Workflow (ADR 0007) auftauchen.
- **Kontext-Link:** Optionaler `sourcePath` erzeugt einen Wikilink-Rueckverweis auf die Seite, aus der heraus der Stub angelegt wurde.
- **Path-Safety:** Traversal-Versuche (`..`, absolute Pfade) werden abgewiesen.

## Konsequenzen

**Positiv**
- Einheitliches Verhalten unabhaengig vom Erstellungspfad — manuelle Anlage, Stub-Aufloesung und Synthese erzeugen identische Seitenstruktur.
- Nahtlose Integration mit Review Workflow: Seed-Seiten erscheinen sofort in der Review Queue.
- Slug-Normalisierung verhindert Dateinamen-Konflikte und Sonderzeichen-Probleme.

**Negativ**
- Sechs feste Kategorien sind opinionated — Nutzer mit anderem Taxonomie-Modell muessen sich anpassen oder die Kategorie nachtraeglich aendern.
- `confidence: low` bei Seed-Seiten ist eine Annahme — manche manuell angelegte Seiten sind von Anfang an hochwertig.

## Alternativen

- **Freie Pfadwahl ohne Kategorie-Routing:** Flexibler, aber verliert die automatische Typ-Zuordnung und erzeugt uneinheitliche Wiki-Strukturen.
- **Kategorie per Prompt/Dialog abfragen:** Mehr Kontrolle, aber Reibung bei der Erstellung — besonders bei Batch-Anlage aus Pending Stubs.
