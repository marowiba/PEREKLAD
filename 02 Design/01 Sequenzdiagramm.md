# PlantUML-Sequenzdiagramm – Kern-Workflow (Version 1.1)

## 1. Verhaltensspezifikation und Interaktionsströme

Das folgende Sequenzdiagramm beschreibt die dynamischen Abläufe der Übersetzungs-App. Es visualisiert die sequenzielle Abfolge von Funktionsaufrufen und Events zwischen den Systemkomponenten (Frontend, Core-Domäne und Infrastruktur-Adapter) entlang des gesamten Dokumenten-Lebenszyklus.

### Kernmerkmale der Modellierung:
- **Asynchrone Entkopplung:** Hintergrundprozesse (OCR-Verarbeitung und API-Anfragen) laufen vollständig blockierungsfrei ab, sodass die Benutzeroberfläche jederzeit reaktiv bleibt.
- **Zustandssicherheit (State Machine):** Statusübergänge werden strikt deterministisch über den `DocStatusStateMachine`-Teilnehmer geschaltet.
- **Isolierter Kontext:** Editor-Interaktionen (wie die Undo/Redo-Historie) sind granular an die jeweilige `paragraphId` gebunden, um unerwünschte Nebeneffekte auf globaler Ebene zu verhindern.
- **Harte Validierung:** Der Export-Prozess erzwingt eine formale Prüfung (Beglaubigungsformel, rechtliche Schablone) im Core, bevor die PDF-Generierung initiiert wird.

---

## 2. PlantUML-Spezifikationscode

```plantuml
@startuml
autonumber
skinparam BoxPadding 10
skinparam ParticipantPadding 10

box "User Interface (Frontend)" #LightBlue
actor "Übersetzerin" as User
participant "MainLayout / Sidebar" as UI
participant "EditorView" as EditUI
participant "PdfPreviewComponent" as PdfUI
participant "DialogManager" as Dialog
end box

box "App Core (Domain)" #LightYellow
participant "WorkspaceOrchestrator" as Core
participant "DocStatusStateMachine" as FSM
participant "EditorEngine" as Engine
database "ParagraphHistoryManager" as History
participant "ExportValidator" as Validator
end box

box "Infrastructure / Adapters" #LightGray
participant "FileSystemAdapter" as Storage
participant "OcrAdapter" as OCR
participant "TranslationAdapter" as Trans
participant "PdfGeneratorEngine" as PDF
participant "OS_PrintService" as OSPrint
end box

== 1. Datei-Import & Asynchrone Hintergrundverarbeitung ==

User -> UI: Drag & Drop: PDF-Datei
UI -> Core: importDocument(filePath)
activate Core

Core -> Storage: readFile(filePath)
activate Storage
Storage --> Core: fileBuffer, metaData
deactivate Storage

Core -> FSM: createDocument(metaData)
activate FSM
FSM -> FSM: State transition to [Importiert]
FSM --> Core: docId, status: Importiert
deactivate FSM

Core --> UI: updateSidebar(docId, status: Importiert)
note right of UI: UI zeigt Dokument mit grauem Spinner an

Core -> FSM: transitionTo(In_Uebersetzung)
activate FSM
FSM --> Core: status: In_Uebersetzung
deactivate FSM
Core --> UI: updateSidebar(docId, status: In_Uebersetzung)

Core -> OCR: extractTextAndLayout(fileBuffer)
activate OCR
note over OCR: Verarbeitet gemischte\nInhalte (UKR/DE)
OCR --> Core: rawTextSegments, layoutTree, confidenceScore
deactivate OCR

alt OCR-Konfidenz < 70%
    Core -> FSM: triggerEvent(OcrConfidenceLow)
    activate FSM
    FSM -> FSM: State transition to [Warnung]
    FSM --> Core: status: Warnung
    deactivate FSM
    Core --> UI: updateSidebar(docId, status: Warnung, icon: ⚠)
end

Core -> Trans: translateSegments(rawTextSegments)
activate Trans
note over Trans: Modularer API-Aufruf\n(DeepL/OpenAI/Gemini)
alt API Error (e.g., HTTP 429)
    Trans -> Trans: Execute Retry with Exponential Backoff
end
Trans --> Core: translatedSegments
deactivate Trans

Core -> Core: createParagraphDataModel(rawTextSegments, translatedSegments)

Core -> FSM: transitionTo(Bereit_zur_Pruefung)
activate FSM
FSM --> Core: status: Bereit_zur_Pruefung
deactivate FSM
Core --> UI: updateSidebar(docId, status: Bereit_zur_Pruefung, icon: 🗗)
deactivate Core

== 2. Editor-Interaktionen (Review-Modus) ==

User -> EditUI: Navigiert zu Absatz (Alt + ↓ / Cmd + Option + ↓)
EditUI -> Engine: setActiveParagraph(paragraphId)
activate Engine

Engine -> Engine: Get internal line/page references
Engine --> PdfUI: centerOnSegment(paragraphId)
note right of PdfUI: PDF-Vorschau scrollt gedämpft\n(Smooth Scroll) zum Originalsegment

Engine --> EditUI: highlightActiveParagraph(paragraphId)
deactivate Engine

User -> EditUI: Ändert Zoom im Editor (Strg + Mausrad)
note over EditUI: Zoom beeinflusst nur Darstellung,\nnicht Paragraph-Datenmodell
EditUI -> EditUI: Check state of Chain-Icon (🔗)
alt Zoom-Sync ist AKTIV
    EditUI -> PdfUI: setZoomLevel(targetPercent)
end

User -> EditUI: Führt Textänderung durch
EditUI -> Engine: updateText(paragraphId, updatedText)
activate Engine
Engine -> History: pushState(paragraphId, updatedText)
Engine --> EditUI: renderUpdatedParagraph(paragraphId)
deactivate Engine

User -> EditUI: Drückt Strg + Z (Undo)
EditUI -> Engine: triggerLocalUndo(paragraphId)
activate Engine
Engine -> History: popPreviousState(paragraphId)
activate History
History --> Engine: previousTextState
deactivate History
Engine --> EditUI: renderUpdatedParagraph(paragraphId)
deactivate Engine
note right of EditUI: Nur der fokussierte Absatz wird modifiziert.\nBisherige Absätze bleiben unangetastet.

== 3. Review-Fortschritt & Export-Workflow ==

User -> EditUI: Markiert Absatz als geprüft (Strg + Enter)
EditUI -> Core: setParagraphChecked(paragraphId)
activate Core
Core -> Core: Recalculate progress metrics (e.g., 12/19)
Core --> UI: updateProgressBar(progressString)
deactivate Core

User -> UI: Drückt F5 (Export / Freigabe)
UI -> Core: triggerExportWorkflow(docId)
activate Core

Core -> Validator: validateDocumentRequirements(docId)
activate Validator
note over Validator: Prüft Vorhandensein von:\n- Beglaubigungsformel\n- Seitenzahlen ("Seite X von Y")

alt Validierung FEHLGESCHLAGEN (z.B. Formel fehlt)
    Validator --> Core: validationErrorsList
    Core --> Dialog: showWarningDialog(validationErrorsList)
    note left of Dialog: Workflow stoppt;\nFehlermeldung blockiert Export
else Validierung ERFOLGREICH
    Validator --> Core: validationPassed
    deactivate Validator
    
    Core -> PDF: generateFinalPdf(documentData, templates)
    activate PDF
    note over PDF: Inkludiert Layout-Schablone,\nStempel-Chips & Schlusskomponente
    PDF --> Core: finalPdfBuffer
    deactivate PDF
    
    Core -> Storage: saveFinalDocument(targetPath, fileNamePattern, finalPdfBuffer)
    activate Storage
    note over Storage: Legt PDF in vordefinierter\nMonats-Struktur ab
    Storage --> Core: fileSavedPath
    deactivate Storage
    
    Core -> FSM: transitionTo(Exportiert)
    activate FSM
    FSM -> FSM: State transition to [Exportiert]
    FSM --> Core: status: Exportiert
    deactivate FSM
    
    Core --> UI: exportCompleted(fileSavedPath)
    deactivate Core
    
    UI -> Dialog: triggerSuccessAndPrintPrompt(fileSavedPath)
    activate Dialog
    User -> Dialog: Bestätigt "Jetzt physisch drucken"
    Dialog -> OSPrint: sendToDefaultPrinter(fileSavedPath)
    deactivate Dialog
    note right of OSPrint: Manueller Hardcopy-Schritt\n(Stempel & Unterschrift außerhalb der App)
end

@enduml

```

---

## 3. Architektur-Konformität

Dieses Verhaltensmodell korrespondiert vollständig mit den strukturellen Festlegungen der vorhergehenden Entwicklungsphasen:

1. **Ports & Adapters:** Komponenten der rechten Box (Infrastruktur) agieren strikt gekapselt hinter ihren definierten Systemschnittstellen (`ITranslationService`, `IOcrService`, `IPdfGenerator`, `IFileSystem`). Eine Auswechslung der konkreten Adapter-Implementierungen hat keinerlei Auswirkungen auf den Kontrollfluss des Kerns.
2. **Datenmodell-Konsistenz:** Der hierarchische Aufbau von `Document` zu segmentierten Einheiten mit fester `Paragraph_ID` ist die informationstechnische Bedingung für den synchronisierten Fokus- und Render-Mechanismus.

