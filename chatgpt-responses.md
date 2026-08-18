Perfect. ✅ DOCX Loader is verified.

Your screenshot confirms:

Loaded 5 documents

and the DOCX was successfully extracted into the common Document structure. The existing TXT and PDF loaders continue to work as well.

Phase 1 ingestion progress

Project Skeleton       ✅
Document Model         ✅
Base Loader            ✅
TXT Loader             ✅
Loader Factory         ✅
Knowledge Scanner      ✅
PDF Loader             ✅
DOCX Loader            ✅

Excel Loader           ⬜  ← Next
Mixed-file ingestion   🟨
Chunking               ⬜
Embedding              ⬜
ChromaDB               ⬜
Retrieval              ⬜
RAG                    ⬜
Chatbot                ⬜

Increment 6 — Excel Loader

This one is particularly important for our actual Phase 1 because your enterprise knowledge will contain things like:

Application inventory

Cluster inventory

Database inventory

Environment mappings

Infrastructure inventory


These will likely be Excel spreadsheets.

Goal

Implement:

.xlsx
  ↓
ExcelLoader
  ↓
Document

For the first version, we'll preserve the sheet name in metadata and convert the spreadsheet rows into text.


---

1. Create excel_loader.py

Create:

app/
└── loaders/
    └── excel_loader.py

Use:

from pathlib import Path

import pandas as pd

from app.loaders.base_loader import BaseLoader
from app.models.document import Document


class ExcelLoader(BaseLoader):
    """
    Loader for Excel (.xlsx) documents.
    """

    def load(self, file_path: Path) -> Document:

        sheets = pd.read_excel(
            file_path,
            sheet_name=None
        )

        sections = []

        for sheet_name, dataframe in sheets.items():

            dataframe = dataframe.fillna("")

            sections.append(
                f"Sheet: {sheet_name}\n"
            )

            sections.append(
                dataframe.to_string(index=False)
            )

            sections.append("\n")

        content = "\n".join(sections)

        return Document(
            source=file_path.name,
            file_type="xlsx",
            content=content,
            metadata={
                "sheets": str(len(sheets))
            }
        )


---

2. Update loader_factory.py

Add:

from app.loaders.excel_loader import ExcelLoader

and:

if extension == ".xlsx":
    return ExcelLoader()

Your factory now supports:

.txt   → TxtLoader
.pdf   → PdfLoader
.docx  → DocxLoader
.xlsx  → ExcelLoader


---

3. Create a sample Excel file

Create:

knowledge/sample.xlsx

For example:

Application	Namespace	Cluster	Environment

Payment API	payment-prod	VKS-PR-01	Production
Customer API	customer-prod	VKS-PR-02	Production


You can create this using Excel itself.


---

4. Run the existing scanner

We don't change KnowledgeScanner.

Run:

python -m scripts.test_scanner

Expected:

Loaded 6 documents

You should see something similar to:

sample.xlsx
xlsx

Sheet: Sheet1

Application    Namespace       Cluster      Environment
Payment API    payment-prod    VKS-PR-01    Production
Customer API   customer-prod   VKS-PR-02    Production


---

Important observation

At this point we'll have completed the first version of the entire document ingestion layer:

Knowledge/
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
         TXT        PDF        DOCX
          │          │          │
          └──────────┼──────────┘
                     ▼
                 XLSX
                     │
                     ▼
              Loader Factory
                     │
                     ▼
             Document Objects

Once Excel passes, don't add more loaders yet.

We'll do one important checkpoint:

Ingestion Layer Validation

We'll test that:

all four formats load;

unsupported files are ignored;

nested directories work;

metadata is preserved;

malformed files don't crash the entire scan;

the scanner produces a clean collection of Document objects.


Then we'll move to the next major subsystem: Chunking.

That is where the raw documents become suitable for embeddings and eventually ChromaDB.
