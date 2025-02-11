Title: Migration de SAS vers R  
Author: Antoine
Date: '2022-11-08'
Category: R
Tags: R, SAS, recodage, migration, codes, scripts
Cover: images/cover_12.png
twitter_image: images/cover_12.png
Summary: Quelques conseils pour réussir sa transition de SAS vers R

[TOC]  

L'utilisation du logiciel d'analyses statistiques SAS est encore assez ancrée dans certaines administrations publiques et grandes entreprises, malgré la concurrence de deux logiciels open-source offrant des performances souvent supérieures : R et Python. La __migration vers un nouveau langage__ présente un certain nombre de difficultés, mais le jeu en vaut la chandelle! Nous proposons dans cet article quelques recommandations pour recoder ses scripts SAS en R et assurer une __transition vers l'open-source__ la plus simple possible.  

# Options de recodage en R   
Une des difficultés de R est aussi une de ses forces : le grand nombre de possibilités qui s'offrent à vous pour coder. Le CRAN (pour _Comprehensive R Archive Network_) comprend un nombre impressionnant de packages (+ de 18000!). Dans ces conditions, il est important de se mettre d'accord a minima sur l'orientation que vous souhaitez prendre, afin d'assurer une certaine homogénéité entre les différents membres de vos équipes. Ces _règles_ peuvent bien sûr varier d'une équipe à l'autre, en fonction de ses besoins spécifiques. On présente ici trois des options les plus couramment utilisées en statistiques : base R, le tidyverse et data.table. On ne rentre pas dans le détail de la syntaxe de chacune de ces options mais vous pouvez approfondir cet aspect en lisant notre [article qui compare les performances de ces trois options](https://blog.statoscop.fr/comparaisons-base-dplyr-datatable.html).  

> 👋 Nous c'est Antoine et Louis de Statoscop, une coopérative de statisticiens / data scientists.
> Vous cherchez un prestataire pour vous accompagner dans votre migration vers R?
<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>   

## Base R   
Base R désigne toutes les fonctions natives de R, qui sont chargées par défaut au lancement du logiciel. Pour l'analyse de données, on s'appuiera notamment sur le format `data.frame`. L'utilisation exclusive de base R aurait l'avantage de ne pas faire dépendre votre code d'autres packages et donc de lui assurer une certaine stabilité. Cependant, pour l'analyse de données, la syntaxe sera vite verbeuse et peu lisible pour des non initiés. De plus, pour des traitements de données plus complexes, vous ne trouverez pas forcément ce qu'il vous faut dans ce que propose base R. Enfin, cette option est souvent [bien moins performante](https://blog.statoscop.fr/comparaisons-base-dplyr-datatable.html) en termes de vitesse d'exécution que les deux que nous vous présentons ensuite.  

## Tidyverse     
Le tidyverse est un [ensemble de packages](https://www.tidyverse.org/) destinés au traitement de données, du nettoyage à l'analyse et la datavisualisation. En particulier, le package `dplyr` propose des fonctions de haut niveau couvrant la très grande majorité des besoins lors de la manipulation des données. Ces fonctions, chaînées avec l'opérateur _pipe_ (`%>%` du package `magritr` ou `|>` de base R depuis la version 4.1.0), offrent un __code très lisible et intuitif__ à écrire. Enfin, elles sont optimisées et offrent de [bonnes performances](https://blog.statoscop.fr/comparaisons-base-dplyr-datatable.html). L'inconvénient du tidyverse est sans doute qu'il concerne beaucoup de packages qui ont chacun également un certain nombre de dépendances à d'autres packages. Cependant, le fait qu'il soit maintenu par les équipes de RStudio (la société s'appelle [Posit](https://posit.co/) depuis peu) est de nature à rassurer sur la stabilité de cette solution. En général on ne recommande de toute façon pas de charger l'ensemble du tidyverse mais seulement les packages que vous utilisez.  

## data.table  
Des trois options, `data.table` est [la plus performante](https://blog.statoscop.fr/comparaisons-base-dplyr-datatable.html). Ce package utilise un format concurrent du `data.frame` de base R, le `data.table`, qui optimise la vitesse d'exécution des instructions. Ce package a aussi l'avantage de ne dépendre d'aucun autre package. En contrepartie, il offre une syntaxe sans doute moins lisible pour les non-initiés, en particulier pour des opérations plus complexes.  

## Notre conseil  
Dans la majorité des cas, le `tidyverse` et en particulier __`dplyr` sera le choix le plus adapté__. Pour des utilisateurs de SAS, en particulier pour ceux connaissant un peu de SQL, les fonctions à utiliser seront plus intuitives. Cette option bénéficie en outre d'une communauté d'utilisateurs très importante permettant de trouver facilement des réponses en ligne aux problèmes que vous pourrez rencontrer. De plus, la vitesse d'exécution sera souvent au moins aussi bonne que celle de SAS. Si l'activité concernée par le recodage a un gros enjeu de temps d'exécution et moins de lisibilité du code, c'est sans doute vers `data.table` qu'il sera intéressant de se tourner. Il peut aussi être possible de choisir le meilleur des deux mondes avec le package [dtplyr](https://dtplyr.tidyverse.org/), combinant la syntaxe de dplyr aux fonctions de data.table. Enfin, il va de soi qu'une certaine souplesse est de mise pour pouvoir utiliser les fonctions de base R quand elles n'ont pas d'équivalent dans l'écosystème choisi, ainsi que d'autres packages pour des besoins spécifiques.  

# Organisation du travail de recodage en R  
Une fois que vous avez choisi comment vous souhaitez coder en R, vous pouvez spécifier vos recommandations dans un document __listant les bonnes pratiques__ de codage en R. Cela pourra servir à encadrer et accompagner le travail des développeurs R et uniformiser autant que possible les différentes manières de coder. En plus de cela, nous recommandons quelques outils simples à adopter pour tirer le meilleur de l'éco-système R.    

## Les projets RStudio  
Les [projets RStudio](https://support.rstudio.com/hc/en-us/articles/200526207-Using-RStudio-Projects) sont des fichiers associés à un dossier de travail que vous aurez créés. Ils permettent de spécifier les chemins vers vos données ou vers le dossier des sorties que vous voulez créer de manière __relative à ce dossier parent__. Leur utilisation permet de bannir de vos scripts l'utilisation de chemins spécifiques à votre machine. Ainsi, on remplacera avantageusement l'appel du chemin `C://Users/antoine/Documents/Mon_projet/data/ma_base.csv` par `data/ma_base.csv` dans les scripts.  C'est donc __un premier pas__ vers la création d'un dossier de travail contenant des scripts réutilisables en l'état par vos collègues, que ce soit ceux qui travaillent dans votre équipe ou ceux qui vous succéderont.  

## Renv pour gérer les packages  
Renv est un gestionnaire de packages R, dont nous parlons déjà dans [cet article](https://blog.statoscop.fr/gestion-des-packages-sur-r-avec-renv.html). Il vous permet de choisir un __environnement stable de packages__ sur un projet donné. En particulier, il vous assure à un moment T que toute une équipe sur un même projet travaille bien avec les mêmes packages et les mêmes versions de ceux-ci. Il vous permet aussi de pouvoir refaire tourner un code avec le même système de packages qu'au moment où il a été créé. C'est donc un autre élément essentiel de la reproductibilité et de la stabilité de vos projets, en particulier si on l'utilise avec un logiciel de gestion de versions...  

## Git ou un autre logiciel de gestion de versions  
Même si ça n'est pas spécifique à R, il est important de profiter de cette migration pour adopter l'utilisation d'un logiciel de gestion de versions si cela n'est pas déjà fait. [Git](https://githowto.com/) est sans doute le plus utilisé aujourd'hui. Il permet de tracer les modifications de votre code pas-à-pas, de revenir à une version antérieure de votre projet sans difficultés, de partager en temps réel avec votre équipe les modifications du code, de gérer aussi simplement que possible des modifications simultanées sur un même projet, et bien d'autres choses encore... La prise en main de Git peut prendre un peu de temps car cela n'est pas forcément intuitif pour qui n'est pas habitué à la gestion de versions mais c'est sans aucun doute absolument nécessaire pour assurer une bonne gestion de vos projets.    

# Accompagnement de la migration de SAS vers R   
Enfin, un aspect important de la migration vers un nouveau langage est la __gestion de la transition__ entre les deux. Que celle-ci soit prise en main en interne ou par un prestataire, il est indispensable de former les équipes en place et de libérer du temps de travail pour accompagner cette transition.  

## En interne   
L'avantage de gérer la migration vers R en interne est qu'elle associe forcément à ce processus les équipes en place. L'inconvénient évident est qu'il est alors indispensable de libérer beaucoup de temps du travail effectif à consacrer à cette tâche. Il est très important en amont de se mettre d'accord sur les points évoqués plus haut afin d'établir une sorte de _charte_ du code qui assurera un minimum de cohérence au sein des équipes. Enfin, il est évidemment indispensable d'accompagner chacun des membres des équipes en fonction de leur familiarité avec le nouveau langage. L'idéal est de proposer des __formations sur mesure__ correspondant aux problématiques métier de chacun.  

## Avec un prestataire   
Si vous faites appel à un prestataire, cela ne signifie pas pour autant que vous ne devrez pas libérer du temps de vos équipes pour la migration. On recommande en effet des échanges étroits et réguliers avec le prestataire, et plusieurs livraisons à intervalles réguliers plutôt qu'une seule à la fin du travail. Ceci doit permettre de s'assurer d'une part que le prestataire ne va pas dans une mauvaise direction et d'autre part que le code apparaît lisible et compréhensible pour les chargés d'étude. C'est aussi l'occasion pour ces derniers de se former et d'échanger avec le prestataire sur ces choix lors du recodage. Il est aussi possible de faire appel à un prestataire simplement pour encadrer votre équipe dans le travail de recodage, optimiser le code à certains points bloquants, faire un audit du travail réalisé...

# Conclusion  
Quitter SAS et aller vers R, ou un autre logiciel, c'est donc du travail et cela nécessite d'investir du temps et des moyens. Mais c'est aussi se libérer d'une __licence beaucoup trop chère__, et s'ouvrir les portes de nouvelles possibilités. Pourquoi ne pas en profiter pour __automatiser vos rapports__ et vos publications avec [R Markdown](https://rmarkdown.rstudio.com/)? Et si vous vous lanciez dans le développement d'une petite [application interactive R Shiny](https://shiny.rstudio.com/gallery/) pour mettre en valeur vos résultats les plus marquants?   
Quoiqu'il en soit, si cet article vous a donné envie d'aller plus loin, vous pouvez [nous contacter sur le site de Statoscop](https://www.statoscop.fr/contact) pour discuter des prochaines étapes!   

 <div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>   