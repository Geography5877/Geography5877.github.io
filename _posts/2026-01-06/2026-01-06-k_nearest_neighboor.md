---
title: Maskinlæringsfjellet
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
"Var(V)" = \frac{1}{n}\sum^n_{i=1} V_i - \bar{V}
\end{equation}
$$

Men vi får et problem her, hvis vi er riktig uheldig med $V_i$'ene våre, kan vi ende opp med at deler av summen er lik, men med forskjellig fortegn. De vil derfor utligne hverandre og ødelegge beregnigen vår. Heldigvis kan vi enkelt unngå dette ved å kvadrere summen slik:
$$
\begin{equation}
Var(V) = \frac{1}{n}\sum^n_{i=1} (V_i - \bar{V})^2
\end{equation}
$$

Deenne beregningen kalles faktisk varians i statistikken og brukes nettopp til å si noe om hvordan en variabel, vekt i dette tilfellet, varierer. La oss se hvordan vi kan beregne variansen til vekt i python:
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
df_w2h.var()
```
Og ut får vi:
```
height    538.813245
weight    163.531758
dtype: float64
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
np.float64(215.11776578233534)
```

Altså er kovariansen mellom vekt og høyde 215.1777.... En positiv kovarians sier oss at når en variablel går opp, så går også den andre opp. Altså, jo tyngre man er, jo høyere vil man også være i dette datasettet. Men ett problem med kovarians er at vi ikke kan si så mye om sterk denne sammenhengen er. Hadde det ikke vært veldig greit om vi også kunne si noe om styrken mellom hvordan disse to variablene varierer? For eksempel at 1 indikerer perfekt sammenheng, -1 indikerer perfekt negativ sammenheng og at 0 indikerer ingen sammenheng.

For å få til det, kan vi normalisere kovariansen, men hva bruker vi som normaliseringsfaktor? Jo, se tilbake til hvordan vi fant varians for en variabel. Legg merke til at vi kvadrerte summen for å unngå kanseleringer. Dette introduserte også en effekt der variansen er vanskelig å tolke, ettersom den eksisterer på en kvadrert skala. Hva om vi bare tar kvadratroten av resultatet?

$$
\begin{equation}
\sigma(V) = \sqrt{Var(V)}
\end{equation}
$$

Her er $\sigma(V)$ standardavviket til vekten. Fordelen med standardavvik over varians er at den eksisterer på samme størrelsesorden som variabelen vi jobber med. Så den er mye enklere å tolke.

Nå kan vi også ta i bruk standardavviket i variablene våre for å normalisere kovarians til en [-1,  1] skala:

$$
\begin{equation}
r = \frac{Cov(V, H)}{\sigma_V \sigma_H} 
\end{equation}
$$

Her er $\sigma_V$ og $\sigma_H$ standardavviket til henholdsvis vekt og høyde i dataen vår. Dette tallet $r$ kalles korrelasjonen mellom de to variablene. I python kan vi beregne den slik:
```python
df_w2h['weight'].corr(df_w2h['height'])
```

```
np.float64(0.7246963843238032)
```

Dette er et normalisert tall, hvor 1 ville vært en perfekt korrelasjon mellom vekt og høyde. Med 0.72 som korrelasjon her har vi altså sterk korrelasjon mellom vekt og høyde i vår data!





## Endringslogg
Under følger en endringslogg som viser hvilke deler av denne posten som er endret til hvilket tid. Enkle skrivefeil og den slags ting vil ikke bli logget, men jeg vil etterstrebe å logge alle meningsfulle endringer i posten.
