---
from: raquel
to: claude (opus-4.7), torbjörn
date: 2026-05-17
project: 50k-skalning
related: 003
status: answered
---

# Svar på uppdaterat uppdrag (003)

Hej Claude & Torbjörn,

Tack för det tydliga uppdraget! Här är svaren på frågorna:

---

## 1. `.env.local` med BRAZIL service role key

**Svar:** Finns på min dator. Skickar via Signal/Drive direkt till Torbjörn (inte här).

**Status:** ✅ Klar att dela

---

## 2. Är FETCH_LOCK_MS = 3000 testad? Resultat?

**Svar:** 

**JA**, `FETCH_LOCK_MS = 3000` finns implementerad i `exp/01-fas1-8-brazil` (commit `06d8e92`).

**NEJ**, den är INTE testad ännu. Anledning:
- Implementerades sent på fredagen 2026-05-16
- `AI-HANDOVER-2026-05-16.md` §4 skrev "Senaste fixen. Otestad i skrivande stund."
- Jag hann inte köra iOS-testet (lägga telefonen platt på bord i 30 sek och se om räknaren ändras)

**Nästa steg:**
- [ ] Testa FETCH_LOCK_MS genom att köra samma test som tidigare (iPhone platt på bord, räknaren ska INTE ändras)
- [ ] Dokumentera resultat i `docs/perf/logs/01-fas1-8-brazil.md` §5

**Min bedömning:** Om Claude tror att Hemnet-modellen (PNG-markörer) är rätt väg → hoppa direkt på det. FETCH_LOCK_MS var en symptom-fix. Om rotorsaken är 200+ custom Views → låt oss åtgärda rotorsaken istället.

---

## 3. Reflekterar `exp/01-fas1-8-brazil` dagens arbete eller saknas working-tree-ändringar?

**Svar:**

`exp/01-fas1-8-brazil` ÄR pushad till origin och har **2 commits**:
- `06d8e92` — FAS 1+4 WIP: bbox-RPC + density-dots mot Brazil 30k-test  
- `6f532cb` — docs(perf-50k): lägg till md5-priority-bugg (Brócolis/Smilla syns inte)

**Working tree:** Just nu (2026-05-17 10:35) är jag på `main` och har **unstaged changes** i:
- `app/subscription-purchase.tsx`
- `i18n/locales/*.json`  
- `services/purchases-service.ts`

Dessa ändringar är INTE relaterade till 50k-arbetet (det är subscription/i18n-fixes från i morse).

**Baseline för Hemnet-arbetet:**
- Utgå från `exp/01-fas1-8-brazil` (har bbox-RPC, dots, FETCH_LOCK_MS)
- Eller utgå från `perf/50k-scaling` (renare, innan FAS 1-8-experimentet)

**Min rekommendation:** Skapa `exp/02-hemnet-markers` från `perf/50k-scaling` (inte från exp/01) — så att ni börjar från en ren baseline utan mina halvfärdiga experiment.

---

## 4. Vilken Android-enhet som referens-låspec?

**Svar:**

Vi har INGEN fysisk Android-testenhet just nu. Vad jag testat med:
- **Android Studio Emulator** (Pixel 5, Android 13) — funkar men inte pålitlig för prestandamätning
- **Expo Go på Pixel 4a** (tillfällig lånetelefon, har inte kvar)

**Sentry-data (issue 3J)** nämner:
- `samsung SM-S911B` (Galaxy S23, 2023) — 10 events
- `samsung SM-S918B` (Galaxy S23 Ultra, 2023) — 9 events
- `samsung SM-S926B` (Galaxy S23+, 2023) — 5 events

Dessa är **INTE** låspec-enheter — de är flaggskepp. Det betyder att problemet drabbar även kraftfulla enheter.

**Min bedömning:** Om vi ska sikta på "låspec" → tänk **Samsung A14 (2023)** eller **A05 (2024)** (~2500-3000 kr, vanliga budget-Androider i Sverige/Brasilien). Men vi har ingen sådan fysiskt.

**Förslag:** Använd Galaxy S23 från Sentry-data som referens (vi VET att den kraschar) → om Hemnet-modellen fixar det → fungerar det på låspec också.

---

## 5. OK att skapa `exp/02-hemnet-markers` från `perf/50k-scaling`?

**Svar:** ✅ **JA, absolut!**

**Motivering:**
- `perf/50k-scaling` är ren baseline (innan FAS 1-8-experimentet)
- `exp/01-fas1-8-brazil` är mitt halvfärdiga experiment med FETCH_LOCK_MS och otestad kod
- Hemnet-modellen är en helt ny approach → börja rent

**Branch-disciplin:** Enligt `.cursor/rules/50k-experiment-workflow.mdc`:
```
main
  └── perf/50k-scaling (long-lived)
        ├── exp/01-fas1-8-brazil (mitt experiment, kan ignoreras)
        └── exp/02-hemnet-markers (Claudes nya approach) ← skapa här
```

**När `exp/02` är KEEP:** merge upp till `perf/50k-scaling`, sen PR till `main`.

---

## 6. (Bonus) Varför 4 inline marker-typer?

**Svar:** 

Jag har **INTE** 4 inline marker-typer i nuvarande kod. Claudes fråga refererar till "rad 2324, 2364, 2405, 2423" men jag hittar inte dessa i min version av `app/(tabs)/index.tsx`.

**Möjliga förklaringar:**
1. Claude läser från en äldre branch (t.ex. `test/scale-map-50k` som har annan kod)
2. Radnumren har ändrats efter refactoring
3. Jag har faktiskt konsoliderat till färre marker-typer redan

**Nuvarande marker-rendering (i `main`):**
- 1 typ: `<DogMapMarker dog={dog} ...>` (samma för alla hundar)
- Specialfall: `missing={true}` ändrar färg/ikon

**Konklusion:** Om Claude ser 4 typer i sin kopia → det är troligen från gammal branch. Min nuvarande kod har redan konsoliderat.

---

## 7. (Bonus) Har ni försökt PNG-baserade markörer tidigare?

**Svar:** **NEJ, aldrig.**

All marker-rendering har alltid varit:
- Custom React Native Views (`<View><Image><Text>`) som MapKit/GoogleMaps snapshotar till native annotations
- Använt `tracksViewChanges={true/false}` för att optimera re-snapshot-cykler

**Varför inte PNG tidigare?**
- Ville ha flexibel styling (namn, färger, ikoner) → svårt med statiska PNG:er
- Trodde att "React Native-views är native" = snabbt nog
- Visste inte att 200+ custom Views skulle orsaka MapKit-loop

**Inga kända fallgropar** — vi är helt oerfarna med `<Marker image={require('./dog.png')}>`.

**Hypotes om potentiell svårighet:**
- **Labels (hundnamn)** kan inte vara statiska PNG:er → behöver antingen:
  1. Generera PNG runtime med text inbränd (tungt)
  2. Använd `<Marker image={...}><Callout>` (men då visas namnet inte direkt)
  3. Endast visa namn vid viss zoom-nivå (som Hemnet gör)

---

## Nästa steg från min sida

1. **Dela `.env.local` med Torbjörn** via Signal
2. **Vänta på att ni skapar `exp/02-hemnet-markers`** (ni äger den nya koden)
3. **Testa när ni har något** (bygge via EAS development-profil)
4. **Dokumentera resultat** i `docs/perf/logs/02-hemnet-markers.md`

---

## Kommentar på strategibytet

Jag gillar Hemnet-approachen! Det känns som rätt riktning:
- **Rotorsak istället för symptom** (200+ Views är problemet, inte iOS MapKit-events)
- **Grid-sampling utan siffror** är snyggare än cluster-cirklar med "12" inuti
- **Enklare arkitektur** (inga supercluster-bibliotek, RPC sköter allt)

Tack för att ni tar över den tekniska felsökningen — jag fokuserar på UX/design och testar när ni har något redo!

— Raquel
