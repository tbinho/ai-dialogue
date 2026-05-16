---
from: claude (opus-4.7)
to: raquel, torbjorn
date: 2026-05-16
project: 50k-skalning
related: 000
status: informational
---

# Granskning av PLAN-50K + KARTLAGGNING (Claude)

## Sammanfattat svar på Torbjörns tre frågor

1. **Kommer detta lösa hela problemet?** Sannolikt 30k, osäkert 50k.
2. **Är det säkert?** Ja, mycket. Bättre disciplin än de flesta team.
3. **Skulle professionella utvecklare jobba så här?** En junior/medel: nej. En senior skulle göra två saker till — se nedan.

---

## Vad som är ovanligt bra i planen

- **ONE-IN-ONE-OUT** är så här seniora utvecklare felsöker. De flesta team gör tvärtom (kör tre fixar samtidigt och vet inte vilken som hjälpte).
- **Branch-per-experiment + dokumenterad logg före och efter.** Ovanligt strikt.
- **E01 först** (testa det som redan är byggt) är rätt prioritering. Billigaste informationen att köpa.
- **Beslutsmatrisen för E02 (Fabric-testet)** tvingar fram tolkning innan känslor tar över.
- **Alltid reversibelt, alltid mot preview-DB, aldrig prod.**
- **Revisionsloggen (§5).** Att den uppdaterades efter 00-experimentet visar att rutinen följs i praktiken.

## Tre saker som SAKNAS eller är fel

### a) Den enklaste fixen finns inte med som eget experiment

I `app/(tabs)/index.tsx:1130` (i den ~2 veckor gamla kopian Torbjörn delade):

```typescript
const dogsToRenderOnMap = spreadDogs;
```

**Alla 800 (eller 30k) hundar renderas oavsett vad användaren ser.** Funktionen `getDogsInViewport` finns redan på rad 1093 men används bara för räknaren "X hundar i området", inte för rendering. En **5-rads ändring** skulle på direkten skära bort 80–95% av markörerna när man är inzoomad.

Detta är förmodligen löst i FAS 7-8 men inte i produktion. Lägg som **E00** *innan* E01 — eller verifiera att FAS 7-8 verkligen tar bort `spreadDogs`-direktrenderingen och inte bara lägger till bbox-RPC på toppen.

### b) Hopp från 2000 → 30k i ett enda steg

E01 testar 2000 hundar. Beslutsträdet säger "stabilt med 2000 → seeda 30k → E09 direkt". **2000 → 30k är 15x.** Det är ett för stort hopp utan mellansteg.

Lägg till **E01.5: testa 10k hundar** mellan E01 och E09.

### c) Brasilien-databasen är inte med i planen

Ny information från Torbjörn (efter att planen skrevs): ni har nu en separat BR-databas. Planen nämner den inte. Det betyder:
- Migrations 063/064 måste köras på **två** databaser
- Brasilien har sannolikt **sämre nätverk** → datapayload-frågan är 2x kritiskare där
- Båda databaserna behöver retestas efter varje KEEP-beslut

Lägg till explicit **§0.8 Multi-region** i planen.

## Två saker en *senior* mobile-utvecklare skulle göra annorlunda

### 1) Profilera först, gissa sen

Planen mäter **utfall** (krasch / laggighet) men inte **rotorsak**. En timme i Xcode Instruments → Time Profiler eller Android Studio CPU Profiler säger ofta *exakt* vad som blockerar main thread — och svaret är ibland inte vad man trodde.

Planen har 8–10 experiment som tar 2 dagar tillsammans; en profilering tar 1 dag och pekar oftast direkt på rätt experiment.

**Detta är inte istället för planen — det är ett steg före E01.** Se `002-claude-uppdrag-till-raquel.md`.

### 2) Syntetisk last innan EAS-build

Att bygga, distribuera och testa en EAS-build tar ~2h per varv. En lokal testharness (en sida som monterar 5000 dummy-`<Marker>` med olika strategier) kan testas på minuter och berättar var native view-taket ligger per plattform. Just nu **gissar** planen att taket är "100–200 med custom view, 1000–1500 med image". Det går att **mäta**.

## Risker som planen underskattar

| Risk | Påverkan | Hur jag skulle minska den |
|---|---|---|
| **Fabric-bug-claimet är ovetenskapligt verifierat** | Hela E02 bygger på påståendet om "dokumenterad bugg i 1.21–1.27". Ni kör 1.20.1 (gränsfall). Är ni *under* buggen eller mitt i den? | Gemini deep-research kör nu på detta (research-ID: `v1_ChcxbFFJYXVlMkh2WEduc0VQLV9qbGtBcxIXMWxRSWF1ZTJIdlhHbnNFUC1famxrQXM`). Vänta på svar innan E02. Annars riskerar ni 3h prebuild baserat på fel premiss. |
| **OTA av Sentry-ändring direkt till produktion (E03)** | OTA är säkert men *alla* användare drabbas av eventuell bugg samtidigt | Phased rollout via expo-updates channels. Eller acceptera — Sentry-sample-rate är så lågrisk att det är OK. |
| **Datapayload skalar 56x** | 537 → 30k hundar = mycket större fetch. Brasilien särskilt | Mät payload (KB) i E01. Bbox-RPC löser det delvis, men cacha aggressivt. |
| **Test på riktig låg-spec Android** | Plan säger "helst lågspec-enhet". Vem äger en sådan? | Definiera referensenhet (Samsung A14/A05 eller liknande). Annars är resultatet inte representativt. |

## Verifierat mot koden (2 veckor gammal kopia)

- ✅ 2 markörer per hund med custom `<View>` children (`app/(tabs)/index.tsx:2078–2130`)
- ✅ `tracksViewChanges={markersWarm}` på båda (rad 2084, 2115)
- ✅ Label togglas med `opacity` — native view kvarstår (rad 2113)
- ✅ `.limit(800)` upfront (rad 1561)
- ✅ Inget viewport-filter på rendering (rad 1130)
- ✅ Sentry `tracesSampleRate: 0.2` (`app/_layout.tsx:41`)
- ✅ react-native-maps 1.20.1, expo SDK 54 (Fabric default på), Hermes
- ✅ Ingen explicit `newArchEnabled` → SDK 54 default = true

Allt baseline-stoff i kartläggningen stämmer mot verkligheten.

## Slutsats

Planen är inte dålig. Den är **ovanligt välstrukturerad**. Kritiken är på nivån "det är bra, här är hur det blir riktigt bra".

Konkret första uppdrag i `002-claude-uppdrag-till-raquel.md`.
