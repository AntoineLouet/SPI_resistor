# Contexte et objectifs du projet

> Document de travail à construire ensemble. Les `❓` sont les décisions à prendre.

## 1. Sujet

Concevoir un **trieur de résistances par reconnaissance à partir de photos** : une machine qui prend en photo une résistance, détermine sa valeur et la dirige vers le bon bac.

## 2. Problème posé

- Les résistances traversantes (THT) se repèrent par un code couleur de 4, 5 ou 6 bandes. Les lire à l'œil est lent et source d'erreurs : daltonisme, bandes marron / rouge / orange proches, usure, éclairage.
- En vrac, un atelier, un FabLab ou un établissement d'enseignement perd du temps à retrier ses composants.
- Les multimètres et LCR-mètres sont fiables mais lents et manuels. Nous voulons une solution **automatique, peu coûteuse et sans contact électrique**.

❓ À confirmer : la cible est-elle les résistances traversantes uniquement, ou aussi les CMS ?

## 3. Objectifs (premier jet à valider)

| Niveau | Objectif |
|--------|----------|
| Minimum viable | Identifier la valeur d'une résistance THT à partir d'une photo prise dans un environnement contrôlé (fond uni, éclairage fixe) |
| Cible | Chaîne complète : prise de vue, reconnaissance, affichage de la valeur, sortie vers le bon bac |
| Extension | Résistances CMS, mesure de contrôle, base de données de stock |

❓ Objectif de performance chiffré (taux de reconnaissance, temps par pièce, nombre de valeurs à trier) à fixer.

## 4. Périmètre

- **Dans le périmètre** (hypothèse) : acquisition d'image, algorithme de reconnaissance, électronique ou logiciel de commande, structure de la machine, tests.
- **Hors périmètre** (hypothèse) : alimentation automatique de pièces en vrac (bol vibrant), industrialisation.

❓ À trancher en équipe.

## 5. Contraintes

- Budget : ❓ (l'an dernier, environ 106 € de matériel)
- Délais : ❓
- Équipe : ❓ personnes
- Moyens disponibles : FabLab (impression 3D), commandes passées par l'école
- Légal : sécurité des machines et électrique (voir état de l'art)

## 6. Architecture envisagée par l'équipe

Caméra sur la machine, flux envoyé à un PC qui traite l'image, puis le PC envoie la valeur à un ESP32 qui pilote les actionneurs de tri.
Analyse, risques et alternatives : voir `03-architecture-et-alternatives.md`.

## 7. Organisation de l'équipe

| Membre | Rôle proposé |
|--------|--------------|
| ❓ | Chef de projet |
| ❓ | Vision / logiciel |
| ❓ | Électronique et commande |
| ❓ | Mécanique / CAO 3D |
| ❓ | État de l'art et rédaction |

## 8. Prochaines étapes

1. Valider les consignes (`00-consignes.md`) avec le tuteur
2. Répondre aux `❓` de ce document
3. Compléter l'état de l'art (`02`) et trancher les alternatives (`03`)
4. Finaliser la note de cadrage, le SWOT, le planning, le RACI, le budget et les risques (`04`)
