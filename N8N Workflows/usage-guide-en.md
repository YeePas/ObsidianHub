# Usage Guide — Obsidian Capture via Telegram

This guide explains how to use the **Obsi8Bot** on Telegram to automatically turn text, voice, and reMarkable notes into Obsidian notes and Todoist tasks.

## The idea in one sentence

You send something to the bot on Telegram — a short message, a voice memo, or a PDF exported from your reMarkable — and the workflow turns it into a clean Markdown note in your Obsidian inbox, with (where relevant) Todoist tasks created automatically. You get a confirmation back in Telegram once it's done.

## 1. Text message

Just send a plain text message to the bot.

- The text is converted directly into a Markdown file.
- The file is saved to your Obsidian inbox (via Dropbox sync).
- You receive a confirmation in Telegram.

Use this for quick, standalone notes, ideas, or anything you want to capture on the go.

## 2. Voice message

Send a voice message to the bot.

- The audio is downloaded and transcribed.
- The transcribed text is converted into a Markdown note in your Obsidian inbox, same as a text message.
- You receive a confirmation in Telegram.

Handy when you'd rather speak than type — while walking, or right after a conversation.

## 3. reMarkable notes (PDF)

This is the most elaborate flow, built for handwritten notes from your reMarkable tablet.

**Steps:**

1. Export your note from the reMarkable (or via `app.remarkable.com`) as a **PDF**.
2. Send that PDF as a document to the bot on Telegram.
3. Optional: add a title in the message caption, formatted as `Titel: <your title>`. If you skip this, the workflow derives a title from the filename or content.

**What happens automatically:**

- The PDF is sent to OpenAI for full OCR/analysis (handwriting, typed text, arrows, boxes).
- The content is turned into a structured Obsidian note:
  - A short summary at the top.
  - Section headers, with work notes grouped by their position on the page.
  - A dedicated **Actiepunten** (action items) section with checkable tasks (`- [ ]`).
  - A dedicated **Illustraties** (illustrations) section with a short mention per drawing (no interpretation of the drawing's content).
- Both the note and the original PDF are saved to your Obsidian inbox (Dropbox).
- At the top of the note there's an **Originele bestanden** section with a clickable link (`[[Remarkable/...]]`) to the original file — a plain reference link, Obsidian does not render it inline automatically.
- Every action item is automatically created as a separate task in **Todoist**.
- You get a confirmation back in Telegram with the note's title.

### The handwriting notation system

The AI interprets a fixed set of conventions in your handwritten notes:

| On the page | Becomes in Markdown |
|---|---|
| Underlined text (first occurrence on the page) | Main title (`#`) |
| Underlined text (every following occurrence) | Subtitle (`##`) |
| A full horizontal line across the row | Start of a new section (the AI comes up with its own heading) |
| An asterisk (`*`) next to a line | Work note → bullet with a dash (`- `) |
| An empty square next to a line | Action item → checkable task (`- [ ]`) |

**Tip:** the more consistently you draw these symbols on your reMarkable, the more consistent the output.

### What doesn't happen

- Drawings/sketches are not described or interpreted in detail — only a line noting a drawing is present, referencing the original file.
- Nothing is invented: unclear or uncertain text is transcribed as best as possible, but no tasks or commitments are made up that aren't actually on the page.

## 4. reMarkable notes (PNG)

Besides the PDF route, there's also a route for standalone PNG images — handy when the color-PDF export doesn't OCR well, or when you'd rather send a single page. This route runs as a separate n8n workflow (`png-flow-n8n-workflow.json`), but uses the same **Obsi8Bot**.

**Steps:**

1. Download the page as a **PNG** (e.g. via `app.remarkable.com/folder/home`).
2. Send that PNG as a document or photo to the bot on Telegram.
3. Optional: add a title in the message caption, formatted as `Titel: <your title>`. If you skip this, the workflow derives a title from the filename or content.

**What happens automatically:**

- The PNG is sent to OpenAI for full OCR/analysis (handwriting, typed text, arrows, boxes), using the same notation system as the PDF flow (see above).
- The content is turned into a structured Obsidian note: a short summary at the top, section headers, a dedicated **Actiepunten** (action items) section with checkable tasks (`- [ ]`).
- At the top of the note there's an **Originele bestanden** section with a clickable link (`[[Remarkable/...]]`) to the original PNG — a plain reference, not rendered inline automatically.
- Both the note and the original PNG are saved to your Obsidian inbox (Dropbox).
- Every action item is automatically created as a separate task in **Todoist**.
- You get a confirmation back in Telegram with the note's title.

See the architecture document for technical details and the list of bugs found and fixed this session.

## FAQ

**Where do my notes end up?**
In the `Obsidian/Inbox/...` folder in your linked Dropbox, which appears in your Obsidian vault via Dropbox sync.

**What if the bot doesn't respond?**
Check whether the n8n workflow is actually **published** (not just saved as a draft) — n8n uses a separate "Publish" step before changes go live.

**Can I send multiple PDFs at once?**
Yes, each message with an attachment triggers its own, independent processing run.
