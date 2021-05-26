Title: Comparaisons base R, dplyr et data.table
Author: Antoine
Date: '2021-03-30'
Slug: comparaisons-base-dplyr-datatable
Category: R
Tags: R, Rstats, dplyr, data.table, benchmark
Cover: images/cover_1.png
twitter_image: images/cover_1.png
Summary: Comparaisons des temps d'exécution de base R, dplyr et data.table sur quelques cas d'étude

Cet article est une mise à jour de l'article du [blog d'Antoine](https://antoinesir.rbind.io/post/comparaisons-base-r-dplyr-data-table/) réalisé en 2018. L'idée est de comparer les performances de trois alternatives dans R pour l'analyse de données :   
- l'utilisation des seules fonctions de base R  
- dplyr  
- data.table  

[TOC]  

# Rappels sur dplyr et data.table
On rappelle ici les principales caractéristiques de ces packages mais pour se former à leur utilisation on peut se référer au [cours de perfectionnement de Martin Chevalier](https://teaching.slmc.fr/perf/presentation_handout.pdf). Pour une exploration de ce qu'englobe le `tidyverse` et notamment une présentation des commandes de `dplyr`, vous pouvez jeter un oeil à [l'introduction à R et au tidyverse](https://juba.github.io/tidyverse/index.html) de J. Barnier. Enfin pour data.table, on trouve des informations utiles sur le cours [Manipulations avancée avec data.table](http://larmarange.github.io/analyse-R/manipulations-avancees-avec-data-table.html) de J. Larmarange et on vous conseille l'excellent article [a gentle introduction to data.table](https://atrebas.github.io/post/2020-06-17-datatable-introduction/).  

## dplyr et le tidyverse  

Le `tidyverse` (contraction de "tidy" et "universe") est un concept initié par Hadley Wickham, chef statisticien de RStudio. Il regroupe un ensemble de packages utiles au traitement statistique et au nettoyage de bases de données. On va s'intéresser ici presque seulement au package `dplyr` (dont les instructions seront appliquées aux `tibbles`, un format de data.frame issu du `tidyverse`), mais vous pouvez parcourir les packages proposés dans le tidyverse sur [le site officiel](https://www.tidyverse.org/).  
  
`dplyr` propose un ensemble d'opérations de traitement de données sous une syntaxe différente de celle utilisée dans les fonctions de base de R. Ce langage présente le double avantage d'être à la fois lisible pour quelqu'un habitué aux langages tels que SAS ou SQL et de proposer des fonctions optimisées qui présentent de bonnes performances en termes de temps d'exécution. La grammaire `dplyr` s'appuie en effet sur des fonctions au nom explicite :  

* `mutate(data, newvar1=fonction(var1,var2...))` et `transmute(data, newvar1=fonction(var1,var2...))` créent de nouvelles variables
* `filter(data, condition)` sélectionne au sein d'une table certaines observations, à la manière de `where` dans SAS.
* `arrange(data, var1, descending var2,...)` trie une base selon une ou plusieurs variables (l'équivalent d'une `proc sort`).
* `select(data, var1 : varX)` sélectionne certaines variables dans une base, à la manière de `keep` dans SAS. 
* `summarise(data, newvar1=mean(var1), newvar2=sum(var2))` réalise toute sorte d'opérations statistiques sur une table.
* `group_by(data, var)` regroupe une table par une variable
* et bien d'autres...  
  
Un aspect pratique de ce langage est que toutes ces opérations peuvent être chaînées à l'aide de l'opérateur `%>%` ("pipe" en anglais, issu du package `magrittr`) dont la syntaxe est la suivante : `data %>% fonction(...)` est équivalent à `fonction(data, ...)`. Cette syntaxe permet de chaîner un grand nombre d'opérations sur une base commune, en limitant le nombre de fois où l'on écrit des tables intermédiaires tout en conservant une grande lisibilité du code. Ce petit exemple vous en convaincra peut-être :  

```R
library(dplyr)
# on crée un data frame avec 100 lignes, 
# chaque individu appartenant à un des 50 groupes
df <- data.frame(id1 = c(1:100), idgpe = sample(50), replace = TRUE)

# on y applique les instructions de dplyr
df %>% as_tibble() %>% mutate(var = rnorm(100)) %>% 
  group_by(idgpe) %>% summarise(var_mean = mean(var)) -> output_tibble
```

Un regard peu habitué contesterait peut-être l'aspect très lisible de l'instruction, mais ça l'est réellement. Le déroulé est le suivant :  

1) on transforme notre data.frame en tibble (pour rappel : format optimisé de data.frame pour dplyr) avec `as_tibble`  
2) on crée une variable `var` avec `mutate`  
3) on agrège par  `idgpe` avec `group_by`   
4) on calcule la moyenne de `var` avec `summarise`, que l'on stocke dans `var_mean`. Comme cette instruction suit un group_by, elle est réalisée à l'intérieur de chaque groupe (défini par `idgpe`), sinon elle aurait été réalisé sur l'ensemble de la table.    

Tout cela est stocké dans une table output_tibble, qui est ainsi un tibble agrégé par `idgpe` et qui a donc 50 lignes. L'intérêt de ce chaînage est qu'il permet une économie de code et d'écriture d'éventuelles tables intermédiaires.  

## Data.table  

Le package `data.table` ne prétend pas, contrairement au `tidyverse`, proposer une syntaxe concurrente à base R mais enrichir celle-ci. Il est axé autour d'un nouveau format d'objet, le data.table, qui est un type de data.frame qui permet une utilisation optimisée de l'opérateur `[`.  
Tout data.frame peut être converti en data.table grâce à la fonction `as.data.table`, ou, de manière plus optimale pour l'utilisation de la mémoire, grâce à la fonction `setDT` qui permet de directement transformer la nature de l'objet sans avoir à en écrire un autre. Il est important d'avoir en tête qu'un data.frame converti en data.table conserve les caractéristiques d'un data.frame. Cependant, l'opérateur `[` appliqué au data.table change de signification et devient :  

```R
DT[i,j,by]
```

Avec `i` qui permet de sélectionner des observations (sans avoir besoin de répéter le nom de la base dans laquelle on se trouve), `j` qui permet de créer ou sélectionner des variables et `by` de regrouper les traitement selon les modalités d'une variable définie. Comme dans `dplyr`, il est possible de chaîner les opérations réalisées comme le montre l'exemple suivant, qui reprend le même cas de figure que celui illustrant le package `dplyr` :  

```R
library(data.table) 
# on convertit notre data frame précédemment créé en data.table
dt <- as.data.table(df)

# on y applique les même instructions
dt[, list(var_mean = mean(rnorm(100))), by = list(idgpe = idgpe)] -> output_dt
```  

Le fait de renseigner les variables au sein de `list()` permet d'avoir une table en sortie au niveau de `idgpe` (donc 50 observations), sans cela la variable est bien moyennée par groupe mais la table en sortie est toujours au niveau `id1` (100 observations).   

## Vitesses d'exécution  

Voilà donc pour les présentations! Allez, on montre le résultat d'un petit `microbenchmark` des deux juste pour voir : 

<table class="dataframe">
<caption>Temps d'exécution en microsecondes</caption>
<thead>
	<tr><th scope=col>expr</th><th scope=col>lq</th><th scope=col>mean</th><th scope=col>uq</th><th scope=col>neval</th></tr>
</thead>
<tbody>
	<tr><td>dplyr     </td><td>9.79270</td><td>13.23297</td><td>14.367579</td><td>100</td></tr>
	<tr><td>data.table</td><td>1.40644</td><td> 2.13729</td><td> 2.546176</td><td>100</td></tr>
</tbody>
</table>

Sur cet exemple, on voit un avantage clair à data.table! Mais on est sur une toute petite table en entrée. On va essayer de se rapprocher de cas plus concrets en s'intéressant à un exemple sur des bases plus importantes.  

# Comparaisons sur une étude de cas simple
Les avantages et inconvénients de ces deux packages sont à l'origine de nombreux débats. Vous pouvez vous en convaincre en suivant [cette discussion sur stackoverflow](https://stackoverflow.com/questions/21435339/data-table-vs-dplyr-can-one-do-something-well-the-other-cant-or-does-poorly). On peut quand même dégager deux compromis :   

* Le choix de l'un ou l'autre des packages dépend beaucoup de ce que l'on va en faire (types d'analyses, taille des données, profils des utilisateurs du code...).   
* Les deux packages sont plus intéressants que base R pour l'analyse de données, que ce soit en termes de facilité d'écriture ou de performances.   

Pour ce deuxième point, on va essayer de s'en convaincre ensemble avec ce petit exemple.

## Notre étude de cas
Pour cet exemple, on utilise les données du package de Hadley Wickham que l'on trouve dans `nycflights13`. En particulier, la base `flights` donne toutes les heures de départ et d'arrivée selon les aéroports de départ et d'arrivée ainsi que les retards au départ et à l'arrivée. La base `weather` donne elle des indications météo, heure par heure, dans chaque aéroport.   
Commençons par charger nos packages (n'oubliez pas de faire `install.packages("nom_pck")` avant si vous ne l'avez jamais fait) et nos données : 


```R
# Les packages nécessaires
library(tidyverse) # Regroupe différents packages, voir https://www.tidyverse.org/ 
library(data.table)
library(microbenchmark) # Pour les calculs de vitesse d'exécution
library(nycflights13) # Pour les données

# data.table pour tests avec data.table
flightsdt <- as.data.table(flights)
weatherdt <- as.data.table(weather)
```
    
## Moyenne des retards et fusion des tables  

Un rapide examen des bases vous montre que la première étape avant toute analyse est comme souvent de regrouper les éléments de flights par heure et aéroport de départ pour pouvoir les fusionner avec la table weather, qui donnent les indications météo minute par minute. On écrit cette instruction de  3 manières différentes :  

__En base R__  

```R
flights_time_hour <- aggregate.data.frame(
    list(arr_delay = flights$arr_delay, 
         dep_delay = flights$dep_delay), 
    list(time_hour = flights$time_hour, 
         origin = flights$origin), 
    mean)
output_base <- merge(weather, flights_time_hour, 
                     by = c("time_hour", "origin"), 
                     sort = FALSE)
```

__En dplyr__  

```R
flights %>% group_by(time_hour, origin) %>% 
  summarise(arr_delay = mean(arr_delay),
            dep_delay = mean(dep_delay)) %>% 
  ungroup() %>% 
  inner_join(weather, by = c("time_hour", "origin")) -> output_dplyr 
```

__En data.table__  

```R
output_DT <- merge(flightsdt[, list(arr_perc_delay = mean(arr_delay),
                                    dep_perc_delay = mean(dep_delay)), 
                             by = c("time_hour", "origin")], 
                   weatherdt, 
                   by = c("time_hour", "origin"))
```

On utilise la fonction `merge` plutôt que `DT1[DT2, on = c("time_hour", "origin"), nomatch = 0]` car on a constaté qu'elle était plus rapide, conformément à ce que montre bien cet [article du Jozef's Rblog](https://jozefhajnala.gitlab.io/r/r006-merge/).  

## Comparaisons des vitesses d'exécution
Chacun jugera de la lisibilité de chacune de ces instructions, qui font toutes la même chose, car c'est finalement assez subjectif. On donne ici les résultats d'un `microbenchmark` de ces instructions : 

<table class="dataframe">
<caption>Temps d'exécution en millisecondes</caption>
<thead>
	<tr><th scope=col>expr</th><th scope=col>lq</th><th scope=col>mean</th><th scope=col>uq</th><th scope=col>neval</th></tr>
</thead>
<tbody>
	<tr><td>Base </td><td>1182.33161</td><td>1396.52780</td><td>1559.42968</td><td>10</td></tr>
	<tr><td>dplyr</td><td> 223.45642</td><td> 313.16457</td><td> 360.95388</td><td>10</td></tr>
	<tr><td>DT   </td><td>  22.83487</td><td>  24.68264</td><td>  26.32068</td><td>10</td></tr>
</tbody>
</table>


Les résultats sont très nettement en faveur des packages `dplyr` et `data.table`. Ce dernier est même assez largement le plus rapide, son avantage s'étant accru depuis la première version de cette article. Sans doute existe-t-il des moyens de plus optimiser l'instruction en base R, mais là n'est pas vraiment la question. On voit qu'avec une syntaxe simple et lisible, `dplyr` et `data.table` font beaucoup mieux en termes de vitesse d'exécution que les fonctions de base R. 

