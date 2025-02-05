Title: Analyse en composantes principales avec Python
Author: Antoine
Date: '2021-04-16'
Slug: acp-python
Category: Python, Stats & ML
Tags: Python, Machine Learning, Statistiques, Data Science  
Cover: images/cover_2.png
Summary: Présentation et exemples d'utilisation de l'ACP en statistiques et data science.

[TOC]

Dans cet article, nous allons essayer de comprendre intuitivement comment fonctionne l'analyse en composantes principales. Nous présenterons ensuite à quoi celle-ci peut servir en prenant les exemples d'une analyse exploratoire des données et d'une problématique de réduction de dimension.   

# Explication introductive

L'analyse en composantes principales est une méthode consistant à transformer des variables corrélées entre elles en nouvelles variables. Chacune de ces nouvelles variables est le résultat d'une combinaison linéaire des anciennes variables. 

*Note : une combinaison linéaire de 3 variables $V_1$, $V_2$ et $V_3$ s'écrit $\alpha_1.V_1 + \alpha_2.V_2 + \alpha_3.V_3$ où les $\alpha_i$ sont des coefficients réels.*

Ces nouvelles variables sont appelées __composantes principales__ et sont, par contruction, décorrélées les unes des autres.  

Autrement dit, l'ACP projette vos données dans un nouvel espace. La première composante principale est construite de manière à capter la plus grande variance possible de vos données, la seconde la part la plus importante de la variance possible __restant à expliquer__, et ainsi de suite.  

Une illustration brillante de ce processus est proposée par <a href="https://www.allisonhorst.com/" target="_blank">Allison Horst</a>. Elle <a href="https://twitter.com/allison_horst/status/1288904459490213888" target="_blank">représente</a> un jeu de données à deux dimensions avec des crevettes et l'analyse en composantes principales comme les passages d'un requin-baleine affamé :  


![Pelican](../images/acp/output_3_0.png)


![Pelican](../images/acp/output_4_0.png)



La problématique du requin-baleine est en effet la même que celle de la création d'une première composante principale : quel axe choisir pour avaler un maximum de crevettes dès le premier passage? L'axe choisi va ressembler à celui-ci :  



![Pelican](../images/acp/output_6_0.png)



Il s'agit pour le requin de choisir la droite de sorte qu'il y ait un maximum de crevettes sur son parcours ce qui revient à ce que les crevettes soient le plus proche possible de cette droite. Mathématiquement, la première composante principale est la combinaison linéaire des deux axes $x$ et $y$ qui maximise l'inertie projetée ce qui revient à minimiser les écarts entre les points et cette droite.

Dans cet exemple, la seconde composante principale sera l'axe perpendiculaire à ce premier axe.

Si les points étaient parfaitement alignés sur une ligne, l'ensemble de la variance serait expliqué par la première composante et on serait parvenus à réduire le nombre de dimensions de notre problème sans perte d'information.


> 👋 Nous c'est Antoine et Louis de Statoscop, une coopérative de statisticiens / data scientists.
> Vous voulez en savoir plus sur ce que l'on fait?
<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>    


# Mise en oeuvre d'une ACP
D'accord, on a projeté notre jeu de données dans un nouvel espace avec des nouvelles "variables" décrites comme combinaisons linéaires des précédentes telles que la première explique la plus grande partie de la variance possible, la seconde la plus grande partie de la variance restant à expliquer, etc... Mais ça nous sert à quoi?   

## Analyse exploratoire de nos données  
La caractéristique des composantes principales par rapport au jeu de données non transformé est que les premières composantes principales ont un fort pouvoir discriminant, puisqu'elles expliquent une grande partie de la variance totale du jeu de données. Ainsi, représenter notre jeu de données par rapport aux deux premiers axes de l'ACP peut permettre de vérifier que ces données permettent bien de distinguer différentes classes.  

Prenons comme exemple la base de données `wine` que l'on peut charger directement depuis le module `sklearn`. Cette base de données contient des résultats d'analyses chimiques de 178 vins de 3 différents producteurs. Ces résultats sont synthétisés par 13 mesures différentes que l'on retrouve dans les données. Pour voir si ces mesures permettent ou non de distinguer les vins des trois producteurs, nous allons commencer par représenter les vins sur l'espace des deux premières composantes principales. Pour cela, on importe les données et on les centre-réduit avant d'appliquer notre ACP avec la fonction `sklearn.decomposition.PCA`. On paramètre celle-ci pour qu'elle nous renvoie seulement les deux premières composantes :  


```python
import numpy as np
# Import fonction ACP
from sklearn.decomposition import PCA

# Import données
from sklearn.datasets import load_wine

values, target = load_wine(return_X_y=True)
target_names = load_wine().target_names
feature_names = load_wine().feature_names
# on standardise nos données : 
from sklearn.preprocessing import StandardScaler
values_cr = StandardScaler().fit_transform(values)

# On paramètre notre PCA pour garder les deux premières composantes
pca=PCA(2)
pca_wine = pca.fit_transform(values_cr)

# en sortie : le même nombre de lignes que les données en entrées
# et le nombre de variables correspondant au nombre de composantes
# conservées
pca_wine.shape
```




    (178, 2)



En sortie, nous obtenons les vecteurs des deux premières composantes dans l'objet `pca_wine`. Notons que nous aurions pu paramétrer la fonction `PCA` de manière à ce qu'elle nous renvoie le nombre de composantes nécessaire à expliquer `X`% de la variance, comme nous le ferons par la suite. Depuis l'objet `pca`, on peut voir le vecteur de la variance expliquée par chaque composante avec `pca.explained_variance_ratio_` et donc la variance totale expliquée par nos deux composantes en sommant les éléments de ce vecteur :  


```python
pca.explained_variance_ratio_.sum()
```




    0.554063383569353



On explique donc 55 % de la variance totale de nos données avec 2 composantes, alors que celle-ci contient 13 variables. Voyons si cela suffit à discriminer nos 3 producteurs visuellement :  

<!--- ![Pelican](../images/acp/output_13_0.png)-->
<img alt="Pelican" src="../images/acp/output_13_0.png" style="max-width:80% !important" >

On constate ici que les 3 producteurs sont bien répartis dans des zones distinctes du plan et ce résultat semble montrer que chacun produit des types de vin caractéristiques.

On peut se convaincre que l'ACP a bien joué son rôle en produisant le même type de schéma avec deux autres variables originales du jeu de données (sans transformation linéaire), disons le degré d'alcool et l'intensité de la couleur. On s'attend bien sûr à ce que les classes soient moins discriminées qu'avec les deux premières composantes principales :  

<img alt="Pelican" src="../images/acp/output_15_0.png" style="max-width:80% !important" >

Ces variables permettent de distinguer des tendances, comme le fait que le producteur 1 produit des vins plutôt moins alcoolisés et dont la couleur est peu intense alors que le producteur 0 produit des vins plus alcoolisés. Mais ces variables seules ne permettent pas de partitionner nos classes aussi clairement qu'avec les deux premières composantes de notre ACP.   

L'ACP ne permet certes pas au premier coup d'oeil de proposer une interprétation des résultats, mais il est néanmoins possible d'étudier comment chaque variable contribue aux composantes avec l'instruction `pca.components_` :  

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Noms des variables</th>
      <th>Composante 1</th>
      <th>Composante 2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>alcohol</td>
      <td>0.144329</td>
      <td>-0.483652</td>
    </tr>
    <tr>
      <th>1</th>
      <td>malic_acid</td>
      <td>-0.245188</td>
      <td>-0.224931</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ash</td>
      <td>-0.002051</td>
      <td>-0.316069</td>
    </tr>
    <tr>
      <th>3</th>
      <td>alcalinity_of_ash</td>
      <td>-0.239320</td>
      <td>0.010591</td>
    </tr>
    <tr>
      <th>4</th>
      <td>magnesium</td>
      <td>0.141992</td>
      <td>-0.299634</td>
    </tr>
    <tr>
      <th>5</th>
      <td>total_phenols</td>
      <td>0.394661</td>
      <td>-0.065040</td>
    </tr>
    <tr>
      <th>6</th>
      <td>flavanoids</td>
      <td>0.422934</td>
      <td>0.003360</td>
    </tr>
    <tr>
      <th>7</th>
      <td>nonflavanoid_phenols</td>
      <td>-0.298533</td>
      <td>-0.028779</td>
    </tr>
    <tr>
      <th>8</th>
      <td>proanthocyanins</td>
      <td>0.313429</td>
      <td>-0.039302</td>
    </tr>
    <tr>
      <th>9</th>
      <td>color_intensity</td>
      <td>-0.088617</td>
      <td>-0.529996</td>
    </tr>
    <tr>
      <th>10</th>
      <td>hue</td>
      <td>0.296715</td>
      <td>0.279235</td>
    </tr>
    <tr>
      <th>11</th>
      <td>od280/od315_of_diluted_wines</td>
      <td>0.376167</td>
      <td>0.164496</td>
    </tr>
    <tr>
      <th>12</th>
      <td>proline</td>
      <td>0.286752</td>
      <td>-0.364903</td>
    </tr>
  </tbody>
</table>
</div>



Ce tableau représente les coefficients de la combinaison linéaire des variables pour chaque composante. Il nous permet par exemple de constater que l'intensité de la couleur et l'alcool jouent fortement et négativement sur la seconde composante. Cela correspond à ce que l'on observait dans les deux graphiques précédents puisque les vins des producteurs 0 et 2 ont des valeurs négatives sur l'axe de la seconde composante (1er graphique) et ce sont bien ceux dont le taux en alcool et l'intensité de la couleur sont les plus importants (2e graphique)

## Utilisation de l'ACP pour la réduction de dimensions  
La propriété de l'ACP de capter une partie importante de la variance des données à partir de moins de variables est particulièrement intéressante dans le domaine du Machine Learning pour être capable de fournir des prédictions avec des modèles plus légers (car utilisant moins de variables) et des résultats au moins aussi performants.  
Pour notre exemple, même si la réduction de dimensions n'est pas un enjeu fondamental vu le faible nombre de variables, nous pouvons tester si nous parvenons à faire un modèle de prédiction de l'origine du vin (producteur 0, 1 ou 2) en réduisant le nombre de dimensions.  
Tout d'abord, commençons par déterminer ce nombre de dimensions. Le graphique suivant nous donne l'évolution de la variance expliquée en fonction du nombre de composantes :   

<img alt="Pelican" src="../images/acp/output_20_0.png" style="max-width:80% !important" >


L'ACP permettrait d'expliquer plus de 70% de la variance totale dès 4 composantes. Pour voir si cela est suffisant pour entraîner un modèle de prédiction, on peut comparer les performances d'un arbre de classification sur les données transformées après PCA et sur les données brutes. On utilise une méthode de validation croisée pour estimer les performances du modèle qui consiste à partitionner les données en 5 groupes et à entraîner les données sur 4 groupes et les tester sur celui restant. On fait cela 5 fois pour parcourir le champ des possibles et on évalue la précision globale du modèle en faisant la moyenne de ces 5 résultats. Cette méthode doit permettre d'estimer la qualité du modèle sur des données sur lesquelles il n'a pas été entraîné et de ne pas prendre en compte le surapprentissage dans son évaluation. Le tableau suivant donne les taux de précision obtenus pour chaque méthode, c'est à dire le nombre de vins correctement classifiés sur le nombre de vins total.


```python
# Plutôt que de renseigner le nombre de composantes 
# on renseigne la valeur minimum de la variance 
# expliquée totale que l'on souhaite
pca=PCA(0.70) 
wine_pca = pca.fit_transform(values_cr)

# On entraîne notre modèle et on l'évalue avec une 
# méthode de validation croisée 
from sklearn.model_selection import cross_val_score
from sklearn.tree import DecisionTreeClassifier
clf = DecisionTreeClassifier(random_state=0)
mean_pca = cross_val_score(clf, wine_pca, target, cv=5).mean()
mean_all = cross_val_score(clf, values, target, cv=5).mean()

pd.DataFrame({"Précision moyenne après ACP" : [mean_pca], 
              "Précision moyenne sans ACP" : [mean_all], 
              "Nombre de composantes" : [wine_pca.shape[1]]})
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Précision moyenne après ACP</th>
      <th>Précision moyenne sans ACP</th>
      <th>Nombre de composantes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.933175</td>
      <td>0.887619</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



On constate que l'ACP n'a pas seulement permis de réduire le nombre de dimensions de notre problème, elle nous offre aussi une précision globale du modèle supérieure. Cela n'est pas toujours le cas - puisque ça dépend de votre problématique, des variables explicatives dont vous disposez et du nombre de composantes que vous retenez - mais ici c'est dû au fait qu'elle permet de réduire le bruit associé aux données en ne conservant qu'une partie de l'information totale. Cela permet ainsi de prévenir les problèmes de surapprentissage, c'est à dire le fait que le modèle explique parfaitement les données d'entraînement mais se généralise mal à de nouvelles données. Ce sujet est abordé dans [cet article de notre blog sur l'arbitrage biais/variance](https://blog.statoscop.fr/larbitrage-biaisvariance-dans-la-modelisation-de-donnees.html).  

C'est tout pour aujourd'hui! Si vous voulez voir d'autres exemples d'utilisation de l'ACP, je vous conseille <a href="https://jakevdp.github.io/PythonDataScienceHandbook/05.09-principal-component-analysis.html" target="_blank">cet article</a> qui aborde notamment le cas du traitement des images, pour lequel il est particulièrement intéressant de réduire le nombre de dimensions. N'hésitez pas à [visiter notre site](https://www.statoscop.fr) et à nous suivre sur [Twitter](https://twitter.com/stato_scop) pour ne pas rater les prochains articles! Vous pouvez trouver le notebook avec l'ensemble du code ayant servi à générer cette note sur le <a href="https://github.com/Statoscop/notebooks-blog" target="_blank">github de Statoscop</a>.   
  

<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>  