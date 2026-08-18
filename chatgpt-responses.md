Excellent. ✅ PDF Loader is now working.

The output confirms that the scanner successfully processed:

app1.txt
app2.txt
sample.txt
sample.pdf

and extracted actual PDF text. So the abstract-class issue is resolved.

Current Phase 1 status

Project Skeleton       ✅
Document Model         ✅
Base Loader            ✅
TXT Loader             ✅
Loader Factory         ✅
Knowledge Scanner      ✅
PDF Loader             ✅

DOCX Loader            ⬜
Excel Loader           ⬜
Mixed-file ingestion   🟨
Chunking               ⬜
Embeddings             ⬜
ChromaDB                ⬜
Retrieval               ⬜
RAG                     ⬜
Chatbot                 ⬜

One important observation: your PDF is clearly a real enterprise/SOP-type document, and pypdf is already extracting its text successfully. That's a good sign for the actual Phase 1 use case.


---

Increment 5 — DOCX Loader

Now we'll add Word documents.

Goal

Support:

.docx → DocxLoader → Document

using the python-docx package that is already part of our offline SDK dependencies.

Brief reason

SOPs, operational procedures, architecture notes, and internal documentation can commonly exist as Word documents. We want them entering the same normalized Document pipeline as TXT and PDF.


---

1. Create

app/
└── loaders/
    └── docx_loader.py

Put this in it:

from pathlib import Path

from docx import Document as DocxDocument

from app.loaders.base_loader import BaseLoader
from app.models.document import Document


class DocxLoader(BaseLoader):
    """
    Loader for DOCX documents.
    """

    def load(self, file_path: Path) -> Document:

        doc = DocxDocument(file_path)

        paragraphs = []

        for paragraph in doc.paragraphs:
            text = paragraph.text.strip()

            if text:
                paragraphs.append(text)

        content = "\n".join(paragraphs)

        return Document(
            source=file_path.name,
            file_type="docx",
            content=content,
            metadata={
                "paragraphs": str(len(paragraphs))
            }
        )


---

2. Update loader_factory.py

Add:

from app.loaders.docx_loader import DocxLoader

Then add:

if extension == ".docx":
    return DocxLoader()

So your factory should now handle:

.txt
.pdf
.docx


---

3. Create a sample DOCX

Create:

knowledge/sample.docx

You can make it in Microsoft Word with:

Enterprise AI SDK

Application: Payment API

Namespace: payment-prod

Cluster: VKS-PR-01

Environment: Production

Save it under knowledge.


---

4. Run the existing scanner

No scanner modification is required.

Run:

python -m scripts.test_scanner

Expected:

Loaded 5 documents

and you should see:

sample.docx
docx

Enterprise AI SDK
Application: Payment API
Namespace: payment-prod
Cluster: VKS-PR-01
Environment: Production


---

Important

Don't modify KnowledgeScanner.

That's the benefit of the architecture we've created:

┌── TxtLoader
                 │
Knowledge ───────┼── PdfLoader
                 │
                 └── DocxLoader
                       ▲
                       │
                 LoaderFactory
                       ▲
                       │
                 KnowledgeScanner

We're simply plugging another loader into the existing pipeline.

Implement DOCX and run the same scanner. Once you get the expected output, we'll move to the Excel loader, which is particularly important for your actual SBI inventories.
