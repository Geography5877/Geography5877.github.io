---
title: Er attention alt vi trenger?
date: 2025-03-31 06:57:00 +0200
categories: [Språkmodeller og deres utfordringer]
tags: [llm, attention, dypdykk]     # TAG names should always be lowercase
math: true
description: I denne posten skal vi se på den viktigste byggesteinen i dagens språkmodeller: Attention mekanismen. Vi skal se hvordan den muliggjorde store språkmodeller og hvilke fordeler og ulemper den bringer med seg.
media_subpath: /assets/images/2025-04-03/
---
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Denne posten er en del av en lengre serie poster om språkmodeller og problemer med disse. Jeg vil starte med å presisere at språkmodeller helt klart har sine bruksområder, men ettersom denne serien ser på problemene med språkmodeller kommer jeg til å fokusere på ulemper og problemer forbundet med hvordan språkmodeller fungerer i dag.
{: .prompt-info }
<!-- markdownlint-restore -->

## Et historisk perspektiv på sekvensmodellering:
I [en tidligere post](https://enklypesalt.com/posts/Hvordan-opplever-kien-en-samtale/) påpeket jeg at det var en ting, mer enn noe annet, som muliggjorde dagens språkmodeller. Nemlig Transformer arkitekturen [^atten-is-all]. Denne arkitekturen er den underliggende arkitekturen alle de mest populære språkmodell som brukes i dag er bygget på. Før attention mekanismen var det rekurrente nevrale nettverk[^lstm][^rnn1][^rnn2] som dominerte språkmodellering. Disse modellene vedlikeholder en intern tilstand i, som oppdateres hver gang en ny input, f.eks. et nyt ord, prosesseres. Slike nettverk viste svært god prestasjon på en rekke oppgaver som innebar behandling av sekvenser, som tekstforståelse, spill og tidsserieanalyser. De har imidlertid også to store ulemper.

Den første ulempen er at for å trene et rekurrent nevralt nettverk, må man sende treningssignalet gjennom hele historien nettverket har behandlet, hver eneste gang. Ettersom vi oppdaterer vektene i et nevralt nettverk basert på hvor stort bidrag hvert "nevron" har på det endelige resultatet, så må man holde styr på hvordan alle foregående ord i en setning påvirket tilstanden i nettverket frem til nettverket produserte sin output. Så la oss si et rekurrent nettverk får setningen "Hunden drakk vann. Hvem drakk vannet?" og skal svare på denne. Da må nettverket gå ord for ord gjennom setningen og iterativt oppdatere sin interne tilstand for hvert ord. Etter alle ordene er prosessert svarer nettverket, forhåpentligvis, "Hunden". Men la oss si den svarer, "en katt". Da må vi oppdatere nettverkets vekter slik at nettverket lærer seg å prosessere ordene i setningen på en bedre måte og dermed forstå at det er "Hunden" som er riktig svar. Ettersom oppdateringen av tilstanden i nettverket skjedde for hvert ord, må vi derfor også gå baklengs gjennom hele setningen for å oppdatere vektene til nettverket. Dette gjør det svært vanskelig å parallelisere et rekurrent netverk, ettersom prosesseringen av ett ord avhenger av resultatet av prosesseringen av de foregående ordene i en slik arkitektur. Det gjør derfor også at å trene et rekurrent nevralt nettverk tar betraktelig lengre tid enn andre arkitekturer. Noe som gjør storskala språkmodellering uforholdsmessig dyrt og tidkrevenede.

Den andre ulempen er at rekurrente nevrale nettverk sliter med å lære seg sammenhenger som gjelder over lang tid. For å forstå det, må vi først innse at tilstanden nettverket vedlikeholder internt er av en begrenset størrelse, avhengig av vårt hardware, er det begrenset mye informasjon vi kan lagre i denne tilstanden. Når en sekvens med ord blir veldig lang, er det ikke plass i denne tilstanden til å holde styr på hvordan alt henger sammen. Resultatet er at vi så en tydelig nedgang i prestasjon jo lengre sekvensene nettverket prosseserte ble.

### Enter the Transformer
I 2017 kom en banebrytende paper med tittelen "Attention is all you need"[^atten-is-all], fra Google Brain teamet. Dette paperet introduserte en ny type nevral nettverk arkitektur kalt en Transformer. Denne arkitekturen var ikke helt ulik andre arkitekturer på den tiden, men den introduserte et nytt konsept, nemlig å bruke en attention mekanisme til å styre flyten av informasjon i nettverket. La oss først se litt nærmere på denne arkitekturen:

#### Attention?
Attention er en operasjon som opererer på tre vektorer:
- $Q$
- $K$
- $V$

Alle disse tre vektorene er en lineær transformasjon av den oprinnelige teksten vår. Hvordan skjer dette? Jo, husk først at det første som skjer med teksten vår er at den blir [tokenized](https://enklypesalt.com/posts/Hvordan-opplever-kien-en-samtale/#tokenization). Den blir altså omgjort til en sekvens med tall. Det neste steget er at hvert token går gjennom en "embedding modell" som transformerer hvert token til en vektor. De lineære transformasjonene som skjer i $Q,K$ og $V$ skjer altså på embeddingen av hvert token. Den lineære operasjonen er rett og slett en matrisemultiplikasjon med denne sekvensen. Hver av $Q, K$ og $V$ har sin egen matrise, noe som gjør at denne operasjonen resulterer i tre separate og uavhengige transformasjoner av den opprinnelige sekvensen. Rent matematisk kan vi formulere det slik som i \eqref{eq:QKV-weights}:

\begin{equation}
\begin{aligned}
& Q = xW^Q \\
& K = xW^K \\
& V = xW^V
\end{aligned}
\label{eq:QKV-weights}
\end{equation}

Her er $W^Q, W^K$ og $W^V$ matrisene jeg nevnte over, og $x$ er den opprinnelige tokeniserte teksten. La oss se et eksempel på en slik beregning. La oss si at vi starter med teksten: "2+2". Som vi har sett [tidligere](https://enklypesalt.com/posts/Hvordan-opplever-kien-en-samtale/#tokenization) blir denne teksten tokenisert til:\\

$\text{tokens} = \begin{bmatrix}17 & 10 & 17\end{bmatrix}$

Vi sender denne så gjennom en embedding modell og får ut embeddingen av hvert token:\\
$x = \begin{bmatrix}1 & -3\end{bmatrix}, \begin{bmatrix 13 & -2 \end{bmatrix}, \begin{bmatrix}1 & -3\end{bmatrix}$

La oss nå si at vi har matrisen: $W^Q = \begin{bmatrix}-1 & 1 \\ 2 & -1\end{bmatrix}$. Vi kan da beregne $Q$ slik:\\

\begin{equation}
\begin{aligned}
Q = xW^Q &= \begin{bmatrix} 1 & -3\\ 13 & -2 \\ 1 & -3\end{bmatrix} \begin{bmatrix}-1 & 1 \\ 2 & -1\end{bmatrix}$\\
&= \begin{bmatrix} -1\cdot 1 + 2 \cdot (-3) & 1 \cdot 1 + (-1)\cdot(-3) \\ (-1) \cdot 13 + 2 \cdot(-2) & 1 \cdot 13 + (-1) \cdot (-2) \\ -1\cdot 1 + 2 \cdot (-3) & 1 \cdot 1 + (-1)\cdot(-3)\end{bmatrix}\\
&= \begin{bmatrix} -7 & 4 \\ -17 & 15 \\ -7 & 4\end{bmatrix}
\end{aligned}
\label{eq:Q-calc}
\end{equation}

La oss også si vi brukte samme metode for å finne: $K = \begin{bmatrix} -8 & -10 \\ 7 & 18 \\ -8 & 10\end{bmatrix}$ og $V = \begin{bmatrix} -10 & -8 \\ 18 & 7 \\ -10 & 8\end{bmatrix}$



Neste steg i attention mekanismen er å beregne det vi kaller et "attention map". Dette er en $n \times n$ matrise som inneholder informasjon om hvordan hvert token i sekvensen vår skal relateres til alle andre tokens i den samme sekvensen. Her er $n$ lengden på sekvensen. Så hvordan beregner vi denne? Jo vi bruker $Q$ og $K$ fra tidligere \eqref{attention}:

\begin{equation}
{\text{AttentionMap}}(Q, K) = \text{softmax}(\frac{QK^{\top}}{\sqrt{d_k}})
\label{eq:attention}
\end{equation}

Hvor: $\text{softmax}(x_i) = \frac{e^{x_i}}{\sum^n_{j=1}e^{x_j}}$ og $d_k$ er en skaleringsfaktor.

La oss starte med å beregne $QK^T$:\\

\begin{equation}
\begin{aligned}
QK^{\top}& =  \begin{bmatrix}-7 & 4 \\ -17 & 15 \\ -7 & 4\end{bmatrix}\begin{bmatrix}-8 & 7 & -8 \\ -10 & 18 & -10\end{bmatrix}\\
&=\begin{bmatrix}(-8) \cdot (-7) + (-10) \cdot 4 & 7 \cdot (-7) + 18 \cdot 4  & (-8) \cdot (-7) + (-10) \cdot 4 \\ (-8) \cdot (-17) + (-10) \cdot 15 & 7 \cdot (-17) + 18 \cdot 15 & (-8) \cdot (-17) + (-10) \cdot 15 \\ (-8) \cdot (-7) + (-10) \cdot 4 & 7 \cdot (-7) + 18 \cdot 4  & (-8) \cdot (-7) + (-10) \cdot 4\end{bmatrix}\\
&=\begin{bmatrix} 16 & 23 & 16 \\ -14 & 151 & -14 \\ 16 & 23 & 16\end{bmatrix}
\end{aligned}
\end{equation}

Skaleringsfaktoren $d_k$ er størrelsen på vektorene vi får ut av embedding modellen. I vårt tilfelle er altså $d_k = 2$. Vi skalerer derfor $QK^{\top}$ slik:

\begin{equation}
\begin{bmatrix} 16 & 23 & 16 \\ -14 & 151 & -14 \\ 16 & 23 & 16\end{bmatrix}/\sqrt{2} = \begin{bmatrix} 11.31 & 16.26 & 11.31 \\ -9.90 & 106.77 & -9.90 \\ 11.31 & 16.26 & 11.31\end{bmatrix}
\end{equation}

Vi kan nå beregne attention mappet slik:\\
\begin{equation}
\begin{aligned}
$\text{softmax}\left(\frac{QK^{\top}}{\sqrt{d_k}}\right)\\
&= \text{softmax}(\begin{bmatrix} 11.31 & 16.26 & 11.31 \\ -9.90 & 106.77 & -9.90 \\ 11.31 & 16.26 & 11.31\end{bmatrix})\\
&= \begin{bmatrix} 3.49\cdot 10^{-42} & 4.92 \cdot 10^{-51} & 3.49\cdot 10^{-42} \\ 2.14 \cdot 10^{-51} & 1.00 & 2.14 \cdot 10^{-51} \\ 3.49\cdot 10^{-42} & 4.92 \cdot 10^{-51} & 3.49\cdot 10^{-42}\end{bmatrix}
\end{aligned}
\end{equation}

Hva er det vi har i denne matrisen? Jo, denne matrisen inneholder hvor viktig hvert token i teksten betyr for hvert av de andre tokenene i teksten. Vi har rett og slett beregnet hvor my den første 2'eren bryr seg om + og den andre 2'eren, samt seg self. Hvor mye + bryr seg om den første og den andre 2'eren osv. Vi kan nå multiplisere dette med $V$ for å transformere inputen vår $x$ med informasjon om hvor mye hvert token i $x$ betyr for hvert av de andre tokenene i teksten. Attention mekanismen vekter rett og slett hvert token med tanke på oppgaven den skal løse.

Legg også merke til at denne beregningen er dynamisk. Altså forandrer den seg hver gang vi får en annen tekst. Som for eksempel når vi får et nytt token inn i sekvensen vår. 

#### Men er ikke en transformer et nevralt nettverk?
Foreløpig har vi bare sett lineære transformasjoner og en softmax i transformer arkitekturen. Er ikke transformeren et nevralt nettverk? Jo det er den, den resterende delen av en transformer er ganske enkelt et helt vanlig nevralt nettverk som kjøres over alle de transformerte tokenene som kommer ut av attention mekanismen, token for token. En transformer flere lag, hvor hvert lag har en attention mekanisme og et slikt nevralt nettverk. Teksten vår mates gjennom alle lagene i transformeren før vi får neste token prediksjonen fra transformeren.

Legg merke til at det i transformeren ikke er noen tilstand som må vedlikeholdes. Dette gjør at en transformer kan paralleliseres! Dette er antakelig hovedårsaken til at transformer arkitekturen er fundamentet i en hver språkmodell. Den kan trenes i parallel, noe som gjør at vi kan trene modellen på massive megder tekst kjapt og effektivt.

### Kontekstvinduet i en transformer
Vi har tidligere nevnt at kontekstvinduet og dets størrelse er [avgjørende for hva en språkmodell kan gjøre](https://enklypesalt.com/posts/Hvordan-opplever-kien-en-samtale/#kontekstvinduet). Nå som vi har en oversikt over hvordan attention beregnes, kan vi se litt nærmere på dette.

Størrelsen på kontekstvinduet avgjøres nemlig av størrelsen på attention mappet vi beregnet tidligere. Hvor store attention maps vi kan få plass til på vårt hardware styrer hvor lange sekvenser vi kan behandle i språkmodellen vår. Vi ser at kontekstvinduet har størrelse $n^2$, der $n$ er tekstlengden. Vi må også huske at modellen vår også består av mange lag, der hvert lag har sitt eget attention map. Da ser vi at vi fort møter en hardwarebegrensning på hvor store individuelle attention maps vi får plass til, og dermed også hvor stort kontekstvindu vi kan ha.

Vi har også også sett at en transformer ikke vedlikeholder noen tilstand, men dynamisk beregner vektingen av tokens hver gang vi endrer teksten vår. Det betyr at når teksten vår blir for stor til å få plass i attention mappet vårt, så faller den helt enkelt ut av modellen og inngår ikke lenger i noen prosessering. Kontekstvinduet inneholder altså bare de siste $m$ tokenene og det er kun disse som brukes for å beregne svarene fra modellen.

Har vi f.eks. et kontekstvindu med størrelse 5 ord og teksten: "Hunden drikker vann", får alle ordene plass i kontekst vinduet. Om vi deretter legger til et par ord i teksten for å få: "Hunden drikker vann. Det liker den.", er det bare " liker den." som får plass i kontekstvinduet (husk at mellomrom og punktum er egne ord). Modellen vil derfor ikke kunne svare på hvem som liker hva i den teksten, ettersom "Hunden drikker vann. Det" har falt ut av kontekstvinduet og ikke blir med i noen beregninger.

Heldigvis er kontekstvinduene i dagens språkmodeller veldig store. Så i de fleste tilfeller vil vi ikke oppleve at vi fyller kontekstvinduet med det første. Skal vi imidlertid prosessere mye tekst, som f.eks. når vi ønsker oss sammendrag av lengre tekst, eller stiller modellen spørsmål om innholdet i en lengre tekst, kan vi møte på dette problemet.





Attention påvirkes av hva som er i kontekstvinduet. 
-> Hvis informasjon faller ut, kan det bety endringer i hvordan all etterfølgende tekst "tolkes" av modellen
-> Ny tekst kan påvirke hvordan gammel tekst tolkes
   - både positivt og negativt (overskriver feil, men gjør det også mulig å "overse" overskriviger)


## Referanser
[^atten-is-all]: Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Łukasz Kaiser, and Illia Polosukhin. 2017. Attention is all you need. In Proceedings of the 31st International Conference on Neural Information Processing Systems (NIPS'17). Curran Associates Inc., Red Hook, NY, USA, 6000–6010
[^rnn1]: Elman, J. L. (1990). Finding structure in time. Cognitive science, 14(2), 179-211.
[^rnn2]: Jordan, M. I. (1997). Serial order: A parallel distributed processing approach. In Advances in psychology (Vol. 121, pp. 471-495). North-Holland.
[^lstm]: Hochreiter, S., & Schmidhuber, J. (1997). Long short-term memory. Neural computation, 9(8), 1735-1780.
