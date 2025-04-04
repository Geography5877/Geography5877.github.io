---
title: Agenter eller dobbeltagenter
date: 2025-04-07 06:57:00 +0200
categories: [Språkmodeller og deres utfordringer]
tags: [llm, agenter]     # TAG names should always be lowercase
math: true
description: I denne posten skal vi se nærmere på agenter; hvordan de fungerer og hvorfor de ikke løser problemene med språkmodeller
media_subpath: /assets/images/2025-04-04/
---
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Denne posten er en del av en lengre serie poster om språkmodeller og problemer med disse. Jeg vil starte med å presisere at språkmodeller helt klart har sine bruksområder, men ettersom denne serien ser på problemene med språkmodeller kommer jeg til å fokusere på ulemper og problemer forbundet med hvordan språkmodeller fungerer i dag.
{: .prompt-info }
<!-- markdownlint-restore -->

## Hva er en agent?
I de siste par årene har mer og mer av oppmerksomheten rundt språkmodeller dreiet i retning agenter. Vi hører nå om store forskningsprosjekter og industrielle satsinger hvor agenter har en sentral rolle. Men hva er egentlig en agent?

Uttrykket agent, ihvertfall i maskinlæring, kommer fra et subfelt som kallser for Reinforcement Learning. Dette er et læringsparadigme hvor vi trener med belønning og straff istedet for med eksempler på hvordan en oppgave skal løses. Modellen skal for eksempel navigere i en labyrint og får belønning når den finner utgangen. I reinforcement learning er en agent, den tingen som velger og utfører handlinger. I labyrint eksempelet er den "tingen" som skal finne veien ut, kalt agenten.

Agenter i språkmodeller stammer fra samme definisjon. Det er egne entiteter som velger og utfører handlinger. Det er flere måter vi kan gjøre en språkmodell om til en agent på, men den aller vanligste består av opptil tre steg, hvor hvert av stegene alene kan resultere i en agent, eller alle stegene kan brukes sammen til å lage en agent:

### Oppgavespesifikt systemprompt
Den mest vanlige tilnærmingnen for å lage en agent, er å gi språkmodellen et oppgavespesifikk systemprompt. That's it.

Som vi har sett [tidligere](https://enklypesalt.com/posts/Hvordan-opplever-kien-en-samtale/#hvordan-ser-en-samtale-egentlig-ut) kan vi endre systempromptet for å få en modell som oppfører seg på en ny måte. Vi kan da for eksempel lage en kundebehandlingsagent ved å endre system promptet til noe sånt som "Du er en fremragende kundebehandler. Du hjelper kunder med alle deres behov. Du er høflig og omtenksom.". Dette er nok til å endre modellens oppførsel til å passe som en kundebehandler. Vi kan gjøre tilsvarende for å få en agent som passer seg som forskningsassistent, eller bibliotekar.

I de aller fleste tilfeller er det til og med den samme modellen som får de forskjellige rollene. Som vi så i [en tidligere post](https://enklypesalt.com/posts/Hvordan-svarer-kien/#autoregressiv-spr%C3%A5kmodellering), mater vi all tekst inn på nytt, hver gang vi interagerer med en språkmodell. Det betyr at vi også mater inn system promptet på nytt hver gang. Så når vi snakker med forskjellige agenter er det egentlig den samme modellen som tar på seg en ny hatt eller parykk for å late som den nå er en helt annen modell. Ikke helt ulikt debatten i Team Antonsen:
{% include embed/youtube.html id='1bXjw7qVqqg' %}

#### Tool-use og MCP
For å gi forskjellige agenter forskjellige egenskaper, utover hattene og parykkene jeg nevnte tidligere, kan vi gi dem tilgang til forskjellige verktøy. I tilfellet forskningsassistent, kan vi for eksempel gi agenten tilgang til å søke på Google Scholar og å lese forskningsartikler. For en bibliotekar ville det kanskje vært fornuftig å gi agenten tilgang til bibliotekets database over bøker, brukere og utlånsoversikt. Vi kan på denne måten begrense tilgangen på verktøy til enkelte agenter og dermed lage mer spesialiserte og kompetente agenter.

#### Oppgavespesifikk trening
For å ytterligere forbedre spesielle agenter, kan vi faktisk gjøre [fine-tuning](https://enklypesalt.com/posts/Hvordan-svarer-kien/#fine-tuning)

### Koordinering og kommunikasjon

