# API‑Adapter‑Interfaces (Ports) Spezifikation – Version 1.1

Dieses Dokument definiert die primären und sekundären **Ports (Schnittstellen)** der App‑Core‑Domäne. Sie bilden die technologische Schutzmauer der hexagonalen Architektur. Alle externen Dienste (OCR, KI‑Übersetzung, Dateisystem und PDF‑Generierung) werden über diese abstrakten Kontrakte entkoppelt.

Die Spezifikation ist als ausführbarer Python‑Code (`abc.ABC`) umgesetzt und integriert das strukturierte `OcrResult`‑Value‑Object.

---

## 1. Systemübersicht: Ports im Hexagon

```
                [ Infrastruktur-Schicht / Adapter ]
                                │
┌───────────────────────────────┴───────────────────────────────┐
│  [OcrAdapter]      [DeepL/OpenAI]       [ReportLab]    [LocalFS]
│       │                  │                   │             │
▼       ▼                  ▼                   ▼             ▼
┌───────────────┐      ┌───────────────┐   ┌───────────────┐ ┌───────────────┐
│  IOcrService  │      │ITranslationSvc│   │ IPdfGenerator │ │  IFileSystem  │
└───────┬───────┘      └───────┬───────┘   └───────┬───────┘ └───────┬───────┘
        │                      │                   │                 │
        └──────────────────────┼───────────────────┴─────────────────┘
                               │
                               ▼
                    [ CORE-DOMÄNE (Business) ]
```

---

## 2. Detaillierte Datentypen für Ports

```python
from dataclasses import dataclass
from typing import List, Dict, Any
from domain.models import Paragraph

@dataclass(frozen=True)
class OcrResult:
    """
    Kapselt das vollständige Ergebnis einer OCR-Analyse.
    Transportiert neben den Domänen-Absätzen auch geometrische Layout-Metadaten
    für die Zwei-Spalten-Synchronisation (Smooth-Scroll/Zentrierung).
    """
    paragraphs: List[Paragraph]
    layout_tree: Dict[str, Any]  # Bounding-Boxes, Seiten-Koordinaten, Rotationsdaten
```

---

## 3. Port‑Spezifikationen (Python‑Interfaces)

```python
from abc import ABC, abstractmethod
from pathlib import Path
from typing import List, Dict, Any

from domain.models import Document, Paragraph
from domain.ports import OcrResult
```

### IOcrService

```python
class IOcrService(ABC):
    """
    Port für optische Zeichenerkennung (OCR).
    """

    @abstractmethod
    def extract_text_and_layout(self, file_buffer: bytes) -> OcrResult:
        """
        Analysiert ein Dokument und liefert ein OcrResult.

        Raises:
            OcrProcessingException
        """
        pass
```

### ITranslationService

```python
class ITranslationService(ABC):
    """
    Port für KI-Übersetzungsdienste (DeepL, OpenAI, Gemini).
    """

    @abstractmethod
    def translate_paragraphs(self, paragraphs: List[Paragraph], target_lang: str = "DE") -> List[Paragraph]:
        """
        Übersetzt die Quellsegmente der Paragraphs.

        Raises:
            TranslationApiException
        """
        pass
```

### IPdfGenerator

```python
class IPdfGenerator(ABC):
    """
    Port für die PDF-Engine.
    """

    @abstractmethod
    def generate_final_pdf(self, document: Document, template_config: Dict[str, Any]) -> bytes:
        """
        Rendert das finale PDF.

        Raises:
            PdfGenerationException
        """
        pass
```

### IFileSystem

```python
class IFileSystem(ABC):
    """
    Port für Dateioperationen (OS-agnostisch).
    """

    @abstractmethod
    def read_file(self, file_path: Path) -> bytes:
        """
        Liest eine Datei.

        Raises:
            StorageAccessException
        """
        pass

    @abstractmethod
    def save_final_document(self, base_directory: Path, file_name: str, file_data: bytes) -> Path:
        """
        Speichert das exportierte PDF in der Monatsstruktur:

        [base_directory]/[YYYY-MM]/[file_name]

        Raises:
            StorageAccessException
        """
        pass
```

---

## 4. Gekapselte Domänen‑Exceptions

```python
class DomainException(Exception):
    pass

class OcrProcessingException(DomainException):
    pass

class TranslationApiException(DomainException):
    pass

class PdfGenerationException(DomainException):
    pass

class StorageAccessException(DomainException):
    pass
```

---

## 5. Vorteile der Version 1.1

1. **OcrResult liefert Layout‑Metadaten** → ermöglicht pixelgenaue PDF‑Zentrierung.  
2. **Deterministischer Dateifluss** → `save_final_document` garantiert gültigen Pfad.  
3. **Strikte Entkopplung** → Core bleibt vollständig unabhängig von SDKs.  

---