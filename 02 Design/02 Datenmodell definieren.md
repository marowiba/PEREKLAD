# Technisches Datenmodell – Übersetzungs‑App (Version 1.1)

Dieses Dokument definiert das finale logische und programmatische Datenmodell für den Core (Domäne) der Übersetzungs-App. Es integriert alle Optimierungen der Architektur-Review (Version 1.1), ist vollständig frei von UI-Framework-Abhängigkeiten (z. B. PySide/Qt) und sichert die hexagonale Entkopplung der Anwendung ab.

Die Implementierung ist in standardisiertem Python (3.10+) verfasst, nutzt starke Typisierung (`typing`) sowie das `dataclasses`-Modul und dient als direkte Vorlage für die Entwicklung.

---

## 1. Klassen-Diagramm & Datenhierarchie

Die Datenstruktur folgt einer strikten Komposition. Ein Dokument verwaltet seine Metadaten und eine geordnete Liste von Absätzen. Absätze können entweder einfacher Fließtext oder spezialisierte Inhaltsblöcke sein.

```
[Document]
│
├──> [DocumentStatus] (Enum)
│
└──> List von [Paragraph]
│
├──> Paragraph_ID (UUID/String)
├──> [ReviewState] (Enum)
├──> OCR-Konfidenz (float, auf Absatzebene)
├──> [ParagraphHistoryManager] (Isolierter Undo/Redo-Stack)
│
└──> Inhalt (Spezialisierter Blocktyp):
    ├──> [TextBlock] (Normaler Text)
    ├──> [TableBlock] (Strukturierte Daten)
    ├──> [StampBlock] (Metadaten / Siegel)
    └──> [CommentBlock] (Übersetzer-Anmerkung)
```

---

## 2. Core-Enums & Value-Objects

```python
from enum import Enum, auto
from dataclasses import dataclass

class DocumentStatus(Enum):
    """Repräsentiert den aktuellen Lebenszyklus-Status eines Dokuments in der State Machine."""
    IMPORTIERT = auto()
    IN_UEBERSETZUNG = auto()
    WARNUNG = auto()
    BEREIT_ZUR_PRUEFUNG = auto()
    IN_PRUEFUNG = auto()
    EXPORTIERT = auto()


class ReviewState(Enum):
    """Definiert den Bearbeitungsstatus eines einzelnen Segments (Paragraphs)."""
    OFFEN = auto()
    GEPRUEFT = auto()


@dataclass(frozen=True)
class ProgressMetrics:
    """Immutables Value-Object zur Repräsentation des aktuellen Review-Fortschritts für die UI."""
    total: int
    checked: int
    open: int

    @property
    def percentage(self) -> float:
        if self.total == 0:
            return 100.0
        return (self.checked / self.total) * 100.0
```

---

## 3. Spezialisierte Inhaltsblöcke (Polymorphes Content-Modell)

```python
from dataclasses import dataclass, field
from typing import List, Dict, Any

@dataclass
class TextBlock:
    source_text: str
    target_text: str


@dataclass
class TableBlock:
    matrix: List[List[Dict[str, str]]]
    rows: int
    columns: int

    def update_cell(self, row: int, col: int, target_text: str) -> None:
        if 0 <= row < self.rows and 0 <= col < self.columns:
            self.matrix[row][col]["target"] = target_text


@dataclass
class StampBlock:
    stamp_description: str
    is_verified: bool = False


@dataclass
class CommentBlock:
    comment_text: str
```

---

## 4. Segmentierter History-Manager (Absatzbezogenes Undo/Redo)

```python
class ParagraphHistoryManager:
    def __init__(self, initial_state: Any):
        self._undo_stack: List[Any] = [initial_state]
        self._redo_stack: List[Any] = []

    def push_state(self, new_state: Any) -> None:
        if self._undo_stack and self._undo_stack[-1] == new_state:
            return
        self._undo_stack.append(new_state)
        self._redo_stack.clear()

    def undo(self) -> Any | None:
        if len(self._undo_stack) > 1:
            current = self._undo_stack.pop()
            self._redo_stack.append(current)
            return self._undo_stack[-1]
        return None

    def redo(self) -> Any | None:
        if self._redo_stack:
            state = self._redo_stack.pop()
            self._undo_stack.append(state)
            return state
        return None
```

---

## 5. Die Kern-Entitäten: Paragraph & Document

```python
import uuid
import copy
from dataclasses import dataclass, field
from typing import Any, List, Dict

@dataclass
class Paragraph:
    paragraph_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    content: TextBlock | TableBlock | StampBlock | CommentBlock = field(default_factory=lambda: TextBlock("", ""))
    review_state: ReviewState = ReviewState.OFFEN
    ocr_confidence: float = 1.0
    history: ParagraphHistoryManager = field(init=False)

    def __post_init__(self):
        self.history = ParagraphHistoryManager(copy.deepcopy(self.content))

    def update_content(self, updated_content: Any, mark_checked: bool = True) -> None:
        self.content = updated_content
        self.history.push_state(copy.deepcopy(updated_content))
        if mark_checked:
            self.review_state = ReviewState.GEPRUEFT


@dataclass
class Document:
    document_id: str
    source_file_path: str
    target_file_path: str | None = None
    status: DocumentStatus = DocumentStatus.IMPORTIERT
    paragraphs: List[Paragraph] = field(default_factory=list)
    metadata: Dict[str, Any] = field(default_factory=dict)

    @property
    def global_ocr_confidence(self) -> float:
        valid_paragraphs = [p for p in self.paragraphs if isinstance(p.content, (TextBlock, TableBlock))]
        if not valid_paragraphs:
            return 1.0
        return sum(p.ocr_confidence for p in valid_paragraphs) / len(valid_paragraphs)

    @property
    def progress_metrics(self) -> ProgressMetrics:
        total = len(self.paragraphs)
        checked = sum(1 for p in self.paragraphs if p.review_state == ReviewState.GEPRUEFT)
        return ProgressMetrics(total=total, checked=checked, open=total - checked)

    def is_exportable(self) -> bool:
        if self.status == DocumentStatus.EXPORTIERT:
            return False
        metrics = self.progress_metrics
        return metrics.total > 0 and metrics.open == 0
```

---

## 6. Architektur-Vorteile von Version 1.1

1. **Konsistenter Lebenszyklus:** `IN_PRUEFUNG` erlaubt präzise Editor‑Zustandskontrolle.  
2. **Umfassende OCR-Überwachung:** Tabellen fließen korrekt in die Qualitätsmetrik ein.  
3. **Immutable Fortschrittsdaten:** `ProgressMetrics` verhindert unbeabsichtigte Manipulationen.  
