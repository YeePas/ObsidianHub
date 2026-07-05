# Obsidian Capture — Telegram → n8n → Obsidian/Todoist

An n8n workflow that turns Telegram messages — plain text, voice memos, and reMarkable handwritten notes (PDF/PNG) — into structured Markdown notes in an Obsidian vault, with action items automatically created as Todoist tasks.

It's a pure **capture** pipeline: nothing gets interpreted or acted on, everything sent through the bot becomes a real note in the inbox.

## How it works

You message the **Obsi8Bot** on Telegram. Depending on what you send, the workflow branches:

| Input | What happens |
|---|---|
| Text message | Converted directly to a Markdown note |
| Voice message | Downloaded, transcribed, converted to a Markdown note |
| reMarkable PDF export | Sent to OpenAI for full OCR/analysis, converted to a structured note with summary, sections, action items, and illustration references |
| reMarkable PNG (single page) | Same idea as the PDF flow, for a single exported page image — runs as a separate n8n workflow, same bot, see [Status](#status) |

Every note is saved to `Obsidian/Inbox/...` in a Dropbox folder that syncs into the Obsidian vault. You get a confirmation message back in Telegram once it's done.

### reMarkable notation system

For handwritten notes, the OCR prompt interprets a fixed set of conventions:

| On the page | Becomes in Markdown |
|---|---|
| Underlined text (first occurrence) | Main title (`#`) |
| Underlined text (every following occurrence) | Subtitle (`##`) |
| A full horizontal line across the row | Start of a new section |
| An asterisk (`*`) next to a line | Work note → bullet (`- `) |
| An empty square next to a line | Action item → checkable task (`- [ ]`) |

Action items are parsed out of the note and created as individual Todoist tasks. Drawings/sketches are noted as present but not interpreted — nothing is invented that isn't actually on the page.

## Architecture

```
Telegram Trigger
├─ Build Obsidian Markdown → Upload Note to Dropbox → Send a text message         (text)
├─ Prepare Voice → Get a file → Transcribe a recording → Code in JavaScript
│      → Upload Note to Dropbox → Send a text message                             (voice)
├─ Prepare Remarkable PDF Preview → Get Remarkable PDF Preview
│      → Get Remarkable PDF Original File
│           ├─ Upload Remarkable PDF Original (Dropbox, raw backup)
│           └─ Prepare PDF Render Command → Analyze Remarkable Full PDF
│                → Build Remarkable OCR Markdown
│                     ├─ Upload Remarkable OCR Markdown → Bevestig Remarkable notitie opgeslagen
│                     └─ Extract Actiepunten → Maak Todoist Taak (reMarkable)      (PDF)
└─ Prepare Remarkable PNG Upload → Get Remarkable PNG File → Prepare PNG Render Command
       → Analyze Remarkable Full PNG → Build Remarkable PNG OCR Markdown
       → Bevestig Remarkable PNG notitie opgeslagen                               (PNG, see Status)
```

**Key technical choices:**

- **OCR model:** OpenAI `gpt-5.4` via the Responses API (`/v1/responses`), called directly with an HTTP Request node (not the built-in n8n OpenAI node), `temperature: 0.1` for consistent output.
- **Storage:** Dropbox app folder `/Obsidian/Inbox/Remarkable/...`, synced to the Obsidian vault.
- **Dropbox auth:** being migrated from a manual "Access Token" (short-lived, ~4 hours, needs regenerating by hand) to **OAuth2** (refresh token, renews itself automatically and doesn't expire). Uses the same Dropbox app — the App key/secret double as the Client ID/Secret. Free, no production approval required for personal use.
- **Link to source:** every note gets an `## Originele bestanden` section near the top with a clickable Obsidian link (`[[Remarkable/filename]]`) back to the original PDF/PNG — a plain reference, not an inline embed.
- **Todoist integration:** action items are parsed out of the markdown and created one by one via an HTTP node to `https://api.todoist.com/api/v1/tasks` (the old `/rest/v2/tasks` endpoint was deprecated as of July 2026).

## Repository contents

| File | Purpose |
|---|---|
| `architectuur-en-roadmap.md` | Architecture and roadmap (Dutch), current state and open issues |
| `gebruikshandleiding-nl.md` | Usage guide (Dutch) |
| `usage-guide-en.md` | Usage guide (English) |
| `png-ocr-prompt.md` | Standalone OCR prompt for the PNG variant |
| `png-flow-n8n-workflow.json` | Importable n8n workflow to build the PNG branch as a separate project |

## Status

The text, voice, and reMarkable PDF branches are live and working in the main "Obsidian" n8n workflow. The PNG branch inside that same workflow is broken (wrong Telegram credential on the file-download node) and disabled.

PNG notes work via a separate, standalone workflow (`png-flow-n8n-workflow.json`), using the same bot. It's been tested node-by-node and is working. Known open items:

- [ ] Fix the Telegram credential on "Get Remarkable PNG File" in the main workflow, or keep using the standalone PNG workflow instead (recommended — no need to fix the broken branch).
- [ ] Apply the `/api/v1/tasks` Todoist endpoint fix to the live "Maak Todoist Taak (reMarkable)" node (already fixed in the PNG workflow).
- [ ] Migrate the Dropbox credential from Access Token to OAuth2, so tokens stop expiring and needing manual regeneration.
- [ ] Add retry/error handling to critical nodes (Dropbox, OpenAI, Todoist) so one failure doesn't take down the whole execution.
- [ ] The ngrok for N8N tunnel URL is not stable; consider a fixed domain.

See `architectuur-en-roadmap.md` for full detail on each item.

## Setup

1. Import the workflow into n8n.
2. Connect credentials: Telegram Bot API (Obsi8Bot), OpenAI API, Dropbox API (OAuth2 recommended over Access Token — see Status), Todoist API.
3. Point the Dropbox upload paths at your own `Obsidian/Inbox/...` folder structure.
4. Publish the workflow — n8n keeps canvas edits as an unpublished draft until you explicitly click **Publish**.
