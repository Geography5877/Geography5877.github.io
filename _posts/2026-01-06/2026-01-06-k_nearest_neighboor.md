---
title: Kunnskapsfjellet
date: 2026-01-06 06:57:00 +0200
categories: [Maskinlæring og statistikk]
tags: [prediksjon, ml, statistikk]     # TAG names should always be lowercase
math: true
description: I denne posten skal vi starte en ny serie der vi skal se på statistikk og maskinlæring og samtidig forhåpentligvis forstå hvordan de fungerer og hva de kan brukes til.
media_subpath: /assets/images/2026-01-06/
---

Nå er det snart et år siden sist jeg postet på denne bloggen. Det har jeg endelig tenkt å gjøre noe med!

Denne bloggen er jo egentlig tenkt brukt som et sted hvor jeg kan skrive om mine interesser og forhåpentligvis produsere noe som også kan være nyttig for andre i samme slengen. I denne posten ønsker jeg å starte en ny serie der vi ser nærmere på statistikk og maskinlæring, der målet er at du som leser skal få bedre innsikt i tankegangen bak modellene som styrer mye av automatikken vi har rundt oss i dag. Samtidig opplever jeg selv også at jeg nokså ofte glemmer detaljer rundt modeller og metoder. Derfor vil serien også tjene som en referanse for meg selv, slik at jeg kan komme tilbake for å få en liten påminnelse. Forhåpentligvis kan vi alle få noe ut av dette :)

## Vi skal løse problemer, ikke lære om verktøy
Det er jo mange forskjellige måter man kunne startet en slik serie. Det finnes uttallige alternativer i statistikkbøker og maskinlæringskurs. Jeg tenker derfor egentlig bare å starte med et tenkt problem og deretter prøve å løse dette. Jeg tror de fleste metodene vi kommer til å møte i denne serien har nettopp dette som opphavshistorie. En person har sittet med et problem de ønsket å løse, men ikke hatt et verktøy tilgjengelig, som kunne løse det. Hen har derfor satt seg ned og tenkt grundig og nøye over problemet og deretter funnet på en ny måte å tenke på for å løse problemet. Gjerne som en liten utvidelse av et eksisterende verktøy eller modell.

Jeg har fått mer og mer tro på at læring burde ta en lignende form. Vi mennesker kommuniserer ofte gjennom historier og vi deler også mye lærdom gjennom historiedeling. Jeg tror derfor at læring generelt kan være mer effektivt om vi kombinerer teori med historier for å både øke engasjementet rundt det vi lærer, men også forstå bedre hvorfor vi lærer det og hvordan man i første omgang fant ut av det. Det er ytterst sjeldent at en ny teknologi oppstår ut av det blå. De aller fleste vitenksapelige fremskritt kommer som et resulatat av å bygge en liten ny bit oppå et fjell av eksisterende kunnskap. I denne serien har jeg lyst til å prøve å bestige dette fjellet gjennom gradvis å støte på nye utfordringer og deretter "gjennoppfinne" tilnærmingene som kan brukes for overkomme disse.

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
![text](k_nearest_dataset.png)
_Vi plotter vekt mot høyde for å se om vi ser tendenser til sammenhenger._

Gjennom ren inspeksjon kan det se ut som at jo mer man veier jo høyere er man. Akkurat slik vi tenkte oss frem til tidligere! Men dette er jo bare et plott og vi får jo bare en viss formening om at vekt henger sammen med høyde. Det hadde vært veldig fint om vi var kunne kvantisere dette på en mer objektiv måte. En måte der vi ser på hvordan vekt og høyde varierer sammen. Det hadde jo vært gull! Men før vi kan gjøre det for to variabler (høyde og vekt), trenger vi en måte å måle hvordan en enkelt variabel varierer. En måte å gjøre dette på er å se på hvor mye variabelen varierer fra gjennomsnittet sitt.

La oss ta vekt som et eksempel. Vi kan si noe om hvor mye vektmålingene våre varierer ved å se på hvor langt de er unna den gjennomsnittlige vekten. La oss kalle gjennomsnittsvekten $\bar{V}$ og den $i$-te vektmålingen $V_i$. Da kan vi beregne hvor mye vekten i dataen vår varierer slik:
$$
\begin{equation}
\frac{1}{n}\sum^n_{i=1} V_i - \bar{V}
\end{equation}
$$



## Endringslogg
Under følger en endringslogg som viser hvilke deler av denne posten som er endret til hvilket tid. Enkle skrivefeil og den slags ting vil ikke bli logget, men jeg vil etterstrebe å logge alle meningsfulle endringer i posten.
