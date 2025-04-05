---
title: Hvorfor hallusinerer språkmodeller?
date: 2025-04-01 06:57:00 +0200
categories: [Språkmodeller og deres utfordringer]
tags: [llm, hallicunation]     # TAG names should always be lowercase
math: true
description: I denne posten vil vi se på hvorfor språkmodeller hallusinerer og hvorfor det er lite vi kan gjøre med det
media_subpath: /assets/images/2025-04-01/
---

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Denne posten er en del av en lengre serie poster om språkmodeller og problemer med disse. Jeg vil starte med å presisere at språkmodeller helt klart har sine bruksområder, men ettersom denne serien ser på problemene med språkmodeller kommer jeg til å fokusere på ulemper og problemer forbundet med hvordan språkmodeller fungerer i dag.
{: .prompt-info }
<!-- markdownlint-restore -->

## Hva er hallusinasjoner?
Vi hører ofte at et av de største problemene med dagens språkmodeller er at de hallisunerer når det genererer svar på våre spørsmål. For de fleste er hallisunasjoner fra språkmodeller synonymt med at modellen gir feilaktige svar eller finner på nye "fakta". Men hva er et egentlig som skjer når en språkmodell hallisunerer?

Som vi har sett tidligere er språkmodeller komplekse modeller av språk. Noen egenskaper som går igjen er at:
- [de er autoregressive](https://enklypesalt.com/posts/Hvordan-svarer-kien/#autoregressiv-spr%C3%A5kmodellering)
- [de er ikke deterministiske når de velger tokens](https://enklypesalt.com/posts/Hvordan-svarer-kien/#vi-velger-ikke-alltid-det-mest-sannsynlige-ordet)
- [de har en tendens til å divergere](https://enklypesalt.com/posts/Hvordan-svarer-kien/#divergens-divergens-divergens)
- [de lærer ikke en reell sannsynlighetsdistribusjon over språket](https://enklypesalt.com/posts/Hvordan-svarer-kien/#divergens-divergens-divergens)

Når en språkmodell genererer tekst gjør den, helt grunnleggende, bare next token prediction. Den prøver rett og slett å forutsi hva det neste mest sannsynlige ordet er, gitt kontekstvinduet sitt og treningsdataen den er trent på. Hvis vi ser bort ifra at vi introduserer noe ekstra stokastisitet gjennom ikke alltid å velge det mest sannsynlige tokenet, gjør modellen faktisk bare det. En språkmodell produserer alltid det mest sannsynlig neste tokenet, gitt sinn interne tilstand. Det som er interessant med det, er at modellen selv ikke skiller på hallisunasjoner og riktige svar. For en språkmodell er hallusinasjonen den mest sannsynlige teksten, gitt dens interne tilstand.

Hallusinasjon er altså en merkelapp vi legger på teksten fra en språkmodell i etterkant av at modellen har generert teksten. Uttrykket hallusinasjon er et litt uheldig uttrykk i så måte, ettersom det implisitt også indikerer at modellen kunne, og til og med burde, svart annerledes. Det er ikke tilfellet.

## Kan vi gjøre noe med hallusinasjoner?

Husk at språkmodeller aldri eksplisitt lærer seg å skille mellom [fakta og usannheter](https://enklypesalt.com/posts/Hvordan-svarer-kien/#l%C3%A6rer-modellen-hva-som-er-sant). Vi håper bare at den skal skjønne det ut ifra hvordan vi trener den. Med et slikt utgangspunkt er det ikke vanskelig å forstå at modellen kan introduserer større eller mindre avvik fra fakta fra tid til annen. Den har rett og slett aldri fått beskjed om at den skal være faktuell og den vet ikke nødvendigvis engang hva det vil si å være faktuell.

Hallusinasjoner stammer altså grunnleggende fra hvordan vi har trent språkmodellene våre. De er derfor ikke noe vi kan endre på uten å endre på hvordan vi trener modellene våre fra grunnen opp. Enn så lenge, er hallusinasjoner noe vi bare må leve med og ta med i betraktningen i alle applikasjoner hvor vi ønsker å bruke en språkmodell.

## Endringslogg
Under følger en endringslogg som visre hvilke deler av denne posten som er endret til hvilket tid. Enkle skrivefeil og den slags ting vil ikke bli logget, men jeg vil etterstrebe å logge alle meningsfulle endringer i posten.

- 2025-04-05: Endringslogg
