---
from: claude (opus-4.7)
to: raquel
date: 2026-05-16
project: 50k-skalning
related: 000, 001
status: awaiting-response
---

# Uppdrag: Profilera först, gissa sen — innan vi kör PLAN-50K E01

## TL;DR

Innan vi exekverar PLAN-50K vill vi titta på Sentry-data vi redan har (20% trace-sampling sedan månader) och lägga in fokuserad instrumentering. Detta tar ~1 dag totalt och pekar oftast direkt på rätt experiment, istället för att vi kör 8 experiment baserat på hypoteser.

Granskningen av planen finns i [001-claude-feedback-pa-PLAN-50K.md](001-claude-feedback-pa-PLAN-50K.md). **Den ersätter inte planen** — den lägger till ett steg före E01.

## Tre uppgifter i prioritetsordning

### Uppgift 1: Sentry-export (15 min) — STARTA HÄR

Exportera/skärmdumpa från sentry.io → projekt `react-native`:

**A. Issue-detaljer (full JSON, "Export" → "Download JSON"):**
- REACT-NATIVE-K (38 användare, App Hang ≥2s, iOS 26.3.1)
- REACT-NATIVE-M
- REACT-NATIVE-Q
- REACT-NATIVE-1S (Background ANR `MapMarker.updateMarkerIcon`, Android 15)
- REACT-NATIVE-15 (SIGBUS `SkPaint::~SkPaint`, Android 10)

**B. Performance → Transactions, senaste 14 dagarna:**
- Filtrera på map-screen / breeding-mode
- Sortera på "Slowest"
- Skärmdump topp 10
- Klicka in på #1 → skärmdump av span waterfall (visar var tiden går)

**C. Performance → "App Start" och "Slow Frames":**
- Skärmdump trend senaste 30 dagarna
- Korrelera med antal aktiva användare för att se om det skalar med last

**Leverans:** Zippa allt och commita i `projects/50k-skalning/data/003-sentry-export-2026-05-16/` (skapa mappen). Beskriv vad som finns i en kort `003-raquel-sentry-export.md` i projektroten.

⚠️ **Innan du committar:** Granska JSON-filerna — om de innehåller riktiga användar-e-postadresser eller andra personuppgifter, anonymisera dem först (search-and-replace till `user-1@example.com` osv).

### Uppgift 2: Instrumentera koden (1h) — efter Sentry-svar

Skapa branch `perf/instrumentation` baserad på `hotfix/map-regressions`. Lägg in fokuserad mätning i `app/(tabs)/index.tsx`:

```ts
// Vid fetchDogs (rad ~1510)
const t0 = performance.now();
const { data } = await queryPromise;
Sentry.metrics.distribution('fetchDogs.duration_ms', performance.now() - t0);
Sentry.metrics.distribution('fetchDogs.row_count', data?.length || 0);
Sentry.metrics.distribution('fetchDogs.payload_kb',
  JSON.stringify(data || []).length / 1024);

// I spreadDogs useMemo (rad ~692)
// Wrap med performance.now() + Sentry.metrics.distribution('spreadDogs.duration_ms', ...)

// I handleRegionChange (rad ~1071)
// Counter: Sentry.metrics.increment('handleRegionChange.calls') (för att se throttle-behov)

// Vid MapView-rendering, räkna markörer per render:
Sentry.metrics.gauge('map.markers_rendered', dogsToRenderOnMap.length)
```

**Säkerhet:** Bara dev/preview-builds, INTE production OTA. Risk: `Sentry.metrics` utan rate-limiting kan kosta. Lägg en dev-only-guard:

```ts
if (__DEV__ || process.env.EAS_BUILD_PROFILE !== 'production') {
  Sentry.metrics.distribution(...)
}
```

### Uppgift 3: Bygg EAS development-build för Android (30 min, passivt)

`eas build --profile development --platform android` på `perf/instrumentation`. Skicka APK-länken till Torbjörn — han sideloadar på en Samsung-lågspec och kör testscenariot i 2 min. Datan dyker upp i Sentry Metrics.

## Vad Claude levererar tillbaka

När (A) Sentry-exporten är inne kommer jag att leverera:

1. **Rotorsaksanalys** — vilket steg i marker-pipeline som faktiskt blockerar main thread (baserat på riktiga stack traces, inte gissning)
2. **Reviderad PLAN-50K v3** — med E00 (viewport-filter), E01.5 (10k-test) och §0.8 (multi-region). Eventuellt omprioritering av E04/E05 om Sentry-datan visar oväntade saker.
3. **Konkret första experiment** med förväntad effekt och hur vi verifierar att rotorsaken faktiskt försvann

## Frågor till dig (svara i `003-raquel-svar.md`)

1. Stämmer Sentry-issue-ID:n ovan fortfarande, eller har Sentry rotated dem?
2. Har ni en preview-DB för Brasilien också, eller bara för EU? Om bara EU — vill vi sätta upp en BR-preview innan vi börjar?
3. Vilken Android-lågspec-enhet kan vi använda som referens? (Samsung A14, A05, något annat?)
4. Vill du köra parallellt med profileringen redan starta E01 (testa FAS 1-8 mot EU-preview)? Eller vill du vänta tills vi vet vad Sentry säger?

## Praktiska saker

- All kommunikation via filer i denna mapp. Numrera kronologiskt.
- Torbjörn agerar brygga ifall något behöver klargöras snabbt mellan oss.
- Gemini deep-research kör parallellt på Fabric-bug-frågan (research-ID i 001). Resultatet kommer som `004-claude-gemini-fabric-research.md` när det är klart.

Hör av dig.

— Claude (Opus 4.7), via Torbjörns Claude Code
