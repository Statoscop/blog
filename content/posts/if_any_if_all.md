Title: if_any et if_all : appliquer la même condition sur plusieurs variables dans dplyr
Author: Antoine
Date: '2025-04-03'
Category: R
Tags: R, dplyr, tidyverse, if_any, if_all, across
Cover: images/cover_23.png
twitter_image: images/cover_23.png
Summary: Quelques exemples d'utilisation de `if_any` et `if_all` dans le traitement de vos données avec R.   

[TOC]


Au début de l'année 2021, [la sortie de dplyr 1.0.4](https://www.tidyverse.org/blog/2021/02/dplyr-1-0-4-if-any/) consacre l'arrivée de deux petits nouveaux : `if_any` et `if_all`. Ces deux fonctions viennent notamment __compléter le verbe `across`__, qui avait été introduit quelques mois plus tôt dans dplyr 1.0.0, et [que nous avons déjà présenté dans un précédent article](https://blog.statoscop.fr/fonctionnement-et-performances-dacross-dans-dplyr.html).  
Ici nous vous proposons donc un rapide tour des possibilités offertes par ces fonctions. Nous commençons par montrer comment ils peuvent être utilisés dans `filter()` pour filtrer facilement notre dataframe __en appliquant la même condition à plusieurs variables__. Puis nous montrons comment ils peuvent être avantageusement __utilisés dans des instructions `mutate()` pour synthétiser les valeurs de plusieurs variables__. 


# Syntaxe de `if_any` et `if_all`  

La syntaxe de ces deux verbes est exactement la même que celle d'`across()` :  

```r
if_any(.cols, .fns, ...)
if_all(.cols, .fns, ...)
```

Comme pour `across`, __le paramètre `.cols` permet de sélectionner les variables__ sur lesquelles on souhaite appliquer notre filtre. Pour cela on utilise les méthodes du `tidyselect` :  
- `where(fn)` pour sélectionner les colonnes sur une condition (`where(is.numeric)` par exemple).  
- `starts_with(match)`, `ends_with(match)`, `contains(match)`, `matches(match)` pour sélectionner les variables sur une caractéristique de leur nom (`starts_with("v_")` par exemple).   
- `c(var1, var2, var3)`, `all_of(c("var1", "var2", "var3"))` ou `any_of(c("var1", "var2", "var3"))` pour sélectionner directement les variables par leur nom.   

Le __paramètre `.fns` permet de définir la fonction de filtrage__ sur les variables sélectionnées. Ce sera soit une fonction déjà existante (`is.na` par exemple), soit une fonction définie en par l'utilisateur en amont ou à la volée (avec la syntaxe `~ .x > 50` par exemple).   

Ces deux fonctions vont __créer en sortie un vecteur booléen__ qui va nous permettre de filtrer nos observations de deux façons :   

- avec `if_any` si la condition définie dans `.fns` est respectée pour au moins une des variables. Cela revient à __coder la condition pour chaque colonne avec l'opérateur ` | ` (`OU`).__   
- avec `if_all` si cette condition est respectée pour toutes les variables. Cela revient à __coder la condition pour chaque colonne avec l'opérateur ` & ` (`ET`).__ 

Voyons concrètement comment nous pouvons les utiliser dans deux cas distincts : __filtrer notre dataframe__ sur une conditions portant sur plusieurs variables à la fois et __créer une nouvelle variable en fonction des valeurs de plusieurs autres variables__.    

> 👋 Nous c'est Antoine et Louis de Statoscop, une coopérative de statisticiens / data scientists.
> Vous voulez en savoir plus sur ce que l'on fait?
<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>        


# Filtrer un dataframe avec `if_any` et `if_all`  

Pour __illustrer l'utilisation de ces deux verbes__, on s'appuie sur [des données Kaggle de notes d'étudiants à différentes matières](https://www.kaggle.com/datasets/simranjitkhehra/student-grades-dataset). On a modifié ces données pour avoir __une colonne pour chaque note de chaque matière__. Notre dataset `stud_grades` a maintenant ce format :  


```
## Rows: 10,000
## Columns: 6
## $ student.id         <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, …
## $ math               <chr> "F", "F", "F", "D", "C", "A", "D", "F", "A", "F", "…
## $ science            <chr> "C", "A", "A", "B", "C", "C", "F", "F", "C", "A", "…
## $ history            <chr> "A", "B", "A", "F", "C", "C", "A", "A", "D", "B", "…
## $ english            <chr> "F", "C", "A", "D", "A", "C", "D", "C", "F", "F", "…
## $ physical_education <chr> "C", "A", "C", "B", "C", "F", "B", "B", "B", "F", "…
```

Les notes possibles sont `A`, `B`, `C`, `D` et `F`.   

On présente plusieurs opérations de filtrage qui peuvent être utiles dans notre exploration des données.  

## Repérer des valeurs en particulier avec `if_any`  

La première question que l'on peut se poser est de savoir s'il manque des informations dans notre dataset. `if_any` va nous permettre de renvoyer les lignes contenant __au moins une valeur manquante__ parmi les variables que l'on sélectionne :  


```r
stud_grades |> 
  filter(if_any(c(math:physical_education), is.na)) |> 
  print()
```

```
## # A tibble: 0 × 6
## # ℹ 6 variables: student.id <int>, math <chr>, science <chr>, history <chr>,
## #   english <chr>, physical_education <chr>
```

On voit tout de suite que le dataframe ne contient pas de valeurs manquantes, d'une manière plus directe et élégante que de coder la condition `is.na(math) | is.na(science) | is.na(history) | is.na(english) | is.na(physical_education)`.   

Dans la même logique on pourrait vouloir identifier __les cas où des élèves ont reçu la note "F"__ :  


```r
stud_grades |> 
  # On définit ici notre fonction "à la volée" avec ~ function(.x)
  filter(if_any(c(math:physical_education), ~ .x == "F")) |> 
  # on renvoie le nombre d'observations
  nrow()
```

```
## [1] 6749
```

On apprend ainsi que 6749 élèves sur 10 000 ont eu au moins un F à une des matières!  

## Repérer les très bons (ou les très mauvais) résultats avec `if_all`  

Avec `if_all`, on peut facilement repérer les excellents résultats en filtrant par exemple sur les élèves __ayant eu des "A" à toutes les matières__ :  


```r
stud_grades |> 
  # notez qu'on sélectionne les mêmes colonnes que précédemment
  # avec une méthode différente
  filter(if_all(where(is.character), ~ .x == "A")) |> 
  print()
```

```
## # A tibble: 2 × 6
##   student.id math  science history english physical_education
##        <int> <chr> <chr>   <chr>   <chr>   <chr>             
## 1       6933 A     A       A       A       A                 
## 2       8749 A     A       A       A       A
```
On voit ainsi directement que seulement deux élèves ont réussi cet exploit, avec une syntaxe bien plus directe que de coder `math == "A" & science == "A" & history == "A" & english == "A" & physical_education == "A"`.    

On peut bien sûr également __repérer les élèves en difficulté__, en regardant ceux qui n'ont eu que des "F" à toutes les matières.

# Créer une variable synthétique avec `if_any` et `if_all` dans `case_when`  

On conclut cet article en vous montrant __comment profiter avantageusement de ces verbes dans une instruction `mutate()`__, dans le but par exemple de __créer une variable synthéthique__ qui prenne en compte l'ensemble de ces notes. Les modalités pourraient être les suivantes :  

1 - _Facilités en sciences et en lettres_ (au moins B dans toutes ces matières)  
2 - _Facilités en sciences, difficultés en lettres_ (au moins B en maths et science, C ou moins dans une matière littéraire)  
3 - _Facilités en lettres, difficultés en sciences_ (au moins B en histoire et anglais, C ou moins dans une matière scientifique)  
4 - _Difficultés en lettres et sciences_ (jamais mieux que C dans aucune de ces 4 matières)   
5 - _Difficultés surtout en sciences_ (jamais mieux que C dans une matière scientifique)   
6 - _Difficultés surtout en lettres_ (jamais mieux que C dans une matière littéraire)   
7 - _Résultats irréguliers_ (aucune des conditions précédentes : donc bons résultats dans seulement au moins une des matières scientifiques et littéraire)   

Ces conditions s'écrivent ainsi :  


```r
stud_grades <- stud_grades |> 
  mutate(
    lvl_sc_lit = case_when(
      if_all(c(math:english), ~ .x %in% c("A", "B")) ~ "Facilités en sciences et en lettres",
      
      if_all(c(math, science), ~ .x %in% c("A", "B")) & 
               if_any(c(history, english), ~ .x %in% c("C", "D", "F")) ~ "Facilités en sciences, difficultés en lettres",
      
      if_all(c(history, english), ~ .x %in% c("A", "B")) & 
               if_any(c(math, science), ~ .x %in% c("C", "D", "F")) ~ "Facilités en lettres, difficultés en sciences",
      
      if_all(c(math:english), ~ .x %in% c("C", "D", "F")) ~ "Difficultés en lettres et sciences", 
      
      if_all(c(math, science), ~ .x %in% c("C", "D", "F")) & 
        if_any(c(history, english), ~ .x %in% c("A", "B")) ~ "Difficultés surtout en sciences", 
      
      if_all(c(history, english), ~ .x %in% c("C", "D", "F")) & 
        if_any(c(math, science), ~ .x %in% c("A", "B")) ~ "Difficultés surtout en lettres",
      
      TRUE ~ "Résultats irréguliers"))
```

On voit ainsi __qu'on peut combiner des conditions de `if_any` et `if_all` avec les `&` et `|`__, ce qui permet une grande flexibilité dans la création de nos conditions. On utilise également le paramètre `TRUE` de `case_when` pour tous les cas ne tombant dans aucune des conditions que nous avons imaginées, c'est à dire les élèves n'ayant pas de profil particulièrement littéraire ou scientifique mais plutôt des résultats très irréguliers dans les différentes matières. Voici comment se répartissent nos élèves dans cette nouvelle variable synthétique :     


```
## # A tibble: 7 × 2
##   lvl_sc_lit                                        n
##   <chr>                                         <int>
## 1 Difficultés en lettres et sciences             1334
## 2 Difficultés surtout en lettres                 1790
## 3 Difficultés surtout en sciences                1678
## 4 Facilités en lettres, difficultés en sciences  1334
## 5 Facilités en sciences et en lettres             259
## 6 Facilités en sciences, difficultés en lettres  1337
## 7 Résultats irréguliers                          2268
```


C'est tout pour aujourd'hui! On espère que cette note vous permettra de mieux exploiter la puissance de ces deux petits verbes bien pratiques. Si vous avez besoin de [conseils en programmation pour l'analyse de données](https://statoscop.fr), n'hésitez pas à continuer votre navigation sur notre site ou à nous suivre sur [BlueSky](https://bsky.app/profile/statoscop.fr) et [Linkedin](https://www.linkedin.com/company/statoscop). Pour retrouver l'ensemble du code ayant servi à générer cette note, vous pouvez vous rendre sur le [github de Statoscop](https://github.com/Statoscop/notebooks-blog).  



<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>  