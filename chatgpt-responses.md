Perfect. ✅ Excel Loader is verified.

Your screenshot shows the Excel file being detected and converted into a Document, including the sheet name and tabular content. So our complete initial ingestion layer is now working.

Phase 1 — Current Status

Project Skeleton        ✅
Document Model          ✅
Base Loader             ✅
TXT Loader              ✅
PDF Loader              ✅
DOCX Loader             ✅
Excel Loader            ✅
Loader Factory          ✅
Knowledge Scanner       ✅

--------------------------------
DOCUMENT INGESTION COMPLETE
--------------------------------

Chunking Engine         ⬜  ← NEXT
Embedding Service       ⬜
ChromaDB                ⬜
Retrieval               ⬜
RAG Prompt              ⬜
Qwen Integration        ⬜
Enterprise Chatbot      ⬜
FastAPI                 ⬜

We have now reached an important architectural boundary:

RAW KNOWLEDGE
                      │
                      ▼
              ┌───────────────┐
              │     Scanner   │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │ Loader Factory│
              └───────┬───────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
       TXT           PDF           DOCX
                                     
                      XLSX
                       │
                       ▼
                 Document[]

Everything above this line is now working.


---

Increment 7 — Introduce the Chunk Model

Before writing the actual chunking algorithm, we'll create the object that represents a chunk.

Goal

Convert:

Document

eventually into:

Chunk 1
Chunk 2
Chunk 3
...

But first we need to define what a Chunk looks like.


---

Brief reason

A Document is the original source.

A Chunk is a smaller searchable unit derived from that source.

Later, one chunk = one embedding/vector.

We also need to retain the relationship:

Chunk → Original Document → Source File

This will eventually allow the chatbot to say where its answer came from.


---

Step 1 — Create chunk.py

Create:

app/
└── models/
    └── chunk.py

Put this inside:

from dataclasses import dataclass, field
from typing import Dict
import uuid


@dataclass
class Chunk:
    """
    Represents a searchable chunk derived from a Document.
    """

    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    document_id: str = ""
    content: str = ""
    chunk_index: int = 0
    metadata: Dict[str, str] = field(default_factory=dict)


---

Step 2 — Why these fields?

For now, only remember the purpose:

id

Unique identity of the chunk.

document_id

Links the chunk back to the original Document.

content

The actual text that will eventually be embedded.

chunk_index

Tells us whether this is chunk 0, 1, 2, etc.

metadata

Carries information from the original document.

For example:

source = ApplicationInventory.xlsx
sheet  = Production

or:

source = Runbook.pdf
page   = 12


---

Step 3 — Verify the model

Create:

scripts/test_chunk.py

Put:

from app.models.chunk import Chunk


chunk = Chunk(
    document_id="document-123",
    content="Application: Payment API",
    chunk_index=0,
    metadata={
        "source": "sample.txt"
    }
)

print(chunk)

Run:

python -m scripts.test_chunk

Expected:

Chunk(
    id='...',
    document_id='document-123',
    content='Application: Payment API',
    chunk_index=0,
    metadata={'source': 'sample.txt'}
)


---

Checkpoint

At this point we should have:

Document
   │
   │ split later
   ▼
Chunk

Don't implement the actual splitting algorithm yet.

First verify that Chunk imports and instantiates correctly, exactly like we did with Document.

Once this passes, Increment 8 will be the actual TextChunker, where we'll take a real Document and produce multiple chunks. That's the first step where we'll need to make a decision about chunk size and overlap.
