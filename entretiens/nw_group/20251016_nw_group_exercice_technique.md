# Test technique

## Propositions

**Remarques globales** : Les schémas d'architectures présentent les différents services qui intéragissent entre eux. La présentation ne fait pas mention de la manière dont les services sont hébergés. Les questions suivantes semblent suggérer qu'on possède une machine pour chaque chose, ce qui donnerait lieu à plusieurs SPOF. L'absence de middleware / API Gateway laisse imaginer que l'on peut mener des attaques type DDOS.

La quantité de messages qui circule dans le système ne doit pas toujours être la même. Les systèmes doivent donc pouvoir mettre en place du scaling horizontal pour compenser les pics de demandes.

On possède une architecture avec plusieurs microservices. Le fait qu'ils soient distincts permet de faire des modifications sur un service sans impacter les autres. La stratégie de branche est peut-être un peu simpliste.
- On a l'impression que les tests et les typages migration ne sont faites que lors du passage en main.
- Feature branch pour éviter que lors du développement on ait des conflits ?

### Risques de sécurité (hors cyber sécurité)

- Le document précise qu'en moyenne, 10% des JBOXs sont en défaut. Il faudrait s'assurer que ce chiffre n'augmente pas. Notamment, quels sont les délais d'intervention sur place en cas de panne détectée ?
- Les JBOX possèdent des informations (logiciels confidentiels et tokens d'authentification). Il faudrait s'assurer qu'on ne puisse pas vandaliser / voler la JBOX physique. Comment s'en assure-t-on ?

### Cybersécurité

**ATTENTION** : Les propositions faites ci-dessous se basent sur des meilleurs pratiques mais il faudrait demander son avis à un expert en cybersécurité / un auditeur cybersécurité avant de mettre en place le produit.

Les access token étant écrit sur les machines, ils ne sont pas renouvelés et l'énoncé ne précise pas si ces access tokens sont uniques par JBOX.

Chaque JBOX possède un token différent, ce qui est une bonne chose. Par contre, un seul token pour telemetry, ce qui est embêtant.

Quels sont les risques possibles ?
- Vol des données.
- Effondrement du système (un utilisateur malveillant parvient à détruire un composant du système).

Propositions classiques : les appels passent par un API Gateway. On refuse les appels non authentifiés. Load balancing.
Un utilisateur vandalise une JBOX. Les composants interagissant entre eux, il faut que notre système soit capable de résister si une JBOX est détruite.

Encryption des données au repos.

Si on possède un authentifiant, on peut polluer les données.

### Monitoring

On constate que toutes les JBOX communiquent avec un point unique (pas de load balancing).

Gossip protocol ?

On va posséder des données de TimeSeries. il existe des systèmes specialisés our cela pour lesquels la recherche de données et le stockage est optimisé. Vérifier si on a besoin d'utiliser un tel système (ex : InfluxDB).

Conserver des données d'utilisation afin de pouvoir faire des analyses et ainsi identifier les régions les plus sollicitées. Eventuellement faire de l'analyse prédictive (ML si on possède assez de données mais comme on peut s'attendre à des consommations similaires d'une année sur l'autre et que nous sommes face à une donnée dont on attend une forte saisonnalité, avoir une BI solide devrait être suffisant dans un premier temps.)

Impression : sait-on si un système meurt ? Il faudrait envoyer une notification toutes les heures pour vérifier si tout le monde est en vie.

## Gestion des logs

### Soucis mémoire

**Diagnostic d'erreur** : le serveur sauvegarde localement tous les logs. Après un certain temps d'exécution, il est surchargé.

**Causes possibles** : Tout est stocké sur une machine unique. 

**Proposition à court terme** : Afin d'éviter que les logs surchargent la machine, on peut imaginer un système qui s'exécute tous les soirs et supprime les logs plus vieux qu'une certaine durée. L'inconvénient est que l'on perd ainsi des informations qui pourraient être utilisées à des fins d'analyse.

**Moyen terme et long terme** : A moyen et long terme, on peut raffiner le système proposé avant en mettant en place des filtres particuliers basés sur la gravité des erreurs (les développeurs voudront sans doute garder les logs d'erreurs graves). On peut aussi imaginer un système où les logs sont archivés dans le cloud toutes les 24h. La machine de stockage ne devra stocker que les logs des dernières 24h, il faudra alors la dimensionner pour qu'elle soit capable d'héberger les logs de 24h seulement.

Eventuellement agréger les données les plus anciennes.

### Soucis requêtes non déployées

**Diagnostic d'erreur** : il y a eu un changement de version d'une partie du système (appelons les deux versions concurrentes v1 et v2). Les v1 et v2 ne sont pas compatibles entre elles.

**Causes possibles** : Les étapes de déploiement sont manuels pour les automates. Il y a eu un oubli.

**Note** : Si la nouvelle v2 n'est pas prête pour certains systèmes, nous commencerons à faire un rollback en v1 pour tous les systèmes concernés.

**Solution moyen terme** : Automatiser les déploiements pour les automates. Chaque message spécifie sa version de message dans le header ou alors on précise la version de l'endpoint à utiliser dans son uri (exemple : https://jbox-37.nw.io/v1/automate/planning). Les messages des versions précédentes sont traités normalement, la consolidation de données est gérée plus tard.

**Solution long terme** : Mettre en place un protocole clair pour les passages de version. Ce protocole doit notamment inclure la liste des systèmes impactés par les passages de version, une stratégie de rollback et une data limite au dela de laquelle les versions précédentes ne seront pas supportées (les versions avec des failles de sécurité doivent être décommissionnées rapidement). Il pourrait être pratique d'envoyer des warning quand un message d'une version bientôt décommissionnée est repéré. Le fait que l'erreur se produise semble indiquer aussi une faille dans les méthodes de tests : les versions v1 et v2 n'ont pas dû être testées ensembles pour qu'on ne se rende pas compte qu'une telle erreur pouvait se produire. Rajouter des tests entre versions différentes.

**Propositions supplémentaires** : Avoir des zones "pilotes" afin de pouvoir tester les nouvelles versions dans un environnement contrôlé.

## Ajout de fonctionnalités

**Note** : Les nouvelles offres sont des nouvelles offres. Elles ne doivent pas impacter les offres existantes. Pour chacunes d'entre elles, il faut créer un repository de code dédié et il faut être capable de desactiver le système en cas de bug.

Tous les schémas d'architecture proposés se fondent sur ceux donnés dans la consigne (sans les améliorations suggérées aux parties précédentes).

### Réserve secondaire

Intercaler un service qui se connecte aux APIs de réserve secondaire de RTE. La plupart des informations sur les JBOX devraient être rattachées à Planning.

Certaines des JBOX ne pourront pas servir de réserve secondaire.

### Rentrée de défaut

2 solutions proposées (on edge + logs)

On voudrait pouvoir identifier les défauts qu'on pourrait identifier automatiquement.

La première étape serait de vérifier quels sont les défauts qui pourraient être automatisés. On veut limiter le nombre de faux positifs et minimiser le temps de calcul.

La meilleure solution serait sans doute d'avoir un capteur au niveau de la JBOX qui reçoit la valeur f et mesure si au bout de trois minutes la valeur ne suit plus la consigne. Si c'est le cas, on envoie un messsage au service de défaut en précisant quelles sont les causes de l'erreur. L'accès direct à l'information de la JBOX permettrait de détecter rapidement les anomalies. Dans des cas critiques, le temps de délai entre la réception des logs et le calcul pourrait poser problème (exemple : cas de surchauffe). Cependant, barder les JBOX de capteurs pourrait augmenter leur coût et les capteurs sont parfois plus fragiles que les systèmes en eux-même. ==> Faux positifs.

Le soucis est qu'il faudra changer les JBOX existantes, ce qui n'est pas quelque chose de souhaitable.

Cette solution pose cependant problème.

On pourrait pour cela utiliser les logs.

**Proposition court terme** : utiliser les logs. Proposition long terme : adapter les nouvelles JBOX pour qu'elles puissent surveiller eux-même leur comportement. On imagine une période de chevauchement entre les deux solutions.

### Question bonus

**Conpréhension du problème** : On aimerait que toutes les JBOX aient un niveau similaire aux alentours de 50 %.

**Proposition court terme** : On garde un recueil indiquant le niveau d'énergie de toutes les JBOX. On demande à la JBOX avec le niveau d'énergie la plus élevé de fournir de l'énergie à celle avec le niveau la plus faible. On s'appuie pour cela sur le planning afin de rediriger les solutions en fonction de la puissance.


### Sources notables

System Design Alex Xu Vol 2, chapitre 5 (Metrics monitoring and Alerting System).
Blog BytebyteGo (https://blog.bytebytego.com/), articles sur les meilleurs pratiques pour les APIs.
Descriptif des APIs RTE (https://data.rte-france.com/)
API Design Roadmap du site GeekForGeek (https://www.geeksforgeeks.org/blogs/api-design-roadmap)
Articles sur les stratégies de branches (https://medium.com/@sreekanth.thummala/choosing-the-right-git-branching-strategy-a-comparative-analysis-f5e635443423)

Les solutions proposées sont les miennes, cependant ChatGOOT a été consulté afin de pouvoir approfondir les concepts de fournisseur primaire / secondaire et répondre à des questions sur le système d'enchère de l'électricité.


EMS = Energy management system.