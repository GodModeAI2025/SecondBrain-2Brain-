# ADR 0009 — ask() Migration auf Streaming API

- **Status:** Accepted
- **Datum:** 2026-04-29

## Kontext

Die `ask()`-Funktion in `src/main/core/claude.ts` nutzte bisher `anthropic.messages.create()` fuer synchrone (nicht-gestreamte) Anfragen. Bei langen Antworten (Ingest grosser Dokumente, Lint-Suggest) kam es zu Timeout-Problemen: die HTTP-Verbindung stand minutenlang ohne Daten, bis die vollstaendige Antwort auf einmal eintraf. Manche Proxy- und Netzwerk-Setups unterbrachen diese idle Connections.

## Entscheidung

`ask()` nutzt jetzt `anthropic.messages.stream()` mit `stream.finalMessage()`:

- Die Verbindung empfaengt kontinuierlich SSE-Events (Token-Deltas), auch wenn der Aufrufer nur die finale Antwort braucht.
- Retry-Logik ist inline in `ask()` statt in einer separaten `withRetries()`-Wrapper-Funktion. Die generische Wrapper-Funktion wurde entfernt.
- `askStreaming()` fuer den Query-Chat bleibt unveraendert — dort wurde bereits gestreamt.

## Konsequenzen

**Positiv**
- Keine idle Connections mehr — Proxies und Load Balancer sehen kontinuierlichen Traffic.
- Einheitliches Verbindungsmuster: sowohl `ask()` als auch `askStreaming()` nutzen jetzt die Streaming-API.
- Retry-Logik ist nahe am Aufrufer und einfacher zu debuggen.

**Negativ**
- Minimaler Overhead durch SSE-Parsing, auch wenn nur die finale Nachricht gebraucht wird. In der Praxis vernachlaessigbar.
- `withRetries()` als generische Utility entfaellt — falls kuenftig andere Funktionen Retries brauchen, muss die Logik erneut implementiert werden.
