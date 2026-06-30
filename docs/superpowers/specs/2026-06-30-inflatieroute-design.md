# Spec — De Inflatieroute (Scouting Hubertus-Brandaan)

**Datum:** 2026-06-30
**Status:** Goedgekeurd (ontwerp), in uitvoering
**Bron-concept:** `Hubertus scouting app concept.zip` — design handoff "De Inflatieroute" (high-fidelity, stijl "Dossier" gekozen).

## Doel

Mobiele web-app voor een vaartocht-/postenspel van Scouting Hubertus-Brandaan
(Explorers/Toppers, 17–18 jaar). Crisis-thema (inflatie). De groep vaart een route
over de Vliet en de Leidse grachten naar het Konijneneiland op de Kagerplassen, met
7 posten. Per post: een cryptische vind-hint + live kaart. De volgende post ontgrendelt
pas nadat de Toppers ter plaatse een grappig **crisis-parool** intypen én de posthouder
een **4-cijferige pincode** invoert. Behaalde "buit" (benzine-reageerbuizen + BBQ-eten)
wordt bijgehouden.

## Beslissingen (door gebruiker bevestigd)

1. **Locatie/hosting:** submap `inflatieroute/` in de bestaande repo
   `NiRobit-NL/scouting-neus-speuren`. Deployt automatisch via de bestaande GitHub Pages
   naar `https://nirobit-nl.github.io/scouting-neus-speuren/inflatieroute/`. De snotneuzen-app
   op de root blijft volledig ongemoeid.
2. **Spelleider-codes:** verborgen spelleider-modus. Activatie via `?spelleider` in de URL
   **of** 5× tikken op het logo. Toont per post de pincode (+ parool) en een reset-knop.
   Geen losse demo-knop.
3. **Stijl:** alleen "Dossier" (manilla papier / inkt-stempels). De donkere "Controlekamer"-
   variant wordt **niet** meegebouwd (YAGNI; eventueel later als thema-toggle).

## Architectuur

- Eén self-contained bestand `inflatieroute/index.html` — vanilla HTML/CSS/JS, geen build-step.
  Spiegelt exact de bestaande snotneuzen-app (`index.html` op root).
- **Geen backend.** Alle data (7 posten + finish) in een JS-array client-side.
- **Leaflet 1.9.4 + OpenStreetMap** via CDN voor de kaart.
- **Google Fonts:** Oswald (koppen) + Special Elite (body/mono).
- **Voortgang in `localStorage`** (`inflatieroute_unlocked`): array van 7 booleans, zodat
  refresh/lege batterij de behaalde posten bewaart.
- Logo: `inflatieroute/assets/hubertus-brandaan.svg` (origineel blauw `#0C68B0`).

### Rendering-aanpak (vanilla, port van het React-concept)

- De Leaflet-kaart wordt **één keer** geïnitialiseerd en blijft gemount; markers worden
  alleen bijgewerkt bij wijziging van `unlocked` (kaart nooit herbouwen → geen flikker).
- Route-, voorraad- en detail-views worden herbouwd via `render()` bij structurele
  state-wijzigingen (tab, open/dicht detail, unlocked, pin, err, gps, reveal).
- Het **parool-invoerveld** gebruikt gerichte updates (geen volledige re-render per
  toetsaanslag) zodat focus/caret behouden blijft; alleen de afhankelijke UI (rand,
  ✓-melding, keypad-enabled) wordt bijgewerkt.

## Schermen

Eén telefoonscherm: slim status-strip + header (titel + voortgangsbalk X/7) + content
met 3 tabs + tabbalk onderaan. Detail-overlay schuift per post over het scherm (slide-up).

1. **Route** (standaard) — koers-banner met cryptische hint naar de actieve post +
   7 post-kaarten (statussen: behaald / actief / vergrendeld) + finale-kaart bij 7/7.
2. **Kaart** — Leaflet, gecentreerd op Leiden/Leidschendam. Alleen **behaalde** posten als
   genummerde blauwe markers (`#14609f`) + gestippelde polyline in volgorde. Bij 7/7 de
   finish-ster (`#a82f24`) op het Konijneneiland. Niet-behaalde posten verborgen.
   `fitBounds` op zichtbare punten; default view `[52.16, 4.49]` zoom 12; `scrollWheelZoom` uit.
3. **Voorraad** — benzine-meter (X/11 reageerbuis-iconen) + BBQ & Buit-lijst van behaalde posten.

### Detail-overlay

- **Actieve post:** vind-hint (envelop-kader) → "noodcoördinaat tonen"-knop (GPS) →
  opdracht → **crisis-parool** (doelzin groot getoond + invoer, genormaliseerd vergeleken)
  → **4-cijfer pincode-keypad** (gedimd/uitgeschakeld tot parool klopt; foute code = rood +
  schud + reset). In spelleider-modus: code + parool zichtbaar.
- **Behaalde post:** "GOEDGEKEURD"-stempel → opdracht → de buit (benzine + eten) →
  koers-hint naar de volgende post (of finish) → "Terug naar route".

## Spel-flow / state

- `unlocked: boolean[7]` (localStorage), `tab`, `open`, `pin`, `err`, `gps`, `parool`,
  `spelleider`.
- Afgeleid: `activeIndex` = eerste niet-behaalde post; `benzineTotal` = som benzine
  behaalde posten (max 11); `paroolOK` = `norm(parool) === norm(post.parool)` met
  `norm(s) = s.toUpperCase().replace(/[^A-Z0-9]/g,'')`.
- Validatie: keypad reageert niet zolang parool niet klopt; 4 juiste cijfers === `post.code`
  → post behaald; foute code → schud (~0.4s) + reset (~0.65s); vergrendelde posten niet aantikbaar.

## Design tokens (Dossier)

- bg `#c9b994` · bg2 `#e7dcc1` · bg3 `#dccfae` · line `#b6a47e`
- text `#2b2417` · dim `#7a6d50`
- accent (stempel-rood) `#a82f24` · accent2 (scout-blauw) `#14609f` · ok (groen) `#2f6b3a`
- fonts: koppen Oswald (600/700, uppercase); body/mono Special Elite
- radius `2px`; panelen met slagschaduw `2px 2px 0 rgba(0,0,0,.10)`; papiertextuur via radial-gradients
- animaties: `shakeX`, `slideUp`, `stampIn`, `pulseDot`/`blip`; hit-targets ≥44px (keypad 48px)

## Data (7 posten + finish)

Letterlijk uit het concept (`InflatiePhone.dc.html` `POSTS`/`FINISH`): num, name, area, code,
gps, parool, title, desc, benzine, food, hint. Totaal benzine bij 7/7 = 11.

## Niet in scope (YAGNI)

- Donkere "Controlekamer"-variant.
- Backend / accounts / multi-device sync.
- PWA-installatie / service worker (kan later).

## Deploy

Werk op branch `inflatieroute-app`; na verificatie in de browser mergen naar `main` en pushen.
GitHub Pages serveert de submap automatisch.
