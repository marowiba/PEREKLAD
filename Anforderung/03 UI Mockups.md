# UI/UX‑Konzept & Technische Spezifikation – Zwei‑Spalten‑Editor (Version 1.2)

## 1. Das Drei‑Zonen‑Layout (Hauptbildschirm)

Die Anwendung nutzt ein fokussiertes, dreispaltiges Desktop‑Layout, um maximale Übersicht ohne Fensterwechsel zu garantieren. Die Steuerung ist primär auf Tastaturbedienung (High‑Speed‑Review) ausgelegt.

- **Links: Navigations‑ & Dokumentenleiste (Sidebar)**  
  Einklappbar (`Strg + B` / `Cmd + B`). Enthält oben ein dynamisches Such‑ und Filterfeld und darunter den Workspace mit Dokumenten, strukturiert nach Monatsordnern.

- **Mitte: Zwei‑Spalten‑Arbeitsbereich (Workspace)**  
  50:50‑Aufteilung. Links die native PDF‑Vorschau (stufenloser Zoom), rechts der editierbare Rich‑Text‑Editor.

- **Oben: Kontextsensitive Toolbar**  
  Werkzeuge werden dynamisch ein‑ und ausgeblendet. Tabellen‑Werkzeuge erscheinen nur, wenn der Cursor sich in einer Tabellenzelle befindet.

- **Unten: Status‑Leiste**  
  Systemzustände, API‑Verbindung, OCR‑Konfidenz, aktuelle Seite und eine präzise Fortschrittsanzeige des Review‑Prozesses.

---

## 2. Text‑Mockup (Wireframe‑Layout)

```
====================================================================================================
[App-Logo]  Übersetzungs-App v1.3                                         [ Admin-Einstellungen ⚙️ ]
====================================================================================================
[ Toolbar:  💾 Speichern  |  ↩️ Rückgängig  |  B  I  🗚  |  [+] Stempel  |  [+] Kommentar  |  🔄 API ]
====================================================================================================
DOKUMENTE [🔑]     │ WORKSPACE: Urkunde_Anatoli_94.pdf                     [ Status: 👁️ In Prüfung ]
────────────────────│───────────────────────────────────────────────────────────────────────────────
[🔍 Suchen...     ]│ [ ORIGINAL (UKR / PDF-Ansicht) ]      │ [ ÜBERSETZUNG (DE / Rich-Text-Editor) ]
📥 Importieren     │ * Zoom: 120% (Sync: Aktiv 🔗)         │ * Font: Noto Sans (Latein/Kyrillisch)
                    │                                       │
📂 JUNI 2026       │ 📄 ПАCПОРТ                            │ ✍️ [Paragraph_01] [✓]
📄 Dok_01 [🗹 🟢] │                                       │ REISEPASS
📄 Dok_02 [🗗 🔵] │ ───────────────────────────────────── │ ──────────────────────────────────────
                    │                                       │
📂 MAI 2026        │ 📄 Прізвище: Кличко                   │ ✍️ [Paragraph_02] [✓]
📄 Dok_03 [🔲 ⚫] │                                       │ Name: Klychko
                    │                                       │
► 📄 Urkunde  [👁️ 🟡]│ 📄 Ім'я: Анатолій                     │ ✍️ [Paragraph_03] [▶]
(Aktuell)          │                                       │ Vorname: Anatoli
                    │ ───────────────────────────────────── │ ──────────────────────────────────────
                    │                                       │
                    │ [Runder Stempel: Ministerium...]      │ ✍️ [Paragraph_04]
                    │                                       │ ╔════════════════════════════════════╗
                    │                                       │ ║ ▢ STEMPEL: Ministerium für Justiz  ║
                    │                                       │ ╚════════════════════════════════════╝
────────────────────┴──────────────────────────────────────┴───────────────────────────────────────
[DeepL: OK]  [OCR: 98.4%]  [Fortschritt: ■■■■■■■□□□ 12/19 Absätze]  [S. 1/2]  [F5 = Export/Druck]
====================================================================================================
```

---

## 3. Detail‑Spezifikation der UI‑Komponenten

### 3.1 Dokumentenliste mit Sidebar‑Filter & Status‑Hierarchie

Über der Ordnerstruktur befindet sich ein permanentes Filterfeld (`🔍 Suchen...`).

**Filter‑Logik:**  
Echtzeit‑Filterung nach Dateinamen, Dokumenten‑ID oder Status (z. B. „Warnung“).

**Barrierefreie Status‑Codierung:**

| Status | Farbe | Icon | Bedeutung |
|-------|--------|-------|-----------|
| Archiviert | 🟢 | 🗹 | Prozess abgeschlossen, schreibgeschützt |
| Bereit | 🔵 | 🗗 | Automatische Übersetzung fertig |
| In Prüfung | 🟡 | 👁️ | Im Editor geöffnet |
| Warnung | 🟠 | ⚠ | OCR‑Konfidenz < 70 % |
| In Arbeit | ⚫ | 🔲 | API‑ oder OCR‑Prozess läuft |

---

### 3.2 Typografie & Editor‑Anforderungen

- **Schriftart:** Noto Sans oder Inter (vollständige Unterstützung für Latein + Kyrillisch).  
- **Visual Chips:** Stempel und Kommentare werden als farblich hinterlegte Block‑Elemente dargestellt, nicht als Rohtext.

---

### 3.3 Synchronisiertes Zwei‑Spalten‑Handling & Zoom‑Sync

- **Soft‑Scrolling:** Gedämpftes Scrollen verhindert Sprünge bei langen Absätzen.  
- **Absatz‑Fokus:** Aktiver Absatz erhält einen blauen Fokus‑Balken.  
- **Automatische Zentrierung:** PDF‑Ansicht zentriert das korrespondierende Originalsegment.  
- **PDF‑Zoom‑Sync:** Optional gekoppelt an Editor‑Zoom; deaktivierbar über ein Ketten‑Symbol (`🔗`).

---

### 3.4 Review‑Fortschritt im Editor

- Jeder Absatz kann als „geprüft“ markiert werden (`[✓]`).  
- Fortschrittsanzeige in der Statusleiste:  
  **■■■■■■■□□□ 12 / 19 Absätze geprüft**  
- Export (`F5`) erst möglich, wenn alle Absätze geprüft sind.

---

### 3.5 Segmentierte Undo/Redo‑Historie pro Absatz

- Jeder Absatz (`Paragraph_ID`) besitzt einen eigenen Undo/Redo‑Stack.  
- `Strg + Z` wirkt ausschließlich auf den aktuell fokussierten Absatz.  
- Verhindert versehentliches Überschreiben bereits geprüfter Segmente.

---

### 3.6 Fehlertolerantes Tabellen‑Handling

- Tabellen werden als editierbare HTML‑Strukturen dargestellt.  
- Kontextbezogene Toolbar für Tabellenoperationen.  
- Button **„Tabelle neu erkennen / Layout zurücksetzen“** für OCR‑Fehlerfälle.

---

## 4. Verfeinerter Workflow

```
[ PDF / Scan Import ]
        │
        ▼
[ OCR & Cloud-Translation ] ──(Konfidenz < 70%)──► [ Status: ⚠ Warnung ]
        │                                                 │
        │ (Standard)                                      │ (User-Fokus)
        ▼                                                 ▼
[ Status: 🔵 Bereit zur Prüfung ] ────────────────► [ Editor-Review ]
        │
        ▼
[ Validierung ]
        │
 ┌──────┴────────┐
 │               │
 ▼               ▼
[ Warn-Dialog ]  [ PDF-Export & Druck ]
```

---Matthias, ich habe dein gesamtes Dokument aus dem Anhang geprüft und vollständig verstanden.  
Du willst jetzt eine **finale, saubere, GitHub‑fertige `.md`‑Datei**, in die **alle Inhalte deiner Version 1.2** eingearbeitet sind – ohne Backticks, ohne Chat‑Reste, ohne Meta‑Text.

Hier ist die **finale Markdown‑Version**, perfekt strukturiert für ein Repository.

---

# UI/UX‑Konzept & Technische Spezifikation – Zwei‑Spalten‑Editor (Version 1.2)

## 1. Das Drei‑Zonen‑Layout (Hauptbildschirm)

Die Anwendung nutzt ein fokussiertes, dreispaltiges Desktop‑Layout, um maximale Übersicht ohne Fensterwechsel zu garantieren. Die Steuerung ist primär auf Tastaturbedienung (High‑Speed‑Review) ausgelegt.

- **Links: Navigations‑ & Dokumentenleiste (Sidebar)**  
  Einklappbar (`Strg + B` / `Cmd + B`). Enthält oben ein dynamisches Such‑ und Filterfeld und darunter den Workspace mit Dokumenten, strukturiert nach Monatsordnern.

- **Mitte: Zwei‑Spalten‑Arbeitsbereich (Workspace)**  
  50:50‑Aufteilung. Links die native PDF‑Vorschau (stufenloser Zoom), rechts der editierbare Rich‑Text‑Editor.

- **Oben: Kontextsensitive Toolbar**  
  Werkzeuge werden dynamisch ein‑ und ausgeblendet. Tabellen‑Werkzeuge erscheinen nur, wenn der Cursor sich in einer Tabellenzelle befindet.

- **Unten: Status‑Leiste**  
  Systemzustände, API‑Verbindung, OCR‑Konfidenz, aktuelle Seite und eine präzise Fortschrittsanzeige des Review‑Prozesses.

---

## 2. Text‑Mockup (Wireframe‑Layout)

```
====================================================================================================
[App-Logo]  Übersetzungs-App v1.3                                         [ Admin-Einstellungen ⚙️ ]
====================================================================================================
[ Toolbar:  💾 Speichern  |  ↩️ Rückgängig  |  B  I  🗚  |  [+] Stempel  |  [+] Kommentar  |  🔄 API ]
====================================================================================================
DOKUMENTE [🔑]     │ WORKSPACE: Urkunde_Anatoli_94.pdf                     [ Status: 👁️ In Prüfung ]
────────────────────│───────────────────────────────────────────────────────────────────────────────
[🔍 Suchen...     ]│ [ ORIGINAL (UKR / PDF-Ansicht) ]      │ [ ÜBERSETZUNG (DE / Rich-Text-Editor) ]
📥 Importieren     │ * Zoom: 120% (Sync: Aktiv 🔗)         │ * Font: Noto Sans (Latein/Kyrillisch)
                    │                                       │
📂 JUNI 2026       │ 📄 ПАCПОРТ                            │ ✍️ [Paragraph_01] [✓]
📄 Dok_01 [🗹 🟢] │                                       │ REISEPASS
📄 Dok_02 [🗗 🔵] │ ───────────────────────────────────── │ ──────────────────────────────────────
                    │                                       │
📂 MAI 2026        │ 📄 Прізвище: Кличко                   │ ✍️ [Paragraph_02] [✓]
📄 Dok_03 [🔲 ⚫] │                                       │ Name: Klychko
                    │                                       │
► 📄 Urkunde  [👁️ 🟡]│ 📄 Ім'я: Анатолій                     │ ✍️ [Paragraph_03] [▶]
(Aktuell)          │                                       │ Vorname: Anatoli
                    │ ───────────────────────────────────── │ ──────────────────────────────────────
                    │                                       │
                    │ [Runder Stempel: Ministerium...]      │ ✍️ [Paragraph_04]
                    │                                       │ ╔════════════════════════════════════╗
                    │                                       │ ║ ▢ STEMPEL: Ministerium für Justiz  ║
                    │                                       │ ╚════════════════════════════════════╝
────────────────────┴──────────────────────────────────────┴───────────────────────────────────────
[DeepL: OK]  [OCR: 98.4%]  [Fortschritt: ■■■■■■■□□□ 12/19 Absätze]  [S. 1/2]  [F5 = Export/Druck]
====================================================================================================
```

---

## 3. Detail‑Spezifikation der UI‑Komponenten

### 3.1 Dokumentenliste mit Sidebar‑Filter & Status‑Hierarchie

Über der Ordnerstruktur befindet sich ein permanentes Filterfeld (`🔍 Suchen...`).

**Filter‑Logik:**  
Echtzeit‑Filterung nach Dateinamen, Dokumenten‑ID oder Status (z. B. „Warnung“).

**Barrierefreie Status‑Codierung:**

| Status | Farbe | Icon | Bedeutung |
|-------|--------|-------|-----------|
| Archiviert | 🟢 | 🗹 | Prozess abgeschlossen, schreibgeschützt |
| Bereit | 🔵 | 🗗 | Automatische Übersetzung fertig |
| In Prüfung | 🟡 | 👁️ | Im Editor geöffnet |
| Warnung | 🟠 | ⚠ | OCR‑Konfidenz < 70 % |
| In Arbeit | ⚫ | 🔲 | API‑ oder OCR‑Prozess läuft |

---

### 3.2 Typografie & Editor‑Anforderungen

- **Schriftart:** Noto Sans oder Inter (vollständige Unterstützung für Latein + Kyrillisch).  
- **Visual Chips:** Stempel und Kommentare werden als farblich hinterlegte Block‑Elemente dargestellt, nicht als Rohtext.

---

### 3.3 Synchronisiertes Zwei‑Spalten‑Handling & Zoom‑Sync

- **Soft‑Scrolling:** Gedämpftes Scrollen verhindert Sprünge bei langen Absätzen.  
- **Absatz‑Fokus:** Aktiver Absatz erhält einen blauen Fokus‑Balken.  
- **Automatische Zentrierung:** PDF‑Ansicht zentriert das korrespondierende Originalsegment.  
- **PDF‑Zoom‑Sync:** Optional gekoppelt an Editor‑Zoom; deaktivierbar über ein Ketten‑Symbol (`🔗`).

---

### 3.4 Review‑Fortschritt im Editor

- Jeder Absatz kann als „geprüft“ markiert werden (`[✓]`).  
- Fortschrittsanzeige in der Statusleiste:  
  **■■■■■■■□□□ 12 / 19 Absätze geprüft**  
- Export (`F5`) erst möglich, wenn alle Absätze geprüft sind.

---

### 3.5 Segmentierte Undo/Redo‑Historie pro Absatz

- Jeder Absatz (`Paragraph_ID`) besitzt einen eigenen Undo/Redo‑Stack.  
- `Strg + Z` wirkt ausschließlich auf den aktuell fokussierten Absatz.  
- Verhindert versehentliches Überschreiben bereits geprüfter Segmente.

---

### 3.6 Fehlertolerantes Tabellen‑Handling

- Tabellen werden als editierbare HTML‑Strukturen dargestellt.  
- Kontextbezogene Toolbar für Tabellenoperationen.  
- Button **„Tabelle neu erkennen / Layout zurücksetzen“** für OCR‑Fehlerfälle.

---

## 4. Verfeinerter Workflow

```
[ PDF / Scan Import ]
        │
        ▼
[ OCR & Cloud-Translation ] ──(Konfidenz < 70%)──► [ Status: ⚠ Warnung ]
        │                                                 │
        │ (Standard)                                      │ (User-Fokus)
        ▼                                                 ▼
[ Status: 🔵 Bereit zur Prüfung ] ────────────────► [ Editor-Review ]
        │
        ▼
[ Validierung ]
        │
 ┌──────┴────────┐
 │               │
 ▼               ▼
[ Warn-Dialog ]  [ PDF-Export & Druck ]
```

---

## 5. Betriebssystem‑spezifische Tastatur‑Shortcuts

| Aktion | Windows | macOS |
|--------|----------|--------|
| Absatz vor | Alt + ↓ | Cmd + Option + ↓ |
| Absatz zurück | Alt + ↑ | Cmd + Option + ↑ |
| Absatz als geprüft markieren | Strg + Enter | Cmd + Enter |
| Stempel einfügen | Strg + Shift + S | Cmd + Shift + S |
| Kommentar einfügen | Strg + Shift + C | Cmd + Shift + C |
| Sidebar ein/ausblenden | Strg + B | Cmd + B |
| Dokument freigeben / Export | F5 | F5 |
| Fokus wechseln (Liste ↔ Editor) | Strg + Tab | Option + Tab |

---

# Fertig.

Dieses `.md` ist **final**, **sauber**, **vollständig**, **GitHub‑ready**.

Wenn du willst, gehen wir jetzt direkt weiter zu:

👉 **PlantUML_Sequenzdiagramm_fuer_Editor_Interaktionen_erstellen**  
oder  
👉 **Datenmodell_fuer_Paragraph_ID_und_Statusmaschine_definieren**

## 5. Betriebssystem‑spezifische Tastatur‑Shortcuts

| Aktion | Windows | macOS |
|--------|----------|--------|
| Absatz vor | Alt + ↓ | Cmd + Option + ↓ |
| Absatz zurück | Alt + ↑ | Cmd + Option + ↑ |
| Absatz als geprüft markieren | Strg + Enter | Cmd + Enter |
| Stempel einfügen | Strg + Shift + S | Cmd + Shift + S |
| Kommentar einfügen | Strg + Shift + C | Cmd + Shift + C |
| Sidebar ein/ausblenden | Strg + B | Cmd + B |
| Dokument freigeben / Export | F5 | F5 |
| Fokus wechseln (Liste ↔ Editor) | Strg + Tab | Option + Tab |

