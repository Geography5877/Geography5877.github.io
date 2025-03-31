---
title: Hvordan Opplever KI'en en samtale?
date: 2025-03-25 06:58:00 +0200
categories: [Språkmodeller og deres utfordringer]
tags: [llm]     # TAG names should always be lowercase
math: true
description: I denne posten vil vi se på hvordan en samtale ser ut fra en språkmodel's perspektiv.
media_subpath: /assets/images/2025-03-28/
---

Denne posten er en del av en lengre serie poster om språkmodeller og problemer med disse. Jeg vil starte med å presisere at språkmodeller helt klart har sine bruksområder, men ettersom denne serien ser på problemene med språkmodeller kommer jeg til å fokusere på ulemper og problemer forbundet med hvordan språkmodeller fungerer i dag.


# Hvordan kan en maskin lese?
La oss starte med et helt grunnleggende spørsmål. Når du skriver noe til en språkmodel, hvordan kan den "lese" hva du skriver? Språkmodellen har jo ikke øyne, så den kan ikke se hva du skriver på samme måte som vi mennesker gjør. Så hvordan er den i stand til å lese og forstå hva du har skrevet når den svarer deg?

## Tokenization
Svaret på dette spørsmålet er at modellen ikke kan lese. Ihvertfall ikke på den måten vi mennesker gjør det. En språkmodel er, kort forklart, en linær algebramaskin. Den maserer og transformerer tekst gjennom bruk av enkle lineære matriseopperasjoner. Men hvordan kan man gjøre matriseopperasjoner på ren tekst? Det kan vi ikke, så for å få det hele til å fungere må vi først transformere rå tekst til tall. Denne prossessen er det vi kaller "tokenization". Når vi skriver tekst til en språkmodel er det første som skjer at teksten vi skriver blir oversatt, eller "tokenized" til tall, slik at det er mulig for en maskin å gjøre matriseopperasjoner på den. Så hvordan ser slike token ut?

For å se det kan du besøke https://platform.openai.com/tokenizer. Her kan du skrive inn tekst og se hvordan teksten forandres til tokens. I bildet under ser vi et eksempel på dette. I tekstboksen har jeg skrevet teksten "Hei, hvor gammel er du?" og under får vi se hvilke deler av teksten som resulterer i egne tokens.

![text](OpenAI_Tokenizer_marked_tokens.png)
_Fargene på teksten indikerer hvilke deler av teksten som resulterer i separate tokens._

Vi kan også få se tokenene direkte:
![text](OpenAI_Tokenizer_text_to_vec.png)
_Her ser vi tokene som korresponderer med teksten vår._

Det er altså en slik sekvens med tokens en språkmodell "leser" og ikke vår tekst direkte. Disse tokene kan vi gjøre matriseoperasjoner på og språkmodellen er derfor i stand til å prossesere dem.

Akkurat hvordan tokens blir generert fra tekst varierer fra modell til modell, men så lenge vi er konsekvente i måten vi gjør det på for en gitt modell vil ikke det by på noen problemer.

## Tokenization fjerner mening
At man fant ut hvordan man kunne transformere tekst til tall var et stort gjennombrudd i maskinlæring, ettersom vi endelig var i stand til å gjøre effektiv maskinlæring på språk. Det er imidlertid også flere baksider ved tokenization. 

Slik vi gjør tokenization i dag, skjer dette på sub-ord. Dette betyr at enkelte ord deles opp etter satte regler for å tokeniseres. For eksempel vil ordet "tokenization" kunne deles opp slik, "token" og "ization", før det transformeres til tall. Dette er en god strategi, ettersom det gjør at vi slipper å lage egne tokens for "tokenization" og "ionization", men vi kan istedet gjennbruke tokenene "token", "ion" og "ization" for å sette sammen ordene til henholdsvis "tokenization" og "ionization". Det gjør at vi trenger betraktelig færre distinkte token, noe som gjør tokenization kjappere og mer effektivt!

### Matematikk blir umulig
Det betyr imidlertid også at tall kan bli veldig rare i en språkmodell. Tall må også transformeres til sine respektive tokens. Så tallet 2 leses ikke som "2" i en språkmodell, men tokenet som korresponderer til "2". Under ser vi et bilde av tokenization av regnestykket 2+2. Her blir merkelig nok tallet 2, som jo allerede er et tall, tokenisert til tokenet "17". Med andre ord ser ikke en språkmodell tall på samme måte som vi gjør. Den ser heller tokenet som representerer tallet.

![text](OpenAI_Tokenizer_2+2.png)
_Når regnestykket blir "tokenized" representeres plutselig tallet "2" med tokenet "17" og pluss får tokenet "10"._

Dette blir enda rarere hvis vi ser på større tall. I regnestykket "2048 + 2048", ser vi at tokeniseringen markerer deler av tallet 2048 som egne tokens. Videre blir også " + " delt opp i to token: " +" og " ". Det har altså betydning om vi inkluderer mellomrom mellom "+" og tallene.

![text](OpenAI_Tokenizer_2048+2048_marked.png)
_Regnestykket 2048 + 2048 får en ganske merkelig tokenisering._

Dette ser vi igjen når vi ser på tokenene som korresponderer med regnestykket. "14397" korresponderer med "204" og "23" korresponderer med "8". "659" korresponderer med " +" og "220" korresponderer med " ". 
![text](OpenAI_Tokenizer_2048+2048_vectorized.png)
_Regnestykket 2048 + 2048 får merkelige tokens._

Vi kan se et enda mer ekstremt eksempel hvis vi bruker desimaltall.
![text](OpenAI_Tokenizer_decimal.png)
_Desimaltall deles opp i flere deler._

Med andre ord, mister regnestykker mye av sin betydning når de tokeniseres. For å illustrere hvordan problemet ville sett ut for mennesker, kan vi kanskje se for oss at vi ikke lenger jobber med tall, men med kjente malerier. Hvis vi nå deler opp tall og regnestykker på samme måte som tokenizeren over og bytter ut hvert token med et kjent maleri, kan vi kanskje få en viss innsikt i hvor mye vannskeligere vi har gjort problemet for språkmodellen. Se for deg at 2048 deles opp i 204 = Mona Lisa og 8 = Skrik. Deretter har vi " +" = Pike med Perleøredobb, mens bare "+" = Stjernenatt og " " = Venus' fødsel osv. Legg merke til at også her er "+" og " +" to helt forskjellige malerier. Slik at 2048+2048 tilsvarer sekvensen, Mona Lisa, Skrik, Stjernenatt, Mona Lisa, Skrik. Mens 2048 + 2048 tilsvarer sekvensen, Mona Lisa, Skrik, Pike med Perleøredobb, Venus' fødsel, Mona Lisa, Skrik. Under kan vi se hvordan dette ser ut.

![text](math-to-paintings.png)
_Disse to sekvensene med malerier representerer det samme regnestykket. Det er ikke spesielt intuitivt eller lett å lære seg._

Man kunne jo tenke seg at dette ikke er det verste i verden. Modellen trenger jo bare å lære seg at Mona Lisa og Skrik i sekvens betyr 2048, men det blir verre. Hvis vi ser på 0.2048+0.2048, brukes fortsatt Mona Lisa og Skrik til å representere tallene bak desimaltegnet. Men nå må vi også lære oss at siden vi har en "0" og et desimaltegn forran, så betyr nå Mona Lisa og Skrik i sekvens at vi først må dele tallene på 10 000 for å få riktig verdi. Altså blir det å lære seg mattematikk delt opp i to problemer. Først må vi altså lære hva alle mulige kombinasjoner av malerier korresponderer til av tall, samt hvordan disse kombinasjonene endrer betydning basert på malerier forran. Deretter må vi lære oss hva operasjonene betyr og hvordan disse kan resultere i nye kombinasjoner av malerier.

Det er et veldig lignende problem en språkmodell står overfor når den skal lære seg matematikk, men med tokens i stedet for malerier. Ettersom en språkmodell kun har en begrenset "lagringsplass" i form av parametrene, eller vekter, er det umulig å lære seg alle kombinasjoner av tokens. Den får også bare sett et begrenset sett med mulige kombinasjoner av tokens mens den trenes, ettersom det vil ta alt for lang tid å trene på alle mulige kombinasjoner. Det er også slik at det ikke er forskjell på tokens som representerer tall og tokens som representerer tekst når vi gjør tokenization. For en språkmodel er derfor sekvenser med tokens som representerer et regnestykke helt likestilt med en sekvens med tokens som representerer ren tekst. Med disse begrensningene tatt i betraktning er det ikke vanskelig å forstå at det å lære seg matematikk kun fra tokens en umulig oppgave for en språkmodell. Det er derfor språkmodeller stort sett kun behersker små og enkle regnestykker som 2+2, men ikke 12836086120856028106+918263806205601283. Den har sett 2+2 så ofte under trening at den har lært seg at 4 følger naturlig, på samme måte som vi mennesker lærer oss at "Ta det med en klype", skal etterfølges av "salt". Den utfører altså ikke beregningen 2+2 når den svarer 4. Den har bare "pugget" at 4 skal følge 2+2.

### Tekst kan misforstås
Det er ikke kun i matematikken tokenization kan by på problemer. I de fleste språk har vi ord som staves likt, men har forskjellig betydning. Et norsk eksempel er "tre". Ordet tre kan representere tallet 3, et tre og det kan kanskje diskuteres om det konseptuelt også er en tredje ting vi tenker på når vi sier _tre_verk (jeg er ingen lingvist, så jeg skal ikke påstå noe her).

I en språkmodell blir imidlertid ordet tre representert av det samme tokenet. Uavhengig av om konteksten tilsier at vi snakker om tallet 3, et tre, at noe er laget av tre eller til og med om "tre" bare er en del av et ord. Under lister jeg opp 5 setninger med deres korresponderende tokens:
tre pluss tre = [4086, 633, 1824, 4360]\\
tre store tre = [4086, 4897, 4360]\\
den er laget av tre = [1660, 1111, 139108, 1452, 4360]\\
treplanke = [4086, 528, 29147]\\
trett = [4086, 1037]

Som vi kan se brukes tokenene 4086 og 4360 til å representere "tre" og " tre" respektivt. Ved første øyekast kan det se ut som om tokenizeren, lar konteksten bestemme om vi snakker om tallet 3 eller et tre. Den koder derfor ikke tallet tre og et tre forskjellig. Problemet oppstår når vi ser videre de siste to eksemplene. Også her er "tre" biten av ordet kodet som tokenet 4086, men nå er det ikke lenger tvetydig hva vi refererer til. I ordet treplanke er det utvedydig at vi refererer til materialet tre. Ordet 3planke gir ingen mening for oss, så her har tokenizationen introdusert ekstra usikkerhet som vi mennesker ikke opplever når vi leser tekst. I ordet "trett" er situasjonen enda verre. Her brukes fortsatt 4086 til å representere tre-biten av ordet, men her er det hverken tallet 3 eller treverk vi refererer til. Tokenizationen har alstå introdusert enda mer usikkerhet. Det blir nå opp til modellen å forstå ut ifra konteksten om vi snakker om tallet 3, treverk eller om vi snakker om noe helt urelatert som tretthet. Videre kan også ordet trett tolkes forskjellig utifra konteksten (er noen trøtte eller snakker vi om et tretthetsbrudd?).

Som følge av måten vi tokeniserer tekst på, er en språkmodell enda mer avhengig av kontekst enn vi mennesker er når den skal tolke og forstå meningen i en tekst. Dette tar oss videre til neste element i hvordan en språkmodell opplever en samtale.

# Kontekstvinduet
Dagens mest populære språkmodeller er alle bygget på Transformer modellen (Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Łukasz Kaiser, and Illia Polosukhin. 2017. Attention is all you need. In Proceedings of the 31st International Conference on Neural Information Processing Systems (NIPS'17). Curran Associates Inc., Red Hook, NY, USA, 6000–6010). Det er flere begrensninger med transformers for språkmodeller, men her skal vi fokusere på kontekstvinduet.

## Kontekstvinduet har begrenset størrelse
En transformer kan kun prossesere et begrenset antall tokens. Bakgrunnen for dette kommer av at det største fremskrittet introdusert i transformer modellen, nemlig "attention mekanismen". Vi skal ikke se på hvordan denne mekanismen fungerer her. Vi skal i stedet nøye oss med å påpeke at en del av denne mekanismen er å beregne hvor viktig hvert token er hvor hvert eneste annet token i en sekvens. Som konsekvens av dette, inneholder enhver attention mekanisme en matrise av størrelse $$n^2$$, hvor $$n$$ er antallet tokens i sekvensen som behandles. 

$$\begin{bmatrix} \text{Hunden} & 0.90 & 0.09 & 0.10 \\ \text{drakk} & 0.10 & 0.50 & 0.50 \\ \text{vann} & 0.30 & 0.40 & 0.40 \\ \text{ } &\text{Hunden} & \text{drakk} &\text{vann} \end{bmatrix}$$

I eksempelet over, ser vi en fiktiv attention beregning for setningen "Hunden drakk vann". Vi ser at matrisen inneholder $9 = 3^2$ tall. Ettersom vi med dagens hardware har begrenset lagringsplass på våre GPUer er det begrenset hvor store matriser vi kan håndtere. Vi er derfor tvunget til å begrense antallet tokens som prosseseres til enhver tid til et antall vi kan håndtere uten å gå tom for minne. Måten vi gjør dette på, er at vi konstruerer et vindu hvor alle tokens som er inne i vinduet prosseseres, mens alle tokens som er utenfor ikke blir prossesert. Vi lar deretter dette vinduet skli over alle tokens i teksten vår, for å prossesere hele teksten. Dette vinduet er det vi kaller _kontekstvinduet_ i en språkmodell.

Desverre er det slik at vi ikke kan huske tokenene som allerede er prossesert. Så i det øyeblikket et token faller ut av tokenvinduet, er det ikke lenger med å påvirker svaret til språkmodellen i det hele tatt. Så om vi deler en tekst med en språkmodell, hvor vi ønsker at modellen skal svare oss på spørsmål angående teksten, må vi først sørge for at teksten vår ikke er for stor til å få plass i kontekstvinduet. La oss igjen se på eksempelet "Hunden drakk vann". Hvis modellen vår har et kontekstvindu som tilsvarer to ord, vil ikke modellen vite hvem som drakk vannet, ettersom "Hunden" falt ut av kontekstvinduet, idet "vann" komm inn i kontekstvinduet. Hvis vi så spør modellen hvem som drakk vannet, vil den ikke vite svaret. Den vil alikevell forsøke å svare, men vil nå måtte gjette basert på statistikk over norsk språk. Den vil derfor kanskje svare "Han" eller "Hun". Dette skyldes et annet, urelatert, problem med språkmodeller vi skal se nærmere på i en senere post.

Det er gjort mange innovasjoner i senere tid, for å utvide størrelsen på kontekstvinduet og modeller som ChatGPT og Gemini har etterhvert fått enormt store kontekstvinduer. Dette er imidlertid fortsatt et stort problem når man ønsker å bruke språkmodeller til å svare på spørsmål fra store mengder tekst, eller gjøre oppsummeringer. Det er også en utfordring når vi nå beveger oss mer og mer i retningen "ressonerende modeller", hvor modellen er trent til å holde en slags indre, resonerende, monolog for å etterprøve seg selv og komme med bedre svar. Slike monologer har en tendens til å bli ekstremt lange, selv for enkle spørsmål, ettersom modellen hopper frem og tilbake mellom forskjellige "teorier" før den til slutt lander og blir enig med "seg selv". Den fyller rett og slett opp sitt eget kontekstvindu med ressonerende intern monolog og forskyver eventuell relevant foregående tekst, slik at den etterhvert faller ut av kontekstvinduet. Derfor er du med jevne mellomrom tvunget til å starte en ny "samtale" med språkmodellen og laste opp teksten på nytt, for å sørge for at relevant tekst igjen er en del av kontekstvinduet når du interagerer med en språkmodell.
