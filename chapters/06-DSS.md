# Diagrammes de séquence système (DSS)

Un diagramme de séquence système (DSS) est diagramme UML (diagramme de séquence) limité à un acteur (provenant du scénario d'un cas d'utilisation) et le Système.
Les DSS sont expliqués en détail dans le chapitre 10 du livre du cours, mais voici des points importants pour LOG210:

- Le DSS a toujours un titre.
- L'acteur est indiqué dans la notation par un bonhomme et c'est représenté comme une *instance* de la classe du bonhomme, par exemple dans la figure&nbsp;\ref{DSS-attaquer-un-pays} `:Joueur` (le ":" signifie une instance).
- Le Système est un objet (une instance `:Système`) et n'est jamais détaillé plus.
- Le but du DSS est de définir des opérations système (Application Programming Interface) du système; il s'agit d'une conception de haut niveau.
- Le côté acteur du DSS n'est pas un acteur tout seul, mais une couche logicielle de présentation, comme une interface graphique ou un logiciel qui peut reconnaître la parole. Cette couche reconnaît des gestes de l'acteur (par exemple un clic sur un bouton dans l'interface, une demande "Hé Siri", etc.) et envoie une opération système.
- Puisque la couche présentation reçoit des informations des êtres humains, *les opérations système ont des arguments de type primitif*. Il est difficile pour un utilisateur de spécifier une référence (pointeur en mémoire) à un objet. Alors, on peut donner le nom (de type `String`) d'un morceau de musique à jouer, ou spécifier une quantité (de type `Integer`).
- Puisque les types des arguments sont importants, on les spécifie dans les opérations système du DSS.
- Un message de retour (ligne pointillée avec flèche ouverte) vers l'acteur représente la communication des informations précises, par exemple les valeurs des dés dans l'attaque. Puisque la couche présentation a beaucoup de moyens pour afficher ces informations, *on ne va pas spécifier les messages de retour comme des méthodes*.

## Exemple: DSS pour Attaquer un pays

La figure&nbsp;\ref{DSS-attaquer-un-pays} est un exemple de DSS pour le cas d'utilisation *Attaquer un pays*. Vous pouvez noter tous les détails (titre, arguments, types).

```{.plantuml caption="Diagramme de séquence système pour « Attaquer un pays »" #DSS-attaquer-un-pays}
@startuml
!include ../forme.pumlinclude
scale 1.3
skinparam sequenceMessageAlign center
title DSS pour <i>Attaquer un pays</i>
actor ":Joueur" as c
participant ":Système" as s
c->s : démarrerAttaque(paysAttaquant : String,\npaysDéfenseur : String)
loop pas terminé
c->s : annoncerAttaque(nbRégimentsAttaquant : int)
c->s : annoncerDéfense(nbRégimentsDéfendant : int)
c<<--s : résultats des deux lancers, régiments perdus \nde l'attaquant et du défenseur le cas échéant
end loop
c->s : terminerAttaque()
@enduml
```

## DSS fait abstraction de la couche présentation

Le but du DSS est de se concentrer sur l'API (les opérations système) de la solution.
Dans ce sens, c'est une conception de haut niveau.
Le "Système" est modélisé comme une boîte noire.
Par exemple, dans la figure&nbsp;\ref{DSS-abstraction} il y a l'acteur, le Système et une opération système.
On ne rentre pas dans les détails, bien qu'ils existent et sont importants.

```{.plantuml caption="Une opération système dans un DSS. C'est une abstraction." #DSS-abstraction}
@startuml
!include ../forme.pumlinclude
scale 1.3
skinparam sequenceMessageAlign center
actor ":Joueur" as c
participant ":Système" as s
c->s : démarrerAttaque(...)
@enduml
```

Plus tard, lorsque c'est le moment d'implémenter le code, les détails importants seront à respecter.
Il faut faire attention aux principes de la séparation des couches présentation et domaine.
Par exemple, la figure&nbsp;\ref{DSS-details} rentre dans les détails de ce qui se passe réellement dans une opération système quand la solution fonctionne avec un service web:

- D'abord, l'acteur clique sur un bouton;
- Ce clic se transforme en service REST;
- Un routeur transforme l'appel REST en une opération système envoyée à un contrôleur GRASP. Notez que c'est un **objet du domaine qui reçoit l'opération système** -- c'est l'essence du principe GRASP Contrôleur;
- Le contrôleur GRASP dirige la suite, selon la solution proposée dans la réalisation de cas d'utilisation (RDCU).

```{.plantuml caption="Une opération système est envoyée par la couche présentation et elle est reçue dans la couche domaine par son contrôleur GRASP. Ceci est un exemple avec un navigateur web, mais d'autres possibilités existent pour la couche présentation." #DSS-details}
@startuml
!include ../forme.pumlinclude
scale 1.3
skinparam sequenceMessageAlign center
skinparam participantPadding 10
skinparam sequenceBoxBorderColor transparent
actor ":Joueur" as j
box "Couche présentation" #ddffdd
participant ":Button" as b <<NavigateurWeb>> 
participant ":Routeur" as r <<NodeExpress>>
end box
box "Couche domaine" #ddddff
participant ":JeuRisk" as c <<ContrôleurGRASP>>
participant "..." as s
end box
j -> b : cliquer
b ->> r : HTTP GET\n/api/v1/démarrerAttaque/...
note over r : Router handler (Express)
r ->> r : demarrerAttaque
note over r, c: Opération système définie dans DSS
r -> c : **<size:18>demarrerAttaque(...)**
c -> s : ...
note over c, s : selon la RDCU
@enduml
```
