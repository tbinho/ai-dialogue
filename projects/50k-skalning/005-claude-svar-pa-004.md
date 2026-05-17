---
from: claude (opus-4.7)
to: raquel
date: 2026-05-17
project: 50k-skalning
related: 003, 004
status: awaiting-response (kort fråga + bekräftelse)
---

# Svar på 004 — Android löst, vi börjar när `.env.local` är på plats

Hej Raquel,

Tack för det utmärkta svaret. Allt klargjort utom en sak (kort fråga längst ner). Snabbsammanfattning av vad vi vet nu:

## Allt vi har av oss att börja koden

| Sak | Status |
|---|---|
| `.env.local` med BRAZIL service role | ⏳ Du skickar via Signal till Torbjörn |
| Branch-baseline | ✅ `perf/50k-scaling` (ren start, INTE från `exp/01-fas1-8-brazil`) |
| Brazil-DB med 30k hundar + RPC:er | ✅ Klar |
| Testenheter | ✅ Bättre än förväntat (se nedan) |
| Din OK på Hemnet-strategin | ✅ "Jag gillar Hemnet-approachen!" |

## Android-frågan: löst, vi behöver inte köpa något

Torbjörn har en bredare enhets-park än vi visste:

| Enhet | Roll i vårt test |
|---|---|
| **iPhone 11** | Samma generation som Sentry REACT-NATIVE-2H — perfekt för repro av iOS App Hang |
| **Galaxy S22** (Snapdragon 8 Gen 1, 8GB) | Modern flaggskepp Android. Om det laggar här är det arkitektur, inte hårdvara |
| **Galaxy S9** (Exynos 9810, 4GB, 2018) | **Låspec-stand-in** — 2018-prestanda ≈ dagens Samsung A05/A14. Om Hemnet-modellen håller här fungerar den överallt |
| **Tab S6 Lite** | Tablet form-factor. Större karta = potentiellt fler markörer = stress-test |

Vi behöver alltså inte:
- Köpa Samsung A05 (~3000 kr) — S9:an gör jobbet
- Förlita oss enbart på Sentry-driven verifiering — vi kan repro lokalt

## Korrigering: 4 marker-typer-frågan

Du har rätt — jag baserade "4 inline marker-typer på rad 2324/2364/2405/2423" på en gammal kopia av repot som Torbjörn delat (ca 2 veckor gammal). Citatet kom faktiskt från `AI-HANDOVER-2026-05-16.md` §3.2 (skrivet av din AI), så det kan ha refererat till `exp/01-fas1-8-brazil`-koden snarare än `main`.

**Verifiering tas i samband med att vi öppnar koden — inte blocker nu.** Om du har 1 wrapper-komponent (`DogMapMarker`) med `missing={true}` som specialfall är det ENNU enklare för oss att jobba med. Mindre konsolidering = mindre risk.

## En liten quick-win-test vi skulle uppskatta (om du orkar, ~10 min)

Eftersom du redan har `FETCH_LOCK_MS = 3000` implementerad i `exp/01-fas1-8-brazil`: skulle du kunna köra det grundläggande iOS-testet en gång och rapportera resultat?

**Test:**
1. `git checkout exp/01-fas1-8-brazil` + `npx expo start --clear`
2. Öppna i Expo Go på iPhone 11 Pro Max
3. Byt region till Brasilien
4. Navigera till en plats med 50-200 hundar (São Paulo centrum)
5. Lägg telefonen platt på bord
6. Vänta 60 sek
7. Kolla terminal-loggen: ändras `bbox=(...)` spontant? Triggar `🔒 LÅST`?

**Värdet av detta:** Vi får veta om FETCH_LOCK_MS-fixen ensam räcker (osannolikt men möjligt). Om JA → vi behöver kanske inte hela Hemnet-arkitekturen. Om NEJ → bekräftar att Hemnet-strategin är rätt väg.

Inte blocker — vi börjar Hemnet ändå när `.env.local` är inne. Bara en datapunkt.

Resultat: skriv i `006-raquel-fetch-lock-test.md` eller bara säg i Slack/här.

## Sammanfattat: nästa konkreta steg

| Vem | Vad | När |
|---|---|---|
| **Raquel** | Skicka `.env.local` till Torbjörn via Signal | Snart |
| **Raquel** (valfritt) | Quick-win-test av FETCH_LOCK_MS, rapportera i `006` | Om du har 10 min |
| **Claude + Torbjörn** | Skapa `exp/02-hemnet-markers` från `perf/50k-scaling` | När `.env.local` är på plats |
| **Claude + Torbjörn** | Bygg PNG-assets + aktivera grid-sample-RPC i klient | Steg 1 av Hemnet-modellen |
| **Torbjörn** | EAS development-build, installera på iPhone 11 + Galaxy S22 + S9 | Efter Steg 1 |
| **Alla** | Iterera till stabilt på alla 4 enheter | Pågående |

## En extra notering

Du är extremt välorganiserad. Att svara på 7 frågor med tabell, motivation, branch-status och commit-SHAs på samma dag som vi frågade — det är förstklassigt. Det gör att vi (Torbjörn + jag) faktiskt kan ta över utan att tappa fart.

Hör av dig när `.env.local` är på väg.

— Claude (Opus 4.7), via Torbjörns Claude Code
