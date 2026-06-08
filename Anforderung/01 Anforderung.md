# Anforderungsbeschreibung – Übersetzungsprozess & Übersetzungs‑App (Version 1.3)

## 1. Zielsetzung
Das System dient der teilautomatisierten Erstellung beeidigter/vereidigter Übersetzungen aus dem Ukrainischen ins Deutsche. Es nimmt PDF-Dokumente entgegen, führt eine KI-gestützte Erstübersetzung inklusive OCR durch, ermöglicht eine manuelle Korrektur im System und erzeugt ein final formatiertes PDF. Das finale Dokument wird anschließend außerhalb der App gedruckt, gestempelt und unterschrieben.  
Die App muss plattformübergreifend auf Windows und macOS laufen.

---

## 2. Benutzerrollen & Berechtigungen
Zur zielgerichteten UI-Gestaltung und Rechteverwaltung werden folgende Rollen definiert:

- **Übersetzerin (Fachnutzer):** Kernrolle. Hat Zugriff auf den Editor, führt das Review durch, korrigiert Texte/Layouts und gibt Dokumente für den Export frei.  
- **Sachbearbeitung (Assistenz):** Übernimmt den Import von Dokumenten, prüft den Status und stößt den finalen Druck/Export an. Kein Zugriff auf die API-Konfiguration.  
- **Administrator:** Konfiguriert die App, hinterlegt und testet API-Schlüssel, verwaltet Pfade für die Monats-Ordnerstruktur und hat Einblick in die System-Logs.

---

## 3. Prozessbeschreibung & Datenfluss (UI-Flow)
Der funktionale Ablauf innerhalb der App folgt einem strikten, linearen Workspace-Konzept:

```
1. Import ──> 2. Automatische Verarbeitung (OCR/API) ──> 3. Zwei-Spalten-Editor ──> 4. Freigabe & Export
```

1. **Input-Annahme:** Eingang von PDF-Dateien (digitaler Text, Scans/Bilder oder hybride Dokumente; Einzel- oder Massenimport) per Drag & Drop.  
2. **System-Verarbeitung:** Automatische Durchführung von OCR, Layout-Analyse und KI-Übersetzung direkt nach dem Import.  
3. **Review & Freigabe:** Manuelle Korrektur von Text- und Formatfehlern im synchronisierten Editor durch die Übersetzerin.  
4. **Output & Export:** Generierung des finalen PDFs und automatische Archivierung in einer vordefinierten Monats-Ordnerstruktur.  
5. **Hardcopy-Schritt:** Manueller Ausdruck des PDFs, Stempelung und Unterschrift außerhalb der App.

---

## 4. Dokumenten-Lebenszyklus (Status-Modell)
Jedes Dokument durchläuft zwingend folgende Status-Kette, die in der Workspace-Übersicht visualisiert wird:

`Importiert` → `In Übersetzung` → `Bereit zur Prüfung` → `In Prüfung` → `Freigegeben` → `Exportiert` → `Archiviert`

---

## 5. App‑Funktionen

### 5.1 Input‑Verarbeitung & Vorbereitung
- **PDF‑Import:** Flexibler Import via Drag & Drop oder Datei-Auswahl.  
- **Multilinguale OCR:** Texterkennung optimiert für gemischte Inhalte (Kyrillisch/Ukrainisch und Lateinisch/Deutsch), insbesondere für Scans, Stempel und Zertifikate.  
- **Layout‑Erkennung:** Erkennung und temporäre Speicherung von Strukturmerkmalen (Tabellen, Absätze, Briefköpfe), um das Original-Layout bestmöglich zu wahren.

### 5.2 Übersetzung & Review-Modus (Editor)
- **API‑Abstraktionsschicht:** Modularer Adapter für verschiedene Anbieter (DeepL, OpenAI GPT‑4o, Google Gemini). Wechsel ohne Codeänderung.  
- **Zwei-Spalten-Editor:** Synchronisierte Ansicht von Original (links) und Übersetzung (rechts) mit gekoppeltem Scrollen.  
- **Layout-Erhaltung:** Anpassung einfacher Formatierungen (Fett, Kursiv, Tabellenstruktur) während des Reviews.  
- **Änderungsverfolgung (Audit Trail):** Schreibgeschützte Protokollierung aller Änderungen.

### 5.3 Output & Formatierung für vereidigte Übersetzungen
Die PDF-Engine muss standardisierte Vorlagen generieren, die folgende Elemente enthalten:

- **Überschrift:** „Übersetzung aus dem Ukrainischen“.  
- **Strukturtreue:** Kennzeichnung nicht übersetzbarer Elemente (z. B. *[Runder Stempel: …]*).  
- **Seitenzählung:** „Seite X von Y der Übersetzung“.  
- **Schlusskomponente:** Platzhalter für Beglaubigungsformel, Ort, Datum, Stempel- und Unterschriftenfeld.  
- **Automatisierter Export:** Speicherung im Monatsordner mit Namensschema `JJJJ-MM-TT_Dateiname_DE.pdf`.

---

## 6. Ausnahme- und Fehlerbehandlung (Exception Handling)
- **Leere/Beschädigte PDFs:** Fehlermeldung und Abbruch.  
- **Schlechte OCR-Qualität:** Warnung bei <70 % Konfidenz.  
- **Netzwerk-/API-Ausfälle:** Retry mit Exponential Backoff; bei Fehlschlag klare Fehlermeldung; lokaler Bearbeitungsstand bleibt erhalten.

---

## 7. Logging & Monitoring
- **System- & Fehlerlogs:** App-Crashes, Datei-Fehler, Netzwerkabbrüche.  
- **API-Nutzungslogs:** Token-/Zeichenzählung pro Request (ohne Textinhalte).  
- **Qualitätslogs:** Durchschnittliche OCR-Konfidenz pro Dokument.

---

## 8. Datenschutz & Sicherheit
- **Lokale Datenhaltung:** Keine dauerhafte Speicherung auf Drittservern.  
- **Datenschutzkonforme APIs:** Business-APIs ohne Training auf Kundendaten.  
- **Temporäre Dateien:** Automatische Bereinigung nach Export oder App-Schließung.

---

## 9. Technische Rahmenbedingungen
- **Plattformen:** Windows 10/11 und macOS (Intel & Apple Silicon).  
- **Technologie-Stack:** Technologie-neutral (Electron/Node.js, Python/PySide, Flutter, .NET MAUI).

---

## 10. Nicht‑funktionale Anforderungen & Qualitätsmetriken

### 10.1 Performance-Benchmarks
- **Standard-Dokumente:** ≤ 30 s für bis zu 5 Seiten.  
- **Große Dokumente:** ≤ 120 s für bis zu 20 Seiten.  
- **Ressourceneffizienz:** UI bleibt responsiv (asynchron).

### 10.2 Qualitäts-KPIs
- **OCR-Genauigkeit:** ≥ 95 % bei Standarddokumenten.  
- **Layout-Abweichung:** Keine strukturellen Verschiebungen; Editor muss Korrekturen ermöglichen.  
- **Usability:** Minimale Klickwege; durchgängiger Workflow ohne Fensterwechsel.

