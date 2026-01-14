Title: Utilisation de case_when dans dplyr : cas des variables facteurs  
Author: Antoine
Date: '2025-09-16'
Category: R 
Tags: R, dplyr, case_when, code, Data Science,
Cover: images/cover_26.png
twitter_image: images/cover_26.png
Summary: Astuces pour bien définir sa variable facteur dans une instruction case_when suite aux dernières mises à jour de dplyr.

[TOC]

Le verbe `case_when` est un __incontournable du traitement de données avec `dplyr`__. Il permet de créer une variable conditionnellement à une ou plusieurs variables existantes. Sa syntaxe est __très lisible et permet à votre code de rester clair__. Cependant, vous avez peut-être déjà eu quelques __problèmes de compatibilité avec les variables facteurs__. Dans cette note, on vous présente ce verbe bien pratique de `dplyr`, ses nouveautés et comment __l'utiliser pour créer directement des variables facteurs__. 

# Syntaxe de case_when dans dplyr et différences avec if_else   

Pour cet article, nous nous reposons sur un dataset complètement fictif qui a la structure suivante :  


```
## tibble [200 × 4] (S3: tbl_df/tbl/data.frame)
##  $ id    : int [1:200] 1 2 3 4 5 6 7 8 9 10 ...
##  $ groupe: chr [1:200] "C" "C" "C" "B" ...
##  $ var1  : int [1:200] NA 20 10 28 19 30 NA 22 20 20 ...
##  $ statut: Factor w/ 2 levels "ko","ok": NA 1 NA 1 1 2 1 NA 2 2 ...
```

Vous pouvez retrouver le code ayant servi à le générer sur le dépôt github des [notes de blog de Statoscop](https://github.com/Statoscop/notebooks-blog).  

Le verbe `case_when` comporte plusieurs différences avec `if_else`, mais deux nous semblent particulièrement importantes :   

- sa syntaxe rend __la lecture de plusieurs conditions__ bien plus aisée    
- par défaut, __il ne renvoie pas NA s'il croise une valeur manquante__ dans la condition   

## Définition de conditions multiples 

Pour __la première différence__, la syntaxe de `case_when` va permettre de __définir les différentes conditions__ et la valeur associée sur une ligne dédiée alors que __celle de `if_else` oblige à chaîner les appels à la fonction__. En fait, `case_when` a même été créé pour permettre de généraliser `if_else` à des multiples conditions puisqu'il est indiqué dans sa description de la page d'aide que c'est une fonction __permettant de vectoriser l'appel à plusieurs `if_else`__. Illustrons cela en codant des deux manières une variable égale à :  

- "cas 1" si groupe = "A" et var1 < 25 
- "cas 2" si groupe = "B" ou "C" et var1 >= 25 
- "cas 3" sinon

Le code avec les deux syntaxes est le suivant :  


```r
df <- df |> mutate(
  
  # syntaxe case_when
  cond_case = case_when(
    groupe == "A" & var1 < 25 ~ "cas 1",
    groupe %in% c("B", "C") & var1 >= 25 ~ "cas 2",
    .default = "cas 3"),
  
  # syntaxe if_else
  cond_ifelse = if_else(
    groupe == "A" & var1 < 25,
    "cas 1",
    if_else(
      groupe %in% c("B", "C") & var1 >= 25,
      "cas 2",
      "cas 3"))
  ) 
```

La syntaxe de case_when permet de __produire un code plus lisible et aéré__. Chaque ligne de condition est évaluée __si aucune des conditions précédentes n'est respectée__. À noter que si l'on ne définissait pas de valeur par défaut explicitement, on aurait à la place des valeurs manquantes.  

## Gestion des valeurs manquantes  

Pour __la seconde différence__, `case_when` considère les valeurs manquantes __comme une valeur à part entière__ alors que `if_else` __renvoie automatiquement une valeur manquante__ s'il trouve une valeur manquante dans la condition. Si l'on observe nos deux variables on constate en effet qu'elles ne sont pas toujours égales :  


```r
df |> head(10) |> 
  knitr::kable()
```



| id|groupe | var1|statut |cond_case |cond_ifelse |
|--:|:------|----:|:------|:---------|:-----------|
|  1|C      |   NA|NA     |cas 3     |NA          |
|  2|C      |   20|ko     |cas 3     |cas 3       |
|  3|C      |   10|NA     |cas 3     |cas 3       |
|  4|B      |   28|ko     |cas 2     |cas 2       |
|  5|C      |   19|ko     |cas 3     |cas 3       |
|  6|B      |   30|ok     |cas 2     |cas 2       |
|  7|B      |   NA|ko     |cas 3     |NA          |
|  8|B      |   22|NA     |cas 3     |cas 3       |
|  9|C      |   20|ok     |cas 3     |cas 3       |
| 10|A      |   20|ok     |cas 1     |cas 1       |  

Ainsi, __lorsque la valeur de la variable `var1` est manquante__, la méthode `if_else` renvoie une valeur manquante alors que `case_when` lui donne la valeur "cas 3" car elle ne correspond pas aux deux premières conditions. Il faut donc __traiter explicitement les valeurs manquantes dans les conditions de `case_when` si l'on souhaite qu'elles ne soient pas prises en compte__.    

> 👋 Nous c'est Antoine et Louis de Statoscop, une coopérative de statisticiens / data scientists.
> Vous voulez en savoir plus sur ce que l'on fait?
<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>      

# Gestion des types facteur avec `case_when`   

La création d'une variable avec `case_when` doit __respecter le fait que la variable créée ait un type unique__. Cela peut poser problème lorsque l'on souhaite créer directement une variable facteur.  

## `.default` pour définir la valeur de base   

Le paramètre `.default` permet de définir la valeur que prend la variable lorsqu'aucune des conditions n'est respectée. Il remplace l'utilisation de la syntaxe `.TRUE ~ valeur` [depuis dplyr 1.1.0](https://cran.r-project.org/web/packages/dplyr/news/news.html). Il peut __correspondre à une valeur__, comme dans l'exemple précédent, __ou à une variable existante__. Il faut veiller à ce que __le type de la variable corresponde bien à celui des modalités définies précédemment__. En effet, on obtient sinon l'erreur suivante :  



```r
df |> mutate(
  statut_bis = case_when(
    is.na(var1) ~ FALSE, # modalité booléen
    .default = statut)) # variable facteur
```

```
## Error in `mutate()`:
## ℹ In argument: `statut_bis = case_when(is.na(var1) ~ FALSE, .default =
##   statut)`.
## Caused by error in `case_when()`:
## ! Can't combine `..1 (right)` <logical> and `.default` <factor<3d285>>.
```
Le message d'erreur nous indique bien l'impossibilité pour `case_when` de __combiner des valeurs booléennes__ (le `FALSE`) __et une variable facteur__ (la variable `statut` donnant la valeur par défaut).  

À noter qu'auparavant, `case_when` sortait également une erreur lorsqu'on définissait des modalités caractères et une variable facteur par défaut. Cela n'est plus le cas [depuis dplyr 1.1.0](https://cran.r-project.org/web/packages/dplyr/news/news.html), puisque `case_when` __transforme automatiquement les variables facteur en variable caractère__. Ainsi, le code suivant fonctionne :  


```r
df |> mutate(
  statut_bis = case_when(
    is.na(var1) ~ "inconnu", # modalité caractère
    .default = statut)) |> # variable facteur convertie automatiquement en caractère
  count(statut_bis) # affichage des modalités de statut_bis
```

```
## # A tibble: 4 × 2
##   statut_bis     n
##   <chr>      <int>
## 1 inconnu        5
## 2 ko            62
## 3 ok            61
## 4 <NA>          72
```

## `.ptype` pour créer directement un facteur 

Il n'est ici pas possible de __définir directement un facteur et l'ordre de ses niveaux__, à moins de le faire dans une nouvelle instruction dédiée. C'est là qu'entre en jeu l'argument `.ptype` de la fonction `case_when`. Par défaut, le type de la variable est en effet __déduit des valeurs définies après les conditions__. L'argument `ptype` permet d'imposer un type spécifique __quand cela est compatible avec les expressions utilisées__. Dans notre exemple précédent, on peut ainsi spécifier que l'on souhaite une variable facteur en sortie, __à condition de bien en définir les niveaux__  :  


```r
df |> mutate(
  statut_bis = case_when(
    is.na(var1) ~ "inconnu", # modalité caractère
    .default = statut,
    .ptype = factor(levels = c("ok", "ko", "inconnu")))) |> 
  count(statut_bis) # affichage des modalités de statut_bis
```

```
## # A tibble: 4 × 2
##   statut_bis     n
##   <fct>      <int>
## 1 ok            61
## 2 ko            62
## 3 inconnu        5
## 4 <NA>          72
```
On a ainsi pu définir notre variable facteur avec l'ordre des niveaux voulu directement dans `case_when`.  

C'est la fin de cet article! N'hésitez pas à [visiter notre site](https://www.statoscop.fr) et à nous suivre sur [BlueSky](https://bsky.app/profile/statoscop.fr) et [Linkedin](https://www.linkedin.com/company/statoscop). Pour retrouver l'ensemble du code ayant servi à générer cette note, vous pouvez vous rendre sur le [github de Statoscop](https://github.com/Statoscop/notebooks-blog).  

<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>    


