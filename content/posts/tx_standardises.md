Title: Taux standardisés : calcul et interprétation avec R
Author: Antoine
Date: '2023-05-11'
Category: R, Stats & ML
Tags: R, Rstats, data science, statistiques  
Cover: images/cover_10.png
twitter_image: images/cover_10.png
Summary: Présentation de la méthode de standardisation directe des taux sur une étude de cas.  


Dans cette note nous présentons la méthode de standardisation directe des taux, qui permet de comparer des fréquences d'évènement entre différentes sous-populations en contrôlant par une ou plusieurs autres variables. Après avoir présenté le principe général de standardisation directe, on présente une mise en oeuvre pas-à-pas de la méthode sur une étude de cas à partir des données des passagers du Titanic.  

# Principe de la standardisation directe  

Les méthodes de standardisation des taux sont principalement étudiés en épidémiologie. Elles permettent en effet de répondre au problème suivant : comment __comparer des indicateurs épidémiologiques__ entre différents pays ayant leurs spécificités démographiques?   
On peut lire par exemple sur le site des [données de la banque mondiale](https://donnees.banquemondiale.org) que le taux de mortalité en 2020 en Australie est de 6 pour 1000 habitants, contre 12 pour 1000 habitants en Allemagne. Or, cet indicateur est bien sûr fortement influencé par la structure d'âge de la population concernée. Pour correctement comparer cet indicateur entre l'Australie et l'Allemagne et éventuellement en tirer des conclusions sur des différences de qualité de vie entre les deux pays, il faudrait __comparer ces taux à structure d'âge constante__.   
C'est précisément ce que permet de faire la standardisation directe. Il est nécessaire pour cela de disposer du détail de l'indicateur par tranche d'âge dans chacun des pays. Dans notre exemple, une fois qu'on dispose des taux de mortalité par tranche d'âge en Australie et en Allemagne, on les  applique à __une structure d'âge de référence__, qui peut être soit celle d'un des deux pays, soit celle du monde entier si on dispose de l'information. On peut alors obtenir un nouveaux taux de mortalité global __corrigé de l'effet lié à l'âge__ et permettant une comparaison plus pertinente entre les pays. Voyons concrètement comment on peut mettre en oeuvre cette méthode sur une étude de cas en R.  

# Premières analyses exploratoires des données  

Pour notre petit exemple, on s'appuie sur un dataset très connu : les [données des passagers du Titanic](https://www.kaggle.com/c/titanic). Les informations qui nous intéressent ici sont la classe économique, l'âge et le fait d'avoir survécu ou non au naufrage. On ne garde que les observations ayant des valeurs renseignées pour ces 3 variables. On veut répondre à la question suivante : __le fait d'être dans une classe économique aisée augmente-t-elle les chances de survie au nauffrage?__  

La première approche de cette question est bien sûr de calculer les taux de survie par classe économique. le résultat est le suivant :  


```r
titanic_data %>% group_by(pclass) %>% 
  summarise(tx_survie = sum(survived) / n())
## # A tibble: 3 × 2
##   pclass tx_survie
##    <int>     <dbl>
##       1     0.637
##       2     0.441
##       3     0.261
```
On constate en effet une forte disparité des taux de survie en fonction de la classe économique, puisque les passagers de 1ere classe sont 63,7% à survivre, contre 44,1% des passagers de seconde classe et 26,1% des passagers de 3e classe.  
Est-on bien sûr cependant que l'on observe __l'effet de la classe économique et pas d'un autre facteur de confusion__? On peut par exemple imaginer que l'âge a un effet important sur la chance de survie, et qu'il est également lié à la classe économique choisie pour le voyage. Vérifions ces hypothèses en croisant la répartition en tranches d'âge dans chaque classe :  


```r
titanic_data %>% 
  group_by(pclass, tr_age) %>% 
  summarise(eff = n()) %>% 
  # À ce stade le df n'est plus groupé que par pclass
  mutate(Proportion = eff / sum(eff)) %>% 
  select(-eff)

## # A tibble: 12 × 3
## # Groups:   pclass [3]
##    pclass tr_age      Proportion
##     <int> <fct>            <dbl>
##        1 0 - 17 ans      0.0528
##        1 18 - 34 ans     0.324 
##        1 35 - 49 ans     0.370 
##        1 50 ans ou +     0.254 
##        2 0 - 17 ans      0.126 
##        2 18 - 34 ans     0.582 
##        2 35 - 49 ans     0.195 
##        2 50 ans ou +     0.0958
##        3 0 - 17 ans      0.212 
##        3 18 - 34 ans     0.605 
##        3 35 - 49 ans     0.158 
##        3 50 ans ou +     0.0259
```

On constate en effet que la classe 3 comporte bien plus de mineurs et bien moins de passagers de plus de cinquante ans que la première classe. Comme on s'y attendait, __plus la classe économique est aisée, plus les passagers ont un âge élevé__. Voyons maintenant si les taux de survie varient en fonction de l'âge :  


```r
titanic_data %>% group_by(tr_age) %>% 
  summarise(tx_survie = sum(survived) / n())
## # A tibble: 4 × 2
##   tr_age      tx_survie
##   <fct>           <dbl>
##  0 - 17 ans      0.526
##  18 - 34 ans     0.377
##  35 - 49 ans     0.409
##  50 ans ou +     0.4
```

Les taux de survie ne semblent pas beaucoup varier en fonction de l'âge, à part pour les mineurs qui ont un taux de survie bien supérieur aux autres. Croisons maintenant les taux de survie en fonction de la classe économique __et__ de la tranche d'âge :  


```r
titanic_data %>% group_by(pclass, tr_age) %>% 
  summarise(tx_survie = sum(survived) / n())
## `summarise()` has grouped output by 'pclass'. You can override using the
## `.groups` argument.
## # A tibble: 12 × 3
## # Groups:   pclass [3]
##    pclass tr_age      tx_survie
##     <int> <fct>           <dbl>
##        1 0 - 17 ans     0.867 
##        1 18 - 34 ans    0.707 
##        1 35 - 49 ans    0.629 
##        1 50 ans ou +    0.514 
##        2 0 - 17 ans     0.879 
##        2 18 - 34 ans    0.414 
##        2 35 - 49 ans    0.333 
##        2 50 ans ou +    0.24  
##        3 0 - 17 ans     0.368 
##        3 18 - 34 ans    0.257 
##        3 35 - 49 ans    0.165 
##        3 50 ans ou +    0.0769
```

On constate là plusieurs choses :  
- Au sein de chaque classe économique il y a un lien très clair entre l'âge et le taux de survie : plus on vieillit pluys celui-ci baisse.  
- Au sein de chaque tranche d'âge, plus la classe économique est élevée plus la chance de survie augmente. La seule exception est la tranche d'âge 0 - 17 ans de la seconde classe qui a un taux de survie légèrement supérieur à la population du même âge de la 1ere classe (87,9% contre 86,7%).  

Au vu de ces éléments il apparaît légitime de calculer des taux standardisés pour comparer des fréquences de survie au sein de chaque classe corrigées de l'effet de l'âge.  

# Calcul pas-à-pas des taux standardisés de survie au naufrage du Titanic    

Au vu des résultats des taux de survie croisés par classe et âge, on sait déjà que la classe économique et l'âge jouent sur la chance de survie. Le calcul des taux standardisés de survie va permettre de quantifier __l'effet de la classe économique isolé de l'effet âge__. Le principe est le suivant : on calcule le taux de survie que l'on observerait dans chaque classe économique __si la répartition des passagers en classe d'âge était la même que celle de la population de référence__. Dans notre exemple, la population de référence est l'ensemble des passagers.

La première étape est de récupérer les effectifs de classe d'âge de la population de référence :  


```r
titanic_data %>% group_by(tr_age) %>% 
  summarise(eff = n()) -> rep_age_ref

rep_age_ref
## # A tibble: 4 × 2
##   tr_age        eff
##   <fct>       <int>
##  0 - 17 ans    154
##  18 - 34 ans   547
##  35 - 49 ans   235
##  50 ans ou +   110
```

On calcule ensuite comme précédemment les taux de survie par tranche d'âge de chaque classe économique et on transforme la table pour avoir une colonne par classe économique  


```r
titanic_data %>% group_by(pclass, tr_age) %>% 
  # taux de survie par classe et tranche d'âge
  summarise(survie = sum(survived) / n()) %>% 
  # on transforme la table en largeur pour avoir trois colonnes
  # donnant le taux de survie
  tidyr::pivot_wider(names_from = pclass, values_from = survie,
                     # option names_glue pour spécifier le nom des variables créées
                     names_glue = "{.value}_{pclass}") -> survie_age_class
## `summarise()` has grouped output by 'pclass'. You can override using the
## `.groups` argument.

survie_age_class
## # A tibble: 4 × 4
##   tr_age      survie_1 survie_2 survie_3
##   <fct>          <dbl>    <dbl>    <dbl>
##  0 - 17 ans     0.867    0.879   0.368 
##  18 - 34 ans    0.707    0.414   0.257 
##  35 - 49 ans    0.629    0.333   0.165 
##  50 ans ou +    0.514    0.24    0.0769
```

Il ne reste plus qu'à fusionner les deux tables ainsi créées et à appliquer chacun des taux de survie aux effectifs par tranche d'âge de la population de référence :  


```r
rep_age_ref %>% inner_join(survie_age_class, by = "tr_age") %>% 
  mutate(
    across(starts_with("survie_"), ~ . * eff)) -> eff_survie_age_class
eff_survie_age_class
## # A tibble: 4 × 5
##   tr_age        eff survie_1 survie_2 survie_3
##   <fct>       <int>    <dbl>    <dbl>    <dbl>
##  0 - 17 ans    154    133.     135.     56.7 
##  18 - 34 ans   547    386.     227.    141.  
##  35 - 49 ans   235    148.      78.3    38.7 
##  50 ans ou +   110     56.5     26.4     8.46
```

> Si l'utilisation d'across ne vous semble pas claire, je ne saurais trop vous recommander [notre précédent article de blog](https://blog.statoscop.fr/fonctionnement-et-performances-dacross-dans-dplyr.html) sur ce verbe bien pratique de dplyr.  

On obtient finalement nos taux de survie standardisés en sommant les effectifs par tranche d'âge et en divisant par l'effectif total :  


```r
eff_survie_age_class %>% 
  summarise(across(starts_with("survie_"), ~ sum(.) / sum(eff)))
## # A tibble: 1 × 3
##   survie_1 survie_2 survie_3
##      <dbl>    <dbl>    <dbl>
##     0.692    0.446    0.234
```

Par rapport aux taux de survie bruts présentés plus haut, on constate que __les inégalités économiques de survie s'aggravent après avoir contrôlé par l'âge__. Le taux de survie standardisés de la classe 1 et de la classe 3 sont en effet de 69,2% et 23,4 % contre 63,7% et 26,1% avant la correction. Cela correspond à ce que l'on pouvait pressentir après l'analyse exploratoire puisque les passagers de la classe 3 sont plus jeunes et que les jeunes ont une meilleure chance de survie au sein de chaque classe économique.   

# Pour conclure  

J'espère que cet exemple vous aura permis de mieux appréhender la question de la standardisation directe. Bien sûr, la méthode utilisée ici comporte des limites, notamment l'utilisation de tranches d'âge très larges qui pourraient fausser le résultat. On peut par exemple penser que les enfants en bas âge ont bénéficié d'une attention toute particulière, et ils se retrouvent ici dans la même tranche d'âge que des adolescents. Mais nous sommes également contraints par les effectifs qui nous empêchent de faire une analyse trop fine de ces subtilités. Si ces questions vous intéressent, vous pouvez vous reporter à notre article sur [le dilemme biais-variance](https://blog.statoscop.fr/le-dilemme-biais-variance-dans-la-modelisation-de-donnees.html).   

Enfin, nous avons passé sous silence une variable explicative très importante : le sexe des passagers. Les taux de survie sont en effet très différents pour les hommes et pour les femmes. Il est possible de reproduire cette analyse en prenant en compte cette variable, croisée également avec l'âge. Pour ce même problème d'effectifs, il sera sans doute nécessaire d'agréger encore plus les tranches d'âge. Vous avez les cartes en main pour le faire, et mieux comprendre les fortunes diverses de Jack et Rose dans le célèbre film de la tragédie...  

C'est la fin de cet article! N'hésitez pas à [visiter notre site](https://www.statoscop.fr) et à nous suivre sur [Twitter](https://twitter.com/stato_scop) et [Linkedin](https://www.linkedin.com/company/statoscop). Pour retrouver l'ensemble du code ayant servi à générer cette note, vous pouvez vous rendre sur le [github de Statoscop](https://github.com/Statoscop/notebooks-blog).  
