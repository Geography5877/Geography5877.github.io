---
title: En strek er også en modell
date: 2026-01-08 06:57:00 +0200
categories: [Maskinlæring og statistikk]
tags: [prediksjon, ml, statistikk, lineær regressjon]     # TAG names should always be lowercase
math: true
media_subpath: /assets/images/2026-01-10/
---

<!-- markdownlint-capture -->
I [forrige post](https://enklypesalt.com/posts/first-modeling-steps/) bygget vi vår første modell for å hjelpe oss å løse problemet med å finne ut hvem som får lov til å ta karusellen vår, gitt vekten deres. Vi utviklet en $k$-nærmeste nabo modell og utviklet en metrikk vi kunne bruke for å evaluere hvor godt modellen presterte ved hjelp av et sett med testdatapunkter. I denne posten skal vi se på en annen, svært vanlig modell fra statistikken, men la oss først gå over problemet vårt enda en gang: Som du sikkert nå husker, har vi rollen som en karuselloperatør i et tivoli. Karusellen vi opererer krever at alle som tar den er over en gitt minimumshøyde (160 cm), men vi har mistet målebåndet vårt og har kun en vekt som hjelpemiddel.

La oss igjen se på dataen vår:
![text](weight_height_dataset.png)
_Her plotter vi vekt (x-aksen) mot høyde (y-aksen)._

Hvis vi virkelig prøver kan det kanskje se ut som om veldig mange av punktene ligger på en linje. Det kan nesten se ut som om punktente i utgansgpunktet har ligget på en linje, men deretter blitt kastet av linjen i forskjellige rettninger. Så kanskje vi kan modellere dataen vår som om den ligger på en linje? Hvis vi bruker en linje som modell kan vi, når vi får en ny vektmåling, flytte oss på x-aksen til vi kommer til vekten vi har målt og deretter flytte oss opp langs y-aksen til vi treffer linjen vår. Vi kan så se inn på y-aksen hvor langt opp vi kom og der har vi høyden modellen vår predikerer!

Under har jeg illustrert fremgangsmåten i en figur. Her måler vi en vekt på 70.0 kg:
Steg 1: Start fra 70 på x-aksen og beveg deg vertikalt til du treffer den blå linjen.
Steg 2: Når du har truffet linjen beveger du deg direkte horisontalt til du treffer y-aksen. Hvor langt opp du kom på y-aksen er modellens prediksjon, her ca 157.5 cm.

![text](finding_y.png)
_Her ser vi hvordan vi starter fra en måling på 70 kg og finner høyden på ca. 157.5 cm._

Dette virker jo enkelt nok når vi har en bein linje å forholde oss til, men i det har jo ikke vi. Linjen er jo modellen vi skal bygge. Så hvordan gjør vi det?

## Å tegne en linje
Før vi kan starte med å bygge en modell av en linje må vi ta en litt nærmere kikk på hva en linje faktisk er, eller hvordan vi kan definere en linje. Hva trenger vi for å kunne tegne en linje? Jo, vi trenger minst to punkter å tegne linjen gjennom. Hvorfor to? Jo du kan jo prøve å tegne en linje som går gjennom bare ett punkt. Hvor skal du starte? Du kan starte hvor som helst og trekke en linje gjennom punktet og du vil ha laget en linje. Du kan så starte en annen plass og trekke en ny linje gjennom punktet igjen. Da har du to linjer, som begge går gjennom det samme punktet, men de er åpenbart ikke den samme linjen. Så for at vi skal kunne definere én spesifikk linje i modellen vår trenger vi minst to punkter. Ett å starte fra og ett vi skal gjennom. 

![text](two_lines.png)
_To linjer går gjennom det samme punktet, men de er åpenbart ikke samme linje. Vi trenger derfor minst to punkter for å definere en linje._

Men kan vi bruke flere enn to punkter? Ja selvfølgelig! Under har jeg tegnet en linje definert av tre punkter:
![text](three_linear_points.png)
_Tre punkter kan definere samme linje._

Her har jeg brukt punktene (1, 1), (2, 2) og (3, 3). Det første tallet i hver parentes sier hvor punktet ligger langs x-aksen og det andre tallet i parentesen sier hvor punktet ligger langs y-aksen. Så det første punktet ligger ett steg ut på x-aksen og ett steg opp på y-aksen. Men hvordan vet vi at denne linjen er en 100% bein? Kan det ikke være at den er litt kurvet? En måte vi kan forsikre oss selv om det på er å beregne hvor mye linjen endrer seg, når vi flytter oss mellom punktene. Hvis denne endringen er lik mellom alle punktene _vet_ vi at linjen vår er bein! Så la oss gjøre det.

Vi starter i det første punktet (1, 1) og flytter oss til neste punkt (2, 2). For å finne endringen langs x-aksen subtraherer vi x-verdiene fra hverandre, og for å finne endringen i y-aksen subtraherer vi y-verdiene fra hverandre. Hvis vi setter beregningen for endringen langs y-aksen i telleren og beregningen for endring langs x-aksen i nevneren i en brøk kan vi beregne den totale endringen for linjen:
$$
\frac{2 - 1}{2-1} = \frac{1}{1} = 1
$$

Vi kan nå gjøre det samme fra punkt to (2, 2) til punkt tre (3, 3):
$$
\frac{3 - 2}{3-2} = \frac{1}{1} = 1
$$

Endringen er 1 i begge tilfeller. Vi har en bein linje! Det vi har beregnet her er stigningstallet til linjene som ligger mellom punktene. Hvis dette stigningstallet er likt, vet vi at alle tre punkter ligger på den samme overordnede linjen. Stigningstallet defineres for en linje generelt slik:

$$
\begin{equation}
a = \frac{y_2 - y_1}{x_2 - x_1}
\end{equation}
$$

Men her snublet vi over noe interessant. Jeg sa nettopp at hvis stigningstallet til linjen mellom punktene er likt, så må også punktene ligge på den samme overordnede linje. Det betyr at vi kan bruke stigningstallet, $a$, til å definere en linje. Men er det nok med bare stigningstallet? Nei, stigningstallet sier jo bare hvor mye linjen endrer seg når vi beveger oss langs x-aksen. Men det finnes uendelig mange linjer som endrer seg like mye. Under har jeg tegnet tre linjer. De har samme stigningstall, men er åpenbart ikke samme linje. Jeg har også inkludert y-aksen i svart.

![text](three_lines_same_a.png)
_Tre linjer med samme stigningstall._

Så vi trenger altså noe mer enn bare stigningstallet. Men hvis vi tar en ny titt på figuren over, så ser vi at selv om alle tre linjene har samme stigningstall, så krysser de y-aksen på forskjellige punkter. Kan vi bruke punktet hvor linjen krysser y-aksen sammen med stigningstallet til å definere linjen? Ja det kan vi faktisk! Vi kan definere en gerell linje som:

$$
\begin{equation}
y = ax + b
\end{equation}
$$

$a$ er da stigningstallet vi definerte i stad. $x$ er hvor langt langs x-aksen vi flytter oss og $b$ er hvor linjen krysses y-aksen. Denne definisjonen av en linje har du antakelig sett tidligere, men jeg syntes det var nyttig å bruke litt tid til å se på hvorfor denne definisjonen stemmer.

## Hvilken linje gir den beste modellen?
Okay, så nå har vi kommet et godt stykke på veien, vi har mange datapunkter og vi vet hvordan vi kan definere en linje. Men nå kommer neste store utfordring. Hvordan finner vi den linjen som best beskriver dataen vår? Eller sagt på en annen måte. Hvordan finner vi den beste modellen, gitt dataen vår og at modellen er en bein linje?

Under har jeg tegnet tre linjer gjennom datasettet vårt. Hvilken av linjene passer dataen vår best? Kan vi kanskje si noe om hvor langt unna linjen vår er fra hvert datapunkt i datasettet vårt? Det burde jo kunne si noe om hvor bra linjen passer til daten. Hvis vi finner avstanden mellom hvert datapunkt og linjen og legger sammen det, vil ikke den linjen som ha lavest summerte avstand også være den linjen som har passer dataen vår best? Dette er verdt å teste ut, men la oss starte med et noen få eksempeldatapunkter og et par linjer for å teste ut dette:

Vi starter med punktene:
- (1, 3)
- (2, 3)
- (2.5, 1)
- (3, 2)

Vi bruker også linjene:
- $y_1 = 1x + 0.0 = x$
- $y_2 = 0.66x + 0.5$

