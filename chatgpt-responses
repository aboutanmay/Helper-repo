Excellent. This is another successful milestone. The scanner is correctly discovering all files recursively, selecting the appropriate loader through the factory, and returning normalized Document objects.

We're no longer building isolated classes—we're building a working ingestion pipeline.

Progress

✅ Project Skeleton
✅ Document Model
✅ Base Loader
✅ TXT Loader
✅ Loader Factory
✅ Knowledge Scanner

⬜ PDF Loader
⬜ DOCX Loader
⬜ Excel Loader
⬜ Chunking Engine
⬜ Embedding Service
⬜ ChromaDB
⬜ Retrieval
⬜ RAG
⬜ Enterprise Chatbot


---

Next Increment: PDF Loader

Now we'll start supporting real enterprise documentation.

Goal

Load PDFs (Runbooks, SOPs, Architecture Docs) into the same Document object.

This is probably the most common document type you'll have in SBI.


---

Step 1

Create

app/
    loaders/
        pdf_loader.py


---

Step 2

Paste

from pathlib import Path

from pypdf import PdfReader

from app.loaders.base_loader import BaseLoader
from app.models.document import Document


class PdfLoader(BaseLoader):
    """
    Loader for PDF documents.
    """

    def load(self, file_path: Path) -> Document:

        reader = PdfReader(file_path)

        text = ""

        for page in reader.pages:
            page_text = page.extract_text()

            if page_text:
                text += page_text + "\n"

        return Document(
            source=file_path.name,
            file_type="pdf",
            content=text,
            metadata={
                "pages": str(len(reader.pages))
            }
        )


---

Step 3

Update loader_factory.py

Currently you have:

if extension == ".txt":
    return TxtLoader()

Add:

from app.loaders.pdf_loader import PdfLoader

and

if extension == ".pdf":
    return PdfLoader()


---

Step 4

Place any simple PDF inside

knowledge/

It doesn't matter what it contains.

For example

knowledge/

sample.pdf


---

Step 5

Run

python -m scripts.test_scanner


---

Expected Output

Now you should see something like

Loaded 4 documents

sample.txt

...

sample.pdf

...

pages : 5

The scanner should automatically detect the PDF without modifying the scanner code.


---

Why this is an important milestone

Notice what just happened:

Previously:

Knowledge Folder

↓

TXT only

Now:

Knowledge Folder

↓

TXT

↓

PDF

↓

Same Document Object

The scanner doesn't know or care whether it's loading TXT or PDF.

That's exactly how we want to build the system.


---

A small architectural suggestion

Before moving to DOCX and Excel, I'd like to make one enhancement.

Currently, PdfLoader returns the entire PDF as one large string.

That works for testing, but it's not ideal for RAG.

After we verify the PDF loader, I'd like to slightly redesign the Document metadata to preserve information such as:

Number of pages

Author (if available)

Title (if available)

Creation date (if available)


This metadata will become very useful later when searching, filtering, and citing enterprise documents, and it's easier to add now before we start indexing hundreds of files.
