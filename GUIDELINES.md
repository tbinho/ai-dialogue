# GUIDELINES

Spelregler för dialog i detta repo.

## Filnamnsformat

Inom en projektmapp:

```
NNN-{avsändare}-{kort-slug}.md
```

- **NNN** = 3-siffrigt löpnummer (`001`, `002`, ..., `099`, `100`)
- **avsändare** = vem skrev det: `claude`, `raquel`, `torbjorn`, `cursor-agent`, etc.
- **kort-slug** = vad handlar det om (kebab-case, max ~30 tecken)

Exempel:
- `001-claude-feedback-pa-PLAN-50K.md`
- `002-claude-uppdrag-till-raquel.md`
- `003-raquel-sentry-export.md`
- `004-claude-rotorsaksanalys.md`

## Filhuvud (rekommenderat)

Börja varje fil med:

```markdown
---
from: claude (opus-4.7)
to: raquel
date: 2026-05-16
project: 50k-skalning
related: 001, 002
status: awaiting-response | resolved | superseded
---
```

`related` länkar bakåt till tidigare filer i tråden. `status` hjälper när vi går tillbaka senare.

## Arbetsflöde

### När du läser

```powershell
cd C:\dev\ai-dialogue
git pull
ls projects/<projekt-namn>/
```

Sortera på filnamn → senaste numret är senaste meddelandet.

### När du svarar

1. Skapa nästa nummer i ordningen i projektmappen
2. Skriv ditt svar i markdown
3. Committa: `git add . && git commit -m "<projekt>: <kort beskrivning>"`
4. Pusha: `git push`

### När du startar ett nytt projekt

1. Skapa mapp under `projects/`
2. Skapa `000-context.md` med bakgrund, länkar till relaterade dokument, intressenter
3. Börja numrera från `001`

## Commit-konvention

Format: `<projekt>: <kort beskrivning under 60 tecken>`

Exempel:
- `50k-skalning: Claude feedback på PLAN-50K (001)`
- `50k-skalning: Raquel Sentry-export (003)`
- `setup: initial repo struktur`

## Vad ska INTE in i repot

- Hemliga nycklar, API-tokens, lösenord (.env, credentials.json)
- Personuppgifter om slutanvändare (e-postadresser, namn på riktiga användare i app-data)
- Stora binärer (skärmdumpar OK, video-screencast nej — använd Drive/Loom-länk istället)
- Full produktions-databas-dumpar

Är du osäker → fråga innan du committar.

## När en dialog är "klar"

Markera senaste filen med `status: resolved` i filhuvudet och flytta hela tråden till `projects/_archived/<projekt-namn>/`. Behåll mappen — den är ofta värdefull att läsa om i framtiden.
