# Gebruikshandleiding — Obsidian-capture via Telegram

Deze handleiding legt uit hoe je de **Obsi8Bot** op Telegram gebruikt om tekst, spraak en reMarkable-notities automatisch in Obsidian en Todoist te krijgen.

## Het idee in één zin

Je stuurt iets naar de bot in Telegram — een berichtje, een spraakmemo, of een geëxporteerde PDF van je reMarkable — en de workflow zet dat om in een nette Markdown-notitie in je Obsidian-inbox, met (waar relevant) automatisch aangemaakte Todoist-taken. Je krijgt in Telegram een bevestiging zodra het klaar is.

## 1. Tekstbericht

Stuur gewoon een tekstbericht naar de bot.

- De tekst wordt direct omgezet naar een Markdown-bestand.
- Het bestand wordt opgeslagen in je Obsidian-inbox (via Dropbox-sync).
- Je krijgt een bevestiging terug in Telegram.

Gebruik dit voor snelle losse notities, ideeën of dingen die je onderweg wil vastleggen.

## 2. Spraakbericht (voice memo)

Stuur een spraakbericht naar de bot.

- De audio wordt opgehaald en getranscribeerd.
- De getranscribeerde tekst wordt (net als bij een tekstbericht) omgezet naar een Markdown-notitie in je Obsidian-inbox.
- Je krijgt een bevestiging terug in Telegram.

Handig als je liever inspreekt dan typt, bijvoorbeeld tijdens het lopen of net na een gesprek.

## 3. reMarkable-notities (PDF)

Dit is de meest uitgebreide flow, bedoeld voor handgeschreven aantekeningen vanaf je reMarkable-tablet.

**Stappen:**

1. Exporteer je notitie vanaf de reMarkable (of via `app.remarkable.com`) als **PDF**.
2. Stuur die PDF als document naar de bot in Telegram.
3. Optioneel: geef in het bijschrift (caption) van het bericht een titel mee, in de vorm `Titel: <jouw titel>`. Doe je dit niet, dan leidt de workflow een titel af uit de bestandsnaam of de inhoud.

**Wat er automatisch gebeurt:**

- De PDF wordt naar OpenAI gestuurd voor volledige OCR/analyse (handschrift, getypte tekst, pijlen, kaders).
- De inhoud wordt omgezet naar een gestructureerde Obsidian-notitie:
  - Een korte samenvatting bovenaan.
  - Koppen per sectie, met werknotities gegroepeerd op basis van hun positie op de pagina.
  - Een aparte sectie **Actiepunten** met aanvinkbare taken (`- [ ]`).
  - Een aparte sectie **Illustraties** met een korte vermelding per tekening (geen inhoudelijke interpretatie van de tekening zelf).
- De notitie én het originele PDF-bestand worden opgeslagen in je Obsidian-inbox (Dropbox).
- Bovenaan de notitie staat een sectie **Originele bestanden** met een klikbaar linkje (`[[Remarkable/...]]`) naar het originele bestand — puur ter verwijzing, Obsidian toont het bestand niet automatisch inline.
- Elk actiepunt wordt automatisch als losse taak aangemaakt in **Todoist**.
- Je krijgt een bevestiging terug in Telegram met de titel van de notitie.

### Het notatiesysteem van je handschrift

De AI interpreteert een aantal vaste conventies in je handgeschreven notities:

| Op de pagina | Wordt in Markdown |
|---|---|
| Onderstreepte tekst (eerste keer op de pagina) | Hoofdtitel (`#`) |
| Onderstreepte tekst (elke volgende keer) | Subtitel (`##`) |
| Volledige horizontale streep dwars over de regel | Start van een nieuwe sectie (eigen koptekst wordt door de AI bedacht) |
| Sterretje (`*`) bij een regel | Werknotitie → bullet met een streepje (`- `) |
| Leeg vierkantje bij een regel | Actiepunt → aanvinkbare taak (`- [ ]`) |

**Tip:** teken je deze symbolen consequent op je reMarkable, hoe consistenter de output.

### Wat níet gebeurt

- Tekeningen/schetsen worden niet inhoudelijk beschreven of geïnterpreteerd — er komt alleen een regel dat er een tekening aanwezig is, met verwijzing naar het origineel.
- Er wordt niets verzonnen: onleesbare of onzekere tekst wordt zo goed mogelijk benaderd, maar er worden geen taken of afspraken bedacht die niet echt op de pagina staan.

## 4. reMarkable-notities (PNG)

Naast de PDF-route is er ook een route voor losse PNG-afbeeldingen — handig als de kleuren-PDF-export niet goed OCR't, of als je liever één losse pagina stuurt. Deze route draait als aparte n8n-workflow (`png-flow-n8n-workflow.json`), maar gebruikt dezelfde **Obsi8Bot**.

**Stappen:**

1. Download de pagina als **PNG** (bijvoorbeeld via `app.remarkable.com/folder/home`).
2. Stuur die PNG als document of foto naar de bot in Telegram.
3. Optioneel: geef in het bijschrift (caption) een titel mee, in de vorm `Titel: <jouw titel>`. Doe je dit niet, dan leidt de workflow een titel af uit de bestandsnaam of de inhoud.

**Wat er automatisch gebeurt:**

- De PNG wordt naar OpenAI gestuurd voor volledige OCR/analyse (handschrift, getypte tekst, pijlen, kaders), met hetzelfde notatiesysteem als de PDF-flow (zie hierboven).
- De inhoud wordt omgezet naar een gestructureerde Obsidian-notitie: korte samenvatting bovenaan, koppen per sectie, een aparte **Actiepunten**-sectie met aanvinkbare taken (`- [ ]`).
- Bovenaan de notitie staat een sectie **Originele bestanden** met een klikbaar linkje (`[[Remarkable/...]]`) naar de originele PNG — puur ter verwijzing, niet automatisch zichtbaar.
- De notitie én de originele PNG worden opgeslagen in je Obsidian-inbox (Dropbox).
- Elk actiepunt wordt automatisch als losse taak aangemaakt in **Todoist**.
- Je krijgt een bevestiging terug in Telegram met de titel van de notitie.

Zie het architectuur-document voor de technische details en het overzicht van wat er deze sessie nog aan bugs is gevonden en gefixt.

## Veelgestelde vragen

**Waar komen mijn notities terecht?**
In de map `Obsidian/Inbox/...` in je gekoppelde Dropbox, die via Dropbox-sync in je Obsidian-vault verschijnt.

**Wat als de bot niet reageert?**
Controleer of de n8n-workflow **gepubliceerd** is (niet alleen opgeslagen als concept) — n8n gebruikt een aparte "Publish"-stap voordat wijzigingen echt live gaan. Controleer ook de Dropbox-credential: bij de oude Access Token-methode verloopt die na een paar uur, waardoor niets wordt opgeslagen. Dit wordt opgelost door over te stappen op OAuth2 (zie architectuur-document).

**Kan ik meerdere PDF's of PNG's tegelijk sturen?**
Ja, elk bericht met een bijlage triggert een eigen, onafhankelijke verwerking.
