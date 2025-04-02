---
title: Er resonnementer løsningen?
date: 2025-04-02 06:58:00 +0200
categories: [Språkmodeller og deres utfordringer]
tags: [llm]     # TAG names should always be lowercase
math: true
description: I denne posten vil vi se på hva som skiller modeller som kan resonnere fra andre språkmodeller og hvordan det kan påvirke resultatene vi får.
media_subpath: /assets/images/2025-04-02/
---
Denne posten er en del av en lengre serie poster om språkmodeller og problemer med disse. Jeg vil starte med å presisere at språkmodeller helt klart har sine bruksområder, men ettersom denne serien ser på problemene med språkmodeller kommer jeg til å fokusere på ulemper og problemer forbundet med hvordan språkmodeller fungerer i dag.

## Hva er en resonnerende språkmodell?
En resonnerende språkmodell er en språkmodell som er trent spesifikt for å generere "tankerekker" før den svarer på et spørsmål eller utfører en oppgave. Slike tankerekker er egentlig bare modellen som snakker til seg selv, men de tjener et spesielt formål. De skal øke det vi kaller test time compute. Så hva er test time compute?

Test time compute er hvor mye regnekraft som brukes når modellen svarer på våre spørsmål, etter den er ferdigtrent. Og her må vi først gjøre en liten digresjon. For hva mener jeg når jeg skiller mellom trening og testing av en modell?

Jo, i maskinlæring og statistikk skiller vi på trening og testing/validering. Under trening kan modellen oppdatere sine interne parametre. For en lineær regressjonsmodell betyr dette å oppdatere $\beta$ parameterne. For en språkmodell betyr et å oppdatere vektene, eller tallene i de lineære matrisene, i modellen. Avhengig av typen modell oppdateres disse forskjellig. Under testing har fryser vi imidlertid modellen. Altså skjer det ikke lenger noen oppdatering av modellens parametre. Det eneste som påvirker hva som kommer ut av modellen, er dataen vi putter inn. Så i en språkmodell betyr dette at modellen ikke endrer seg over tid mens vi bruker den. Det er kun kontekstvinduet (<https://enklypesalt.com/posts/Hvordan-opplever-kien-en-samtale/#kontekstvinduet>) som endrer seg. En språkmodell lærer derfor ingenting når vi snakker med den. Grunnen til at den kan svare på spørsmål med informasjon vi tidligere har gitt den, er at denne informasjonen fortsatt er i kontekstvinduet. Så fort vi fyller kontekstvinduet med nok tekst til at den relevante informasjonen ikke lenger er en del av dette vinduet, mister også modellen evnen til å svare på det spørsmålet med den informasjonen, fordi informasjonen ikke lenger er tilgjengelig modellen.

Når vi nå snakker om test time compute, snakker vi om å øke mengden regnekraft som brukes når vi snakker med modellen, ikke når vi trener den. Så hvordan gjør man det? Vel, det finnes flere måter å gjøre dette på. Den kanskje enkleste måten er å generere flere svar til det samme spørsmålet. Vi kan deretter velge det svaret som går oftest igjen. Denne teknikken kalles ofte Majority Voting. En annen måte er å be modellen bruke en stegvis tilnærming, der den først skal legge en plan for hvordan den skal svare på spørsmålet. Vi kan så be den evaluere planen og eventuelt forbedre den før vi manuelt mater modellen med den endelige planen og ber den utføre den. Dette kaller vi Chain of Thought (CoT) resonnering. Resultatet er at modellen genererer flere tokens og argumentasjonen er ofte at flere tokens gir bedre utgangspunkt for å ende opp med riktig svar. Men stemmer det?

Flere tokens er ikke i seg selv noen løsning. En språkmodell som plaprer i vei har ikke større sjanse for å generere et korrekt svar enn en som er kortfattet. Vi har jo tidligere også sett at sjansen for å generere et korrekt svar reduseres med antallet ord som genereres (<https://enklypesalt.com/posts/Hvordan-svarer-kien/#divergens-divergens-divergens>). Hovedforbedringen en slik tilnærming gir er at den gjør at modellen ikke lengre er bundet av sin autoregressive natur. Som vi så i (<https://enklypesalt.com/posts/Hvordan-svarer-kien/#autoregressiv-spr%C3%A5kmodellering>) har ikke dagens språkmodeller muligheten til å rette opp i feil i en tekst underveis mens den genererer den. Gjør den en feil, blir den med videre inn i genereringen av neste ord. Dette gjør at feil forplanter seg og driver modellen i en uheldig retning med svaret istt. Med CoT og iterativ forbedring gir vi modellen muligheten til å rette opp feil i sine tidligere svar. Når vi først ber modellen generere en plan og deretter ber den iterativt forbedre planen, kan den oppdage egne feil og rette dem opp. Det er dette vi kaller "resonnement" i språkmodeller.

I majority voting tilfellet er det en statistisk sannhet at gjennomsnitssvaret vil tendere mot å oftere være riktig. Når vi derfor velger det svaret som går igjen oftest, velger vi det statistisk beste svaret. Dette fordrer selvsagt at modellen ikke har systematiske biaser rundt spørsmålet vi stiller. Man har sett at at majority voting resulterer i en drastisk forbedring av modellens svar[^orpo]. Dette underbygger forøvrig også divergensproblematikken jeg presenterte i (<https://enklypesalt.com/posts/Hvordan-svarer-kien/#divergens-divergens-divergens>). Hvis divergens ikke eksisterte, ville man ikke sett en slik forbedring kun av å generere svaret flere ganger og bruke majority voting. I resten av denne posten skal vi imidlertid se på modeller som "resonnerer" og ikke på majority voting.

### Hvordan trener vi en resonnerende språkmodell?
Det finnes selvfølgelig mange måter å gjøre dette på, men en modell som fikk mye oppmerksomhet i 2024 var DeepSeek-r1 [^deepseek]. Her startet man med en modell som allerede var pre-trained. Man samlet deretter inn et stort antall eksempler med spørsmål og svar som inneholdt chain of thought resonnering og trente modellen til å svare på samme måte. Disse CoT eksemplene inneholder også eksempler hvor det oppdages at man er på feil spor og retter det opp igjen. Til slutt brukte de Reinforcement Learningsteknikker[^ppo][^grpo] for å oppnå eksepsjonelt gode resultater, med lavere compute ressurser en sine konkurrenter.


## Fungerer det?
Det enkle svaret er ja. Svarene vi får fra pråkmodeller blir betraktelig bedre av å inkludere resonnementer. Ettersom vi frigjør modellen fra sin iboende autoregressive natur får vi svar som kan rette sine egne feil og komme på rett kurs. Men er det en god løsning?

Svaret på det er mer nyansert. Resonnering er jo i stor grad en løsning på et problem vi har skapt for oss selv gjennom måten vi i utgangspunktet bygget den underliggende språkmodellen. I tillegg introduserer det en drastisk økning i kostnader både i kroner og øre, samt i energikonsupsjon ettersom modellen nå bruker mange flere tokens på å komme frem til sitt endelige svar.

Det er rett og slett et ganske uelegant plaster på et grunnleggende problem vi selv har bygget inn i språkmodellene våre. Det er jo fundamentet det er noe galt med. Burde vi ikke heller forsøke å fikse det enn å bygge masse rekkverk og støttehjul rundt det vi bygger oppå? Resonnement er jo på ingen måte en garanti for at man får riktig svar til slutt heller. Så vi sliter fortsatt med mye av den samme problematikken resonnemenmodeller prøver å løse, bare i en mindre grad.

Det kan vel kanskje sies at de eneste som garantert tjener på resonnementmodeller er tilbyderne av modellene. Ettersom kunder betaler for antallet tokens generert, er jo en resonnementmodell en gullgruve for de som tilbyr dem.

## Referanser
[^orpo]: Hong, J., Lee, N., & Thorne, J. (2024). Orpo: Monolithic preference optimization without reference model. arXiv preprint arXiv:2403.07691.
[^ppo]: Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal policy optimization algorithms. arXiv preprint arXiv:1707.06347.
[^grpo]: Shao, Z., Wang, P., Zhu, Q., Xu, R., Song, J., Bi, X., ... & Guo, D. (2024). Deepseekmath: Pushing the limits of mathematical reasoning in open language models. arXiv preprint arXiv:2402.03300.
[^deepseek]: Guo, D., Yang, D., Zhang, H., Song, J., Zhang, R., Xu, R., ... & He, Y. (2025). Deepseek-r1: Incentivizing reasoning capability in llms via reinforcement learning. arXiv preprint arXiv:2501.12948.
