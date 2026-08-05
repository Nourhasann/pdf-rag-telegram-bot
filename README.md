# PDF-Powered Telegram Chatbot (RAG with Gemma) + Customer Requests

A Google Colab notebook that turns two PDF documents into a working Telegram bot:

1. **Chat Q&A** — answers questions about a narrative PDF (e.g. an SLA) using Retrieval-Augmented Generation (RAG) with **Gemma-2b-it**.
2. **`/request`** — walks a customer through name → item → quantity, matches each item to a known category from a second, data-table PDF using Gemma, and logs it to Excel (a shared master log plus a per-customer file).
3. **`/requests`** — sends back the current shared customer requests log.

## How it works

- **Chat PDF** is split into overlapping text chunks, embedded with `all-MiniLM-L6-v2`, and indexed with FAISS for retrieval.
- **Data PDF** tables are extracted with a custom `pdfplumber`-based parser that builds its own row/column grid from the page's rectangle shapes (rather than relying on default line detection), splits on section headers, and stitches multi-page tables back together. This gives the known category list used to match customer requests.
- **Gemma-2b-it** handles both the RAG answers and the category-matching for customer requests.
- **Telegram** integration is via `pyTelegramBotAPI`.

## Requirements

- Google Colab with a GPU runtime
- A Hugging Face account with access to [`google/gemma-2b-it`](https://huggingface.co/google/gemma-2b-it) (accept the license on the model page first)
- A Telegram bot token from [@BotFather](https://t.me/BotFather)

## Setup

1. Open the notebook in Google Colab.
2. Add two Colab Secrets (🔑 icon in the left sidebar → **Add new secret**), with **Notebook access** turned on:
   - `HF` — your Hugging Face access token
   - `telegram` — your Telegram bot token
3. Run the cells in order from top to bottom.
   - The first cell installs dependencies and restarts the Colab runtime automatically — this is expected (you may briefly see "Session crashed"). Continue running from the next cell once it's back.
4. When prompted, upload your two PDFs:
   - A narrative document (used for chat Q&A)
   - A data-table document (used to match categories for `/request`)
5. Update `SLA_PDF_PATH` and `DATA_PDF_PATH` in the notebook to match your filenames if they differ from the defaults.
6. Once the final cell is running, your bot is live — message it on Telegram.

## Customizing table extraction

The data-PDF table parser splits pages using a list of section-header strings (`section_markers`). If your document has a different layout, adjust that list to match the text that precedes each table/section in your own PDF.

## Notes

- No API keys are hardcoded — both tokens are pulled from Colab Secrets at runtime.
- The bot cell blocks and polls continuously; keep the Colab session open for the bot to keep responding.
- This notebook was built and tuned against a specific pair of SLA-style PDFs; extraction logic (especially the table splitting) may need tweaking for documents with a different layout.

## Data

This bot works with **two PDFs that you supply yourself** — they are not included in this repo and shouldn't be committed to it (see `.gitignore` below).

**PDF 1 — the "chat" document (narrative/text)**
Any text-heavy document you want to ask questions about — a contract, agreement, policy doc, manual, etc. The bot reads all the text, breaks it into overlapping chunks, and searches those chunks to answer questions grounded in that document.

**PDF 2 — the "catalog" document (structured table)**
A document containing one or more tables listing categories/items — for example a price list, service catalog, or product/work-type table. The notebook expects at least one table with a `Work Type` and `Limit` column (adjust `find_main_lead_time_table()` if your table uses different column names). This becomes the list of valid categories that customer-submitted item requests get matched against.

**Example of the expected shape of PDF 2's main table:**

| Work Type | Limit | ... |
|---|---|---|
| Category A | ... | ... |
| Category B | ... | ... |

**Data generated while the bot runs (also not committed):**
- `customer_requests.xlsx` — a running log of every request from every user
- `request_<chat_id>.xlsx` — a per-user file that accumulates that user's requests over time

None of this — your source PDFs or the generated Excel logs — should ever end up in the repo, since they'll contain real business or customer data. The `.gitignore` below takes care of that automatically as long as you keep your files in `*.pdf` / `*.xlsx` form.



This project generates and/or consumes files that may contain private business or customer data. **Do not commit these:**

```
*.pdf
*.xlsx
```

Add that to a `.gitignore` before pushing, and double-check your commit history doesn't already contain any real PDFs, Excel exports, or tokens.

## License

_Add a license of your choice (e.g. MIT) here._
