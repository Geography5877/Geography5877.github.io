---
title: Maskinlæringsfjellet
date: 2026-01-06 06:57:00 +0200
categories: [Maskinlæring og statistikk]
tags: [prediksjon, ml, statistikk]     # TAG names should always be lowercase
math: true
description: I denne posten skal vi starte en ny serie der vi skal se på statistikk og maskinlæring og samtidig forhåpentligvis forstå hvordan disse metodene fungerer og hva de kan brukes til. Koden for denne posten finnes [her](https://github.com/haakom/enklypesalt/tree/main/2026-01-06_correlation) 
media_subpath: /assets/images/2026-01-06/
---
<!-- markdownlint-capture -->

Nå er det snart et år siden sist jeg postet på denne bloggen. Det har jeg endelig tenkt å gjøre noe med!

Denne bloggen er jo egentlig tenkt brukt som et sted hvor jeg kan skrive om mine interesser og forhåpentligvis produsere noe som også kan være nyttig for andre i samme slengen. I denne posten ønsker jeg å starte en ny serie der vi ser nærmere på statistikk og maskinlæring, der målet er at du som leser skal få bedre innsikt i tankegangen bak modellene som styrer mye av automatikken vi har rundt oss i dag. Samtidig opplever jeg selv også at jeg nokså ofte glemmer detaljer rundt modeller og metoder. Derfor vil serien også tjene som en referanse for meg selv, slik at jeg kan komme tilbake for å få en liten påminnelse. Forhåpentligvis kan vi alle få noe ut av dette :)

## Vi skal løse problemer, ikke lære om verktøy
Det er jo mange forskjellige måter man kunne startet en slik serie. Det finnes utallige alternativer i statistikkbøker og maskinlæringskurs. Jeg tenker derfor egentlig bare å starte med et tenkt problem og deretter prøve å løse dette. Jeg tror de fleste metodene vi kommer til å møte i denne serien har nettopp dette som opphavshistorie. En person har sittet med et problem de ønsket å løse, men ikke hatt et verktøy tilgjengelig, som kunne løse det. Hen har derfor satt seg ned og tenkt grundig og nøye over problemet og deretter funnet på en ny måte å tenke på for å løse problemet. Gjerne som en liten utvidelse av et eksisterende verktøy eller modell.

Jeg har fått mer og mer tro på at læring burde ta en lignende form. Vi mennesker kommuniserer ofte gjennom historier og vi deler også mye lærdom gjennom historiedeling. Jeg tror derfor at læring generelt kan være mer effektivt om vi kombinerer teori med historier for å både øke engasjementet rundt det vi lærer, men også forstå bedre hvorfor vi lærer det og hvordan man i første omgang fant ut av det. Det er ytterst sjeldent at en ny teknologi oppstår ut av det blå. De aller fleste vitenskapelige fremskritt kommer som et resultat av å bygge en liten ny bit oppå et fjell av eksisterende kunnskap. I denne serien har jeg lyst til å prøve å bestige dette fjellet gjennom gradvis å støte på nye utfordringer og deretter "gjennoppfinne" tilnærmingene som kan brukes for overkomme disse.

## Vårt første problem
Så la oss starte på veien opp kunnskapsfjellet med en første utfordring. La oss se for oss at vi jobber som tivolivakt som slipper mennesker ombord på en karusell. Karusellen har et høydekrav for at en person skal få lov til å ta den, men vi har mistet målebåndet vårt. Det eneste verktøyet vi har tilgjengelig er en vekt. Kan vi bruke en persons vekt til å si oss noe om høyden til personen? For å svare på dette spørsmålet kan vi i første omgang bruke det vi kjenner om menneskekroppen fra før. Vi vet at jo høyere en person er, jo mer bein må det finnes i kroppen til personen. Det finnes rett og slett ofte mer av en person jo høyere personen er. Så det er jo et godt utgangspunkt. For å se nærmere på dette kan vi ta i bruk data. Under genererer jeg et datasett hvor vi har målt høyde og vekt for en rekke tilfeldig utvalgte mennesker. Vi har kun disse to målene og ingenting annet å gå ut ifra i dette datasettet:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from matplotlib.colors import ListedColormap

# Define a custom colorblind friendly colormap
custom_cmap = ListedColormap([
    '#e69f00',  
    '#0072b2', 
    '#cc79a7', 
    '#f0e442', 
    '#56b4e9', 
    '#009e73', 
    '#000000'
])

# Seed random generator for reproducibility
rng = np.random.default_rng(42)

# Set n samples we want
n_samples = 256

# Generate data
weights_m = rng.normal(80, 10, n_samples//2)
weights_f = rng.normal(65, 10, n_samples//2)

heights_m = weights_m + rng.normal(100, 20, n_samples//2)
heights_f = weights_f + rng.normal(90, 10, n_samples//2)

#is_male = np.ones(n_samples, dtype=np.int8)
#is_male[n_samples//2:]-=1

# Create dataframe from data
df_w2h = pd.DataFrame({"height": np.concatenate([heights_m, heights_f]), 
                       "weight": np.concatenate([weights_m, weights_f])})
print(df_w2h.head())
```

Når vi kjører denne koden får vi generert et dataset og vi ser på noen eksempler fra datasettet:
```
       height     weight
0  191.742492  83.047171
1  162.077038  69.600159
2  184.828053  87.504512
3  161.907731  89.405647
4  155.726173  60.489648
```

Vi kan også plotte dataen for å se hvordan den "ser ut". Under plotter jeg vekt langs x-aksen (horisontalen) og høyde langs y-aksen (vertikalen). Ser du noen sammenheng?
![text](weight_height_dataset.png)
_Vi plotter vekt mot høyde for å se om vi ser tendenser til sammenhenger._

Gjennom ren inspeksjon kan det se ut som at jo mer man veier jo høyere er man. Akkurat slik vi tenkte oss frem til tidligere! Men dette er jo bare et plott og vi får jo bare en viss formening om at vekt henger sammen med høyde. Det hadde vært veldig fint om vi var kunne kvantifisere dette på en mer objektiv måte. En måte der vi ser på hvordan vekt og høyde varierer sammen. Det hadde jo vært gull! Men før vi kan gjøre det for to variabler (høyde og vekt), trenger vi en måte å måle hvordan en enkelt variabel varierer. En måte å gjøre dette på er å se på hvor mye variabelen varierer fra gjennomsnittet sitt.

La oss ta vekt som et eksempel. Vi kan si noe om hvor mye vektmålingene våre varierer ved å se på hvor langt de gjennomsnittlig er unna den gjennomsnittlige vekten. La oss kalle gjennomsnittsvekten i dataen vår $\bar{V} = \frac{1}{N}\sum^N_{i=1} V_i$ der den $i$-te vektmålingen er $V_i$ og det totale antallet målinger er $N$. Da kan vi beregne hvor mye vekten i dataen vår varierer slik:
$$
\begin{equation}
'Var(V)' = \frac{1}{N}\sum^N_{i=1} V_i - \bar{V}
\end{equation}
$$

Det vi gjør er å beregne gjennomsnittlig vektavvik fra gjennomsnittsvekten. La oss se på et konkret eksempel for å gjøre ting litt klarere:

La oss si vi har 4 vektmålinger:
```
1
2
3
2
```
Vi finner da gjennomsnittsvekten slik:
$$
\bar{V} = \frac{1+2+3+2}{4} = 2
$$

Vi kan deretter beregne avviket fra gjennomsnittsvekten for hver vektmåling slik:

$$
\begin{aligned}
avvik(V_1) = 1 - 2 = -1 \\
avvik(V_2) = 2 - 2 = 0 \\
avvik(V_3) = 3 - 2 = 1 \\
avvik(V_4) = 2 - 2 = 0
\end{aligned}
$$

Vi kan nå beregne gjennomsnittlig vektavvik fra gjennomsnittsvekten slik:
$$
'Var(V)' = \frac{-1 + 0 + 1 + 0}{4} = 0
$$

Men dette virker jo veldig rart. Hvordan kan gjennomsnittlig vektavvik fra gjennomsnittsvekten være 0 når vi har 2 som gjennomsnittsvekt og flere målinger som ikke er eksakt 2? Grunnen til at dette skjer er hvordan vi beregner gjennomsnittlig avvik her. Vi har rett og slett vært uheldig med målingene våre, slik at de utligner hverandre når vi summerer dem. Legg merke til at de eneste vektmålingene våre som ikke er 2, er 1 og 3. Når vi trekker fra gjennomsnittet fra disse målingene får vi -1 og 1. Når vi så summerer dette, vil disse utligne hverandre og gi oss 0, så summen vår blir 0.

Dette er problematisk fordi måten vi beregner gjennomsnittlig avvik fra gjennomsnittet nå ikke reflekterer virkeligheten. Finnes det en måte vi kan unngå dette? Ja, heldigvis! Hvis vi tar en titt til, så legger vi merke til at grunnen til at vi får dette problemet er at fortegnet er motsatt for -1 og 1. Hvis vi bare kan kvitte oss med de motsatte fortegnene, så har vi jo ikke lenger dette problemet. En måte vi kan kvitte oss med fortegnet på er å kvadrere! Hvis vi kvadrerer etter vi har beregnet avviket fra en spesifikk vektmåling fra gjennomsnittsvekten kan vi ikke ende opp med negative fortegn! La oss se hva som skjer når vi bruker denne tilnærmingen:
$$
\begin{aligned}
avvik(V_1)^2 = (1 - 2)^2 = 1 \\
avvik(V_2)^2 = (2 - 2)^2 = 0 \\
avvik(V_3)^2 = (3 - 2)^2 = 1 \\
avvik(V_4)^2 = (2 - 2)^2 = 0
\end{aligned}
$$

Da får vi følgende gjennomsnittlig avvik fra gjennomsnittsvekten:

$$
Var(V) = \frac{1 + 0 + 1 +0}{4} = 0.5
$$

Nå fikk vi et annet svar! Vi finner at gjennomsnittlig avvik fra gjennomsnittsvekten er 0.5. Dette virker mye mer rimelig! Denne måten å beregne gjennomsnittlig avvik fra gjennomsnittet på er faktisk så vanlig i statistikk at den har fått et eget navn. Vi kaller den variansen (variance) i dataen vår. Vi kan formulere den mer generelt slik:

$$
\begin{equation}
Var(V) = \frac{1}{N}\sum^N_{i=1} (V_i - \bar{V})^2
\end{equation}
$$

La oss se hvordan vi kan beregne variansen til vekt i python:
```python
# Compute \bar{V}
mean_w = sum(df_w2h.weight)/len(df_w2h)

# Compute (V_i - \bar{V})^2
deviations = [(wi - mean_w)**2 for wi in df_w2h.weight]

# Compute variance
variance = sum(deviations) / len(df_w2h)
print(variance)
```
Ut får vi dette:
```
162.7917952074835
```

Heldigvis kan vi også bruke pandas til å hjelpe oss. Vi kan beregne variansen for hver kolonne i en pandas dataframe slik:
```python
df_w2h.weight.var()
```
Og ut får vi:
```
163.53175791297207
```

Legg merke til at variansen som beregnes av pandas ikke er eksakt den samme verdien vi beregnet for hånd. Den er imidlertid veldig nær. Vi slår oss derfor til ro med dette i denne posten, men vit at det er grunner til at denne forskjellen oppstår.

Okay, så vi er i stand til å finne et tall som sier oss noe om hvordan en variabel varierer, men hva med to variabler. Hvordan kan vi si noe om hvordan de varierer sammen? Jo, da kan vi benytte eksakt samme metode, men istedet for å kvadrere, så multipliserer vi variablene:
$$
\begin{equation}
Cov(V, H) = \frac{1}{n}\sum^n_{i=1} (V_i - \bar{V})(H_i - \bar{H})
\end{equation}
$$

Denne beregningen kalles kovarians i statistikken og sier oss nettopp hvordan to variabler varierer sammen, eller ko-varierer.

I python kan vi beregne dette slik:
```python
df_w2h['weight'].cov(df_w2h['height'])
```
Som gir oss:
```
215.11776578233534
```

Altså er kovariansen mellom vekt og høyde 215.1777.... En positiv kovarians sier oss at når en variabel går opp, så går også den andre opp. Altså, jo tyngre man er, jo høyere vil man også være i dette datasettet. Men ett problem med kovarians er at vi ikke kan si så mye hvor om sterk denne sammenhengen er. Er 215.1777... en stor kovarians? Det er faktisk litt uklart. Hadde det ikke vært veldig greit om vi også kunne si noe om styrken mellom hvordan disse to variablene varierer? For eksempel at 1 indikerer perfekt sammenheng, -1 indikerer perfekt negativ sammenheng og at 0 indikerer ingen sammenheng.

For å få til det, kan vi normalisere kovariansen slik at den kun eksisterer på intervallet $[-1, 1]$, men hva bruker vi som normaliseringsfaktor for å få til det? Jo, se tilbake til hvordan vi fant varians for en variabel. Husk at vi kvadrerte summen for å unngå kanselleringer. Det var jo veldig effektivt for å fjerne problemet, men det introduserte også en utilsiktet effekt som gjorde variansen vanskelig å tolke, akkurat som kovarians. Er en varians på $163.531.... kg^2$  mye eller lite? Vektene vi har målt varierer jo sånn ca. mellom 50 og 100 kg. Hva er egentlig $kg^2$? Det er en ekkel størrelsesenhet å forholde seg til. Det vi egentlig ønsker oss fra beregningen vår er jo et tall som beskriver hvor mye en variabel varierer på en måte vi intuitivt forstår. Helst på samme skala som dataen vår.  Kan vi bli kvitt denne kvadreringen på en måte? Ja, vi kan jo bare bruke inversoperasjonen for kvadrering, nemlig å ta kvadratroten:

$$
\begin{equation}
\sigma_V = \sqrt{Var(V)}
\end{equation}
$$

Hvis vi gjør det får vi at $\sigma_V = \sqrt{163.5317...} = 12.7879...$. Denne beregningen brukes kanskje enda oftere i statistikk en varians og kalles standardavviket (standard deviation). Ved å beregne variabiliteten i dataen vår på denne måten, skjønner vi mye bedre hvordan den varierer fra gjennomsnittet. Vektmålingene våre rangerer altså fra rundt 50 kg til 100 kg, gjennomsnittsvekten vi har målt er $\bar{V} = 71.275 kg$ og den varierer i gjennomsnitt 12.7879... kg fra denne gjennomsnittsvekten.

Så hvordan bruker vi standardavviket til å fikse problemet vårt med kovarians? Jo, vi kan dele kovariansen på produktet av standardavviket til variablene våre:

$$
\begin{equation}
r = \frac{Cov(V, H)}{\sigma_V \sigma_H} 
\end{equation}
$$

Her er $\sigma_V$ og $\sigma_H$ standardavviket til henholdsvis vekt og høyde i dataen vår.  Også denne beregningen brukes svært ofte i statistikken og har derfor også sitt eget navn. $r$ kalles korrelasjonen, eller korrelasjonskoeffisienten, mellom de to variablene. I python kan vi beregne den slik:
```python
df_w2h['weight'].corr(df_w2h['height'])
```

```
0.7246963843238032
```

Dette er et normalisert tall, hvor 1 ville vært en perfekt positiv korrelasjon mellom vekt og høyde og -1 ville vært en perfekt negativ korrelasjon mellom vekt og høyde. Her har vi en korrelasjon på 0.72. Det er altså sterk positiv korrelasjon mellom vekt og høyde i vår data!

Nå har vi allerede kommet ganske langt! Vi kan kvantifisere hvordan vekt og høyde endrer seg sammen og har funnet at det er en sammenheng der. Vi kan også si at denne sammenhengen er en sterk positiv sammenheng, så jo høyere man er jo mer veier man antakelig! Så hvordan bruker vi bruke korrelasjonen til å avgjøre om en person kan få lov til å ta karusellen vår, gitt at vi får målt personens vekt? Vel, det kan vi jo ikke... Korrelasjonen gir oss ingen god måte å si noe om hva en persons høyde er, gitt en vekten. For å kunne gjøre det, trenger vi en modell. Det er her min fascinasjon for statistikk og maskinlæring egentlig starter. Disse metodene lar oss bygge modeller vi kan bruke til å gjøre prediksjoner og komme med prognoser eller evaluere sammenhenger og til og med si noe om hva som forårsaker hva og til hvilken grad! Dette er noe vi mennesker gjør naturlig hele tiden. Vi gjør nesten alltid antakelser og handler basert på disse. Maskinlæring og statistikk lar oss gjøre det samme i et godt definert rammeverk, med matematisk robusthet og med en datamaskins regnekraft. De er rett og slett fantastiske verktøy og hjelpemidler som kan hjelpe oss til å gjøre bedre valg i våre liv. Så videre i denne serien skal vi se på hvordan vi bygger forskjellige typer modeller og bruker dem!

## Endringslogg
Under følger en endringslogg som viser hvilke deler av denne posten som er endret til hvilket tid. Enkle skrivefeil og den slags ting vil ikke bli logget, men jeg vil etterstrebe å logge alle meningsfulle endringer i posten.
