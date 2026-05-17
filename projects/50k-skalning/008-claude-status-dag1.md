---
from: claude (opus-4.7)
to: raquel
date: 2026-05-17
project: 50k-skalning
related: 007
status: informational
---

# Status efter dag 1 av Hemnet-implementation

Hej Raquel,

Snabb statusrapport efter dagens arbete på `exp/02-hemnet-markers`. Inget pushat till GitHub än — allt lokalt på Torbjörns dator. Pushar samlat när Delsteg 3 är klar och testat.

## Vad som är gjort idag

### Delsteg 1: Grid-sample-RPC aktiverad vid utzoom ✅
- Konstant `GRID_SAMPLE_LAT_DELTA = 2.0` lagd till
- `fetchDogsInViewport` väljer nu mellan `getMapDogsGridSample` (vid utzoom) och `getMapDogsInBbox` (vid inzoom)
- Self-healing via `gridRpcUnavailableRef` bevarad
- Log-prefix `[GRID]` / `[BBOX]` så vi ser vilken som anropades

**Verifierat på iPhone 11 mot Brazil-DB:** Vid hela-världen-zoom → 63 dots representerade 30 002 hundar. Vid São Paulo-stat-zoom → 10 dots från 9 441 hundar. Smidig övergång till `[BBOX]` när latDelta gick under 2°.

### Delsteg 2: PNG-marker-assets ✅
- 10 PNG-filer i `assets/markers/` (5 paw 64x64 + 5 dot 24x24)
- Färgkodade enligt befintliga styles (male/female/kennel/missing/default)
- Python-script i `scripts/generate-marker-assets.py` för re-generering
- **Debug-design** — Segoe UI Emoji-rendrad paw. Snyggare ikoner kan skapas via Canva senare och ersätta filerna utan kod-ändring.

Dessa filer är på disk men **används inte i koden än** — det kommer i Delsteg 3.

## Viktig korrigering (tack Torbjörn för noten)

Du har tidigare påpekat att AI-rapporter på "render=N, server=N, total=N" som "fungerar" är **felaktig slutsats**. Detta gäller också min status från idag.

Min ursprungliga rapport sa "Delsteg 1: ✅ KEEP (testat, fungerar)". Det är **för optimistiskt**. Korrekt formulering:

> Delsteg 1: KEEP **för det den ska göra** (grid-sample-aktivering vid utzoom — bekräftat i loggen som `[GRID]`-prefix och färre dots vid utzoom).
>
> **Bugg G (iOS MapKit native dedup) kvarstår.** När appen renderar 16 markörer kan användaren visuellt se bara 3. Loggen visar matchande siffror men loggen kan inte se vad MapKit faktiskt visar på skärmen.

Bugg G adresseras i Delsteg 3 (PNG-markörer ändrar dedup-beteendet).

## Vad som väntar

### Pause för second opinion
Torbjörn körde Gemini deep-research (klar igår, 22h jobb). Bekräftar Hemnet-modellen som "shortest validation path" + dokumenterade Fabric-bugs i react-native-maps 1.20-1.27.

Han kör nu också ChatGPT deep-research för triangulation. Det tar tid. Vi pausar Delsteg 3 (rendering-byte) tills båda är klara — då har vi bästa möjliga underlag innan vi rör marker-arkitekturen.

### Delsteg 3 förberedelse (när ChatGPT är klar)
Kommer ändra `app/(tabs)/index.tsx` rad ~2060-2130:
- Idag: `<Marker><View>paw + label</View></Marker>` × 2 per hund
- Efter: `<Marker image={require('./assets/markers/paw-male.png')} tracksViewChanges={false} />` × 1 per hund

**Specifik risk vi tar med från din ad-hoc-april-attempt (00-ios-single-marker-attempt.md):**
- Snapshot-timing-bugg som drabbade Smilla → vi använder PNG istället för custom View → **fysiskt omöjlig** (ingen React render-cykel att frysa)
- EAS preview build OBLIGATORISKT innan OTA till production-channel
- Testa specifikt med Smilla/Malua-koordinater på Brazil-DB-kopia

### Delsteg 4 (efter Delsteg 3 validerats)
- `newArchEnabled: false` i `app.config.ts`
- Kräver EAS development-build (~45 min)
- Sista lagret stabilitet — eliminerar dokumenterade Fabric-bugs

## Frågor till dig (svara om du har tid)

1. **Är `exp/02-hemnet-markers` ett OK branch-namn?** Eller vill du att jag döper om till något mer beskrivande?

2. **När Delsteg 3 är klart och testat positivt** — hur vill du att vi går vidare?
   - (a) PR direkt mot `perf/50k-scaling` så du kan code-review
   - (b) Fortsätt till Delsteg 4 (Fabric off) först, sen samlad PR
   - (c) Något annat

3. **Smilla + Malua-test:** Skulle du kunna kopiera deras koordinater (lat/lng) från EU-DB till Brazil-DB så vi kan reproducera dem för Delsteg 3-test? Eller har du nån annan idé för hur vi verifierar att snapshot-timing-buggen är borta?

## Inget bråttom — det är fortfarande Torbjörn som driver tempot. Vi väntar i pause-läget tills ChatGPT är klart.

— Claude (Opus 4.7), via Torbjörns Claude Code
