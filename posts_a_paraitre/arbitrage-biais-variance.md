# Idée post arbitrage biais/variance

## Le biais  
Ecart à la "vraie" valeure : faire un graphique avec des points bleus foncés et 1/10e des points plus clairs : c'est l'échantillon. Montrer les deux droites de régression (ou les 2 moyennes pour commencer), montrer ce qu'il se passe quand l'échantillon est biaisé = les points bleus clairs pas répartis pareil que le reste de la population.  

## la variance  
Définition de la variance de manière générale. définition de la variance d'un modèle : graphiques avec droites plus ou moins ajustés (fonction `loess` de R)

## arbitrage biais/variance  
Intuition : plus le modèle a de variance plus le risque de biais est important, et vice-versa. Revenir sur les premiers graphs de la partie variance et faire un modèle avec faible variance et un modèle avec forte variance. expliquer que problème de modèle à faible variance sont qu'ils donnent peu d'infos (donc ça sert à rien non plus de minimiser la variance sous principe de minimiser le biais)
But : maximiser la variance pour avoir l'estimation la plus précise possible mais minimiser le biais.

## lien avec machine learning   
pb fondamental en apprentissage supervisé 
expliquer principe de train/test (on met de côté la validation)
forte variance = surapprentissage = biais important : illustrer avec graphs d'évolution des train et test errors en fonction de la complexité du modèle.
