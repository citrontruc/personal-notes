# Unity

unityhub --no-sandbox

Voir le projet stocké sur github avec le rapport de projet.

## Tweening

Methode afin de pouvoir générer les mouvements intermédiaires pour un objet. Il existe une librairie gratuite appelée dotween qui fait cela.

## Continuous collision detection

La gestion des collisions en unity se fait en regardant par frame s'il y a une collision. Il est donc possible de soit avoir un objet dans un autre ou alors de passer au travers un obstacle si on va trop vite. En cas de problème, utiliser la CCD (continuous collision detection) afin de pouvoir évaluer les mouvements intermédiaires.

Note: Raycast est sans doute meilleur et oins couteux que CCD.
