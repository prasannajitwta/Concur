# Simple Plan: Chat + DOCX Editing in One Webpage

## Direct answers
- **Yes**, you can do this with **only Blob Storage** for now.
- **Cosmos DB is not required** for MVP.
- **Yes**, you can use a single **LLM API call with tool/function calling** (agent-like flow) to perform tasks like edit section, add table, add image.
- **Yes**, show the document in the **same webpage** by rendering a preview and refreshing after each edit.

## MVP architecture (keep it simple)
1. **Frontend (single page)**
   - Left: chat panel.
   - Right: document preview panel.
2. **Python backend (FastAPI)**
   - Endpoint: `POST /chat-edit`
   - Endpoint: `GET /document/{docId}` (latest DOCX link)
3. **Storage**
   - Azure Blob only:
     - `documents/{docId}/v1.docx`, `v2.docx`, ...
     - `documents/{docId}/images/...`
4. **LLM + tools**
   - LLM receives user message + document outline.
   - LLM returns tool calls (JSON ops), backend applies them using `python-docx`.

## Minimal edit operations
Use a very small operation set:
- `replace_section(heading, new_text)`
- `add_table(heading, columns, rows)`
- `add_image(heading, image_blob_url, width)`

This is enough for most business edits.

## Simple request flow
1. User types: “Update Scope section and add pricing table”.
2. Backend downloads latest `.docx` from Blob.
3. Backend extracts headings/outline.
4. LLM API call returns operations JSON.
5. Python applies operations and uploads new version (`v{n+1}.docx`).
6. Frontend reloads preview on right side (same page).

## Same-page document rendering options
Pick one of these:
1. **Best simple option**: convert DOCX -> HTML/PDF on backend and show in right panel.
2. **Download/view option**: show “latest version” link + embedded viewer iframe if available.

For fast MVP, many teams render to PDF and embed it in the page.

## Do you need Cosmos later?
Add Cosmos only when you need:
- chat history search,
- multi-user collaboration locks,
- task tracking/analytics,
- section index caching at scale.

Until then, Blob + in-memory processing is enough.

## Recommended stack (minimal)
- FastAPI
- python-docx
- Azure Blob SDK
- LLM API (OpenAI/Azure OpenAI function/tool calling)
- Optional: websocket for “editing…” live status

## Final recommendation
Start with **Blob-only + FastAPI + python-docx + LLM tool calling**. 
Keep all editing in one chat flow and refresh the document preview in the same webpage after every save.
