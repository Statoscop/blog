Title: Les séries temporelles avec Python (1/4) - Introduction
Author: Louis
Date: '2021-05-04'
Slug: timeseries-1
Category: Python, Séries Temporelles
Tags: Python, Machine Learning, Statistiques, Data Science, Séries temporelles, Datetime
Cover: images/cover_3.png
Summary: Introduction à la manipulation de données temporelles avec Python

[TOC]

Cet article introductif est le premier d'une séquence de 4 posts sur les séries temporelles.   

1. **Introduction à la manipulation de données temporelles avec Python**
2. Visualisations et opérations sur les séries temporelles
3. Éléments théoriques et exemples
4. Analyse, modélisation et prédiction

Avant d'entrer dans le vif du sujet, nous allons donc nous pencher sur le fonctionnement, le stockage et la manipulation des données temporelles avec Python.  

La libraire `Pandas` a été développée dans un contexte de données financières et son nom est une contraction de *Panel Datas* (données de panel), c'est-à-dire des données pour lesquelles on a, pour un même individu, des observations au cours du temps. `Pandas` contient donc par essence de nombreux outils pour travailler avec les dates, le temps et des données indexées en fonction du temps.

> *"Une série temporelle, ou série chronologique, est une suite de valeurs numériques représentant l'évolution d'une quantité spécifique au cours du temps."*  
> <a href="https://fr.wikipedia.org/wiki/S%C3%A9rie_temporelle" target="_blank">Maître Wiki</a>

Une série temporelle peut être vue comme un cas très particulier des données panel puisqu'il s'agit de l'évolution d'une seule observation au cours du temps.

Nous allons donc nous intéresser à ces données temporelles et pour commencer, un peu de vocabulaire et d'anglicisme :  
>- **Timestamps** correspond à un moment précis (par exemple *03 juin 2020, 14:15:16*)  
>- **Time intervals** et **periods** correspondent à une durée ayant des dates de début et de fin précises (par exemple, *l'année 2020*)  
>- **Time deltas** ou **durations** correspondent à une durée exacte (par exemple *123.45 secondes*)  

On va voir comment utiliser et manipuler ces objets date/time avec Python d'une part, puis avec `Pandas` plus particulièrement.

# Les dates (et heures) avec Python

Python a de nombreuses représentations et formats possibles des dates, heures, durées...un petit tour d'horizon s'impose.

## Les packages `datetime` and `dateutil`

Les objet natifs Python pour les dates/times sont dans le module built-in `datetime`. En utilisant également le package `dateutil`, on peut facilement et rapidement effectuer bon nombre d'opérations sur objets temporels.

Par exemple, on peut construire une date manuellement.


```python
from datetime import datetime
datetime(year=2021, month=4, day=30)
```




    datetime.datetime(2021, 4, 30, 0, 0)



Ou avec le module `dateutil`, on peut lire différents format de dates à partir de chaînes de caractères.


```python
from dateutil import parser
d1 = parser.parse("30 of April, 2021, 14:00")
d2 = parser.parse("30/04/2021 14:15:16", dayfirst=True)
d1, d2
```




    (datetime.datetime(2021, 4, 30, 14, 0),
     datetime.datetime(2021, 4, 30, 14, 15, 16))



Une fois qu'on manipule un objet `datetime`, on peut facilement effectuer des opérations sur cet objet. Par exemple, récupérer en chaîne de caractères le jour de la semaine correspondant, le mois, l'année, etc...

Pour en savoir plus sur `strftime` et les codes standards de format de string pour afficher les dates ce sera dans la <a href="https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior" target="_blank">section strftime</a> de la doc Python <a href="https://docs.python.org/3/library/datetime.html" target="_blank">datetime</a>.
Une autre aide utile sur les manipulation de dates est la <a href="http://labix.org/python-dateutil" target="_blank">documentation en ligne de dateutil</a>.


```python
d1.strftime("%A"), d1.strftime("%a"), d1.strftime("%d"), d1.strftime("%B"), d1.strftime("%Y")
```




    ('Friday', 'Fri', '30', 'April', '2021')



__À savoir__ : il existe un package utile <a href="http://pytz.sourceforge.net/" target="_blank">`pytz`</a> contenant des outils pour pouvoir travailler avec les *timezones*.

L'intérêt de `datetime` et `dateutil` est leur flexibilité et leur facilité d'utilisation/syntaxe : on peut faire plus ou moins tout ce qu'on veut en utilisant les objets `datetime` et leurs méthodes built-in.

Un bémol c'est la gestion des grandes quantités de données : de la même manière que les listes numériques ne sont pas optimales comparées à des arrays `numpy` de type numérique, les listes d'objets `datetime` ne sont pas optimales comparées à des tableaux ayant un type date/time.

## Le type de données date des tableaux numpy : `datetime64`

Pour pallier cet écueil, un type de données temporelles natif a été ajouté à `numpy`. Le dtype `datetime64` encode les dates sous forme d'entiers 64-bits et par conséquent les tableaux ayant des données de ce type sont très compacts (pour rappel, un tableau numpy ne peut contenir qu'un seul type de données).

Le dtype `datetime64` nécessite toutefois un format particulier comme par exemple `"YYYY-MM-DD"` ou `"YYYY-MM"`. D'autre éléments peuvent être précisés et pour en savoir plus, c'est <a href="https://numpy.org/doc/stable/reference/arrays.datetime.html" target="_blank">sur la documentation numpy</a>.


```python
import numpy as np
date = np.array('2021-04-30', dtype=np.datetime64)
date
```




    array('2021-04-30', dtype='datetime64[D]')



Une fois qu'on a une date dans le bon format, on peut facilement effectuer des opérations vectorielles dessus :


```python
date + np.arange(7)
```




    array(['2021-04-30', '2021-05-01', '2021-05-02', '2021-05-03',
           '2021-05-04', '2021-05-05', '2021-05-06'], dtype='datetime64[D]')



Grâce au type de données uniforme dans les tableaux `datetime64`, ce genre d'opérations vectorielles peut être effectué bien plus rapidement qu'en travaillant directement avec les objets `datetime` de Python, particulièrement pour de grands tableaux.

Une particularité des objets `datetime64` et `timedelta64` est qu'ils sont construits sur une *unité fondamentale de temps (UFT)*. Comme le `datetime64` est limité à une précision de 64-bit, le nombre de datetime qu'on peut encoder est $2^{64}$ fois l'UFT. En d'autres termes, `datetime64` nécessite un arbitrage entre *résolution temporelle* et *intervalle maximal de temps* .  
Par exemple, si on veut une précision de 1 nanoseconde, alors on pourra avoir la place pour encoder $2^{64}$ nanosecondes, soit un peu moins de 600 ans.

`numpy` déduit l'UFT désirée à partir de l'input mais on peut bien sûr aussi le spécifier.


```python
# un datetime basé sur le jour
np.datetime64('2021-04-30')
```




    numpy.datetime64('2021-04-30')




```python
# un datetime basé sur la minute
np.datetime64('2021-04-30 14:15')
```




    numpy.datetime64('2021-04-30T14:15')




```python
# un datetime basé sur la nanoseconde en fixant l'unité fondamentale de temps
np.datetime64('2021-04-30 14:15:16', 'ns')
```




    numpy.datetime64('2021-04-30T14:15:16.000000000')



À noter aussi, la timezone est par défaut celle de l'ordinateur qui exécute le code.

Le tableau suivant issu de <a href="http://docs.scipy.org/doc/numpy/reference/arrays.datetime.html" target="_blank">la documentation numpy de datetime64</a> liste les codes disponibles ainsi que les échelles relatives et absolues qu'ils permettent d'encoder.

|Code  | Signification | Durée relative | Durée absolue           |
|------|---------------|----------------|-------------------------|
| `Y`  | Year          | ± 9.2e18 ans   | [9.2e18 av.J-C, 9.2e18] |
| `M`  | Month         | ± 7.6e17 ans   | [7.6e17 av.J-C, 7.6e17] |
| `W`  | Week          | ± 1.7e17 ans   | [1.7e17 av.J-C, 1.7e17] |
| `D`  | Day           | ± 2.5e16 ans   | [2.5e16 av.J-C, 2.5e16] |
| `h`  | Hour          | ± 1.0e15 ans   | [1.0e15 av.J-C, 1.0e15] |
| `m`  | Minute        | ± 1.7e13 ans   | [1.7e13 av.J-C, 1.7e13] |
| `s`  | Second        | ± 2.9e12 ans   | [ 2.9e9 av.J-C, 2.9e9]  |
| `ms` | Millisecond   | ± 2.9e9 ans    | [ 2.9e6 av.J-C, 2.9e6]  |
| `us` | Microsecond   | ± 2.9e6 ans    | [290301 av.J-C, 294241] |
| `ns` | Nanosecond    | ± 292 ans      | [ 1678, 2262]           |
| `ps` | Picosecond    | ± 106 jours    | [ 1969, 1970]           |
| `fs` | Femtosecond   | ± 2.6 heures   | [ 1969, 1970]           |
| `as` | Attosecond    | ± 9.2 secondes | [ 1969, 1970]           |

Dans la "vraie vie", on utilise généralement `datetime64[ns]` car cela permet d'encoder une échelle de temps avec des dates actuelles et une précision suffisament fine.

Finalement, on retiendra que le type `datetime64` règle certains défauts du type built-in de Python `datetime`, cependant il manque plusieurs des méthodes et fonctions bien utiles fournies par `datetime` et surtout `dateutil`.

## Dates et times avec `pandas`: le meilleur des 2 mondes

`Pandas` se base sur les outils vus à l'instant pour fournir un objet `Timestamp` qui combine la facilité d'utilisation de `datetime`/`dateutil` avec l'efficacité de stockage et de calcul vectoriel de``numpy.datetime64``.

À partir de ces objets `Timestamp`, `Pandas` peut construire une index `DatetimeIndex` qu'on peut utiliser pour indexer les données d'une `Series` ou d'un `DataFrame`.

Par exemple, on peut utiliser les outils de `Pandas` pour refaire les opérations qu'on a fait juste avant, à savoir: lire une chaîne de caractères contenant un format de date flexible, utiliser les codes de format pour récupérer le jour de la semaine ou encore effectuer des opérations vectorielles comme avec `numpy`.


```python
import pandas as pd
date = pd.to_datetime("30 of April, 2021")
date
```




    Timestamp('2021-04-30 00:00:00')




```python
date.strftime('%A')
```




    'Friday'




```python
print(date.day, date.day_name())
print(date.year)
print(date.month, date.month_name())
```

    30 Friday
    2021
    4 April



```python
date + pd.to_timedelta(np.arange(7), 'D')
# à noter ici l'utilisation de to_timedelta pour transformer 
# le tableau d'entiers en tableau de durées
```




    DatetimeIndex(['2021-04-30', '2021-05-01', '2021-05-02', '2021-05-03',
                   '2021-05-04', '2021-05-05', '2021-05-06'],
                  dtype='datetime64[ns]', freq=None)




```python
# ici on crée une liste d'entier entre 0 et 24 avec un intervalle de 6 
# que l'on transforme en durée en heures avant de les ajouter à notre timestamp
d = pd.to_datetime("30 of April, 2021, 14:00")
d + pd.to_timedelta(np.arange(0,25,6), 'h')
```




    DatetimeIndex(['2021-04-30 14:00:00', '2021-04-30 20:00:00',
                   '2021-05-01 02:00:00', '2021-05-01 08:00:00',
                   '2021-05-01 14:00:00'],
                  dtype='datetime64[ns]', freq=None)




```python
# un dernier exemple avant le suivant
d = pd.to_datetime("30 of April, 2021, 14:00")
d + pd.to_timedelta(np.arange(20, 71, 10), 'm')
```




    DatetimeIndex(['2021-04-30 14:20:00', '2021-04-30 14:30:00',
                   '2021-04-30 14:40:00', '2021-04-30 14:50:00',
                   '2021-04-30 15:00:00', '2021-04-30 15:10:00'],
                  dtype='datetime64[ns]', freq=None)



# Séries temporelles avec `pandas`

## L'indexation par le temps

L'intérêt des time series de `pandas` réside dans l'utilisation d'une indexation des données par des *timestamps*. On crée donc un objet `DatetimeIndex` pour ensuite indexer la série.


```python
index = pd.DatetimeIndex(['2020-03-30', '2020-04-30', '2021-03-30', '2021-04-30'])
data = pd.Series([0, 1, 2, 3], index=index)
data, data.index
```




    (2020-03-30    0
     2020-04-30    1
     2021-03-30    2
     2021-04-30    3
     dtype: int64,
     DatetimeIndex(['2020-03-30', '2020-04-30', '2021-03-30', '2021-04-30'], dtype='datetime64[ns]', freq=None))



Une fois qu'on a une `Series`, on peut utiliser les index datetime comme pour n'importe quel autre index et notamment avoir recours au *slicing*.


```python
data['2020-03-30':'2021-03-30']
```




    2020-03-30    0
    2020-04-30    1
    2021-03-30    2
    dtype: int64



Par ailleurs certaines opérations spécifiques aux `DatetimeIndex` permettent d'obtenir des slicing différents comme par exemple utiliser une année pour récupérer toutes les observations de cette année ou un date seuil pour récupérer toutes les données avant/après cette date.


```python
data['2020'], data['2020-04'], data['2020-05-01':]
```




    (2020-03-30    0
     2020-04-30    1
     dtype: int64,
     2020-04-30    1
     dtype: int64,
     2021-03-30    2
     2021-04-30    3
     dtype: int64)



## Les structures de données `pandas` pour les séries temporelles

Nous allons maintenant introduire les structures de données fondamentales de `pandas` pour travailler avec les séries temporelles :  
1. pour les *timestamps*, il y a le type `Timestamp` : l'idée est que ça remplace le type natif de Python `datetime` tout en étant construit sur le type `numpy.datetime64` qui est plus efficace  
2. pour les *time Periods*, il y a le type `Period` : il permet d'encoder des durées de fréquences fixes basées sur `numpy.datetime64`  
3. pour les *time deltas* ou *durations*, il y a le type`Timedelta` : c'est un remplaçant plus efficace du type natif de Python `datetime.timedelta` basé sur `numpy.timedelta64`  

Les structures d'index associées sont respectivement les `DatetimeIndex`, `PeriodIndex` et `TimedeltaIndex`. On retiendra que parmi ces différents objets, les structures de date/time les plus utilisées sont les `Timestamp` et `DatetimeIndex`.

Même si on peut très bien appeler ces classes d'objets directement, généralement on passe par la fonction `pd.to_datetime()` qui permet de lire une grande variété de formats de chaîne de caractères. Si on passe une seule date à `pd.to_datetime()`, on obtient un `Timestamp`. Si on lui passe une série de dates, on obtient un `DatetimeIndex`.


```python
dates = pd.to_datetime([datetime(2021, 4, 30), '6th of May, 2021', '2021-Jun-7', '06-10-2021', '20210429'])
dates
```




    DatetimeIndex(['2021-04-30', '2021-05-06', '2021-06-07', '2021-06-10',
                   '2021-04-29'],
                  dtype='datetime64[ns]', freq=None)



Tout objet `DatetimeIndex` peut être converti en `PeriodIndex` avec la fonction `to_period()` en ajoutant un code de fréquence (par exemple `'D'` pour une fréquence quotidienne ou `'M'` pour une fréquence mensuelle) :


```python
dates.to_period('M')
```




    PeriodIndex(['2021-04', '2021-05', '2021-06', '2021-06', '2021-04'], dtype='period[M]', freq='M')




```python
dates.to_period('D')
```




    PeriodIndex(['2021-04-30', '2021-05-06', '2021-06-07', '2021-06-10',
                 '2021-04-29'],
                dtype='period[D]', freq='D')



Enfin, un objet `TimedeltaIndex` peut être par exemple créé lorsqu'on soustrait 2 dates :


```python
dates - dates[1]
```




    TimedeltaIndex(['-6 days', '0 days', '32 days', '35 days', '-7 days'], dtype='timedelta64[ns]', freq=None)



## Les séquences avec `pd.date_range()`

Pour pouvoir créer des séquences régulières de dates, `pandas` contient un certain nombre de fonctions : `pd.date_range()`, `pd.period_range()` et `pd.timedelta_range()`.
Les `range()` de Python et `arange()` de numpy prennent comme paramètres un premier élément, un dernier élément (non-inclus) et éventuellement un pas.  
De la même manière, `pd.date_range()` prend une date de départ, une date de fin (qui elle est inclue !) et éventuellement une fréquence (qui vaut 1 jour par défaut).


```python
pd.date_range('2021-04-30', '2021-05-06')
```




    DatetimeIndex(['2021-04-30', '2021-05-01', '2021-05-02', '2021-05-03',
                   '2021-05-04', '2021-05-05', '2021-05-06'],
                  dtype='datetime64[ns]', freq='D')



On peut aussi spécifier simplement un point de départ et un nombre de Periods et on peut utiliser `freq` pour modifier la fréquence.


```python
# pour avoir 8 timestamps chacune correspondant à un jour à partir d'aujourd'hui
pd.date_range('2021-04-30', periods=8)
```




    DatetimeIndex(['2021-04-30', '2021-05-01', '2021-05-02', '2021-05-03',
                   '2021-05-04', '2021-05-05', '2021-05-06', '2021-05-07'],
                  dtype='datetime64[ns]', freq='D')




```python
# pour avoir 4 timestamps chacune correspondant à un décalage de 2 heures à partir d'aujourd'hui 14h
pd.date_range('2021-04-30 14h', periods=4, freq='2H')
```




    DatetimeIndex(['2021-04-30 14:00:00', '2021-04-30 16:00:00',
                   '2021-04-30 18:00:00', '2021-04-30 20:00:00'],
                  dtype='datetime64[ns]', freq='2H')



N'hésitez pas à jeter un oeil à `pd.period_range()` et `pd.timedelta_range()` qui fonctionnent de manière similaire.


```python
pd.period_range('2021-04', periods=8, freq='M')
```




    PeriodIndex(['2020-06', '2020-07', '2020-08', '2020-09', '2020-10', '2020-11',
                 '2020-12', '2021-01'],
                dtype='period[M]', freq='M')




```python
pd.timedelta_range(0, periods=10, freq='H')
```




    TimedeltaIndex(['00:00:00', '01:00:00', '02:00:00', '03:00:00', '04:00:00',
                    '05:00:00', '06:00:00', '07:00:00', '08:00:00', '09:00:00'],
                   dtype='timedelta64[ns]', freq='H')



Vous l'aurez compris, pour bien comprendre ce qu'il se passe et toutes les possibilités, il faut avoir une idée des codes de fréquences...

## Fréquences et décalages (offset)

Le concept de fréquence ou de décalage (on parlera généralement d'offset)  est fondamental pour les outils `pandas` de séries temporelles.
On a déjà croisé les codes `M` (month), `D` (day) et `H` (hour) pour définir des fréquences, on va résumé les codes Pandas dans le tableau suivant.

| Code   | Description EN        | Description     FR        |
|--------|-----------------------|---------------------------|
| ``D``  | Calendar day          | Jour de la semaine        |
| ``W``  | Weekly                | Semaine                   |
| ``M``  | Month end             | Fin du mois               |
| ``Q``  | Quarter end           | Fin du trimestre          |
| ``A``  | Year end              | Fin de l'année            |
| ``H``  | Hours                 | Heures                    |
| ``T``  | Minutes               | Minutes                   |
| ``S``  | Seconds               | Secondes                  |
| ``L``  | Milliseonds           | Milliseondes              |
| ``U``  | Microseconds          | Microsecondes             |
| ``N``  | nanoseconds           | nanosecondes              |
| ``B``  | Business day          | Jour ouvrable             |
| ``BM`` | Business month end    | Fin ouvrable de mois      |
| ``BQ`` | Business quarter end  | Fin ouvrable de trimestre |
| ``BA`` | Business year end     | Fin ouvrable d'année      |
| ``BH`` | Business hours        | Heures ouvrables          |

Les fréquences mensuelles, trimestrielles et annuelles pointent à la fin de la période par défaut. En ajoutant un suffixe `S` à la fin du nom, elles pointeront à la place au début de la période.

| Code    | Description EN         | Description FR              |
|---------|------------------------|-----------------------------|
| ``MS``  | Month start            | Début de mois               |
| ``QS``  | Quarter start          | Début de trimestre          |
| ``AS``  | Year start             | Début d'année               |
|``BMS``  | Business month start   | Début ouvrable de mois      |
|``BQS``  | Business quarter start | Début ouvrable de trimestre |
|``BAS``  | Business year start    | Début ouvrable d'année      |

On peut aussi modifier le mois utilisé pour marquer un code trimestriel ou annuel en ajoutant les 3 lettres du mois en suffixes:  
- ``Q-JAN``, ``BQ-FEB``, ``QS-MAR``, ``BQS-APR``, ...  
- ``A-JAN``, ``BA-FEB``, ``AS-MAR``, ``BAS-APR``, ...  

De la même manière, le "jour seuil" d'une fréquence hebdomadaire peut être modifié en ajoutant en suffixes les 3 lettres du jour:``W-SUN``, ``W-MON``, ``W-TUE``, ``W-WED``, etc.

Enfin, comme vu un peu plus haut, les codes peuvent être combinés avec des valeurs numériques pour spécifier d'autres fréquences. Par exemple, pour une fréquence de 2h30min, on peut faire:


```python
pd.timedelta_range(0, periods=9, freq="2H30T")
```




    TimedeltaIndex(['00:00:00', '02:30:00', '05:00:00', '07:30:00', '10:00:00',
                    '12:30:00', '15:00:00', '17:30:00', '20:00:00'],
                   dtype='timedelta64[ns]', freq='150T')



Tout ça provient en fait du module `pd.tseries.offsets`.


```python
# Par exemple, pour créer un décalage de jour ouvrable, on peut faire :
from pandas.tseries.offsets import BDay
pd.date_range('2021-04-30', periods=7, freq=BDay())
```




    DatetimeIndex(['2021-04-30', '2021-05-03', '2021-05-04', '2021-05-05',
                   '2021-05-06', '2021-05-07', '2021-05-10'],
                  dtype='datetime64[ns]', freq='B')




```python
# Par exemple, pour récupérer le dernier jour ouvrable du mois, on peut faire :
from pandas.tseries.offsets import BMonthEnd
pd.date_range('2021-04-30', periods=8, freq=BMonthEnd())
```




    DatetimeIndex(['2021-04-30', '2021-05-31', '2021-06-30', '2021-07-30',
                   '2021-08-31', '2021-09-30', '2021-10-29', '2021-11-30'],
                  dtype='datetime64[ns]', freq='BM')



Pour en savoir plus, il y a la <a href="https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#dateoffset-objects" target="_blank">section "DateOffset"</a> de la documentation `pandas`.

Vous trouverez, comme pour l'ensemble de nos posts, le code de ce notebook sur le <a href="https://github.com/Statoscop/notebooks-blog" target="_blank">github de Statoscop</a>. Dans le prochain article, nous commencerons à manipuler, visualiser et effectuer certaines opérations classiques sur les séries temporelles avant de conclure sur une petite étude de cas. À très vite !
