Excellent. ✅ Chunk model is verified.

The output confirms that Chunk is importing and instantiating correctly:

Chunk(
    id='...',
    document_id='document-123',
    content='Application: Payment API',
    chunk_index=0,
    metadata={'source': 'sample.txt'}
)

So we're ready for the first actual chunking implementation.

Increment 8 — Text Chunker

Goal

Take one Document and split its content into multiple Chunk objects.

Document
   │
   │ split
   ▼
┌─────────┐
│ Chunk 0 │
├─────────┤
│ Chunk 1 │
├─────────┤
│ Chunk 2 │
└─────────┘

Brief reason

LLMs and embedding models work better when we retrieve relevant portions of a document instead of sending an entire large document every time.

For our first implementation, we'll use character-based chunking. We won't optimize chunk size/overlap yet; we'll make the mechanism work first and tune it later.


---

1. Create the chunker

Create:

app/
└── chunking/
    └── text_chunker.py

Use:

from app.models.document import Document
from app.models.chunk import Chunk


class TextChunker:
    """
    Splits a Document into smaller text chunks.
    """

    def __init__(self, chunk_size: int = 500, overlap: int = 50):
        self.chunk_size = chunk_size
        self.overlap = overlap

    def chunk(self, document: Document) -> list[Chunk]:

        if not document.content:
            return []

        chunks = []

        start = 0
        chunk_index = 0

        while start < len(document.content):

            end = start + self.chunk_size

            chunk_text = document.content[start:end]

            chunks.append(
                Chunk(
                    document_id=document.id,
                    content=chunk_text,
                    chunk_index=chunk_index,
                    metadata={
                        **document.metadata,
                        "source": document.source,
                        "file_type": document.file_type
                    }
                )
            )

            chunk_index += 1

            start = end - self.overlap

        return chunks


---

2. Create the test

Create:

scripts/test_chunker.py

from pathlib import Path

from app.loaders.loader_factory import LoaderFactory
from app.chunking.text_chunker import TextChunker


file_path = Path("knowledge/sample.txt")

loader = LoaderFactory.get_loader(file_path)

document = loader.load(file_path)

chunker = TextChunker(
    chunk_size=50,
    overlap=10
)

chunks = chunker.chunk(document)

print(f"Document: {document.source}")
print(f"Total chunks: {len(chunks)}")

for chunk in chunks:

    print("\n" + "=" * 50)

    print(f"Chunk index: {chunk.chunk_index}")
    print(f"Chunk ID: {chunk.id}")
    print(f"Document ID: {chunk.document_id}")
    print(f"Content:\n{chunk.content}")


---

3. Run

From the project root:

python -m scripts.test_chunker


---

Expected result

You should see something approximately like:

Document: sample.txt
Total chunks: 4

==================================================
Chunk index: 0
Chunk ID: ...
Document ID: ...
Content:
Enterprise AI SDK

Application Name: Payment AP

==================================================
Chunk index: 1
Chunk ID: ...
Document ID: ...
Content:
ayment API

Namespace: payment-prod
...

The exact boundaries will depend on your sample.txt.

Important thing to check

Notice:

Chunk 0
     │
     └── document_id = same Document ID

Chunk 1
     │
     └── document_id = same Document ID

That relationship is intentional.


---

One thing I want you to observe

We're deliberately using:

chunk_size=50
overlap=10

only for testing.

Do not worry yet about whether 500, 1000, 512, etc. is the "correct" production value.

Later we'll evaluate chunking specifically for your:

PDFs

Excel inventories

SOPs

architecture documents

runbooks


and choose a better strategy.

For now, we are validating:

Document
    ↓
TextChunker
    ↓
List[Chunk]

Once this test passes, we'll have completed the Document → Chunk stage and can move toward the embedding layer.
