# SecondBrain (2Brain) Desktop

Dein **Second Brain** auf dem Desktop — lokal-first, KI-gestuetzt, Git-versioniert.
Wirft rohe Notizen, PDFs, DOCX oder HTML-Dokumente hinein und laesst Claude daraus ein strukturiertes, verlinktes Markdown-Wiki bauen. Deine Daten bleiben auf deiner Festplatte, jede Aenderung ist ein Git-Commit.

> Projekt-Website: [**docs/index.html**](docs/index.html) · GitHub Pages: `main` + `/docs`

## Features

- **Upload & Konvertierung** — Drag & Drop von PDF, DOCX, HTML, TXT, CSV, JSON. Automatische Markdown-Konvertierung.
- **KI-Ingestion** — Claude analysiert neue Quellen und erzeugt/aktualisiert Wiki-Seiten mit Entitaeten, Konzepten und Synthesen.
- **Takeaway-Diskussion** — Pro extrahierter Kernaussage ein Sparring-Chat; optional als Synthese-Seite speichern.
- **Wiki-Browser** — Markdown-Rendering mit klickbaren Wikilinks, Frontmatter-Badges und Baumnavigation.
- **Knowledge Graph** — Interaktive Netzwerk-Visualisierung aller Wiki-Seiten und deren Verlinkungen.
- **KI-Query** — Fragen ans Wiki stellen mit BM25-Ranking (persistenter Index), Token-Streaming und Quellenangaben.
- **Gesundheitscheck + Lernvorschlaege** — Mechanische Reparatur plus KI-generierte Fragen, Luecken und Synthese-Kandidaten.
- **Output-Generierung** — Benutzerdefinierte Synthesen aus dem Wiki; Marp-Praesentationen werden in der App als Slide-Deck gerendert.
- **Built-in Skills** — Mitgelieferte Prompt-Bausteine (z.B. `marp-presentation`) per Klick installierbar.
- **Git-Sync** — Automatische Versionierung und Synchronisation ueber ein Git-Repository.
- **Multi-Projekt** — Mehrere Wissensbasen in einem Repository verwalten.

Architektur- und Feature-Entscheidungen sind in [`docs/adr/`](docs/adr/) dokumentiert.

## Voraussetzungen

- Node.js 20+
- Anthropic API-Key (Claude)
- Git-Repository mit Token-Zugang (optional, fuer Sync)

## Setup

```bash
npm install
npm run dev
```

Beim ersten Start fuehrt ein Setup-Wizard durch:
1. API-Key eingeben
2. Git-Repository klonen (oder ueberspringen)
3. Projekt erstellen oder auswaehlen

## Entwicklung

```bash
npm run dev          # Build + Electron starten
npx tsc --noEmit     # TypeScript pruefen
npm run lint         # ESLint
npm test             # Tests ausfuehren
```

## Architektur

```
Renderer (React)  <--IPC-->  Main (Node.js)  <--Git-->  Remote Repo
     |                            |
  Zustand Stores            Core-Module
  React Router              (Vault, Claude, Prompts)
  react-force-graph-2d      Services
  markdown-it               (Git, Project, Settings, Convert)
```

**Main-Prozess:** Dateizugriff, Claude API, Git-Operationen, Konvertierung.
**Renderer:** React-UI mit Zustand State Management, kein direkter Node.js-Zugriff.
**Preload:** Typisierte Bridge zwischen Main und Renderer (`window.api`).

## Zwei `index.html` — nicht verwechseln

Das Repo enthaelt bewusst zwei HTML-Einstiegspunkte mit unterschiedlichem Zweck:

- **`./index.html`** — Vite-Entry fuer den Electron-Renderer. Laedt `src/renderer/main.tsx` und wird **nur** von der Desktop-App zur Laufzeit benoetigt. Nicht manuell oeffnen.
- **`./docs/index.html`** — Standalone Landingpage (Englisch, ohne Build-Step). Wird ueber GitHub Pages (Source: `main` + `/docs`) als Projekt-Website ausgeliefert.

Wer beim Klonen direkt `index.html` im Browser oeffnet, sieht nur den Lade-Placeholder der App. Die eigentliche UI laeuft ueber `npm run dev` in Electron.

## Lizenz

Apache License 2.0 — siehe [LICENSE](LICENSE).
