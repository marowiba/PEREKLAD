# Technischer Architektur‑Entwurf – Übersetzungs‑App (High‑Level, Version 1.1)

## 1. App‑Framework Evaluierung & Empfehlung
Für eine Desktop‑App auf Windows und macOS, die lokale Dateioperationen, OCR, PDF‑Generierung und externe APIs kombiniert, kommen zwei primäre Frameworks in Frage.

### Empfehlung: Python PySide6 oder Electron (TypeScript/Node.js)

| Kriterium | Python + PySide6 | Electron + React/Vue |
| :--- | :--- | :--- |
| **Vorteile** | Sehr gute Bibliotheken für lokale Verarbeitung (OCR, PDF), schlank, performant | Modernes UI‑Ökosystem, starke Web‑Tooling‑Pipeline |
| **Nachteile** | UI‑Styling weniger flexibel als Web‑CSS | Hoher RAM‑Verbrauch durch Chromium |

**Architektur‑Entscheidung:**  
Das Design bleibt technologie‑neutral, aber strikt optimiert für eine klare Trennung von UI und Core‑Logik (Ports & Adapters).

---

## 2. Architektur‑Pattern: Ports & Adapters (Hexagonal)
Die App folgt einem Hexagonalen Architekturmodell, um API‑Austauschbarkeit, Testbarkeit und Plattformneutralität sicherzustellen.

### Core (Domäne)
- Dokumenten‑Lebenszyklus und Status-Verwaltung  
- Status‑Validierung entlang der Prozesskette  
- Formale und rechtliche Regeln für vereidigte Übersetzungen  
- *Keine Abhängigkeit von externen Diensten (DeepL, OpenAI), Dateipfaden oder dem UI-Framework*

### Ports (Schnittstellen / Abstraktionen)
- `ITranslationService`  
- `IOcrService`  
- `IPdfGenerator`  
- `IFileSystem`

### Adapters (Konkrete Implementierungen)
- **Translation:** `DeepLAdapter`, `OpenAiGptAdapter`, `GeminiAdapter`  
- **OCR:** `TesseractLocalAdapter`, `GoogleVisionCloudAdapter`  
- **PDF-Engine:** `WeasyPrintAdapter`, `ReportLabAdapter`  
- **Infrastruktur:** OS-spezifische Dateisystem-Adapter für Windows/macOS  

---

## 3. Modul‑Struktur & Komponenten

```
[  UI / User Interface  ]
│
▼
[ App-Core / Orchestrator ] <──> [ Dokumenten-Status-Maschine ]
│
├─► [ OCR-Modul ] ──────────► IOcrService
├─► [ Translation-Modul ] ──► ITranslationService
├─► [ PDF-Engine ] ─────────► IPdfGenerator
└─► [ Storage-Modul ] ──────► IFileSystem
```

### 3.1 OCR‑Service
- Extraktion kyrillischer & lateinischer Zeichen aus Scans und Dokumenten  
- Unterstützung für lokale (Offline) oder optionale Cloud‑OCR  
- Berechnung der Erkennungsgenauigkeit (Konfidenzwert) pro Seite  
- Triggerung des Events `OcrConfidenceLow` bei einem Konfidenzwert unter 70 %  

### 3.2 Translation‑Service (API‑Adapter‑Layer)
- Dynamischer API‑Manager lädt die in den Einstellungen aktiven Adapter  
- Implementierung flexibler Schnittstellen zu DeepL, OpenAI (GPT‑4o) oder Google Gemini  
- Ausfallsicherheit durch Circuit Breaker und automatisches Retry (3 Versuche mit exponentiellem Backoff bei HTTP 429/500)  

### 3.3 Editor‑Engine
- Logische Kopplung und synchronisiertes Scrolling der UI‑Ansichten  
- Paragraph‑basiertes Datenmodell zur internen Repräsentation  
- Garantierte Zuordnung Original‑Segment ↔ Übersetzungs‑Segment über eine eindeutige `Paragraph_ID`  

### 3.4 PDF‑Engine
- Generierung des finalen Dokuments via HTML‑Templates oder Layout‑Bibliotheken  
- Automatisches Einfügen der landesspezifischen Beglaubigungsformel  
- Standardisierte Seitenzahlen im Format „Seite X von Y der Übersetzung“  
- Dynamische Layout‑Berechnung für den Platzbedarf von Stempel und Unterschrift am Dokumentenende  

---

## 4. Logging & Monitoring

### Log‑Streams
- **Audit‑Trail:** Revisionssichere Protokollierung aller manuellen Benutzeraktionen im Editor  
- **Technical Log:** Erfassung von API‑Antwortzeiten, Netzwerk‑Verbindungsstatus und Fehlercodes  
- **Quality Log:** Fortlaufende Dokumentation der OCR‑Konfidenzwerte zur System‑Optimierung  

### Datenschutz‑Filter & Speicherung
- Regex‑basierte Maskierung zur Anonymisierung personenbezogener Daten in den technischen Logs  
- Rollierende Logfiles (max. 5 Dateien à 10 MB)  
- **Speicherort Windows:** `%APPDATA%/TranslationApp/logs/`  
- **Speicherort macOS:** `~/Library/Application Support/TranslationApp/logs/`  

---

## 5. Deployment & Update‑Modell

### Windows
- Verteilung über Standalone‑Installer (`.exe` via Inno Setup oder `.msi`)  
- *Zwingende Voraussetzung:* Signierung via Code‑Signing‑Zertifikat zur Vermeidung von SmartScreen‑Warnungen  

### macOS
- Verpackung als native `.app` innerhalb eines `.dmg`‑Containers  
- *Zwingende Voraussetzung:* Notarisierung über den Apple Notarization Service zur Gatekeeper‑Kompatibilität  

### Updates
- Manueller Update‑Check innerhalb der administrativen Einstellungen  
- Abgleich einer verschlüsselten Versionsdatei auf dem Server  
- Kein automatisches Hintergrund‑Update zur Gewährleistung der Prozesskontrolle in regulierten Umgebungen  

---

## 6. Zusammenfassung der Architektur‑Vorteile
- **API‑Austauschbarkeit:** Anbieterwechsel (LLM/Übersetzung) ohne Codeänderung im Kern  
- **Plattformneutralität:** Identische Geschäftslogik für Windows und macOS  
- **Hohe Testbarkeit:** Core‑Logik vollständig isoliert über Ports testbar  
- **Betriebsstabilität:** Robuste Fehlerbehandlung durch Circuit Breaker & Retry  
- **Revisionssicherheit:** Lückenlose Nachvollziehbarkeit von Textanpassungen durch Audit‑Trail  
