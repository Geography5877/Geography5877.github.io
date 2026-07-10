---
title: Separate kjønn og multippel regressjon
date: 2026-07-10 06:57:00 +0200
categories: [Maskinlæring og statistikk]
tags: [prediksjon, ml, statistikk, lineær regresjon]     # TAG names should always be lowercase
math: true
media_subpath: /assets/images/2026-07-10/
---

<!-- markdownlint-capture -->
I [forrige post](https://enklypesalt.com/posts/how-good-is-the-fit/) så vi på hvordan vi kan evaluere hvor godt en modell passer til dataen den er trent på. Vi fikk også evaluert modellen vi har så langt ved å bruke denne metoden, men kan vi gjøre en enda bedre jobb?

La oss igjen se på dataen vi har:
![text](weight_height_dataset.png)
_Her plotter vi vekt (x-aksen) mot høyde (y-aksen)._

Så langt har vi jo sett på dataen vår uten å ta hensyn til at vi også har informasjon om kjønnet til hver måling. Heldigvis har vi også tilgang til denne. Hvis vi fargelegger dataen vår basert på kjønn får ser datasettet vårt slik ut:

![text](weight_height_dataset_gendered.png)
_Vi plotter igjen vekt (x-aksen) mot høyde (y-aksen)._

Her ser vi noe interessant. Nemlig at menn generelt veier mer enn kvinner. Samtidig ser vi også at kvinner ser ut til å være tettere samlet enn menn. Altså ser vi at det er større spredning i y-aksen blant menn enn det er hos kvinner. Dette er jo informasjon vi bure kunne bruke i modellen vår. Om vi vet at en vektmåling kommer fra en mann, så bør vi faktisk være mindre sikre på høyden om vi vet at den kommer fra en kvinne. Hvordan går vi frem for å bygge en modell for dette?

Jo den enkleste tilnærmingen er å modellere kvinner og menn med forskjellige modeller. Hvis vi så måler vekten til en mann, bruker vi mannemodellen og hvis vi måler vekten til en kvinne bruker vi kvinnemodellen. La oss prøve det først:

Først trener vi en modell for menn:
```python
import statsmodels.api as sm
X_carousel = sm.add_constant(male.weight)
y_carousel = male.height
model = sm.OLS(y_carousel, X_carousel).fit()
print(model.summary())
```

Da får vi denne modellen:
```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                 height   R-squared:                       0.204
Model:                            OLS   Adj. R-squared:                  0.196
Method:                 Least Squares   F-statistic:                     25.36
Date:                Fri, 10 Jul 2026   Prob (F-statistic):           2.14e-06
Time:                        14:51:12   Log-Likelihood:                -444.29
No. Observations:                 101   AIC:                             892.6
Df Residuals:                      99   BIC:                             897.8
Df Model:                           1                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const         90.1413     18.154      4.965      0.000      54.119     126.163
weight         1.1318      0.225      5.036      0.000       0.686       1.578
==============================================================================
Omnibus:                        1.856   Durbin-Watson:                   2.208
Prob(Omnibus):                  0.395   Jarque-Bera (JB):                1.692
Skew:                          -0.316   Prob(JB):                        0.429
Kurtosis:                       2.938   Cond. No.                         741.
==============================================================================
```

Deretter kan vi trene en modell for kvinner:

```python
import statsmodels.api as sm
X_carousel = sm.add_constant(male.weight)
y_carousel = male.height
model = sm.OLS(y_carousel, X_carousel).fit()
print(model.summary())
```

Da får vi denne modellen:

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                 height   R-squared:                       0.487
Model:                            OLS   Adj. R-squared:                  0.483
Method:                 Least Squares   F-statistic:                     112.0
Date:                Fri, 10 Jul 2026   Prob (F-statistic):           8.21e-19
Time:                        14:51:12   Log-Likelihood:                -452.22
No. Observations:                 120   AIC:                             908.4
Df Residuals:                     118   BIC:                             914.0
Df Model:                           1                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const         90.3801      5.976     15.124      0.000      78.546     102.214
weight         0.9801      0.093     10.584      0.000       0.797       1.163
==============================================================================
Omnibus:                        0.193   Durbin-Watson:                   1.566
Prob(Omnibus):                  0.908   Jarque-Bera (JB):                0.178
Skew:                          -0.088   Prob(JB):                        0.915
Kurtosis:                       2.935   Cond. No.                         400.
==============================================================================
```

Som vi kan se, ser det ut til at begge modellene har så og si det samme kryssningspunktet med y-aksen. Dette ser vi ved å sammenligne coefficienten 'const' i modellene.  Der modellene skiller seg fra hverandre er i vektkoeffisienten. Her ser vi at modellen mener at høyere vekt tilsvarer en høyere mann enn en kvinne. Det er jo ikke helt urimelig det. Men la oss også se på $R^2$ metrikken for modellene når vi nå først er i gang. I [forrige post](https://enklypesalt.com/posts/how-good-is-the-fit/) så vi hvordan vi kunne beregne denne ved bruk av kode, men om vi tar en nærmere titt på outputten for modellene vår kan vi se at vi har 'R-squared' helt oppe til høyre. Dette er faktisk den samme $R^2$ metrikken vi har beregnet tidligere. Vi fikk den altså gratis ut da vi brukte dette biblioteket. Okay, så om vi nå sammenligner $R^2$ for menn og kvinner, så ser vi at menn har betraktelig lavere $R^2$ enn kvinner. Hvorfor det?

Jo det kommer nettopp av denne spredningen vi observerte tidligere. $R^2$ er et mål på hvor mye av variansen som forklares av modellen vår. Siden det er mye mer varians blant menn enn kvinner, og modellene fortsatt er nokså like, så er det ikke så rart at vi får en lavere $R^2$ for menn. Men må vi egentlig trene to modeller? Kunne vi ikke bare trent én modell, men latt den også få vite kjønnet for hvert datapunkt?

Jo det kan vi selvfølgelig! For å gjøre det kan vi inkludere enda en variabel i modellen vi allerede har. Modellen vi har jobbet med til nå har sett slik ut:

$$
\begin{equation}
Y = \beta_0 + \beta_1 X
\end{equation}
$$

Hvor $\beta_0$ er kryssningspunktet med y-aksen og $\beta_1$ er stigningstallet til linjen vår. Vi kan nå utvide modellen slik at den blir seende slik ut:

$$
\begin{equation}
Y = \beta_0 + \beta_1 X_1 + \beta_2 X_2
\end{equation}
$$

Her putter vi inn vektmålingen vår i $X_1$ og kjønnet i $X_2$. Slik vi har satt opp dataen vår er kjønnet gitt av en binær variabel. Den kan altså kun være 0 eller 1. Om vi ser bort ifra hvor realistisk dette er, så kan vi si at variablen skrur av og på $\beta_2$. Hvis, $X_2 = 0$, så har $\beta_2$ ingen påvirkning på modellen vår, siden $0 \times \beta_2 = 0$. Men hvis $\X_2 = 1$, så får $\beta_2$ full effekt. La oss trene en slik modell:

```python
import statsmodels.api as sm
X_carousel = sm.add_constant(df_w2h[['weight', 'is_male']])
y_carousel = df_w2h.height
model = sm.OLS(y_carousel, X_carousel).fit()
print(model.summary())
```

```
                            OLS Regression Results                            
==============================================================================
Dep. Variable:                 height   R-squared:                       0.558
Model:                            OLS   Adj. R-squared:                  0.554
Method:                 Least Squares   F-statistic:                     137.4
Date:                Fri, 10 Jul 2026   Prob (F-statistic):           2.46e-39
Time:                        14:51:12   Log-Likelihood:                -917.94
No. Observations:                 221   AIC:                             1842.
Df Residuals:                     218   BIC:                             1852.
Df Model:                           2                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const         86.7535      6.985     12.420      0.000      72.987     100.520
weight         1.0371      0.107      9.656      0.000       0.825       1.249
is_male       10.9980      2.751      3.998      0.000       5.577      16.419
==============================================================================
Omnibus:                       10.215   Durbin-Watson:                   2.076
Prob(Omnibus):                  0.006   Jarque-Bera (JB):               13.336
Skew:                          -0.339   Prob(JB):                      0.00127
Kurtosis:                       3.994   Cond. No.                         497.
==============================================================================
```

Nå kan vi også ta en titt på modellene våre ved å plotte linjene de korresponderer til. Under har jeg gjort nettopp det:

La oss igjen se på dataen vi har:
![text](three_ols_models.png)
_Tre modeller, men 4 linjer. Oransje og blå linje korresponderer til hver sin separate modell, mens den lilla og den svarte linja korresponderer til henholdsvis $\X_2 = 0$ og $X_2 = 1$_

Hvis vi ser på linjene ser vi at de to modellene som er trent hver for seg, oransje og blå, ser ut til å starte fra så og si samme punkt, men de har forskjellig stigningstall. Dette stemmer veldig godt med det vi så tidligere da vi så på koeffisientene til de to modellene. Men når vi ser på den kombinerte modellen, ser vi noe annet. Her ser vi at stigningstallet ser helt likt ut, mens hvor linjene krysser y-aksen er forskjellig. Hva har skjedd her?

Jo, når vi trener en slik kombinert modell med en binær variabel skjer det noe litt interessant. Vi tvinger modellen til å finne ett stigningstall som skal passe til begge kjønnene. Hus, vi har bare $\beta_1$ som varierer med $X_1$. Vi sier også at koeffisienten til den binære variabelen, $X_2$, gir oss hvor stor forskjellen i høyde mellom mann og kvinne er. Dette kommer direkte til syne gjennom endring i hvor linjen krysser y-aksen. Det blir kanskje enklere å se dette om vi omformulerer funksjonen for linja vår litt. Vi starter med:

$$
\begin{equation}
Y = \beta_0 + \beta_1 X_1 + \beta_2 X_2
\end{equation}
$$

Men siden, $X_2$ kun kan være $0$ eller $1$, så kan vi slå den sammen med $\beta_0$ for å lage en ny koeffisient $\tilde{\beta}_0 = \beta_0 + \beta_2 \times X_2$. Hvis $\X_2 = 0$ får vi $\tilde{\beta}_0 = \beta_0$, men hvis $X_2 = 1$ får vi $\tilde{\beta}_0 = \beta_0 + \beta_2$. Altså, får vi nettopp den forskyvningen av kryssningspunktet på y-aksen vi ser! Men hvordan kan vi si hvilken modell som er best?

  Jo, vi kan igjen bruke $R^2$ verdien, men nå er det litt vanskelig her. Den kombinerte modellen har jo én $R^2$ verdi som korresponderer til begge kjønn, mens hos de separate modellene har vi én $R^2$ verdi per kjønn. Hvordan kan vi kombinere disse? Heldigvis er ikke det så vanskelig. For å kombinere $R^2$ verdiene, så beregner vi en ny $R^2$ verdi som kombinerer begge de separate modellene. Det kan vi gjøre slik:
  
$$
\begin{equation}
R^2_{comb} = 1 - \frac{SS_{Mres} + SS_{Fres}}{SS_{tot}}
\end{equation}
$$

Vi legger altså bare sammen kvadratsummene for hver av modellene før vi deler på den totale kvadratsummen. Igjen kan vi bruke python til å gjøre det for oss:

```python
# 1. Get the total unexplained error from both models combined
total_ssr = model_m.ssr + model_f.ssr

# 2. Calculate the total variance of the entire height dataset
# We subtract the global mean from every actual height, then square and sum them
global_mean = df_w2h['height'].mean()
total_sst = ((df_w2h['height'] - global_mean) ** 2).sum()

# 3. Calculate the combined R-squared
combined_r2 = 1 - (total_ssr / total_sst)

print(f"Combined Separate Models R2: {combined_r2:.4f}")
```

```
Combined Separate Models R2: 0.5586
```

Vi finner altså en $R^2$ for separate modeller som er så og si nøyaktig den samme som for den kombinerte modellen! Så har vi funnet den beste modellen da?

La oss først ta en titt på residualene våre:
![text](residual_plot.png)
_Residualene for modellene vi har trent (de to separate er kombinert i én modell (oransje) her._

Nå kommer vi inn på en viktig del av OLS regressjon. I slike modeller antar vi at variansen residualene våre er lik i dataen vår. Men som vi ser av residualene over er ikke det tilfellet her. Vi har jo faktisk sett at menn har betraktelig høyere varians enn kvinner. Så en grunnleggende antakelse i modellen vår er brutt. Når vi jobber med data som ikke har lik varians i feilleddene kaller vi dataen Heteroskedastisk.

Heldigvis kan vi vekte residualene når vi trener modellen slik at vi vekter menn og kvinner forskjellig underveis i treningen. Men hvordan finner vi vektene? Jo først starter vi fra den kombinerte modellen vi allerede har. Vi kan deretter gruppere residualene basert på kjønn og beregne variansen for disse gruppene. Vi kan så bruke denne variansen til å vinne vektene slik:

$$
\begin{equation}
\text{weight} = \frac{1 }{\text{estimated variance}}
\end{equation}
$$

I kode ser det slik ut:
```python
# Calculate the variance of the residuals for males and females separately
# This extracts the information straight from your data points!
group_variances = model.resid.groupby(df_w2h['is_male']).var()

# . Create the weight vector (Weight = 1 / Variance)
data_driven_weights = 1.0 / df_w2h['is_male'].map(group_variances)
print(data_driven_weights)
```

```
0      0.002550
1      0.002550
2      0.002550
3      0.002550
4      0.002550
         ...   
216    0.008998
217    0.008998
218    0.008998
219    0.002550
220    0.002550
```

Vi har altså at menn skal vektes med 0.00255, mens kvinner skal vektes 3-4 ganger så høyt, med 0.008998. Vi kan nå bruke disse vektene i minste kvadrat beregningen vår som vi så i en [tidligere post](https://enklypesalt.com/posts/linear-regression/): 

$$
\begin{equation}
\text{minimize} \sum_{females} 0.009(y_i - \hat{y}_i)^2 + \sum_{males} 0.003(y_i - \hat{y}_i)^2
\end{equation}
$$

I python ser det slik ut:

```python
X = sm.add_constant(df_w2h[['weight', 'is_male']])
y = df_w2h['height']

# Fit the final, optimized WLS model
wls_model = sm.WLS(y, X, weights=data_driven_weights).fit()
print(wls_model.summary())
```

```
                            WLS Regression Results                            
==============================================================================
Dep. Variable:                 height   R-squared:                       0.581
Model:                            WLS   Adj. R-squared:                  0.577
Method:                 Least Squares   F-statistic:                     151.1
Date:                Fri, 10 Jul 2026   Prob (F-statistic):           6.70e-42
Time:                        16:40:00   Log-Likelihood:                -896.71
No. Observations:                 221   AIC:                             1799.
Df Residuals:                     218   BIC:                             1810.
Df Model:                           2                                         
Covariance Type:            nonrobust                                         
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const         88.9742      5.532     16.082      0.000      78.070      99.878
weight         1.0022      0.086     11.716      0.000       0.834       1.171
is_male       11.5770      2.616      4.425      0.000       6.421      16.733
==============================================================================
Omnibus:                        1.666   Durbin-Watson:                   1.875
Prob(Omnibus):                  0.435   Jarque-Bera (JB):                1.575
Skew:                          -0.206   Prob(JB):                        0.455
Kurtosis:                       2.978   Cond. No.                         447.
==============================================================================
```

Her ser vi at den vektede modellen scorer best på $R^2$ metrikken av alle modellene vi har trent. Så har vi nå funnet den beste modellen? Ja, nå har vi faktisk det! 

Ettersom vi også har laget datasettet vår, så vet vi at det ikke er mulig å finne en bedre modell enn denne. Vi lagde datasettet slik:


```python
# Seed random generator for reproducibility
rng = np.random.default_rng(42)

# Set n samples we want
n_samples = 256

# Generate data
weights_m = rng.normal(80, 10, n_samples//2)
weights_f = rng.normal(65, 10, n_samples//2)

heights_m = weights_m + rng.normal(100, 20, n_samples//2)
heights_f = weights_f + rng.normal(90, 10, n_samples//2)

is_male = np.ones(n_samples, dtype=np.int8)
is_male[n_samples//2:]-=1

# Create dataframe from data
df_w2h = pd.DataFrame({"height": np.concatenate([heights_m, heights_f]), 
                       "weight": np.concatenate([weights_m, weights_f]),
                       "is_male": is_male})
df_w2h = df_w2h[~df_w2h['weight'].between(75, 80, inclusive='left')]
new_row = pd.DataFrame({'height': [130, 220], 'weight': [75.1, 79.9], 'is_male': [1.0, 1.0]})
df_w2h = pd.concat([df_w2h, new_row], ignore_index=True)
```

Her sier vi at det er ca 10 i cm forskjell mellom menn og kvinner i utgangspuntket, kun basert på kjønn. Dette ser vi i linjene hvor vi definerer "heights_m_" og "height_f". Her setter vi gjennomsnittlig forskjell i høyde som $(100-90) + \text{vekt} = 10  + \text{vekt}$. Det finner vi også igjen i modellen vår ved å kombinere 'const' og $\beta_2$/'is_male'. Da ser vi at kvinner har formelen $89.97 + \text{vekt}$, mens menn har $\89.97 + 11.58 + \text{vekt}$. Så og si nøyaktig det vi definerte da vi genererte dataen vår!

Vi ser også at modellen vår har funnet et stigningstall: $\beta_1 = 1.0022 \approx 1$. I datageneratoren vår har vi nettopp satt stigningstallet til $1$. Vi har bare ikke eksplisitt skrevet det. Den fulle funksjonen for dataen vår er derfor slik:


$$
\begin{equation}
Y = 1 \times \text{weight}_f + 90 + 10 \times \text{is_male} +  \text{støy}$
\end{equation}
$$

Og den vektede modellen vår fant
 
$$
\begin{equation}
Y = 1.0022 \times \text{weight}_f + 89.97 + 11.58 \times \text{is_male} +  \text{støy}$
\end{equation}
$$

Gitt måten vi har generert dataen på i disse eksemplene, har vi faktisk funnet den beste modellen! Men burde vi bruke denne modellen?

Det er et spørsmål vi ikke skal svare på her.

## Kode
Koden for denne posten finnes [her](https://github.com/haakom/enklypesalt/tree/main/2026-07-10_separate_genders).

## Endringslogg
Under følger en endringslogg som viser hvilke deler av denne posten som er endret til hvilket tid. Enkle skrivefeil og den slags ting vil ikke bli logget, men jeg vil etterstrebe å logge alle meningsfulle endringer i posten.
