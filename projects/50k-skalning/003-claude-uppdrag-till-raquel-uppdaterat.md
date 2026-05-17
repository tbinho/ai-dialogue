---
from: claude (opus-4.7)
to: raquel
date: 2026-05-17
project: 50k-skalning
related: 000, 001, 002
status: awaiting-response
supersedes: 002 (vissa delar)
---

# Uppdaterat uppdrag — vi byter strategi och tar över felsökningen

## Vad som hänt sen 002

Torbjörn delade `RaquelSandblad/para-hund`-repot. Jag (Claude) har klonat ner det, läst:
- `docs/perf/TORBJORN-ONBOARDING.md`
- `docs/perf/AI-HANDOVER-2026-05-16.md`
- `docs/perf/logs/00-sentry-baseline.md`
- `docs/perf/logs/01-fas1-8-brazil.md`
- `.cursor/rules/` (workflow, säkerhet, multi-region, ask-before-build)

Stort respekt — dokumentationen är förstklassig. Jag har därför mer kontext nu och kan kasta ut delar av 002:

- **Inte Sentry-export** — du har redan tagit den (00-sentry-baseline.md), tack
- **Inte instrumentering** — analysen i AI-HANDOVER är så pass djup att vi inte behöver mer mätning för att veta vad som händer
- **Inte de fem Sentry-issue-ID:n (K, M, Q, 1S, 15)** som 002 nämnde — stale, ersatt av 2H/3J/3H

## Strategibyte: Hemnet-modellen

Efter att ha tittat på Hemnets faktiska app + läst er kod säger jag följande utan att linda in det:

**iOS phantom event-loopen är ett symptom, inte rotorsak.** Rotorsaken är att vi har 200+ custom-View-markörer på skärmen samtidigt, vilket gör MapKit-snapshot-loopen så tung att MapKit triggar sina egna "settle"-events som ser ut som user-input → loop. Med max 50-100 PNG-baserade markörer kan loopen inte uppstå.

Detta är inte gissning. Det är vad Hemnet, Booking, Airbnb, Zillow gör. Ingen rendrar 30k custom Views på en karta.

**Designval (från Torbjörn 2026-05-17):** vi använder **grid-sampling istället för supercluster-cirklar med siffror**. Estetiskt val — vill inte ha "5" eller "12" inbränt i ikoner. Algoritmiskt skiljer det sig från klassisk clustering: vi visar **representativa hundar** per grid-cell (1 hund = 1 dot), inte sammanslagna grupper med antal.

**Grundregel som måste hålla:** 1 hund visas ALLTID som en dot, oavsett zoom-nivå. En ensam hund i ett område får aldrig bli ett tomt område på kartan. Grid-sampling garanterar detta automatiskt — ensamma hundar hamnar i sin egen grid-cell och visas alltid.

### Vald arkitektur

| Zoom-nivå | Antal markörer | Markörtyp | Algoritm |
|---|---|---|---|
| Utzoom (hela land) | 200-400 dots | PNG-dot per grid-cell | Server-side `get_map_dogs_grid_sample` med stor cell (~0.5°) |
| Region/stad | 100-300 dots | PNG-dot per grid-cell | Samma RPC, mindre cell (~0.1°) |
| Stadsdel | 50-150 | PNG paw-ikon utan namn | `get_map_dogs_in_bbox` (alla synliga) |
| Gata | 20-50 | Custom View med paw + namn | `get_map_dogs_in_bbox` (alla synliga) |

Hemnet visar samma princip i Torbjörns skärmdumpar — fast med siffror i clustren, vilket vi alltså INTE gör.

### Konkret nästa steg (i ordning)

1. **PNG-assets för dot/paw** (3-4 PNG:er i några färger för kön/typ/missing/kennel). Debug-design räcker — vi byter till snygga ikoner sen, det är 5 min jobb.
2. **Aktivera `get_map_dogs_grid_sample` i klienten vid utzoom** — RPC:n finns på Brazil-DB (migration 085), anropas inte. Cell-storleken beräknas dynamiskt från `latitudeDelta`.
3. **Konsolidera 4 inline marker-typer till 1 per hund**
4. **Stäng av Fabric explicit** (`newArchEnabled: false` i `app.config.ts`)
5. Test mot Brazil-DB → phantom event-loopen ska försvinna automatiskt
6. **Bara om steg 1-5 inte räcker:** migrera till `@rnmapbox/maps`

**Inga supercluster-bibliotek behövs** — grid-sample-RPC:n sköter aggregeringen server-side med stark garanti att ensamma hundar visas. Färre rörliga delar, mindre kod som kan gå sönder.

Allt på `exp/02-hemnet-markers` (ny branch från `perf/50k-scaling`), enligt branch-disciplinen.

## Vad jag behöver från dig (svara i `004-raquel-svar.md`)

| # | Fråga | Varför |
|---|---|---|
| 1 | `.env.local` (inkl. `SUPABASE_SERVICE_ROLE_KEY_BRAZIL`) — säkrast via Drive/Signal/krypterad kanal, inte här i repot | Torbjörn behöver kunna `npx expo start --clear` |
| 2 | Är FETCH_LOCK_MS = 3000 (den senaste fixen i AI-HANDOVER §4) testad? Resultat? | Avgör om vi behöver göra om grundtestet eller hoppa direkt på Hemnet-steg 1 |
| 3 | AI-HANDOVER §7 säger "alla ändringar ligger som unstaged i working tree, inga commits idag". Vi kunde checka ut `exp/01-fas1-8-brazil` från origin — så den ÄR pushad. Reflekterar branchen dagens arbete eller saknas working-tree-ändringar? | Vi behöver veta att vi jobbar på rätt baseline |
| 4 | Vilken Android-enhet ska vi sikta på som referens-låspec? Samsung A14? A05? Sentry 3J nämner S931B (Galaxy S24) — är det er testenhet? | Bestämmer var vi sätter ceiling för testet |
| 5 | OK om jag (via Torbjörn) skapar `exp/02-hemnet-markers` från `perf/50k-scaling` när vi börjar koden? Eller vill du att vi fortsätter på `exp/01-fas1-8-brazil`? | Branch-disciplin |

## Bonusfråga (svara om du har tid)

6. **Varför har 4 inline marker-typer (rad 2324, 2364, 2405, 2423 i `app/(tabs)/index.tsx`)** byggts ut från 2-april-versionens 2 typer? Finns det ett funktionellt skäl vi missar (t.ex. olika ikoner för kennel vs missing-dog vs male/female) som vi måste behålla?

7. **Har du försökt PNG-baserade markörer (`<Marker image={...}>`) någon gång?** Om ja — vad gick fel? (Inte blocker — vi kör ändå. Men sparar oss en återupptäckts-cykel om någon fallgrop redan är känd.)

## Vad ni absolut INTE behöver göra
- Skicka mer Sentry-data (00-sentry-baseline är tillräcklig)
- Bygga något åt oss — Torbjörn kör allt själv från sin dator (med Claude Code)
- Skriva mer dokumentation åt oss — vi läser repot direkt

## Bonus-möjlighet (FYI, inget åtagande)

När huvudproblemet är löst kan **heatmap-vy** läggas till som alternativ visning — `react-native-maps` har inbyggd `<Heatmap>`-komponent som GPU-renderar 30k+ punkter utan problem. Användaren skulle kunna toggla mellan "Heatmap / Dots / Lista". 1-2 dagars jobb senare. Inget vi fokuserar på nu, men värt att veta att det finns som möjlighet.

## Kommunikationsformat
- Svara i `004-raquel-svar.md` i denna mapp
- Repot är nu **publikt** så du kan läsa direkt på github.com utan att klona
- Om du vill svara via repot: klicka "Add file" → "Create new file" i webgränssnittet och döp den till `004-raquel-svar.md`
- Eller bara svara i Slack/mail — Torbjörn flyttar in det manuellt om det behövs
- Torbjörn pingar mig när du svarat

Och tack för den otroligt välstrukturerade dokumentationen i para-hund-repot. Det sparar oss veckor.

— Claude (Opus 4.7), via Torbjörns Claude Code
