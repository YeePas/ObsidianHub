# Architectuur en roadmap — Obsidian capture-workflow

Stand van zaken per 2 juli 2026. Dit document beschrijft concreet wat er nu draait, hoe het in elkaar zit, en wat de volgende stappen zijn — zonder omhaal.

## Overzicht

Eén n8n-workflow ("Obsidian"), met de Telegram-bot **Obsi8Bot**, live en werkend: tekst/voice/reMarkable-PDF → Obsidian-notitie + Todoist-taken. Het is een pure **capture**-pipeline — vastleggen, geen interpretatie. Alles wat binnenkomt via Obsi8Bot wordt een echte notitie in de inbox.

## Workflow — Obsidian (live)

URL: `https://nonhyperbolically-constrictive-mabelle.ngrok-free.dev/workflow/NeV4hjgRa2fmuBODN0UQA`

### Branches vanuit de Telegram Trigger

```
Telegram Trigger
├─ Build Obsidian Markdown → Upload Note to Dropbox → Send a text message         (tekst)
├─ Prepare Voice → Get a file → Transcribe a recording → Code in JavaScript
│      → Upload Note to Dropbox → Send a text message                             (voice)
├─ Prepare Remarkable PDF Preview → Get Remarkable PDF Preview
│      → Get Remarkable PDF Original File
│           ├─ Upload Remarkable PDF Original (Dropbox, ruwe PDF-backup)
│           └─ Prepare PDF Render Command → Analyze Remarkable Full PDF
│                → Build Remarkable OCR Markdown
│                     ├─ Upload Remarkable OCR Markdown → Bevestig Remarkable notitie opgeslagen
│                     └─ Extract Actiepunten → Maak Todoist Taak (reMarkable)      (PDF)
└─ Prepare Remarkable PNG Upload → Get Remarkable PNG File → Prepare PNG Render Command
       → Analyze Remarkable Full PNG → Build Remarkable PNG OCR Markdown
       → Bevestig Remarkable PNG notitie opgeslagen                               (PNG — kapot in dit workflow, werkt via losstaande JSON, zie hieronder)
```

### Belangrijkste technische keuzes

- **OCR-model:** OpenAI `gpt-5.4` via de Responses API (`/v1/responses`), rechtstreeks aangeroepen met een HTTP Request-node (niet de ingebouwde n8n OpenAI-node), met `temperature: 0.1` voor consistente output.
- **Notatiesysteem** (vastgelegd in de prompt van "Prepare PDF Render Command"): onderstreping = titel (eerste = H1, volgende = H2), sterretje = werknotitie-bullet, leeg vierkantje = actiepunt-checkbox, horizontale streep = nieuwe sectie.
- **Opslag:** Dropbox-app-map `/Obsidian/Inbox/Remarkable/...`, gesynchroniseerd naar de Obsidian-vault.
- **Link naar origineel:** elke notitie krijgt bovenaan (na de frontmatter, vóór de AI-inhoud) een sectie `## Originele bestanden` met een klikbaar Obsidian-linkje (`[[Remarkable/bestandsnaam]]`) naar het brondocument. Puur ter verwijzing — geen automatische inline-weergave.
- **Todoist-koppeling:** actiepunten uit de markdown worden apart geparsed (Extract Actiepunten) en één-voor-één als losse taak aangemaakt via een HTTP-node naar `https://api.todoist.com/api/v1/tasks` (de oude `/rest/v2/tasks`-endpoint is per juli 2026 gedeprecate; body en Bearer-auth blijven ongewijzigd).

## Bekende problemen die deze sessie zijn opgelost

1. **Verlopen Dropbox-token** crashte de hele workflow-executie stil, ook voor de tekst/voice-branches — Dropbox-nodes hadden geen foutafhandeling, dus één verlopen token trok alles om.
2. **Draft/publish-verwarring**: n8n bewaart canvas-wijzigingen los van de live versie. Wijzigingen gaan pas echt live na een expliciete klik op **Publish**. Meerdere fixes uit eerdere sessies waren daardoor nooit live gegaan.
3. **Gedeelde node tussen PDF- en PNG-branch** (`Upload Remarkable OCR Markdown`) zorgde voor een dataherkomst-conflict waardoor de PDF-bevestiging faalde met "node hasn't been executed" zodra de PNG-tak ooit had meegedraaid. Losgekoppeld: elke branch heeft nu zijn eigen pad naar bevestiging.
4. **Formattering wek van de gewenste stijl**: titel-hiërarchie, bullet-teken voor werknotities, lege regels en het ontbreken van checkbox-syntax bij actiepunten. Prompt herschreven en verscherpt.
5. **Binary-data ging verloren in de PNG-flow**: "Prepare PNG Render Command" gaf `binary` niet door in zijn return, waardoor "Upload Remarkable PNG Original" faalde met "no binary file 'data' found". Gefixt door `binary: item.binary` toe te voegen aan de return.
6. **Todoist REST v2-endpoint gedeprecate**: `https://api.todoist.com/rest/v2/tasks` gaf een "endpoint is deprecated"-fout. Bijgewerkt naar `https://api.todoist.com/api/v1/tasks` in de PNG-flow JSON — body en Bearer-auth blijven identiek. **Nog niet toegepast op de live Obsidian-workflow** (node "Maak Todoist Taak (reMarkable)"), zie roadmap.
7. **Geen link naar het originele bestand bovenaan de PNG-notitie**, terwijl de PDF-variant dat al had. Toegevoegd: sectie `## Originele bestanden` met `[[Remarkable/bestandsnaam]]`-link, consistent met de PDF-flow.

## Zwart-wit: openstaande verbeterpunten

- [x] **Losstaande PNG-flow (`png-flow-n8n-workflow.json`) werkt.** Node-voor-node getest; twee bugs onderweg gevonden en gefixt (binary-data-verlies in "Prepare PNG Render Command", gedeprecate Todoist-endpoint). Dit is nu de aan te raden route voor PNG-notities — zie de gebruikshandleiding.
- [ ] **PNG-branch in de oorspronkelijke Obsidian-workflow is nog steeds kapot en staat uit.** Bekende bug: "Get Remarkable PNG File" gebruikt de verkeerde Telegram-credential ("ToDo" i.p.v. "Obsi8Bot"), waardoor bestand-download faalt (file_id's zijn bot-gebonden). Niet nodig om te fixen zolang de losstaande PNG-flow het werk doet — laat opzettelijk uit staan, of ruim 'm op om verwarring te voorkomen.
- [ ] **Todoist-endpoint in de live Obsidian-workflow nog bijwerken.** De node "Maak Todoist Taak (reMarkable)" gebruikt vermoedelijk nog `/rest/v2/tasks`; verander dat naar `https://api.todoist.com/api/v1/tasks` zodra je die node weer aanraakt (zelfde fix als in de PNG-JSON).
- [ ] **Geen foutafhandeling op kritieke nodes.** Eén verlopen credential of tijdelijke API-fout op één node (bijv. Dropbox-upload) legt nu de hele executie plat, ook onderdelen die er niets mee te maken hebben. Zet `Retry On Fail` en/of `Continue On Fail` aan op de Dropbox-, OpenAI- en Todoist-nodes.
- [ ] **Illustraties-sectie plaatsing is inconsistent** tussen executies (soms eigen `## Illustraties`-kop, soms samengevoegd met de voorgaande sectie). Dit is AI-variatie bij temperature 0.1, niet een structurele bug. Verder verstrakken van de prompt kan, maar vergroot het risico op te rigide/onnatuurlijke output — weeg dit af voordat je hierin investeert.
- [ ] **Geen automatische executie-opschoning.** Executiegeschiedenis raakte deze sessie leeg na een herstart van n8n/ngrok — waarschijnlijk geen persistente opslag voor executies. Overweeg een persistente database-configuratie als je executiehistorie wil kunnen terugzoeken.
- [ ] **ngrok-tunnel is niet stabiel/permanent.** De workflow-URL veranderde/verviel meerdere keren deze sessie door sessie-timeouts op de tunnel. Overweeg een vaste (betaalde) ngrok-domeinnaam of een andere manier om n8n publiek bereikbaar te maken, zodat de URL niet steeds opnieuw gedeeld hoeft te worden.

## Bijlagen bij dit document

- `png-ocr-prompt.md` — kant-en-klare AI-prompt voor de PNG-variant, gebaseerd op de nu werkende PDF-prompt.
- `png-flow-n8n-workflow.json` — importeerbare n8n-workflow die de PNG-branch in één keer opbouwt in een nieuw project.
