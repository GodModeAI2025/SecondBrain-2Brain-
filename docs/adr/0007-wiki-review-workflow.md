# ADR 0007 — Frontmatter-gesteuerter Wiki Review Workflow

- **Status:** Accepted
- **Datum:** 2026-04-29

## Kontext

Beim Ingest erzeugt Claude Wiki-Seiten mit `reviewed: false` im YAML-Frontmatter. Der Output-Generator ueberspringt diese Seiten bereits (Compound-Loop-Schutz, damit KI-Output nicht unkontrolliert in den naechsten KI-Input fliesst). Aber: Nutzer hatten keine Uebersicht, welche Seiten Review brauchen, und keinen effizienten Weg, sie abzuarbeiten. Gleichzeitig fehlte bei Outputs die Transparenz, warum bestimmte Quellen uebersprungen werden.

## Entscheidung

**Review Queue** aus Frontmatter-Signalen mit Prioritaets-Scoring:

- `src/main/core/wiki-review.ts` — `buildWikiReviewQueue(pages)` bewertet jede Wiki-Seite anhand von fuenf Gruenden: `unreviewed` (50 Prio), `stale` (40), `seed` (25), `low-confidence` (15), `uncertain` (15). Mehrfach-Gruende addieren sich.
- `src/renderer/pages/ReviewPage.tsx` — Dedizierte Seite mit Filter-Tabs, Inline-Markdown-Preview und Frontmatter-Editor. Nutzer kann `reviewed`, `status`, `confidence` direkt aendern, ohne die Wiki-Seite im Editor oeffnen zu muessen.
- `src/main/core/wiki-frontmatter.ts` — `applyWikiFrontmatterPatch()` aendert Frontmatter-Felder atomar und setzt `updated` automatisch. Nur tatsaechlich geaenderte Felder werden geschrieben.
- Sidebar-Badge zeigt die Anzahl der Review-Items. `useWikiStore.reviewQueue` wird beim Projektwechsel automatisch aktualisiert.

**Output Source Readiness** fuer Compound-Loop-Transparenz:

- `inspectSourceReadiness(projectPath, sourcesPattern)` in `output.ipc.ts` zaehlt pro Output: gefundene Quellen, bereite Quellen, uebersprungene (unreviewed) Quellen.
- `OutputSourceReadiness`-Typ in `api.types.ts` wird bei `output:list` mitgeliefert.
- OutputPage zeigt ein Readiness-Panel mit Ampel (ok/warn/empty), Liste der uebersprungenen Dateien, und direktem Link zur ReviewPage.

**Single Source of Truth ist das YAML-Frontmatter** — kein separater Review-Status in einer Datenbank oder Config-Datei. Alles lebt in der Wiki-Datei selbst und folgt damit dem Git-Sync-Regime des Projekts.

## Konsequenzen

**Positiv**
- Human-in-the-Loop-Qualitaetsgate: KI-generierte Seiten muessen explizit reviewed werden, bevor sie in Outputs einfliessen.
- Transparenz: Nutzer sieht vor der Output-Generierung, welche Quellen uebersprungen werden und warum.
- Kein Zustandssplit: Frontmatter ist die einzige Quelle, funktioniert mit Git-Sync, Backup, und manuellem Editieren.
- Prioritaets-Scoring lenkt den Nutzer automatisch zu den wichtigsten Seiten.

**Negativ**
- Frontmatter muss konsistent gepflegt werden — ein fehlerhaftes `reviewed:`-Feld bricht den Review-Flow.
- Zusaetzliche UI-Komplexitaet durch ReviewPage und Readiness-Panel.
- `inspectSourceReadiness` liest alle Source-Dateien bei jedem `output:list` — bei grossen Wikis spuerbar (akzeptabel, da Output-Anzahl typisch < 10).

## Alternativen

- **Separate Review-Datenbank (SQLite/JSON):** Staerker strukturiert, aber erzeugt Sync-Probleme bei Git-Projekten und Zustandssplit.
- **Review nur im Wiki-Editor:** Funktioniert, aber kein Ueberblick ueber alle offenen Items; jede Seite muss einzeln geoeffnet werden.
- **Automatisches Review nach Timeout:** Wuerde den Compound-Loop-Schutz aushebeln — eine Seite ist nicht dadurch korrekt, dass sie alt genug ist.
