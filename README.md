# Suivi proactif des commandes clients

Un workflow n8n qui tourne chaque matin à 7 h, lit les commandes non livrées dans
l'ERP, décide qui doit être prévenu de quoi, et produit deux sorties : des emails
clients et une file de dossiers pour le support.

## Voir les pages

**→ https://alexisbadard-design.github.io/n8n-test/**

| | |
|---|---|
| [Rejeu de dix matins](https://alexisbadard-design.github.io/n8n-test/demo-10-jours.html) | 30 commandes, 10 matins, chaque décision et son motif |
| [La file du support](https://alexisbadard-design.github.io/n8n-test/tickets.html) | les alertes en tickets, avec l'historique de chaque dossier |

---

## L'idée

> `2 jours ouvrés de préparation + 1 jour de livraison` ⇒ pour tenir la date promise,
> tous les éléments doivent être arrivés **3 jours ouvrés avant**. Une commande encore
> en attente fournisseur après ce point est en retard **arithmétiquement**, même si la
> date annoncée n'est pas encore passée.

C'est ce calcul — et non le statut seul — qui déclenche l'information du client. Il
permet de prévenir jusqu'à trois jours ouvrés **à l'avance**, pendant que le client
peut encore replanifier. C'est la différence entre répondre au symptôme (« où en est
ma commande ») et au vrai grief (« on ne m'a pas prévenu »).

Tout est compté en **jours ouvrés** : week-ends et jours fériés français. Une commande
confirmée un vendredi ne s'expédie pas le samedi.

## Les trois règles qui gouvernent le reste

**Une date quand on la maîtrise, un engagement de recontact sinon.** Bloqué chez le
fournisseur, on n'a aucune visibilité : annoncer une date serait une promesse fabriquée
à partir d'une hypothèse qu'on sait fausse. On annonce alors une date de *recontact* —
une promesse qu'on tient entièrement.

**Le silence est une décision, et il est motivé.** Un email par commande et par jour au
maximum, jamais le week-end, trois messages automatiques puis un humain reprend la main.
Chaque commande sur laquelle le workflow se tait porte la raison de ce silence.

**Un engagement raté se répare, deux ne se réparent pas.** Une seule date ferme peut
être ajustée automatiquement, d'un jour. Au-delà, aucun email : un dossier s'ouvre pour
que quelqu'un reprenne le client à la main.

## Ce que contient ce dépôt

| Fichier | |
|---|---|
| `workflow.json` | L'export n8n, importable tel quel. 26 nœuds. |
| `dataset.json` | Les 30 commandes de test. Les dates sont relatives à la date d'exécution : le jeu reste valable dans le temps. |
| `etat-initial.csv` | La table d'état de départ, à importer dans la feuille Google Sheets pour que les commandes déjà relancées aient un passé cohérent. |
| `docs/` | Les deux pages ci-dessus : données, styles et scripts embarqués, aucun serveur. |

## Import

1. n8n → **Workflows → Import from File** → `workflow.json`
2. Vérifier Settings → *Timezone* : `Europe/Paris` (déjà réglé dans l'export)
3. **Execute workflow** (déclencheur « Exécution manuelle »)

Export réalisé depuis **n8n 2.37**. Les `typeVersion` employés (Schedule Trigger 1.4,
Set 3.5, Switch 3.4) demandent une instance récente ; sur une instance plus ancienne,
ces nœuds apparaîtraient à mettre à jour.

Le flux tourne immédiatement, **sans aucun credential** : la mémoire est tenue en
interne et rien ne sort. Pour rejouer une journée précise : `Config → DATE_EVALUATION`.

### Mise en service

Le premier matin en production, toutes les commandes sont vues pour la première fois :
`statut_depuis` vaut aujourd'hui pour toutes, et une commande en préparation depuis
deux jours est traitée comme si elle venait d'arriver. S'y ajoute le stock de retards
jamais notifiés, qui dépasse aussitôt le coupe-circuit — le premier jour, rien ne
partirait.

Ce n'est pas un défaut, c'est une bascule à conduire : une semaine en `dry_run` pour que
la mémoire mûrisse et que le support relise les sorties, seuil de coupe-circuit relevé
le premier jour, puis activation — en commençant par M5, le message le plus inoffensif.

### Rien n'est envoyé

Le mode `dry_run` est actif : les emails décidés sont écrits dans une feuille de
prévisualisation au lieu de partir. Les nœuds d'envoi — SendGrid, Google Sheets, Jira —
sont livrés **désactivés, credentials vides**. Passer en production est un changement de
valeur dans le nœud `Config`, rien d'autre.

VLD est une entreprise fictive : sociétés, contacts et numéros de commande du jeu de
test sont inventés, et les domaines en `.example` ne sont joignables par personne.
