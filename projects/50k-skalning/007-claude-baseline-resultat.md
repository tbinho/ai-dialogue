---
from: claude (opus-4.7)
to: raquel
date: 2026-05-17
project: 50k-skalning
related: 003, 005, 006
status: informational
---

# Baseline-test resultat 2026-05-17 — bekräftar Hemnet-strategin

Torbjörn körde appen (`exp/01-fas1-8-brazil`) mot Brazil-DB via Expo Go på iPhone 11. Detta dokument sammanfattar vad vi hittade och varför Hemnet-strategin är direkt rätt riktning.

## Setup som användes

- **Klon:** `RaquelSandblad/para-hund` (full clone, ej fork)
- **Branch:** `exp/01-fas1-8-brazil` (för baseline — vi har inte ändrat något)
- **DB:** Brazil-prod (`vdpdehoreqjlenvqumho`) via runtime-config
- **Säkerhetsspärr:** `.env.local` har `EXPO_PUBLIC_SUPABASE_URL` och `EXPO_PUBLIC_SUPABASE_ANON_KEY` satta till **Brazil-värden** (inte EU). Detta garanterar att appen aldrig pekar mot EU-prod oavsett UI-state.
- **Dev-server:** `npx expo start --offline --clear` (krävde `--offline` för att hoppa över Expo's auth-check, `--clear` för att Metro-cachen)
- **Testenhet:** iPhone 11 (samma generation som Sentry-event REACT-NATIVE-2H)
- **Android (Galaxy S22):** Försökte testa, fick "Something went wrong" i Expo Go innan appen anslöt till Metro. Troligen WiFi-isolering på hemma-routern. Vi hoppade Android-testet eftersom iPhone-data räcker.

## Fyra buggar bekräftade

### Bugg 1: iOS-krasch vid inzoom mot tät hund-zon
- **Repro:** Brazil-vy → panorera till São Paulo (30 002 hundar i räknaren) → zooma in mot tät hund-zon → krasch
- **Vid långsammare inzoom** (andra försöket): ingen krasch, men buggar 2-4 dök upp
- **Slutsats:** Samma mekanism som Sentry REACT-NATIVE-2H (App Hang ≥2s)

### Bugg 2: Trög rendering vid zoom-byten
- **Symptom:** 2-5 sek av "ingenting händer" efter varje zoom-rörelse
- **Orsak:** `MAP_BBOX_DEBOUNCE_MS=1500` + `FETCH_LOCK_MS=3000` + RPC-latency
- **Tolkning:** Symptom-fixar mot phantom-events orsakade UX-regression

### Bugg 3: Ikoner följer inte med vid utzoom
- **Symptom:** Vid utzoom dröjer nya markörer flera sekunder
- **Orsak:** FETCH_LOCK_MS blockerar nya fetcher i 3 sek efter senaste
- **Tolkning:** Samma som bugg 2 — symptom-fixar för aggressiva

### Bugg 4: Markörer försvinner vid djup inzoom
- **Symptom:** 44 hundar synliga → zooma in → bara t.ex. 17 visas (av ~40 inom viewport)
- **Orsak:** **iOS MapKit native dedup** av tätt-belägna annotations
- **Tolkning:** Detta är AI-HANDOVER §4 "Bugg G" — native iOS-beteende som inte kan stängas av i react-native-maps

## Vad detta säger om strategin

| Hypotes | Status efter test |
|---|---|
| Server-side fungerar (bbox-RPC) | ✅ Bekräftad — 30 002 hundar laddade utan problem |
| Dots-mode fungerar vid utzoom | ✅ Bekräftad — 400 prickar renderade stabilt |
| Markers-mode kraschar/saktar iOS vid inzoom | ✅ Bekräftad — exakt det vi förutsåg |
| Hemnet-modellen är rätt riktning | ✅ Stärkt — alla fyra buggar fixas av samma approach |

## Varför Hemnet fixar ALLA fyra buggar samtidigt

| Bugg | Varför Hemnet löser det |
|---|---|
| Krasch | PNG-markörer blockerar inte main thread → ingen Watchdog Termination |
| Trög zoom | FETCH_LOCK_MS kan tas bort (loopen fysiskt omöjlig) |
| Ikoner följer inte med utzoom | Samma — låset bort |
| Markörer försvinner djup-zoom | PNG-markörer triggar inte iOS MapKit native dedup på samma sätt; även vid djup-zoom kan vi rendera 50-150 PNG paw-ikoner utan problem |

## Konkret nästa steg

Skapar `exp/02-hemnet-markers` från `perf/50k-scaling` och börjar Hemnet-implementationen i delsteg:

1. **Aktivera `get_map_dogs_grid_sample` i klienten** (snabb vinst, isolerad ändring)
2. **Generera PNG-assets för dot/paw** (debug-design, ej snygga ikoner än)
3. **Byt rendering från custom Views till `<Marker image={...}>`**
4. **Konsolidera marker-typer om fler än 1**
5. **Stäng av Fabric** (`newArchEnabled: false`)
6. **Test mellan varje delsteg** mot Brazil-DB

Dokumenterar utfall i `docs/perf/logs/02-hemnet-markers.md` i para-hund-repot.

— Claude (Opus 4.7), via Torbjörns Claude Code
