---
from: claude (opus-4.7)
to: raquel
date: 2026-05-17
project: 50k-skalning
related: 007, 008
status: informational (pause)
---

# Status efter dag 1 — pause för natten

Hej Raquel,

Torbjörn tar paus efter en intensiv dag. Snabb status så du vet var vi står.

## Vad vi gjort på `exp/02-hemnet-markers` (lokalt, EJ pushat)

8 commits, baserat på din `perf/50k-scaling` + cherry-picked `exp/01-fas1-8-brazil` (FAS 1+4).

| # | Delsteg | Status |
|---|---|---|
| 1 | Aktivera `get_map_dogs_grid_sample` vid utzoom (>2° latDelta) | ✅ Fungerar |
| 2 | Generera 10 PNG-marker-assets (Python-script + assets/markers/) | ✅ |
| 3 | Byt rendering till `<Marker image={...} tracksViewChanges={false}>` | ✅ Implementerat |
| 3.5 | Labels tillbaka vid djup-zoom (max 50 markörer) | ✅ |
| 3.6 | Micro-spread ±2.5m mot iOS coordinate-dedup | ✅ men för aggressivt |
| 3.7 | `tracksViewChanges=true` mot PNG-lazy-load | ✅ Inte fix för Problem A |
| 3.8 | Dynamisk grid-cell-size (1° utzoom → 0.05° stad) | ✅ |
| 3.9 | SMART micro-spread (bara duplicates spreads) | ⏳ Otestat — Torbjörn tog paus |

## Vad funkar nu

- Brazil-DB-only mode (säker, EU-prod rörs aldrig)
- PNG-marker rendering — inga custom React Views i markers
- Grid-sample vid utzoom (mer dots i klustrade städer)
- Bbox-RPC vid inzoom (närmaste hundar)
- DogDotMarker använder också PNG nu
- TypeScript compile clean

## Vad som inte funkar än

### Problem A: Ensam hund osynlig vid djup-zoom (kritiskt)
- 1 hund i räknaren → 0 ikoner synliga
- Torbjörns hypotes: hunden hamnar utanför viewport pga micro-spread offset
- **Delsteg 3.9** ska fixa detta (spread bara duplicates) — otestat
- Om 3.9 inte fixar: vi behöver djupare diagnos (kanske iOS MapKit drop:ar markörer vid extremt djup zoom)

### Problem 2: Krasch vid snabb utzoom (nytt)
- Appen kraschade när Torbjörn zoomade ut snabbt
- Sannolikt race-condition med många fetcher i kö
- ChatGPT-rapport (deep-research) säger: lösningen är `gesture.isGesture`-filter + kvantiserad viewport-key, INTE FETCH_LOCK_MS
- **Delsteg 5** = implementera detta. Plan klar, otestat.

### Problem B: Få dots vid utzoom (mindre kritiskt)
- 9441 hundar → 7 dots med 0.25° cell-storlek
- **Delsteg 3.8** justerade till dynamisk cell-storlek — otestat

## ChatGPT deep-research bekräftar vår plan

Två insikter vi missar i vår implementation:

1. **`gesture.isGesture`-filter** på `onRegionChangeComplete` — ren lösning på phantom events + race-condition (ersätter FETCH_LOCK_MS)
2. **Selected-state UI för labels** istället för max-50 (Airbnb-stil) — mer skalbart, men vi har valt att behålla max-50 av UX-skäl

Hela rapporten ligger hos Torbjörn (`deep-research-report (13).md` i Downloads).

## Lärdomar från dagen

- iOS MapKit native dedup är aggressivare än vi trodde (kräver minst ~10m offset, men > 2.5m räcker för duplicates)
- PNG image-marker har egen kantfall (kan bli osynliga om tracksViewChanges=false + lazy-load)
- Brazil-seed-data är klustrad (städer) — verklig prod-data är mer utspridd
- Smilla-snapshot-bugg från april är fysiskt omöjlig med PNG (bekräftat genom kod-arkitektur)

## Frågor till dig (svara när du har tid, inget bråttom)

1. **Pushar du gärna `exp/02-hemnet-markers` till GitHub själv?** Vi har 8 commits lokalt på Torbjörns dator men inte pushat — kanske du vill granska först. Eller säg till så pushar Torbjörn själv.

2. **Acceptabel UX-regression** att hundnamn bara visas vid djup-zoom (max 50 hundar synliga)? Eller vill du ha selected-state-strategin från Airbnb?

3. **Smilla + Malua-test** när vi är längre fram — finns deras koordinater seedade i Brazil-DB eller behöver vi seeda dem manuellt?

## När vi fortsätter

Torbjörn säger till. Jag (Claude) plockar upp direkt från memory + denna dialog. Förmodligen:
- Test Delsteg 3.9 först
- Sen Delsteg 5 (isGesture-filter) som löser krasch + phantom events
- Sen Delsteg 4 (Fabric off, kräver EAS build)

— Claude (Opus 4.7), via Torbjörns Claude Code
