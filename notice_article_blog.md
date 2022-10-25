Dans cette note, on reprend les différentes étapes de création d'une note de blog avec Pélican.  

# Générer un markdown depuis RStudio ou Jupyter Notebook  
On utiliser les IDE usuels, faisant tourner R ou Python, pour écrire notre note de blog. On travaille alors sur le dépôt git `notebooks-blog` (et sur la branche "encours" éventuellement). Une fois arrivé à une version quasi finalisée, on récupère le markdown correspondant et les fichiers statiques.  

## Depuis RStudio  
Depuis RStudio, on peut générer directement un markdown en knittant normalement le document Rmarkdown et en ajoutant le paramètre `keep_md = True` dans l'en-tête. Par exemple, pour l'article sur ggplot, cela donne :  

```
title: "Ggplot2 : mise en forme d'un barplot"  
author: "Antoine"  
date: "19/08/2022"  
output:   
    html_document:  
        keep_md: true  
```

Ainsi, après chaque _knit_, on obtient en sortie le fichier `titre_article.md` contenant le texte entier de l'article et les éventuels fichiers statiques dans un dossier `titre_article_files`.  

## Depuis Jupyter Notebook  
Depuis Jupyter Notebook, on exporte également l'article une fois finalisé au format Markdown grâce à `Fichier -> Télécharger au format ->  Markdown (.md)`. On obtient en sortie le fichier markdown et un dossier contenant les éventuels fichiers statiques.  

# Créer l'article avec Pélican   
## Préparer l'article dans l'architecture Pélican  
Une fois que l'on a récupéré notre markdown et nos fichiers statiques on se met sur le dépôt git du blog Pélican. On place le fichier markdown dans `content/posts/`. On place les fichiers statiques dans `content/images/nom_article/` (dossier que l'on aura créé). On peut également en profiter pour déposer dans `content/images` l'image qui servira de couverture et de miniature lors du partage sur les réseaux sociaux. On l'appelle généralement `cover_num_article.png`.  
Dans l'article, il reste tout d'abord à modifier les chemins d'appel aux fichiers statiques en chemins relatifs en remplaçant le chemin par défaut par le suivant :  
`![Pelican](../images/nom_article/output_X.png)`. On peut ensuite créer l'en-tête (que l'on aurait aussi pu créer dès la version rmarkdown ou jupyter notebook) sous le format suivant (exemple de l'article ACP) :  

```
Title: Analyse en composantes principales avec Python
Author: Antoine
Date: '2021-04-16'
Slug: acp-python
Category: Python, Stats & ML # plusieurs catégories possibles
Tags: Python, Machine Learning, Statistiques, Data Science  
Cover: images/cover_2.png
Summary: Présentation et exemples d'utilisation de l'ACP en statistiques et data science.
```

Enfin, il faut bien penser __à mettre la balise `[TOC]` en tout début d'article__.  

## Mettre l'article en ligne  
Dans le terminal, on active l'environnement de développement contenant les différentes librairies dont on a besoin (en particulier celles spécifiques à Pélican) avec `conda activate env_blog`. Pour générer le html à partir du `.md` on tape l'instruction `pelican content`. Enfin, pour déployer en local la nouvelle version du blog on lance `pelican --listen` qui permet de tester la manière dont apparaît la note sur le serveur local (à tester en navigation privée pour limiter les problème de cache).  
Quand tout semble en place, il est conseillé de faire un commit + push du dépôt tel qu'il est, __puis__ d'envoyer l'instruction `bash .bashdeploy` qui envoie l'ensemble des instructions permettant de versionner la nouvelle version sur la branche de déploiement.  
Et voilà! L'article devrait apparaître dans la minute sur https://blog.statoscop.fr!