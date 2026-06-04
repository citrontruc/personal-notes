# Refacto

- [Refacto](#refacto)
  - [Why would you do that?](#why-would-you-do-that)
  - [Une maison est abandonnée quand la première fenêtre est cassée](#une-maison-est-abandonnée-quand-la-première-fenêtre-est-cassée)
  - [Digression sur les espaces et les régions](#digression-sur-les-espaces-et-les-régions)
  - [Faut-il séparer les interfaces et les interfaces de leur implémentation ?](#faut-il-séparer-les-interfaces-et-les-interfaces-de-leur-implémentation-)
  - [Distinction importante pq ?](#distinction-importante-pq-)
  - [Comment commencer ?](#comment-commencer-)
  - [Comment Peer review une refacto?](#comment-peer-review-une-refacto)
  - [Design pattern de refacto](#design-pattern-de-refacto)
  - [Renommer ses variables et ses classes](#renommer-ses-variables-et-ses-classes)
  - [Et les performances dans tout ça ?](#et-les-performances-dans-tout-ça-)
  - [Les risques](#les-risques)
  - [Sources](#sources)

## Why would you do that?

On refactorise pour une ou deux raisons: simplifier ou élargir.

- Simplifier: on passe des objets légèrement trop gros ou on fait des opérations trop compliquées et on veut essayer une approche plus directe (changement typique : je me trimballe un foreach alors que je peux en faire une requête LINQ. Je me trimballe toute la classe character alors que j'ai juste besoin de sa position). ==> Pour rendre plus simple à comprendre.
- Elargir: on possède des fonctions définies dans un cas précis et on veut généraliser le cas afin de réutiliser des composants ou préparer des futurs changements. (changement typique : Je peux traiter A ou B les traitements sont similaires sur 50 % du trajet, au lieu de passer A ou B, je peux passer une interface commune). ==> Pour rendre plus simple à modifier.

Thèmes connexes : TDD & SOLID. DRY. Design patterns.

## Une maison est abandonnée quand la première fenêtre est cassée

Le désordre encourage le désordre. Si on possède du code sale, tout le monde va devoir construire autour et il sera donc plus compliqué d'extraire les composants viciés. Toujours plus facile de régler quand les dégâts sont contrôlables.

## Digression sur les espaces et les régions

regions sont créées pour une raison mais très compliqué de les respecter. Chez SG, ils ont une région "méthodes privées et méthodes publiques" mais pas toujours clair pour maj. ça amène à monter et descendre dans le code.

Espaces entre des zones de code ?

## Faut-il séparer les interfaces et les interfaces de leur implémentation ?

Dépend si les interfaces contiennent de la logique. Depends de si le but de la classe est d'être une implémentation de l'interface ou si le but de la classe est de servir pour autre chose et qu'elle utilise l'interface pour cela.

Ex : le but d'un formatteur est de formatter ==> on met ensemble. le but d'une DependencyKey est d'être consommée par un graphe.

## Distinction importante pq ?

Car les outils utilisés sont souvent différents en fonction de l'intention :

| Simplifier | Elargir |
| --- | --- |
| - LINQ | - Generique |
| - IA / Peer review | - interface |
| - Linter | - Variabilisation |

Dans les deux cas, on possède aussi des Design pattern associés.

A noter d'ailleurs que les deux n'ont pas aussi le même impact sur les équipes. Certains refactoring sont invisibles et d'autres demandent de brasser beaucoup de codes.

## Comment commencer ?

Start by fixing architecture before fixing the code.

Once the architecture is clearer, start with Open/Closed and Liskov substitution principle. Remove elements that are unnecessary and then elements that are not supposed to be here. You can then start adding tests.

Toujours commencer par la fin. Si on a un pipeline de traitement, mettre en place un objet temporaire pour faire une conversion ou quelque chose comme ça et fixer tous les éléments après ce changement. Il faut qu'on puisse lancer des tests à tout moment de l'opération.

Le refactoring doit avoir un impact nul sur les résultats ! Noter les résultats et identifiez les tests qui doivent être invariants.

Audit the code by looking at how it works. Diagnose its most important flaws and treat them in this order.

- Start by identifying god objects or bloated functions. We have to deal with them first.
- Create logical entities to regroup values (record or classes). Rename variables to make things more explicit.
- Separate large pile of code in multiple smaller functions / classes.
- Use Interface Segregation & Dependency inversion. Identify the methods that look alike, create objects to put them inside and create unifying interfaces.

## Comment Peer review une refacto?

Digression sur les commit hooks (lancer les tests, compilation automatique, pre-commit hook).

La refacto se fait sur une branche à part.

## Design pattern de refacto

Strangler fig, stack canary.

## Renommer ses variables et ses classes

On possède une "requête A". Si on introduit une requête B, comment va s'appeler la variable ? Séparer une fonction en plusieurs petites fonctions n'est utile que si on trouve un nom logique aux petites fonctions.

## Et les performances dans tout ça ?

Avoir des boucles en plus, des opérations séparées ou des switch plus loin peut causer des dégradations de performances (ou du moins plus que si on avait quelque chose de clair par types). Un exemple : je dois reprendre du code pour qu'il fonctionne avec un type A et un type B (ByRef et Priceable) et je dois vérifier que les Priceable ne sont pas trop gros. Cela fait que je ne peux mener ce test que quand j'ai récupéré les priceables, c'est à dire plus tard (potentiellement, j'ai déjà fait des opérations intermédiaires). Coût de ces opérations n'est pas peanuts mais si les cas sont rares, osef. Les gens sont assez mauvais pour estimer les coûts de tête.

De plus, il est plus facile d'estimer les performances de codes bien organisé car les chemins de chaque cas d'usage est mieux défini. On peut aussi estimer le temps total en sommant les temps des opérations.

## Les risques

- Une refacto sans tests et sans points de repère est un desossement du code. A tous les coups, on va casser quelque chose. Faire des tests de comportements. Si on teste des éléments trop précis, on ne peut pas faire de refacto.
- Une refacto ne possède pas que des critères "écrits", on possède aussi des critères non écrits (garder une latence < 5s, garder une utilisation mémorielle raisonnable...) Nombre d'appels en db est un kpi à superviser par exemple.
- Une refacto sans objectif est stérile. Une refacto intervient dans le but d'introduire une nouvelle fonctionnalité. on déplace du code et on oblige toute l'équipe à réapprendre comment se servir du code sans améliorer les choses. Si un arbre tombe mais que personne ne le voit tomber, est-ce que l'arbre est tombé? ==> On n'a pas besoin de refactoriser du code mort / qui est sensé partir.
- Checkpoints trop espacés (risque générique de classe vs générique de fonctions).
- La perfection. Le bon produit est celui qui sort dans les temps et pour un budget donné. Vous ne perdez pas de ventes sur le fait que votre méchanisme pour changer de mot de passe est lent.

Don't try to be too smart from the get go. Start by not changing to much and then fix elements one after another. When this is done, start fixing performances in the code.

## Sources

Sources : Cours de refactoring sur PluralSight
<https://refactoring.guru/> Site sur la refacto qui présente de façon rigolote les différents design pattern.
The Pragmatic Programmer
Clean Architecture
Martin Fowler Refactoring
