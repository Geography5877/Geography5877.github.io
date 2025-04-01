---
title: Hvordan svarer KIen?
date: 2025-03-31 06:57:00 +0200
categories: [Språkmodeller og deres utfordringer]
tags: [llm]     # TAG names should always be lowercase
math: true
description: I denne posten vil vi se på hvordan en språkmodel er i stand til å skrive tekst
media_subpath: /assets/images/2025-03-31/
---
Denne posten er en del av en lengre serie poster om språkmodeller og problemer med disse. Jeg vil starte med å presisere at språkmodeller helt klart har sine bruksområder, men ettersom denne serien ser på problemene med språkmodeller kommer jeg til å fokusere på ulemper og problemer forbundet med hvordan språkmodeller fungerer i dag.

## Hvordan kan en model skrive?
Tidligere i denne serien har vi sett på hvordan språkmodeller kan lese tekst. I denne posten skal vi se nærmere på hvordan de kan gjøre det motsatte. Nemlig hvordan de genererer tekst.

Som vi har sett tidligere, opererer ikke språkmodeller på tekst direkte, men på tokens. Det er nemlig i token universet en språkmodel lever. Så når en språkmodel skriver tekst, er det ikke egentlig tekst den skriver, men tokens. Vi bruker bare den samme teknikken vi brukte for å tokenize vår tekst i utgangspunktet, til å reversere prosessen for å konvertere språkmodellens tokens til tekst vi kan lese. Men hvordan vet språkmodellen hvilke tokens den skal skrive?

For å svare på det, må vi se nærmere på hvordan man trener språkmodeller. Dagens språkmodeller trenes vanligvis i tre steg.
- Pre-training
- Fine-tuning
- Reinforcement Learning from Human Feedback

### Pre-training
La oss starte med pre-training. I dette steget er målet å skape en språkmodell som har god generell forståelse for språk. Vi ønsker å bygge en modell, som skjønner både semantikk og syntaks i språket den trenes i. Det er fortsatt usikkert om dagens modeller faktisk har god forståelse av semantikken i språket (det er nok på et mer metafysisk plan), men syntaksen har man i dag i stor grad mestret. Så hvordan bygger vi slike pre-trained, eller _foundation_ modeller i dag?

#### Next token prediction
Så og si alle språkmodeller som brukes i dag, pre-traines med en teknikk som kalles next token prediction. I denne settingen er målet for modellen å forutsi hva neste token i en setning, eller sekvens med tekst skal være, gitt de foregående tokenene. For enkelhets skyld, vil jeg i den resterende teksten bruke ord i stedet for tokens. Dette er en forenkling med minimal betydning i denne sammenhengen, men se min forrige blogpost (<https://enklypesalt.com/posts/Hvordan-opplever-kien-en-samtale/#tokenization>) for en forklaring på hvorfor dette ikke gjelder generelt.

Så språkmodellen kan da for eksempel få setningen "Det er bedre med en fugl i hånden enn ti på", hvor oppgaven er å forutsi hva neste ord skal være. I dette tilfellet "taket". Modellen mates med millioner av slike tekstsekvenser, hentet fra hele internett, og den blir hele tiden bedt om å forutsi neste ord. Tanken bak dette er at hvis en modell etter hvert blir god på å forutsi det neste ordet, så betyr det også at den har lært fundamentale deler av et språk. For eksempel, hvis modellen får sekvensen: "Hunden drakk vann. Den som drakk vann var", så kreves det at modellen forstår at å drikke er verbet, at hunden er subjektet og at vann er objektet. Deretter må den bruke denne informasjonen for å "skjønne" at det manglende ordet i neste setning er "hunden". God prestasjon i next token prediction krever altså at modellen til en hvis grad lærer språk.

Okay, greit. Men hvordan velger den hvilket ord som skal være det neste ordet? Jo, språkmodellen er utstyrt med et vokabular fra starten av. Dette vokabularet endrer seg ikke i løpet av treningen. I realiteten består vokabularet av tokens, og det vil derfor være mulig for modellen å bygge ord bestående av flere tokens, men dette er en digresjon. For hvert neste ord den skal forutsi må den velge blant ordene i dette vokabularet. Måten den gjør det på er å lage en sannsynlighetsfordeling over alle ordene i vokabularet sitt. Du kan se for deg at modellen lager et stolpediagram, der hver stolpe korresponderer med ett av ordene i modellens vokabular. Høyden på alle stolpene skal summere til 100% (eller 1.0 om du vil). Det ordet med høyest prosentverdi er det ordet som blir valgt (dette er forøvrig også en sannhet med modifikasjoner vi skal komme tilbake til). Når modellen trenes med next token prediction, lærer den egentlig hvordan den skal allokere prosentandeler til hvert ord i vokabularet sitt.

Altså lærer egentlig en språkmodell en sannsynlighetsfordeling over ord i språket, gitt de foregående ordene i en sekvens, når vi gjør next token prediction. Sannsynligheten blir informert av modellens forståelse for språket generelt og innholdet i kontekstvinduet (<https://enklypesalt.com/posts/Hvordan-opplever-kien-en-samtale/#kontekstvinduet>). Så målet med next token prediction er egentlig at modellen skal lære seg riktig sannsynlighetsfordeling over vokabularet sitt, gitt kontekstvinduet. Dette er naturligvis en utrolig vanskelig oppgave, så for å til dette må vi bygge store modeller som trenes på milliarder av ord, men til slutt ender vi opp med en modell som faktisk er nokså god på denne oppgaven.

#### Autoregressiv språkmodellering
Okay, så modellen kan forutsi neste ord i en sekvens, men hvordan kan vi gå derfra til å bygge hele setninger eller enda lengre svar? Jo, for å få til det gjør vi et ganske enkelt, men veldig effektivt grep. For hvert nye ord som blir generert av modellen, så limer vi det bare på enden av sekvensen vi matet inn i modellen i forrige steg. Da får vi en ny sekvens, med ett nytt ord. Vi mater så denne nye sekvensen inn i språkmodellen for å få den til å forutsi neste ord der igjen. Vi gjentar dette helt til modellen velger et spesielt "avslutt tekst" ord som det mest sannsynlige ordet. På den måten kan modellen generere lange avhandlinger med intern konsistens og mening.

Det fine med dette er at prosessen med å lage arbitrært lange sekvenser med ord er svært enkel og styres av modellen. Men det er også en ganske stor ulempe. Denne prosessen krever at hele modellen brukes for hvert eneste ord som skal genereres. Som jeg nevnte tidligere, kreves det store modeller for å få en god forståelse av språk. Store modeller krever også mye energi ettersom det er mye regnekraft som må til for å beregne alle operasjonene som skjer inne i modellen. At vi nå krever at hele modellen må kjøres på hele sekvensen med ord, hver gang vi skal generere et nytt ord, betyr at språkmodeller i dag er enorme energisluk. Heldigvis forskes det på hvordan vi kan bruke teknikker som speculative decoding [^spec-dec] for å kunne generere flere ord på en gang, men vi er fortsatt langt unna å kunne kalle språkmodeller effektive, når det kommer til behov for regnekraft og energi.

### Fine-tuning
Etter vi har gjort next token prediction trening med milliarder av ord har vi forhåpentligvis fått en god modell for språk, eller språkmodell som de er mer kjent som. Denne modellen er imidlertid ikke spesielt god til å følge instruksjoner, som å svare på spørsmål eller utføre oppgaver som å lage en oppsummering av tekst. De er heller ikke så høflige og imøtekommende som vi gjerne ønsker at en slik modell skal være. For å oppnå dette utfører vi fine-tuning eller instruction tuning. I dette steget trener vi modellen på kuraterte eksempler som demonstrerer spørsmål og svar og oppgaveløsning i noen tusen til millioner eksempler. Vi trener også på lengre setninger og ikke bare neste ord i dette steget. Effekten av dette er at modellens forståelse for hva som er mest sannsynlige neste ord, skiftes mot et spørsmål-svar- og oppgaveløsningsparadigme, samt mot et mer høflig og imøtekommende valg av ord. Fine-tuning gir oss rett og slett en likandes modell som villig hjelper oss med alt vi lurer på. 

### Reinforcement Learning from Human Feedback (RLHF)
Etter fine-tuning steget har vi en språkmodell som ikke bare har god forståelse for språk, men også er i stand til å svare på spørsmål og utføre oppgaver samtidig som den også er imøtekommende og hyggelig. Vi mangler imidlertid en ting. Svar med god kvalitet. Svarene vi får fra modellen nå kan være veldig hyggelige og ofte også riktige, men de kan fremstå veldig kjedelige, være veldig lange og fortsatt ha for høy andel uriktige svar.

For å rette på dette bruker vi et tredje steg. Her lar vi språkmodellen generere mange alternative svar til det samme spørsmålet. Vi lar deretter et menneske rangere alle svarene. Vi kan da bruke trene modellen slik at den genererer svar som ender høyt i slike rangeringer. Dette tar naturligvis veldig lang tid når vi skal lage mange slike treningseksmepler, ettersom mennesket må lese og rangere mange svar. For å få dette til å gå fortere kan vi isteden trene en annen språkmodell til å etterligne valgene til mennesket. Når denne etterligningsmodellen er i stand til å etterligne menneskelige valg godt nok, kan vi så bruke denne til å rangere svar generert fra den første fine-tunede språkmodellen. 

Tanken her er at etterligningsmodellen lærer seg hva mennesker liker i svar fra språkmodeller og derfor kan gjøre tilsvarende rangeringer på nye spørsmål og svar samlinger. Vi kan da bytte ut mennesket med etterligningsmodellen når vi trener den fine-tunede språkmodellen. Ettersom vi kan lage tusenvis av kopier av etterligningsmodellen kan vi nå få treningsprosessen til å gå fort nok til at vi kan trene språkmodellen vår til å generere svar som mennesker liker og verdsetter. Dette kaller vi ofte alignment.

Vi har nå en modell som:
- har god forståelse av språk (pre-training)
- er høflig og svarer på spørsmål (fine-tuning)
- svarer på en måte som stemmer godt med menneskelige mål og ønsker (RLHF, alignment)

Etter RLHF steget har vi faktisk fått den modellen som ligger bak alle ikke-resonerende språkmodeller som ChatGPT, Mistral og Claude. Det som i størst grad skiller disse modellene fra hverandre er hvilke data de er trent på, hvor lenge de er trent og enkelte forskjeller i den underliggende modellarkitekturen. Men i grove trekk kan vi si at dette er oppskriften på å lage en en god språkmodell i dag.

### Vi velger ikke alltid det mest sannsynlige ordet
Tidligere sa jeg at det at modellen velger det ordet med høyest prosentverdi når den forutsier neste ord i en sekvens. Det er en sannhet med modifikasjoner. La oss se nærmere på dette. Et problem med å velge det mest sannsynlige ordet hver gang, er at vi ofte ender opp med svar av lav kvalitet og gjenntakende tekst. Du kan teste dette ved å sette temperaturen til en språkmodell til 0.0. For å omgå dette gjør man i dag et probabilistisk utvalg blant det mest sannsynlige ordene. En av de mest vanlige metodene for dette kalles temperature sampling [^temp-sampl].

Vanligvis beregnes sannsynligheten for hvert ord $$x_i$$ ved hjelp av en softmax operasjon: \eqref{eq:softmax}. Vi tvinger altså summen av alle sannsynlighetene til å bli 1.0.

$$
\begin{equation}
\text{softmax}(x_i) = \frac{e^{x_i}}{\sum^n_{j=1}e^{x_j}}
\label{eq:softmax}
\end{equation}
$$

I temperature sampling legger vi til en ekstra parameter $\tau$ i denne ligningen, slik at vi får ligningen \eqref{eq:temperature-sampling}.
$$
\begin{equation}
\text{softmax}(x_i) = \frac{e^{x_i/\tau}}{\sum^n_{j=1}e^{x_j/\tau}}
\label{eq:temperature-sampling}
\end{equation}
$$

Ved å sette $\tau > 1.0$ får vi en "mykere" sannsynlighetsfordeling, som igjen gir større sannsynlighet for å velge et annet ord enn det mest sannsynlige. Dette fører til at vi får mer diverse og interessante svar fra modellen ettersom vi introduserer enkelte mindre sannsynlige ord i setningene som genereres. Siden vi gjør autoregressiv generering av tekst fører dette til at retningen setningen beveger seg i forandres litt hver gang vi genererer et nytt ord. Dette forklarer hvorfor en språkmodell sjeldent gir ord for ord, samme svar på et identisk spørsmål.

### Divergens, divergens, divergens
Dette er imidlertid også en av forklaringene på hvorfor en språkmodell kan rote seg bort i helt feil svar en gang man stiller et spørsmål, men gi riktig svar, hvis man starter en ny samtale og stiller samme spørsmål. Siden vi noen ganger velger et annet ord enn det mest sannsynlige ordet, sett fra språkmodellens perspektiv, kan vi endre hele betydningen av det modellen "prøver" å svare. Vi kan se på det slik:
1. For hvert nye ord vi skal forutsi, velger vi det riktige ordet med en sannsynlighet $$S$$ og et annet ord med sannsynlighet $$1-S$$. $$S$$ styres av vår spesifikke metode for valg av ord fra modellens sannsynlighetsfordeling og hvor god denne sannsynlighetsfordelingen er.
2. Hver gang vi velger et annet ord enn det riktige ordet, endrer vi "retningen" en setning tar i større eller mindre grad bort fra den riktige setningen.
3. Ettersom vi mater den nye setningen inn i språkmodellen for at den skal forutsi neste ord igjen, vil endringen i retning fra steg 2. også påvirke hvordan sannsynlighetsfordelingen for dette ordet vil se ut. Altså er dette en feedback loop.
4. Ettersom modellen er autoregressiv, så har kan den ikke gå tilbake og rette opp en tidligere feil. Den er rett og slett tvunget til å fortsette å generere ord basert på den setningen den allerede har skrevet. 

Har vi vært uheldig med tidligere ord i setningen vår, vil vi kunne få veldig rare svar i et forsøk, men helt rimelige svar i det neste. Denne effekten, der språkmodellen kan gi vidt forskjellige svar, gitt nøyaktig samme utgangspunkt kaller vi _divergens_. 

Hva er sannsynligheten for at et svar med lengde $n$ er korrekt, om vi antar at sannsynligheten for å velge et feil ord er uavhengig? Hvis vi igjen bruker sannsynligheten for feil ord = $$1-S$$, kan vi uttrykke dette i \eqref{eq:prob-wrong} [^lecun].

$$
\begin{equation}
P(\text{riktig setning}) = (1-S)^n
\label{eq:prob-wrong}
\end{equation}
$$

Hvis vi skal generere en tekst med 100 ord og har en sannsynlighet for å velge riktig ord på 99.999% får vi følgende: $$P(\text{riktig setning}) = (1.0-\frac{99.999\%}{100.0\%})^{100} \approx 0.0\%$$. Altså er det tilnærmet 0.0% sannsynlighet for at vi generer den helt riktige teksten, selv med en ekstremt høy sannsynlighet for å velge riktig ord i hvert forsøk. Her skal det sies, at det vi anser som riktig tekst er en ord-for-ord riktig tekst. Om vi løsner opp kravet litt og kun krever at teksten skal ha samme betydning som den vi mener er riktig, er sannsynligheten for at modellen klarer det mye større.

Men husk at modellen vår er i en feedback loop. Det betyr at sannsynligheten for å velge feil ord ikke er uavhengig i hvert steg, slik vi la til grunn for beregningen over. Sannsynlighetsfordelingen for ord påvirkes av de tidligere ordene som ble generert, siden disse nå er en del av teksten som brukes for å generere neste ord. Så om vi velger feil ord et sted i teksten, vil neste ord ha større sannsynlighet for feil enn om vi ikke hadde valgt feil ord. Ordet etter der vil ha enda større sannsynlighet osv. Dette fører til at vi, selv med vårt løsnede krav for hva vi anser som en riktig setning, vil oppleve at sannsynligheten for at teksten vår blir feil øker eksponentielt med lengden av teksten.

Dette er et stort problem som stammer helt tilbake til hvordan vi trente modellen i pre-training steget. Husk at i pre-training steget trente vi modellen til å forutsi neste ord. Men hvordan vet vi, som trener modellen, hva det mest sannsynlige ordet er? Ettersom vi kjenner hele setningen, så sier vi bare at det mest sannsynlige neste ordet er det ordet som faktisk står som neste ord i setningen. Og ikke nok med det. Vi sier også at dette ordet har 100% sannsynlighet, og at ingen andre ord er sannsynlige i et hele tatt. Det er jo en veldig merkelig tilnærming. La oss si at vi bruker setningen: "Jeg gikk en tur på stien." som et treningseksempel til modellen. Vi fjerner ordet "stien" og modellens oppgave er nå å forutsi det mest sannsynlige ordet, gitt "Jeg gikk en tur på" som kontekst. Er det rimelig å si at "stien" er det eneste mulige alternativet her? Er ikke "veien" eller "fortauet" eller en hel del andre ord også ord som burde få en viss sannsynlighetsprosent? Ved å trene modellen til kun å tro at "stien" er det eneste riktige ordet, lærer den ikke en reell sannsynlighetsfordeling over alle ordene i vokabularet sitt, men en svært "biased" fordeling som, i dette tilfellet, prioriterer norske sanger alt for høyt.

Ettersom alle dagens språkmodeller er trent på denne måten, lider også alle av slike biaser. Vi har heller enda ingen god måte å løse det på. Foreløpig er det eneste man gjør å trene på ekstremt mye tekst, og satse på at vi får så mange forskjellige eksempler at modellen lærer seg noe som ligner på en rimelig sannsynlighetsfordeling. Personlig, tror jeg faktisk ikke vi vil være i stand til å løse det. Jeg tror rett og slett vi fundamentalt må endre måten vi trener modellene våre på, fra pre-training og hele veien opp.

## Referanser
[^spec-dec]: Leviathan, Y., Kalman, M., & Matias, Y. (2023, July). Fast inference from transformers via speculative decoding. In International Conference on Machine Learning (pp. 19274-19286). PMLR.
[^temp-sampl]: Hinton, G., Vinyals, O., & Dean, J. (2015). Distilling the knowledge in a neural network. arXiv preprint arXiv:1503.02531.
[^lecun]: Yann LeCun: <https://www.youtube.com/watch?v=xnFmnU0Pp-8>
