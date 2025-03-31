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

Tidligere i denne serien har vi sett på hvordan språkmodeller kan lese tekst. I denne posten skal vi se nærmere på hvordan de kan gjøre det motsatte. Nemlig hvordan de genererer tekst.

Som vi har sett tidligere, opererer ikke språkmodeller på tekst direkte, men på tokens. Det er nemlig i token universet en språkmodel lever. Så når en språkmodel skriver tekst, er det ikke egentlig tekst den skriver, men tokens. Vi bruker bare den samme teknikken vi brukte for å tokenize vår tekst i utgangspunktet, til å reversere prosessen for å konvertere språkmodellens tokens til tekst vi kan lese. Men hvordan vet språkmodellen hvilke tokens den skal skrive?

For å svare på det, må vi se nærmere på hvordan man trener språkmodeller. Dagens språkmodeller trenes vanligvis i tre steg.
- Pre-training
- Fine-tuning
- Reinforcement Learning from Human Feedback

### Pre-training
La oss starte med pre-training. I dette steget er målet å skape en språkmodell som har god generell forståelse for språk. Vi ønsker å bygge en model, som skjønner både semantikk og syntaks i språket den trenes i. Det er forstatt usikkert om dagens modeller faktisk har god forståelse av semantikken i språket (det er nok på et mer metafysisk plan), men syntaksen har man i dag i stor grad mestret. Så hvordan bygger vi slike pre-trained, eller _foundation_ modeller i dag?

#### Next token prediction
Så og si alle språkmodeller som brukes i dag, pre-traines med en teknikk som kalles next token prediction. I denne settingen er målet for modellen å forutsi hva neste token i en setning er, gitt de foregående tokenene. For enkelhets skyld, vil jeg i den resterende teksten bruke tokens og ord om hverandre. Dette er en forenkling med minimal betydning i denne sammenhengen, men se mitt forrige blogpost for en forklaring på hvorfor dette ikke kan gjøres generelt.

Så språkmodellen kan da for eksempel få setningen "Det er bedre med en fugl i hånden enn ti på", hvor oppgaven er å forutsi hva neste ord vil være. I dette tilfellet "taket". Modellen mates med millioner av slike setninger, hentet fra hele internett, og den blir hele tiden bedt om å forutsi neste ord. Tanken bak dette er at hvis en modell etter hvert blir god på å forutsi det neste ordet, så betyr det også at den har lært fundamentale deler av et språk. For eksempel, hvis modellen får setningen: "Hunden drakk vann. Den som drakk vann var", så kreves det at modellen forstår at å drikke er verbet i setningen, at hunden er subjektet og at vann er objektet i setningen. Deretter må den bruke denne informasjonen til å vite at det manglende ordet i neste setning er "hunden". God prestasjon i next token prediction krever altså at modellen lærer seg språk.

Okay, det gir jo mening, men hvordan velger den ordet? Jo, språkmodellen har et begrenset vokabular. For hvert neste ord den skal forutsi må den velge i dette vokabularet. Måten den gjør det på er å lage en sannsynlighetsfordeling over alle ordene i vokabularet sitt. Du kan se for deg at modellen lager et stolpediagram, der hver stolpe korresponderer med ett av ordene i modellens vokabular. Høyden på alle stolpene skal summere til 100% (eller 1.0 om du vil). Det ordet med høyeste prosentverdi er det ordet som blir valgt (dette er forøvrig også en sannhet med modifikasjoner vi skal komme tilbake til). Når modellen trenes med next token prediction, lærer den egentlig hvordan den skal allokere prosentandeler til hvert ord i vokabularet sitt.

#### Autoregressiv språkmodellering
Vi har nå fått en forståelse for hvordan modellen velger neste ord, men hvordan får den til å bygge hele setninger eller enda lengre svar? For å få til det gjør vi et ganske enkelt, men veldig effektivt grep. For hvert nye ord som blir generert av modellen, så limer vi det bare på enden av setningen vi allerede har og mater denne nye setningen inn i språkmodellen for å få den til å forutsi neste ord der igjen. Vi gjenntar dette helt til modellen velger et spesielt "avslutt setning" ord. På den måten kan modellen generere lange avhandlinger med intern konsistens og mening.

#### Fine-tuning
Etter vi har gjort next token prediction trening med millioner av setninger har vi forhåpentligvis fått en god model for språk, eller språkmodell som de er mer kjent som. Denne modellen er imidlertid ikke spesielt god til å utføre instruksjoner, som å svare på spørsmål. De er heller ikke høfflige og imøtekommende slik vi gjerne ønsker av en slik modell. For å oppnå dette utfører vi fine-tuning trening, dette steget kalles også instruction tuning noen ganger. Her trener vi modellen på kuraterte eksempler hvor vi demonstrerer spørsmål og svar i noen tusen til millioner eksempler. Vi trener også på lengre setninger og ikke bare neste ord i dette steget. Effekten av dette er at modellens forståelse for hva som er mest sannsynlige neste ord, skiftes mot et spørsmål-svar paradigme, samt mot et mer høfflig og imøtekommende valg av ord.

#### Reinforcement Learning from Human Feedback (RLHF)
Etter fine-tuning steget har vi en språkmodell som ikke bare har god forståelse for språk, men også er i stand til å svare på spørsmål og er imøtekommende og hyggelig. Vi mangler imidlertid en ting. Svar med god kvalitet. Svarene vi får fra modellen nå kan være veldig hyggelige og ofte også riktige, men de kan fremstå veldig kjedelige og de kan ha for høy andel uriktige svar enn det vi kan akseptere for en språkmodell som skal være hjelpsom for mennesker.

For å rette på dette bruker vi et tredje steg. Her lar vi språkmodellen generere mange alternative svar til det samme spørsmålet. Vi lar deretter et menneske velge hvilket svar som er best, gitt spørsmålet. Dette tar naturligvis veldig lang tid ettersom mennesket må lese og velge blant mange svar. Ettersom vi ønsker å lære modellen hvilke svar mennesket liker, kan vi ikke vente så lang tid det tar å generere et stort nok treningssett med denne metoden. Vi trener derfor en annen språkmodell til å etterligne valgene til mennesket, gitt de samme svarene. Når denne etterligningsmodellen er i stand til å etterligne menneskelige valg godt nok, kan vi istedet bruke denne til å velge mellom nye svar generert fra modellen. 

Tanken her er at etterligningsmodellen lærer seg hva mennesker liker i svar fra språkmodller og derfor kan gjøre tilsvarende valg på nye spørsmål og svar samlinger. Vi kan da bytte ut mennesket med etterligningsmodellen når vi trener den fine-tunede språkmodellen. Ettersom vi kan lage tusenvis av kopier av etterligningsmodellen kan vi nå få treningsprosessen til å gå fort nok til at vi kan trene en språkmodel som:
- har god forståelse av språk (pre-training)
- er høfflig og svarer på spørsmål (fine-tuning)
- svarer på en måte som stemmer godt med menneskelige mål og ønsker (RLHF, alignment)

Etter RLHF steget har vi faktisk fått den modellen som ligger bak alle ikkeresonerende språkmodeller som ChatGPT, Mistral og Claude. For resonerende modeller endrer man på RLHF steget eller legger til et nytt steg, som også lærer modellen å "resonere" før den gir sitt endelige svar.

### Vi velger ikke alltid det mest sannsynlige ordet
I forrige avsnitt sa jeg at det at vi velger det ordet med høyest prosentverdi, er en sannhet med modifikasjoner. La oss se nærmere på dette. Et problem med å velge det mest sannsynlige ordet hver gang er at vi ofte ender opp med tekst med lav kvalitet og gjenntakende tekst. Du kan teste dette ved å sette temperaturen til en språkmodell til 0.0. For å omgå dette gjør man i dag et probabilistisk utvalg blant det mest sannsynlige ordene. En av de mest vanlige metodene for dete kalles temperature sampling.

Vanligvis beregnes sannsynligheten for hvert ord $$x_i$$ ved hjelp av en softmax operasjon: $$\text{softmax}(x_i) = \frac{e^{x_i}}{\sum_j=1^ne^{x_j}}$$. Vi tvinger altså summen av alle sannsynlighetene til å bli 1.0.

I temperature sampling legger vi til en ekstra parameter $$\tau$$ i denne ligningen, slik at vi får: $$\text{softmax}(x_i) = \frac{e^{x_i/\tau}}{\sum_j=1^ne^{x_j/\tau}}$$

Effekten av dette er at vi har større sannsynlighet for å velge et annet ord enn det mest sannsynlige. Dette fører til at vi får mer diverse og interessante svar fra modellen ettersom vi introduserer enkelte mindre sannsynlige ord i setningene som genereres. Ettesom setningene blir matet inn i modellen for å forutsi neste ord igjen fører dette til at retningen setningen beveger seg i forandres litt hver gang vi genererer en setning. Dette forklarer hvorfor en språkmodell sjeldent gir ord for ord, samme svar på et identisk spørsmål stilt i to nye chatter.

## Man kan ikke stole på språkmodellene
