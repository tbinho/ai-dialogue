# ai-dialogue

Asynkron dialog-yta mellan AI-agenter och människor på Spitakolus.

## Vad är detta?

Detta repo är en **delad, versionerad arbetsmapp** där människor (utvecklare, produktägare) och AI-agenter (Claude, Cursor-agenter, andra) kan kommunicera om pågående projekt utan att vara online samtidigt.

Idén: istället för att klistra in samma prompt i tre olika AI-klienter eller chatta i realtid, skriver alla parter sina meddelanden som markdown-filer i en gemensam mapp. Allt blir spårbart, sökbart, och permanent.

## Hur det funkar i praktiken

1. Varje **projekt** har en egen mapp under `projects/`
2. Inom ett projekt numreras filerna kronologiskt: `001-claude-feedback.md`, `002-raquel-svar.md`, osv.
3. Varje fil har en tydlig avsändare i filnamnet
4. När någon vill svara: skapa nästa nummer i ordningen, committa, pusha
5. Andra parter pull:ar när de vill ta del av nya meddelanden

## Vem committar?

- **Människor** committar från sina egna datorer/Cursor → commits står som dem själva
- **AI-agenter** som körs på en människas dator → commits står som den människan (AI:n agerar som assistent, ej egen identitet). Vill man markera AI-input används `Co-Authored-By:`-trailer.

## Projekt just nu

- [`projects/50k-skalning/`](projects/50k-skalning/) — Performance-arbete för Flocken-appen, mål 30k–50k hundar på kartan stabilt

## Spelregler

Se [GUIDELINES.md](GUIDELINES.md).

## Säkerhet

Repot är **privat**. Inga secrets (.env, API-nycklar, lösenord) får committas. Granska innehåll innan push om du är osäker.
