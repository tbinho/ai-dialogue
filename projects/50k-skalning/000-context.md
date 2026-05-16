---
from: torbjorn
to: alla
date: 2026-05-16
project: 50k-skalning
status: active
---

# 50k-skalning — projektkontext

## Problemet

Flocken-appens karta börjar lagga (mer på Android än iOS) när det är många hundar synliga. Just nu finns ~537 publicerade hundar i produktion. Vi behöver hantera **30 000–50 000 hundar** stabilt på både iOS och Android.

Vi har nu också appen för Europa (SE/NO/DK) och Brasilien — samma app, separata databaser.

## Tidigare arbete

Det är **mycket** redan gjort. Se referensdokument:

- **`PLAN-50K.md`** — den officiella 4-fas-strategin med experiment-flöde (E01–E10), branch-disciplin (`exp/NN-slug`), beslutsmatris (KEEP/REVERT/PARTIAL), revisionslogg
- **`KARTLAGGNING-50K-2026-04-21.md`** — tidslinje över allt som testats sedan december 2025, hypotes-matris, lista över vad som fungerar vs vad som orsakat problem

Båda dokumenten ligger hos Torbjörn (`C:\Users\torbj\Downloads\`). De delas i Slack/Drive innan vi börjar.

## Status (2026-05-16)

- **Produktion live:** release 1.1.5 (iOS buildNumber 88, Android versionCode 78). Endast aktiv perf-change är label-tröskel 0.20/0.30 + iOS `tracksViewChanges=false` + Android warmup 800ms.
- **FAS 1–8 byggt** på `release/hotfix-1.1.5` men **otestat i produktion-likande miljö**. Migrations 063+064 applicerade på preview-DB `hotfix-map-preview` (project ref `eelyjejyvohgysqfpimr`) som har 2000 seedade hundar.
- **Sentry just nu:**
  - REACT-NATIVE-K: 38 användare, 60 events, App Hang ≥2s, iOS 26.3.1
  - REACT-NATIVE-M/Q: 26+13 användare, samma stack
  - REACT-NATIVE-1S: Background ANR `MapMarker.updateMarkerIcon`, Android 15
  - REACT-NATIVE-15: SIGBUS `SkPaint::~SkPaint`, Android 10

## Tekniska nyckel-fakta

| Sak | Värde |
|---|---|
| react-native | 0.81.5 |
| react-native-maps | 1.20.1 |
| react-native-map-clustering | 4.0.0 (installerad, USE_MAP_CLUSTERING=false) |
| Expo SDK | 54 |
| New Architecture (Fabric) | Default ON (ingen explicit override i `app.config.ts`) |
| Hermes | ON (Android jsEngine: 'hermes', iOS default) |
| Sentry tracesSampleRate | 0.2 |
| Markörer per hund idag | 2 (paw + label, båda custom `<View>`-children) |

## Intressenter

- **Torbjörn (`tbinho`)** — grundare, icke-utvecklare, agerar koordinator
- **Raquel (`RaquelSandblad`)** — chefsutvecklare, gör koden
- **Claude Opus 4.7 (extern)** — review + AI-assist, kör via Torbjörns Claude Code

## Vad vi jobbar mot nu

Innan vi exekverar PLAN-50K (E01 → E10) vill vi **profilera först, gissa sen**. Sentry har redan 20% sampling = vi har data, vi har bara inte tittat på den.

Nästa filer i mappen:
- `001-claude-feedback-pa-PLAN-50K.md` — Claudes granskning av planen + kartläggningen
- `002-claude-uppdrag-till-raquel.md` — konkret första uppdrag (Sentry-export + instrumentering + dev-build)
