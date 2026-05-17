---
from: claude (opus-4.7)
to: raquel
date: 2026-05-17
project: 50k-skalning
related: 007, 008, 009
status: informational (slutsumma dag 1)
---

# Slutsumma dag 1 — vad vi gjort, vad som funkar, vad som inte funkar

Hej Raquel,

Torbjörn tar paus efter en lång dag. Slutsumma så du har full bild.

## 10 commits på `exp/02-hemnet-markers` (lokalt, EJ pushat ännu)

```
ed5f533  Delsteg 5: isGesture-filter + viewport-quantization
d084807  Delsteg 3.9: SMART micro-spread - bara duplicates
748492d  Delsteg 3.8: dynamisk grid-cell-size
84d3491  Delsteg 3.7: tracksViewChanges=true
cdc6432  Delsteg 3.6: micro-spread mot iOS coordinate-dedup
2fb2c72  Delsteg 3.5: hund-namn-labels tillbaka (max 50)
e11d479  Delsteg 3: byt rendering till PNG-image-markörer
0fde7a3  Delsteg 2: PNG-marker-assets (10 PNGs + Python-script)
1a9a835  Delsteg 1: aktivera get_map_dogs_grid_sample vid utzoom
18e4c3b  Testlogg-skelett 02-hemnet-markers
+ Raquels cherry-picked FAS 1+4 commit som baseline
```

## Vad funkar nu (testat på iPhone 11 + Brazil-DB)

| Sak | Status |
|---|---|
| Krasch vid inzoom | ✅ Borta (Delsteg 3 + 5) |
| Krasch vid snabb pan | ✅ Borta (Delsteg 5 isGesture-filter) |
| PNG-marker rendering | ✅ Implementerat |
| Grid-sample vid utzoom | ✅ Dynamisk cell-storlek (Delsteg 3.8) |
| Bbox-RPC vid inzoom | ✅ Oförändrat från din FAS 1 |
| Brazil-DB-only mode | ✅ EU-prod aldrig rörd |
| Phantom region-events | ✅ Filtrerade bort (Delsteg 5) |

## Vad som INTE funkar än

### Problem A: Ensam hund osynlig vid djup-zoom (kritiskt)
- Räknaren visar t.ex. 1 hund i området, men 0 ikoner syns
- 10 hundar → 3 ikoner (samma mönster som vid bugg G du dokumenterade)
- Torbjörn testade efter alla våra fixar (3.6 → 3.7 → 3.9) — samma symptom

**Hypotes:** Detta är förmodligen **react-native-maps Fabric-bug**:
- GitHub issues #5909 (Custom image Markers not rendering) och #5912 (Marker image prop fails under New Architecture) — båda OPEN
- Symptom matchar EXAKT vad ChatGPT + Gemini deep-research dokumenterar
- Vi kör Expo SDK 54 med `newArchEnabled: true` (default)
- Bibliotekets eget statement: "Fabric-stöd kommer först i 1.26.1+" — vi kör 1.20.1

### Bonus-bekräftelse
ChatGPT-rapporten (deep-research klart ikväll) bekräftade hela Hemnet-strategin som "shortest validation path" och flaggade specifikt Fabric som risk. Båda externa AI:er (Gemini + ChatGPT) säger samma sak.

## Nästa steg: Delsteg 4 (Fabric off)

**Vad det innebär:**
1. Ändra `app.config.ts`: lägg till `newArchEnabled: false`
2. Köra `npx expo prebuild --clean`
3. Bygga EAS **development**-build (45 min)
4. Installera på iPhone 11 + ev. Galaxy S22
5. Verifiera om Problem A försvinner

**Säkerhet:**
- Development-build → INTE production-channel
- Berör inte några riktiga användare
- Brazil-DB-only kvarstår
- Reverterbart (`newArchEnabled: true` igen om något bryts)

**Sannolikhet att lösa Problem A:** 50-60% enligt Gemini + ChatGPT-rapporter.

**Risk om det inte fungerar:** Hemnet-modellen otillräcklig → vi måste överväga `@rnmapbox/maps`-migration (3-6 veckor, ingen garanti att lösa heller).

## Frågor till dig

### Fråga 1: Vill du köra EAS build själv eller ska Torbjörn göra det?
- Du har EAS-konto-tillgång + kan trigga från din maskin
- Torbjörn kan också om du delar EXPO_TOKEN, men du har erfarenheten

### Fråga 2: Pusha `exp/02-hemnet-markers` till GitHub nu?
- 10 commits lokalt på Torbjörns dator
- Vill du granska först, eller bara pusha så du kan checka ut och bygga?

### Fråga 3: Reanimated 4.1.1 på legacy architecture
- Reanimated 4 är optimerat för Fabric
- Med `newArchEnabled: false` kan den bli något långsammare
- Har du sett att andra paket (Reanimated, others) kräver Fabric i er kodbas?

### Fråga 4: Om Delsteg 4 inte löser — vad är din ståndpunkt om Mapbox-migration?
- 3-6 veckor jobb
- Ej garanti att lösa
- Stort tekniskt skifte
- Värt att börja diskutera nu så vi har plan B klar

## Sammanfattning för Torbjörn

**Vad vi VET nu jämfört med morgonen:**
- Hemnet-modellen är genomförd ✅
- Krash-problem är lösta ✅
- "Ensam hund osynlig"-buggen är förmodligen Fabric-relaterad
- Vi har en konkret nästa-steg-plan (Delsteg 4)
- Vi har **två externa AI-validerings** av hela strategin

**Vad vi INTE löst:**
- Den specifika rendering-buggen kvarstår
- Kräver native build för att verifiera nästa hypotes

**Är detta framgång eller misslyckande?**
Båda. Vi har gjort något grundläggande nytt (PNG-arkitektur) som du inte gjort på 6 månader. Vi har också inte LÖST grundproblemet helt. Realistiskt: Hemnet-modellen behöver 2-3 dagar att fullt validera. Vi har gjort dag 1.

— Claude (Opus 4.7), via Torbjörns Claude Code
