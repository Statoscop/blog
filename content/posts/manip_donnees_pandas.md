Title: Manipulation de données avec pandas
Author: Antoine
Date: '2022-10-12'
Slug: manip-donnees-pandas
Category: Python
Tags: Python, Statistiques, Data Science, index, loc, iloc, colonnes, lignes
Cover: images/cover_11.png
Summary: Tour d'horizon des manières de filtrer des observations et sélectionner des colonnes avec pandas

[TOC]   


Dans cet article, on va essayer de clarifier les différentes options qui s'offrent à vous pour manipuler vos données avec Python en utilisant la librairie `pandas`. On se place dans le cas d'un tableau de données _tidy_, c'est à dire où chaque __colonne__ représente une __variable__, ou caractéristique, et chaque __ligne__ une __observation__. Dans ce cadre, on présente les options possibles pour :  

- __Filtrer les observations__, c'est à dire conserver certaines lignes.   
- __Sélectionner des variables__, c'est à dire conserver certaines colonnes.     

En particulier, on mettra l'accent sur les différences entre les fonctions `loc` et `iloc` et le rôle particulier des index. Pour illustrer cet article, on s'appuie sur la base de données `titanic`, [disponible notamment sur Kaggle](https://www.kaggle.com/datasets/heptapod/titanic). On importe la version csv de cette base avec `read_csv` et on affiche ses caractéristiques :    


```python
import pandas as pd

titanic = pd.read_csv('Data/titanic-survival.csv')
print(titanic.shape)
titanic.tail()
```

    (1309, 14)





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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1304</th>
      <td>3</td>
      <td>0</td>
      <td>Zabour, Miss. Hileni</td>
      <td>female</td>
      <td>14.5</td>
      <td>1</td>
      <td>0</td>
      <td>2665</td>
      <td>14.4542</td>
      <td>NaN</td>
      <td>C</td>
      <td>NaN</td>
      <td>328.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1305</th>
      <td>3</td>
      <td>0</td>
      <td>Zabour, Miss. Thamine</td>
      <td>female</td>
      <td>NaN</td>
      <td>1</td>
      <td>0</td>
      <td>2665</td>
      <td>14.4542</td>
      <td>NaN</td>
      <td>C</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1306</th>
      <td>3</td>
      <td>0</td>
      <td>Zakarian, Mr. Mapriededer</td>
      <td>male</td>
      <td>26.5</td>
      <td>0</td>
      <td>0</td>
      <td>2656</td>
      <td>7.2250</td>
      <td>NaN</td>
      <td>C</td>
      <td>NaN</td>
      <td>304.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1307</th>
      <td>3</td>
      <td>0</td>
      <td>Zakarian, Mr. Ortin</td>
      <td>male</td>
      <td>27.0</td>
      <td>0</td>
      <td>0</td>
      <td>2670</td>
      <td>7.2250</td>
      <td>NaN</td>
      <td>C</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1308</th>
      <td>3</td>
      <td>0</td>
      <td>Zimmerman, Mr. Leo</td>
      <td>male</td>
      <td>29.0</td>
      <td>0</td>
      <td>0</td>
      <td>315082</td>
      <td>7.8750</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Cette base contient 1309 observations (ici des passagers) et 14 variables qui permet de les caractériser. Voyons comment sélectionner des sous-ensembles de ces données.    

> 👋 Nous c'est Antoine et Louis de Statoscop, une coopérative de statisticiens / data scientists.
> Vous voulez en savoir plus sur ce que l'on fait?
<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>  

# Filtrer des observations   
On s'intéresse tout d'abord à la sélection sur les lignes. Différentes options sont possibles :  

## À partir de leur position  avec `iloc`  

Dans `pandas`, il est possible de sélectionner certaines lignes directement en renseignant leur position avec la syntaxe `data_frame.iloc[positions_lignes, positions_colonnes]`. En ayant bien en tête qu'en Python, la première observation correspond à la position 0, on peut par exemple sélectionner les lignes 1 à 3 :   


```python
titanic.iloc[0:3, :] 
# "0:3" lignes de la position 0 à la position 3 exclue
# ":" pour sélectionner l'ensemble des colonnes
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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>Allen, Miss. Elisabeth Walton</td>
      <td>female</td>
      <td>29.0000</td>
      <td>0</td>
      <td>0</td>
      <td>24160</td>
      <td>211.3375</td>
      <td>B5</td>
      <td>S</td>
      <td>2</td>
      <td>NaN</td>
      <td>St Louis, MO</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>Allison, Master. Hudson Trevor</td>
      <td>male</td>
      <td>0.9167</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>11</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>0</td>
      <td>Allison, Miss. Helen Loraine</td>
      <td>female</td>
      <td>2.0000</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
  </tbody>
</table>
</div>



On note que l'écriture `a:b` sélectionne les lignes de la position `a` __inclue__ à la position `b` __exclue__. Les index négatifs partent de la fin de la base, ainsi l'index `-1` correspond à la dernière observation. Il est possible aussi de renseigner les positions souhaitées dans une liste, ou de faire des combinaisons plus complexes en utilisant `range` et la concaténation de listes avec `+`, ci-dessous les lignes 1 à 2, 10 à 12 et l'avant-dernière à la dernière  :  


```python
titanic.iloc[[0, 1] + list(range(9, 12)) + list(range(-2, 0)), :]
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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>Allen, Miss. Elisabeth Walton</td>
      <td>female</td>
      <td>29.0000</td>
      <td>0</td>
      <td>0</td>
      <td>24160</td>
      <td>211.3375</td>
      <td>B5</td>
      <td>S</td>
      <td>2</td>
      <td>NaN</td>
      <td>St Louis, MO</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>Allison, Master. Hudson Trevor</td>
      <td>male</td>
      <td>0.9167</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>11</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1</td>
      <td>0</td>
      <td>Artagaveytia, Mr. Ramon</td>
      <td>male</td>
      <td>71.0000</td>
      <td>0</td>
      <td>0</td>
      <td>PC 17609</td>
      <td>49.5042</td>
      <td>NaN</td>
      <td>C</td>
      <td>NaN</td>
      <td>22.0</td>
      <td>Montevideo, Uruguay</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1</td>
      <td>0</td>
      <td>Astor, Col. John Jacob</td>
      <td>male</td>
      <td>47.0000</td>
      <td>1</td>
      <td>0</td>
      <td>PC 17757</td>
      <td>227.5250</td>
      <td>C62 C64</td>
      <td>C</td>
      <td>NaN</td>
      <td>124.0</td>
      <td>New York, NY</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1</td>
      <td>1</td>
      <td>Astor, Mrs. John Jacob (Madeleine Talmadge Force)</td>
      <td>female</td>
      <td>18.0000</td>
      <td>1</td>
      <td>0</td>
      <td>PC 17757</td>
      <td>227.5250</td>
      <td>C62 C64</td>
      <td>C</td>
      <td>4</td>
      <td>NaN</td>
      <td>New York, NY</td>
    </tr>
    <tr>
      <th>1307</th>
      <td>3</td>
      <td>0</td>
      <td>Zakarian, Mr. Ortin</td>
      <td>male</td>
      <td>27.0000</td>
      <td>0</td>
      <td>0</td>
      <td>2670</td>
      <td>7.2250</td>
      <td>NaN</td>
      <td>C</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1308</th>
      <td>3</td>
      <td>0</td>
      <td>Zimmerman, Mr. Leo</td>
      <td>male</td>
      <td>29.0000</td>
      <td>0</td>
      <td>0</td>
      <td>315082</td>
      <td>7.8750</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Vous aurez peut-être remarqué qu'il est aussi possible de filtrer sur les observations avec `.loc` en renseignant leur position. C'est en fait dû au fait que par défaut, l'index des lignes correspond à leur position. Mais si vous voulez filtrer en fonction de la __position__ des lignes, il est préférable d'utiliser `iloc`. À noter aussi que la syntaxe `a:b` dans le cas où vous utilisez `.loc` __inclue l'élément à la position `b`__ : 


```python
titanic.loc[0:3, :]
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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>Allen, Miss. Elisabeth Walton</td>
      <td>female</td>
      <td>29.0000</td>
      <td>0</td>
      <td>0</td>
      <td>24160</td>
      <td>211.3375</td>
      <td>B5</td>
      <td>S</td>
      <td>2</td>
      <td>NaN</td>
      <td>St Louis, MO</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>Allison, Master. Hudson Trevor</td>
      <td>male</td>
      <td>0.9167</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>11</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>0</td>
      <td>Allison, Miss. Helen Loraine</td>
      <td>female</td>
      <td>2.0000</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>0</td>
      <td>Allison, Mr. Hudson Joshua Creighton</td>
      <td>male</td>
      <td>30.0000</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>NaN</td>
      <td>135.0</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
  </tbody>
</table>
</div>



Si le dataframe avait eu un autre index que celui de la position par défaut, cette syntaxe n'aurait fonctionné qu'avec `iloc`:  


```python
# on crée un index correspondant au nom des passagers
titanic_index_name = titanic.set_index("name")

# titanic_index_name.loc[0:3, :] renvoie une erreur
titanic_index_name.iloc[0:3, ]
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
      <th>pclass</th>
      <th>survived</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
    <tr>
      <th>name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Allen, Miss. Elisabeth Walton</th>
      <td>1</td>
      <td>1</td>
      <td>female</td>
      <td>29.0000</td>
      <td>0</td>
      <td>0</td>
      <td>24160</td>
      <td>211.3375</td>
      <td>B5</td>
      <td>S</td>
      <td>2</td>
      <td>NaN</td>
      <td>St Louis, MO</td>
    </tr>
    <tr>
      <th>Allison, Master. Hudson Trevor</th>
      <td>1</td>
      <td>1</td>
      <td>male</td>
      <td>0.9167</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>11</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>Allison, Miss. Helen Loraine</th>
      <td>1</td>
      <td>0</td>
      <td>female</td>
      <td>2.0000</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
  </tbody>
</table>
</div>



## À partir de leur index avec `loc`   
Cette dernière remarque nous amène tout naturellement vers la seconde manière de filtrer les observations : à partir de leur index. En effet, il est possible avec `pandas` de créer un index qui permet de sélectionner des lignes en fonction de leur label. Ici nous avons créé le dataframe `titanic_index_name` qui identifie chaque ligne avec le nom du passager à laquelle elle correspond. On peut alors sélectionner une ou plusieurs lignes en utilisant ces labels, que l'on renseigne dans une liste :  


```python
titanic_index_name.index
```




    Index(['Allen, Miss. Elisabeth Walton', 'Allison, Master. Hudson Trevor',
           'Allison, Miss. Helen Loraine', 'Allison, Mr. Hudson Joshua Creighton',
           'Allison, Mrs. Hudson J C (Bessie Waldo Daniels)',
           'Anderson, Mr. Harry', 'Andrews, Miss. Kornelia Theodosia',
           'Andrews, Mr. Thomas Jr',
           'Appleton, Mrs. Edward Dale (Charlotte Lamson)',
           'Artagaveytia, Mr. Ramon',
           ...
           'Yasbeck, Mr. Antoni', 'Yasbeck, Mrs. Antoni (Selini Alexander)',
           'Youseff, Mr. Gerious', 'Yousif, Mr. Wazli', 'Yousseff, Mr. Gerious',
           'Zabour, Miss. Hileni', 'Zabour, Miss. Thamine',
           'Zakarian, Mr. Mapriededer', 'Zakarian, Mr. Ortin',
           'Zimmerman, Mr. Leo'],
          dtype='object', name='name', length=1309)




```python
# Une ligne : 
titanic_index_name.loc["Allen, Miss. Elisabeth Walton", :]

# Plusieurs lignes :  les index de la famille Allen
liste_allen = [i for i in titanic_index_name.index if i.startswith("Allen")]
titanic_index_name.loc[liste_allen, :]
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
      <th>pclass</th>
      <th>survived</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
    <tr>
      <th>name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Allen, Miss. Elisabeth Walton</th>
      <td>1</td>
      <td>1</td>
      <td>female</td>
      <td>29.0</td>
      <td>0</td>
      <td>0</td>
      <td>24160</td>
      <td>211.3375</td>
      <td>B5</td>
      <td>S</td>
      <td>2</td>
      <td>NaN</td>
      <td>St Louis, MO</td>
    </tr>
    <tr>
      <th>Allen, Mr. William Henry</th>
      <td>3</td>
      <td>0</td>
      <td>male</td>
      <td>35.0</td>
      <td>0</td>
      <td>0</td>
      <td>373450</td>
      <td>8.0500</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Lower Clapton, Middlesex or Erdington, Birmingham</td>
    </tr>
  </tbody>
</table>
</div>



À noter que si vous sélectionnez une seule ligne, renseigner le label correspondant directement dans une chaîne de caractères vous renverra une `Serie`, soit un vecteur de valeurs pour `pandas` alors que le renseigner dans une liste vous renverra un `DataFrame` avec une seule ligne :  


```python
print(type(titanic_index_name.loc["Allen, Miss. Elisabeth Walton", :]),
      type(titanic_index_name.loc[["Allen, Miss. Elisabeth Walton"], :]))
```

    <class 'pandas.core.series.Series'> <class 'pandas.core.frame.DataFrame'>


## À partir de conditions avec `loc` et `[]`  
Enfin, et c'est sans doute le cas qui est le plus utilisé dans la pratique, on peut filtrer sur les observations à partir d'une condition. L'idée est de garder seulement les lignes correspondant à la condition.   
Imaginons par exemple qu'on veuille le tableau contenant uniquement les passagères. La condition `titanic.sex == "female"` va renvoyer un vecteur de booléens de la taille du nombre de lignes de `titanic` renseignant `True` ou `False` en fonction de si la ligne correspond à un passager ou une passagère :  


```python
titanic.sex == "female"
```




    0        True
    1       False
    2        True
    3       False
    4        True
            ...  
    1304     True
    1305     True
    1306    False
    1307    False
    1308    False
    Name: sex, Length: 1309, dtype: bool



On peut utiliser cette condition pour sélectionner un sous-ensemble du dataframe original avec `.loc` ou directement avec la syntaxe `data[condition]`.  
__Attention :__ Si vous souhaitez créer un nouveau dataframe correspondant à ce sous-ensemble, pensez à utiliser la méthode `.copy()`, sinon vous aurez le warning _SettingWithCopyWarning: A value is trying to be set on a copy of a slice from a DataFrame_ lorsque vous modifierez ce nouveau dataframe. C'est dû au fait que ces modifications affectent également le dataframe original si une copie n'a pas été créée.


```python
# avec .loc
titanic_femmes_loc = titanic.loc[titanic.sex == "female", :].copy()
# avec []
titanic_femmes_croch = titanic[titanic.sex == "female"].copy()

# On teste qu'on a bien le même dataframe et on affiche les 1eres lignes
print(titanic_femmes_loc.equals(titanic_femmes_croch))
titanic_femmes_loc.head(4)
```

    True





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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>Allen, Miss. Elisabeth Walton</td>
      <td>female</td>
      <td>29.0</td>
      <td>0</td>
      <td>0</td>
      <td>24160</td>
      <td>211.3375</td>
      <td>B5</td>
      <td>S</td>
      <td>2</td>
      <td>NaN</td>
      <td>St Louis, MO</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>0</td>
      <td>Allison, Miss. Helen Loraine</td>
      <td>female</td>
      <td>2.0</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>0</td>
      <td>Allison, Mrs. Hudson J C (Bessie Waldo Daniels)</td>
      <td>female</td>
      <td>25.0</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1</td>
      <td>1</td>
      <td>Andrews, Miss. Kornelia Theodosia</td>
      <td>female</td>
      <td>63.0</td>
      <td>1</td>
      <td>0</td>
      <td>13502</td>
      <td>77.9583</td>
      <td>D7</td>
      <td>S</td>
      <td>10</td>
      <td>NaN</td>
      <td>Hudson, NY</td>
    </tr>
  </tbody>
</table>
</div>



Il est bien sûr possible de faire des conditions plus complexes avec les opérateurs `&` (opérateur logique "ET") et `|` (opérateur logique "OU"), par exemple les femmes de plus de 60 ans :  


```python
my_cond = (titanic.sex == "female") & (titanic.age >= 60)
titanic.loc[my_cond, :].head(4)
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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6</th>
      <td>1</td>
      <td>1</td>
      <td>Andrews, Miss. Kornelia Theodosia</td>
      <td>female</td>
      <td>63.0</td>
      <td>1</td>
      <td>0</td>
      <td>13502</td>
      <td>77.9583</td>
      <td>D7</td>
      <td>S</td>
      <td>10</td>
      <td>NaN</td>
      <td>Hudson, NY</td>
    </tr>
    <tr>
      <th>43</th>
      <td>1</td>
      <td>1</td>
      <td>Bucknell, Mrs. William Robert (Emma Eliza Ward)</td>
      <td>female</td>
      <td>60.0</td>
      <td>0</td>
      <td>0</td>
      <td>11813</td>
      <td>76.2917</td>
      <td>D15</td>
      <td>C</td>
      <td>8</td>
      <td>NaN</td>
      <td>Philadelphia, PA</td>
    </tr>
    <tr>
      <th>61</th>
      <td>1</td>
      <td>1</td>
      <td>Cavendish, Mrs. Tyrell William (Julia Florence...</td>
      <td>female</td>
      <td>76.0</td>
      <td>1</td>
      <td>0</td>
      <td>19877</td>
      <td>78.8500</td>
      <td>C46</td>
      <td>S</td>
      <td>6</td>
      <td>NaN</td>
      <td>Little Onn Hall, Staffs</td>
    </tr>
    <tr>
      <th>78</th>
      <td>1</td>
      <td>1</td>
      <td>Compton, Mrs. Alexander Taylor (Mary Eliza Ing...</td>
      <td>female</td>
      <td>64.0</td>
      <td>0</td>
      <td>2</td>
      <td>PC 17756</td>
      <td>83.1583</td>
      <td>E45</td>
      <td>C</td>
      <td>14</td>
      <td>NaN</td>
      <td>Lakewood, NJ</td>
    </tr>
  </tbody>
</table>
</div>



# Sélectionner des variables  

## À partir de leur position avec `iloc`  

De manière complètement analogue à ce que l'on faisait avec les lignes, il est possible de sélectionner certaines variables en indiquant dans `iloc` une liste de positions, par exemple pour sélectionner la première et la dernière colonne :  


```python
titanic.iloc[:, [0, -1]].head()
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
      <th>pclass</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>St Louis, MO</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
  </tbody>
</table>
</div>



À noter que cette notation ne fonctionnera pas ni avec `.loc`, ni avec `[]`, qui ont besoin des labels des variables.  

## À partir de leur nom avec `loc` et `[]`  
Tout comme on pouvait filtrer les lignes à partir de leur label quand on définissait un index, il est possible de sélectionner certaines variables en les renseignant dans une liste avec `.loc` et `[]` :  


```python
# avec .loc
titanic_select_loc = titanic.loc[:, ["pclass", "sex", "age"]].copy()

# avec []
titanic_select_croch = titanic[["pclass", "sex", "age"]].copy()

print(titanic_select_loc.equals(titanic_select_croch))
titanic_select_loc.head(4)
```

    True





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
      <th>pclass</th>
      <th>sex</th>
      <th>age</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>female</td>
      <td>29.0000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>male</td>
      <td>0.9167</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>female</td>
      <td>2.0000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>male</td>
      <td>30.0000</td>
    </tr>
  </tbody>
</table>
</div>



Notons qu'avec `loc` on peut également utiliser la syntaxe `"label1" : "labeln"` pour sélectionner toutes les colonnes se situant entre deux colonnes inclues :  


```python
titanic.loc[:, "pclass":"sex"].head()
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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>Allen, Miss. Elisabeth Walton</td>
      <td>female</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>Allison, Master. Hudson Trevor</td>
      <td>male</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>0</td>
      <td>Allison, Miss. Helen Loraine</td>
      <td>female</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>0</td>
      <td>Allison, Mr. Hudson Joshua Creighton</td>
      <td>male</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>0</td>
      <td>Allison, Mrs. Hudson J C (Bessie Waldo Daniels)</td>
      <td>female</td>
    </tr>
  </tbody>
</table>
</div>



Enfin, de la même manière qu'avec les lignes, sélectionner une seule colonne en renseignant son label directement ou dans une liste renverra une `Serie` ou un `DataFrame` :  


```python
print(type(titanic.loc[:, "pclass"]), type(titanic.loc[:, ["pclass"]]))
```

    <class 'pandas.core.series.Series'> <class 'pandas.core.frame.DataFrame'>


## À partir de conditions avec `loc`   
Enfin, il est possible de sélectionner certaines colonnes en fonction de conditions, représentées par un vecteur de booléens. En général, on va créer ce vecteur en s'appuyant sur les méthodes `pandas` permettant de caractériser les colonnes de notre dataframe, comme `.columns` et `.dtypes` :   


```python
titanic.columns, titanic.dtypes
```




    (Index(['pclass', 'survived', 'name', 'sex', 'age', 'sibsp', 'parch', 'ticket',
            'fare', 'cabin', 'embarked', 'boat', 'body', 'home.dest'],
           dtype='object'),
     pclass         int64
     survived       int64
     name          object
     sex           object
     age          float64
     sibsp          int64
     parch          int64
     ticket        object
     fare         float64
     cabin         object
     embarked      object
     boat          object
     body         float64
     home.dest     object
     dtype: object)



On peut par exemple sélectionner les variables renseignant des entiers (`cond1`), ou celles commençant par la lettre b :  


```python
cond1 = titanic.dtypes == "int64"
titanic.loc[:, cond1].head()
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
      <th>pclass</th>
      <th>survived</th>
      <th>sibsp</th>
      <th>parch</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
cond2 = [i for i in titanic.columns if i.startswith("s")]
titanic.loc[:, cond2].head()
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
      <th>survived</th>
      <th>sex</th>
      <th>sibsp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>female</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>male</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>female</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>male</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>female</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



# Synthèse : j'utilise quoi du coup?    

## `iloc` ou `loc` ? 
En résumé, les syntaxes `data.iloc[i, j]` et `data.loc[i, j]` suivent la même logique en permettant dans le même appel de filtrer sur les lignes en i et sur les colonnes en j. Le choix entre `iloc` et `loc` est seulement guidé par le fait que votre sélection porte sur :  

- les __positions des lignes ou des colonnes__. Dans ce cas vous devez utiliser `iloc`.  
- les __noms des colonnes, les index des lignes, ou un vecteur de booléens__. Dans ce cas utilisez `loc`.  

Rappelez vous que sélectionner par les positions avec `.loc` n'est possible que quand les index par défaut sont les positions des lignes. Mais cela porte à confusion. Prenons par exemple notre table triée par âge pour laquelle on n'a pas réinitialisé les index :  


```python
titanic_sorted = titanic.sort_values("age").copy()
titanic_sorted.head(5)
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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>763</th>
      <td>3</td>
      <td>1</td>
      <td>Dean, Miss. Elizabeth Gladys "Millvina"</td>
      <td>female</td>
      <td>0.1667</td>
      <td>1</td>
      <td>2</td>
      <td>C.A. 2315</td>
      <td>20.5750</td>
      <td>NaN</td>
      <td>S</td>
      <td>10</td>
      <td>NaN</td>
      <td>Devon, England Wichita, KS</td>
    </tr>
    <tr>
      <th>747</th>
      <td>3</td>
      <td>0</td>
      <td>Danbom, Master. Gilbert Sigvard Emanuel</td>
      <td>male</td>
      <td>0.3333</td>
      <td>0</td>
      <td>2</td>
      <td>347080</td>
      <td>14.4000</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Stanton, IA</td>
    </tr>
    <tr>
      <th>1240</th>
      <td>3</td>
      <td>1</td>
      <td>Thomas, Master. Assad Alexander</td>
      <td>male</td>
      <td>0.4167</td>
      <td>0</td>
      <td>1</td>
      <td>2625</td>
      <td>8.5167</td>
      <td>NaN</td>
      <td>C</td>
      <td>16</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>427</th>
      <td>2</td>
      <td>1</td>
      <td>Hamalainen, Master. Viljo</td>
      <td>male</td>
      <td>0.6667</td>
      <td>1</td>
      <td>1</td>
      <td>250649</td>
      <td>14.5000</td>
      <td>NaN</td>
      <td>S</td>
      <td>4</td>
      <td>NaN</td>
      <td>Detroit, MI</td>
    </tr>
    <tr>
      <th>1111</th>
      <td>3</td>
      <td>0</td>
      <td>Peacock, Master. Alfred Edward</td>
      <td>male</td>
      <td>0.7500</td>
      <td>1</td>
      <td>1</td>
      <td>SOTON/O.Q. 3101315</td>
      <td>13.7750</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Si l'on veut la première et la 5e ligne de ce nouveau dataframe, il faut bien utiliser `iloc` et non `loc` qui renverra les index 0 et 5 de l'ancien dataframe `titanic` :  


```python
titanic_sorted.iloc[[0, 4], :]
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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>763</th>
      <td>3</td>
      <td>1</td>
      <td>Dean, Miss. Elizabeth Gladys "Millvina"</td>
      <td>female</td>
      <td>0.1667</td>
      <td>1</td>
      <td>2</td>
      <td>C.A. 2315</td>
      <td>20.575</td>
      <td>NaN</td>
      <td>S</td>
      <td>10</td>
      <td>NaN</td>
      <td>Devon, England Wichita, KS</td>
    </tr>
    <tr>
      <th>1111</th>
      <td>3</td>
      <td>0</td>
      <td>Peacock, Master. Alfred Edward</td>
      <td>male</td>
      <td>0.7500</td>
      <td>1</td>
      <td>1</td>
      <td>SOTON/O.Q. 3101315</td>
      <td>13.775</td>
      <td>NaN</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
titanic_sorted.loc[[0, 4], :]
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
      <th>pclass</th>
      <th>survived</th>
      <th>name</th>
      <th>sex</th>
      <th>age</th>
      <th>sibsp</th>
      <th>parch</th>
      <th>ticket</th>
      <th>fare</th>
      <th>cabin</th>
      <th>embarked</th>
      <th>boat</th>
      <th>body</th>
      <th>home.dest</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>Allen, Miss. Elisabeth Walton</td>
      <td>female</td>
      <td>29.0</td>
      <td>0</td>
      <td>0</td>
      <td>24160</td>
      <td>211.3375</td>
      <td>B5</td>
      <td>S</td>
      <td>2</td>
      <td>NaN</td>
      <td>St Louis, MO</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>0</td>
      <td>Allison, Mrs. Hudson J C (Bessie Waldo Daniels)</td>
      <td>female</td>
      <td>25.0</td>
      <td>1</td>
      <td>2</td>
      <td>113781</td>
      <td>151.5500</td>
      <td>C22 C26</td>
      <td>S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Montreal, PQ / Chesterville, ON</td>
    </tr>
  </tbody>
</table>
</div>



On aurait aussi pu utiliser le paramètre `ignore_index` dans `sort_values()` pour réinitialiser directement les index.  

## `[]` ou `loc`?  
Les crochets n'explicitent pas autant s'ils sélectionnent sur des lignes ou des colonnes. Si vous renseignez une liste de labels ils vont sélectionner des colonnes, et si vous renseignez une liste de booléens ils vont filtrer sur les lignes. Il est possible de chaîner les opérations sur les lignes ou les colonnes. Ainsi : `titanic[titanic.sex == "female"]["name"]` donnera le même résultat que `titanic.loc[titanic.sex == "female", "name"]`, c'est à dire les noms (colonne "name") des passagères (lignes correspondant aux femmes). 
Il vous revient donc de choisir ce qui vous convient le mieux. Dans le cas où on sélectionne à la fois sur les lignes et les colonnes, la syntaxe de `loc` a l'avantage d'être plus lisible. Dans le cas où on sélectionne seulement des colonnes ou seulement des lignes, l'usage des crochets permettra de taper les instructions légèrement plus rapidement. Enfin quand on veut extraire une série d'un dataframe, on utilisera avantageusement la syntaxe `data.nom_col`, équivalente à `data["nom_col"]`.  
  
C'est la fin de cet article! N'hésitez pas à [visiter notre site](https://www.statoscop.fr) et à nous suivre sur [Twitter](https://twitter.com/stato_scop) et [Linkedin](https://www.linkedin.com/company/statoscop). Pour retrouver l'ensemble du code ayant servi à générer cette note, vous pouvez vous rendre sur le [github de Statoscop](https://github.com/Statoscop/notebooks-blog).  

  
<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>  