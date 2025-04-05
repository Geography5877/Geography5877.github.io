---
title: Agenter eller dobbeltagenter
date: 2025-04-04 06:57:00 +0200
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

Uttrykket agent, i hvert fall i maskinlæring, kommer fra et subfelt som kaller for Reinforcement Learning. Dette er et læringsparadigme hvor vi trener med belønning og straff isteden for med eksempler på hvordan en oppgave skal løses. Modellen skal for eksempel navigere i en labyrint og får belønning når den finner utgangen. I reinforcement learning er en agent, den tingen som velger og utfører handlinger. I labyrint eksempelet er den "tingen" som skal finne veien ut, kalt agenten.

Agenter i språkmodeller stammer fra samme definisjon. Det er egne entiteter som velger og utfører handlinger. Det er flere måter vi kan gjøre en språkmodell om til en agent på, men den aller vanligste består av opptil tre steg, hvor hvert av stegene alene kan resultere i en agent, eller alle stegene kan brukes sammen til å lage en agent:

### Oppgavespesifikt system prompt
Den mest vanlige tilnærmingen for å lage en agent, er å gi språkmodellen et oppgavespesifikk system prompt. That's it.

Som vi har sett [tidligere](https://enklypesalt.com/posts/Hvordan-opplever-kien-en-samtale/#hvordan-ser-en-samtale-egentlig-ut) kan vi endre system promptet for å få en modell som oppfører seg på en ny måte. Vi kan da for eksempel lage en kundebehandlingsagent ved å endre system promptet til noe sånt som "Du er en fremragende kundebehandler. Du hjelper kunder med alle deres behov. Du er høflig og omtenksom.". Dette er nok til å endre modellens oppførsel til å passe som en kundebehandler. Vi kan gjøre tilsvarende for å få en agent som passer seg som forskningsassistent, eller bibliotekar.

I de aller fleste tilfeller er det til og med den samme modellen som får de forskjellige rollene. Som vi så i [en tidligere post](https://enklypesalt.com/posts/Hvordan-svarer-kien/#autoregressiv-spr%C3%A5kmodellering), mater vi all tekst inn på nytt, hver gang vi interagerer med en språkmodell. Det betyr at vi også mater inn system promptet på nytt hver gang. Så når vi snakker med forskjellige agenter er det egentlig den samme modellen, som bruker system promptet som en hatt eller parykk for å late som den nå er en helt annen modell. Ikke helt ulikt debatten i Team Antonsen:
{% include embed/youtube.html id='1bXjw7qVqqg' %}

#### Tool-use og MCP
For å gi forskjellige agenter forskjellige egenskaper, utover hattene og parykkene, kan vi også gi dem tilgang til forskjellige verktøy. I tilfellet forskningsassistent, kan vi for eksempel gi agenten tilgang til å søke på Google Scholar og å lese forskningartikler. For en bibliotekar ville det kanskje vært fornuftig å gi agenten tilgang til bibliotekets database over bøker, brukere og utlånsoversikt. Vi kan på denne måten begrense tilgangen på verktøy til enkelte agenter og dermed lage mer spesialiserte og kompetente agenter.

#### Oppgavespesifikk trening
For å ytterligere forbedre spesielle agenter, kan vi faktisk gjøre [fine-tuning](https://enklypesalt.com/posts/Hvordan-svarer-kien/#fine-tuning) med eksempler fra den spesifikke rollen vi ønsker at agenten skal ha. Her kan vi lage svært spissede agenter med spesialkompetanse. Vi kan også gi disse tilgang til tool-use for å gjøre agenten komplett.

### Koordinering og kommunikasjon
Som du sikkert skjønner, kan vi enkelt lage oss en haug med forskjellige agenter, alle med sine parykker, verktøy og spesialkompetanse, som vi så kan bruke for å løse forskjellige oppgaver. Men vi er fortsatt selv den som må ta beslutningen om når vi skal bruke en agent og hvilken agent vi skal bruke. Men slik trenger det ikke være. Vi kan også lage oss en sjefsagent eller koordineringsagent, som har som oppgave å delegere oppgaver til de forskjellige andre agentene. Denne agenten er selvfølgelig ikke noe mer spennende enn den samme modellen som har fått et system prompt om at den har som oppgave å delegere oppgaver.

#### En organisasjon av agenter
Vi kan også lage mellomlederagenter og så videre og så videre. Vi ser altså at vi kan lage oss en hel organisasjon av agenter, hvor vi bare forholder oss til lederen. Vi kan da stille diverse spørsmål om forskning, om medisin eller om været, og lederagenten vil delegere oppgaven nedover i organisasjonen slik at den blir løst av den best skikkede agenten. Det er i det minste slik agenter blir markedsført.

#### Kommunikasjon og feilkommunikasjon
Denne agentorganisasjonen høres jo ganske lovende ut. Du har tilgang til utallige eksperter som kan hjelpe deg med hva det skulle være. Men la oss ikke glemme at hver av disse ekspertene er en fullverdig språkmodell, med alle feil og mangler en språkmodell innehar. I hvert kommunikasjonssteg i aksjonsflyten som oppstår når et agentsystem skal svare deg, er det muligheter for feiltolkning. La oss ta det enkleste eksempelet der vi har to agenter. En leder og en spesialist. Da kan mulige uavhengige feilkilder være:

1. Lederagenten kan ordlegge seg på en uheldig måte når den kommuniserer med spesialisten.
2. Spesialisten kan feiltolke hva lederagenten kommuniserer. (Husk at disse har forskjellige system prompts og vil derfor tolke samme tekst forskjellig).
3. Spesialisten kan velge å bruke et verktøy på feil måte. (En språkmodell gjør ofte trivielle feil når den skal bruke verktøy. Selv ikke modeller trent spesielt på koding er feilfrie kodere).
4. Spesialisten kan feiltolke svaret den får av verktøyet den bruker. Da vil den også videreformidle denne feiltolkningen oppover i kjeden igjen.
5. Spesialisten kan ordlegge seg uheldig når den rapporterer tilbake til lederagenten.
6. Lederagenten kan feiltolke spesialisten.
7. Lederagenten kan ordlegge seg feilaktig når den skal rapportere tilbake til deg.

Vi ser altså at det er utrolig mange feilkilder, selv i dette enkle eksempelet. I hvert ledd i en agentorganisasjon er det 1+5+2 uavhengige feilkilder, avhengig om det er flere mulige agenter å velge mellom (+1), feil i kommunikasjon mellom agenter (+5) og feil bruk av verktøy (+2). Legg også merke til at feilkildene kan kombineres for å lage enda flere mulige kombinasjoner. Dette skalerer raskt til uforholdsmessig mange feilkilder i en større agentorganisasjon. Jeg har også demonstrert i tidligere poster at hver og én av disse feilene vil skje med relativt høy frekvens, gitt dagens språkmodeller. Og når det skjer, er feilsporing svært vanskelig. Den eneste fornuftige handlingen for en bruker vil rett og slett være å bare prøve igjen og håpe at stokastisiteten i systemet gir et bedre svar denne gangen.

### Kostnad og energibruk
En annen utfordring med agent-baserte systemer er kostnadshåndtering. Husk at vi som brukere betaler for hvert token som genereres og hvert token som leses. Men nå leses og genereres tokens om hverandre i parallell og av mange agenter samtidig, bare for å løse én enkelt forespørsel fra en bruker. Kostnadene ved dette vil rakst kunne vokse seg enorme. La oss også tillate at hver agent kan være trent til å "[resonnere](https://enklypesalt.com/posts/Er-resonnementer-losningnen/)", så har vi garantert at de som leverer språkmodeller har en stabil inntekt. 

La oss heller ikke glemme energikonsumpsjonen som kommer med all denne kommunikasjonen, frem og tilbake, mellom alle agentene våre. Heldigvis kan vi velge å bruke mindre modeller for mer spesialiserte agenter, så der har vi i det minste spart en del av energiregnskapet, men det er ikke på langt nær godt nok.

Og alt dette for å få et svar som har så mange og obskure feilkilder at vi må sette av egen tid kun for å etterprøve svaret vi får. Vi hører ofte at nevrale nettverk er "black box" systemer. Vel, agenter må i så fall være "black hole" systemer. Her forsvinner alt av explainability inn i et sluk av stokastisitet og divergens på en måte som vil gjøre det komplett umulig å forstå systemene, både når de oppfører seg riktig og når de oppfører seg feil. En av de mest lovende aspektene ved automatisering med datamaskiner var at de var deterministiske. Gitt samme input til et dataprogram fikk man alltid det samme svaret tilbake. I den retningen vi beveger oss i dag, med agenter og det som verre er, mister vi alt dette, og til hvilken pris? At vi brenner opp kloden i samme slengen?

## Endringslogg
Under følger en endringslogg som viser hvilke deler av denne posten som er endret til hvilket tid. Enkle skrivefeil og den slags ting vil ikke bli logget, men jeg vil etterstrebe å logge alle meningsfulle endringer i posten.

- 2025-04-05: Endringslogg
