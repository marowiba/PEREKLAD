# AGENTS.md

## Projektüberblick
Dieser Workspace enthält die Entwurfs- und Anforderungsdokumentation für eine App zur teilautomatisierten Übersetzung und Qualitätsprüfung ukrainischer Dokumente.

## Wichtige Referenzen
- [01 Anforderung/01 Anforderung.md](01%20Anforderung/01%20Anforderung.md) – Zielsetzung, Rollen, Prozess, Statusmodell und Qualitätskriterien.
- [01 Anforderung/02 Technische Architektur.md](01%20Anforderung/02%20Technische%20Architektur.md) – Architekturziel, Komponenten, Datenfluss und technische Richtlinien.
- [01 Anforderung/03 UI Mockups.md](01%20Anforderung/03%20UI%20Mockups.md) – UI- und UX-Anforderungen.
- [02 Design/01 Sequenzdiagramm.md](02%20Design/01%20Sequenzdiagramm.md) – Kern-Workflows und Interaktionsabläufe.
- [02 Design/02 Datenmodell definieren.md](02%20Design/02%20Datenmodell%20definieren.md) – Domänenmodelle und Statusstruktur.
- [02 Design/03 API‑Adapter‑Interfaces.md](02%20Design/03%20API%E2%80%91Adapter%E2%80%91Interfaces.md) – Ports, Adapter und Domain-Exceptions.
- [Arbeitsablauf/next.md](Arbeitsablauf/next.md) – Nächste technische Schritte und Priorisierung.

## Arbeitsregeln für AI-Agenten
- Nutze diese Dokumente als primäre Quelle der Wahrheit. Neue Implementierungen oder Änderungen sollten mit den vorhandenen Anforderungen und dem Design übereinstimmen.
- Behalte die bestehende Fachsprache bei: Übersetzungsprozess, OCR, Review, Export, Monatsordnerstruktur und Statusmodell.
- Halte die Architektur konsistent mit der hexagonalen Struktur: Core-Domäne, Ports und Adapter.
- Wenn du Code vorschlägst oder implementierst, arbeite klein und modular; vermeide Annahmen, die nicht in den Entwurfsdokumenten belegt sind.
- Wenn ein Punkt unklar ist, prüfe zuerst die relevanten Unterlagen im Ordner 01 Anforderung und 02 Design, bevor du Änderungen vornimmst.

## Erwartete Projektstruktur
- 01 Anforderung/: Anforderungen, Architektur und UI/UX-Dokumentation.
- 02 Design/: Sequenzen, Datenmodell und technische Schnittstellen.
- Arbeitsablauf/: Planung und nächste Umsetzungsschritte.

## Kurzfassung für neue Agenten
Dieses Repo ist aktuell ein Spezifikations- und Entwurfsprojekt. Die Hauptaufgabe besteht darin, Anforderungen, Architektur und Design sauber zu pflegen und bei Implementierungsarbeiten mit den bestehenden Dokumenten konsistent zu bleiben.
