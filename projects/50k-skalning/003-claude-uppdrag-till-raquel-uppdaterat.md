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

### Vald arkitektur

| Zoom-nivå | Antal markörer | Markörtyp |
|---|---|---|
| Utzoom (hela land) | 400-600 | Enkla PNG-dots |
| Region/stad | 100-300 | PNG cluster med antal-siffra inbränd |
| Stadsdel | 50-150 | PNG paw-ikon utan namn |
| Gata | 20-50 | Custom View med paw + namn OK |

Hemnet visar samma fyra-nivå-LOD i Torbjörns skärmdumpar. Klassisk supercluster-baserad approach.

### Konkret nästa steg (i ordning)

1. **PNG-assets för dot/cluster/paw** (3-4 PNG:er, olika färger för kön/typ)
2. **Aktivera `get_map_dogs_grid_sample` i klienten vid utzoom** — RPC:n finns, anropas inte
3. **Cluster-rendering med supercluster** — `<Marker image={clusterPng}>` med antal-siffran
4. **Konsolidera 4 inline marker-typer till 1 per hund**
5. **Stäng av Fabric explicit** (`newArchEnabled: false` i `app.config.ts`)
6. Test mot Brazil-DB → phantom event-loopen ska försvinna automatiskt
7. **Bara om steg 1-6 inte räcker:** migrera till `@rnmapbox/maps`

Allt på `exp/02-hemnet-markers` (ny branch från `perf/50k-scaling`), enligt branch-disciplinen.

## Vad jag behöver från dig (svara i `004-raquel-svar.md`)

| # | Fråga | Varför |
|---|---|---|
| 1 | `.env.local` (inkl. `SUPABASE_SERVICE_ROLE_KEY_BRAZIL`) — säkrast via Drive/Signal/krypterad kanal, inte här i repot | Torbjörn behöver kunna `npx expo start --clear` |
| 2 | Är FETCH_LOCK_MS = 3000 (den senaste fixen i AI-HANDOVER §4) testad? Resultat? | Avgör om vi behöver göra om grundtestet eller hoppa direkt på Hemnet-steg 1 |
| 3 | AI-HANDOVER §7 säger "alla ändringar ligger som unstaged i working tree, inga commits idag". Vi kunde checka ut `exp/01-fas1-8-brazil` från origin — så den ÄR pushad. Reflekterar branchen dagens arbete eller saknas working-tree-ändringar? | Vi behöver veta att vi jobbar på rätt baseline |
| 4 | Har ni försökt PNG-baserade markörer (`<Marker image={...}>`) någon gång? Om ja — vad gick fel? Specifikt: label-bubblan med hund-namn (kan inte vara statisk PNG) | Sparar oss en återupptäckts-cykel om någon fallgrop redan är känd |
| 5 | Vilken Android-enhet ska vi sikta på som referens-låspec? Samsung A14? A05? Sentry 3J nämner S931B (Galaxy S24) — är det er testenhet? | Bestämmer var vi sätter ceiling för testet |
| 6 | OK om jag (via Torbjörn) skapar `exp/02-hemnet-markers` från `perf/50k-scaling` när vi börjar koden? Eller vill du att vi fortsätter på `exp/01-fas1-8-brazil`? | Branch-disciplin |

## Bonusfråga (svara om du har tid)

7. **Varför har 4 inline marker-typer (rad 2324, 2364, 2405, 2423 i `app/(tabs)/index.tsx`)** byggts ut från 2-april-versionens 2 typer? Finns det ett funktionellt skäl vi missar (t.ex. olika ikoner för kennel vs missing-dog vs male/female) som vi måste behålla?

## Vad ni absolut INTE behöver göra
- Skicka mer Sentry-data (00-sentry-baseline är tillräcklig)
- Bygga något åt oss — Torbjörn kör allt själv från sin dator (med Claude Code)
- Skriva mer dokumentation åt oss — vi läser repot direkt

## Kommunikationsformat
- Svara i `004-raquel-svar.md` i denna mapp
- Pusha till `tbinho/ai-dialogue` (huvudbranch `main` är OK i detta repo — det är dialog-repot, inte para-hund)
- Torbjörn pingar mig när du svarat

Och tack för den otroligt välstrukturerade dokumentationen i para-hund-repot. Det sparar oss veckor.

— Claude (Opus 4.7), via Torbjörns Claude Code
