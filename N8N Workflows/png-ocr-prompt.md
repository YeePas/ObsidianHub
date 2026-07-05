# OCR-prompt voor de PNG-variant

Deze prompt is direct afgeleid van de nu werkende en geverifieerde PDF-prompt uit de "Obsidian"-workflow (sessie van 2 juli 2026), aangepast voor een losse PNG-afbeelding in plaats van een meerpagina-PDF. Gebruik hem in een HTTP Request-node naar `https://api.openai.com/v1/responses`, als `input_text` naast een `input_image`-blok met de PNG als base64 data-URI.

## Waarom een aparte prompt voor PNG

Een PNG is meestal één losse afbeelding (foto of screenshot van een pagina), terwijl de PDF-prompt uitgaat van een heel document. Het notatiesysteem (onderstreping, sterretje, vierkantje, streep) blijft identiek — alleen de openingsinstructie en de aanname "één pagina in plaats van een heel document" zijn aangepast.

## De prompt (kant-en-klaar, plakken in een Code-node als JS-array of direct als tekst)

```
Je analyseert één PNG-afbeelding van een reMarkable-pagina met handgeschreven notities, schetsen en eventueel ingeplakte afbeeldingen.
Lees handschrift, getypte tekst, pijlen en kaders zo goed mogelijk. Sla losse tekeningen/schetsen/illustraties over: analyseer of beschrijf de inhoud daarvan niet, noteer alleen dat er een tekening aanwezig is.
Gebruik alleen wat zichtbaar is. Verzin geen ontbrekende tekst, meetings of taken.
Doe altijd een beste-poging tot transcriptie, ook bij gekleurde schetsen of rommelig handschrift. Gebruik onzeker/onleesbaar alleen als echt niets te onderscheiden is, en beschrijf het element dan toch zo goed mogelijk in plaats van het over te slaan.

Notatie systeem:
- Onderstreepte tekst = titel. De EERSTE onderstreepte tekst op de pagina is de hoofdtitel en wordt Markdown H1 (één #). Elke volgende onderstreepte tekst is een subtitel en wordt Markdown H2 (##).
- Een sterretje bij een regel = werknotitie. Schrijf dit in de markdown als bullet met een liggend streepje "- ", gebruik nooit een letterlijk sterretje.
- Een leeg vierkantje bij een regel = actiepunt.
- Een volledige horizontale streep over de regel = start van een nieuwe sectie zonder eigen titel; bedenk zelf een korte passende sectienaam op basis van de notities die volgen.

Maak nette Obsidian Markdown in het Nederlands.
Structuur:
1. Begin met een sectie "## Korte samenvatting" gevolgd door een lopende tekst van 2-4 zinnen (geen bullets).
2. Headers per sectie, met de werknotities gegroepeerd op basis van nabijheid op de pagina.
3. Actiepunten: verzamel alle regels met een leeg vierkantje in een aparte sectie. Gebruik voor elke taak exact het formaat "- [ ] Sectienaam: actiegerichte zin." Laat de vierkante haakjes nooit weg.
4. Illustraties: geen inhoudelijke analyse. Zet alleen een korte regel per sectie met een tekening, bijvoorbeeld "Tekening aanwezig, zie origineel PNG." Ga niet in op vorm, kleur of betekenis.

Voeg geen HTML, JSON of code fences toe.
Zet geen lege regel tussen een header (# of ##) en de bullet- of taken-lijst die daarna volgt.
Begin niet met bestandsnamen zoals File 16 of page headings. Titelcontext: <TITEL_VARIABELE>
```

Vervang `<TITEL_VARIABELE>` in de laatste regel door de titel-variabele uit je Code-node (zelfde patroon als in de PDF-flow: caption-titel, anders bestandsnaam, anders fallback "reMarkable notitie").

## Aanbevolen API-instellingen

- **model:** `gpt-5.4`
- **temperature:** `0.1` (consistente, voorspelbare output — geverifieerd in de PDF-flow)
- **image detail:** `high` (belangrijk voor leesbaarheid van klein handschrift)
- **content-blok:**
  ```json
  {
    "type": "input_image",
    "image_url": "data:image/png;base64,<BASE64_PNG_DATA>",
    "detail": "high"
  }
  ```
- Overweeg de afbeelding vóór verzending naar grijswaarden te converteren als de originele kleuren-PDF-export matig OCR't (reden waarom de PNG-route ooit is bedacht) — dit kan met een losse image-processing node vóór de analyse-stap.

## Link naar het origineel

De regel `## Originele bestanden` met een `[[Remarkable/bestandsnaam]]`-link naar de PNG wordt **niet** door deze AI-prompt gegenereerd, maar door de omringende Code-node ("Build Remarkable PNG OCR Markdown") toegevoegd vóór de AI-tekst wordt aangeplakt — consistent met hoe de PDF-flow dat doet. Zie `png-flow-n8n-workflow.json` voor de exacte code.

## Verschil met de PDF-prompt

| Aspect | PDF-prompt | PNG-prompt |
|---|---|---|
| Content-type | `input_file` (PDF, base64) | `input_image` (PNG, base64) |
| Aanname | Heel document, meerdere pagina's mogelijk | Eén losse pagina/afbeelding |
| Notatiesysteem | Identiek | Identiek |
| Structuur-instructies | Identiek | Identiek |
