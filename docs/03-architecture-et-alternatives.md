# Architecture et alternatives étudiées

> Section du rapport : « Choix technologiques ». Principe : **on compare plusieurs solutions avant de choisir**, puis on justifie.
> Les notes des matrices sont **provisoires**. Elles viennent de l'analyse de Claude et doivent être discutées et corrigées en équipe.
> Sources numérotées [n] : voir `02-etat-de-l-art.md`, section « Références ».

## 1. Architecture imaginée par l'équipe (solution A)

```
 Caméra (sur la machine) --flux--> PC (traitement d'image) --valeur--> ESP32 --> microcontrôleurs --> tri de la résistance
```

### Ce qui est bon
- Le PC offre toute la puissance de calcul : OpenCV, Python et, si besoin, un réseau de neurones. La mise au point est facile (affichage, enregistrement d'images).
- Le découpage « vision / commande » est classique. Des projets de tri par couleur séparent de la même façon un sous-système de vision (Pi + OpenCV), un sous-système de commande (Arduino) et un diviseur mécanique [1].
- L'ESP32 est peu cher et bien documenté.

### Points à clarifier ou à risque
| # | Point | Pourquoi c'est important |
|---|-------|--------------------------|
| 1 | **Un PC est nécessaire pour faire fonctionner la machine** | La démo en soutenance dépend d'un ordinateur, de ses pilotes et de ses mises à jour. Prévoir un PC dédié ou un plan B. |
| 2 | **« L'ESP32 gère des microcontrôleurs »** | Un ESP32 sait déjà piloter plusieurs servos, un driver de moteur pas à pas, des LED, etc. Des microcontrôleurs supplémentaires ne sont probablement pas nécessaires : à justifier ou à supprimer. |
| 3 | **Liaison PC ↔ ESP32** | La liaison USB série est la plus simple et la plus fiable. Le Wi-Fi ajoute de la latence et des pannes. À comparer. |
| 4 | **Le plus dur est la mécanique, pas la vision** | Présenter *une seule* résistance à la caméra, dans la bonne position, puis la diriger vers le bon bac au bon moment (synchronisation) : c'est le point critique. Les solutions d'alimentation et de tri doivent être étudiées avec autant de soin que la reconnaissance. |
| 5 | **Éclairage** | Les projets existants signalent que les reflets et le mélange de corps clairs et sombres compliquent l'analyse [2]. Boîte fermée et éclairage maîtrisé : conditions du succès. |
| 6 | **Pas de mesure de contrôle** | La vision lit la valeur *marquée*, pas la valeur réelle. Une résistance abîmée ou mal marquée sera mal triée (voir variante F). |

## 2. Alternatives sur chaque bloc

### 2.1 Calcul et acquisition
| Option | Description | Avantages | Inconvénients |
|--------|-------------|-----------|---------------|
| **A. Caméra USB + PC** (équipe) | Le PC traite, l'ESP32 commande | Puissance, simplicité de développement | Dépendance au PC, machine non autonome |
| **B. Caméra + Raspberry Pi** | Le Pi traite et commande (GPIO ou ESP32) | Autonome et compact. Existe dans des projets de tri [2][3] | Moins puissant. Un projet a dû optimiser son algorithme, trop lent sur Pi [2] |
| **C. ESP32-CAM seul** | Tout sur la puce | Très bon marché, très compact | Pas de benchmark OpenCV trouvé. En streaming, 10 à 18 images/s à basse résolution avec 140 à 240 ms de latence [4]. Reconnaissance fine peu réaliste |
| **D. Carte hybride Linux + microcontrôleur** | Une seule carte avec les deux (ex. Arduino UNO Q, cité dans un projet de tri [5]) | Un seul module pour vision et commande | Écosystème plus récent. À vérifier : prix, disponibilité, performances |

**Piste recommandée (à valider)** : **A simplifiée** (caméra USB + PC + ESP32 en USB série) pour le développement, avec **B** comme plan d'évolution vers une machine autonome. Les deux partagent le même code Python.

### 2.2 Algorithme de reconnaissance
| Option | Précision rapportée | Données nécessaires | Remarque |
|--------|---------------------|---------------------|----------|
| **Vision classique (OpenCV)** : segmentation, HSV, contours, décodage | Réalisé sans apprentissage dans un projet de tri [2] | Aucune, mais un calibrage | Rapide, explicable, partiellement exploitable avant que la pièce soit entièrement dans le champ [2] |
| **CNN de classification** | ~92 % à l'entraînement et ~90 % en validation (conditions contrôlées), environ 72 % en mauvais éclairage ou mal aligné [6] | ~1 200 images dans cette étude | Sensible aux conditions |
| **Détection par réseau (YOLO) + décodage** | Proposé comme travail futur dans [6]. Jeux d'images annotés publics sur Roboflow Universe [7] | Un jeu de données annoté (existant ou à constituer) | Robuste, mais plus lourd |
| **Hybride** | À étudier | | Détection de la résistance par réseau, puis lecture classique des bandes |

**Piste recommandée** : commencer par la **vision classique** dans une boîte à éclairage contrôlé (rapide à obtenir, explicable dans le rapport), en collectant des images dès le début. Si la précision est insuffisante, passer à l'**hybride / YOLO**. Cela permet de présenter une comparaison chiffrée dans le rapport.

À régler dès le départ : le code couleur ne se lit que dans un sens, il faut gérer 4, 5 et 6 bandes, et la dernière bande donne la tolérance [3]. On peut aussi contraindre le résultat aux valeurs des séries normalisées (voir IEC 60063 dans les références).

### 2.3 Alimentation (une résistance à la fois)
| Option | Principe | Avantages | Inconvénients |
|--------|----------|-----------|---------------|
| **1. Manuelle** | L'utilisateur dépose une pièce à la fois | Très simple, parfait pour une v1 | Peu automatique |
| **2. Goulotte à gravité + échappement** | Les pièces glissent, une servo-vanne en laisse passer une | Automatique, peu coûteuse | Les pièces peuvent se coincer ou se chevaucher |
| **3. Tapis / convoyeur** | Les pièces avancent sous la caméra | Débit élevé. Utilisé dans un projet de tri [3] | Plus de mécanique et de motorisation |
| **4. Bol vibrant / trémie** | Alimentation en vrac | Totalement automatique | Complexe, risque de dérive du projet |

### 2.4 Tri vers le bon bac
| Option | Principe | Avantages | Inconvénients |
|--------|----------|-----------|---------------|
| **a. Goulotte + servo-volet** | Un volet oriente la pièce vers 2 à 4 bacs | Très simple et peu coûteux | Nombre de bacs limité |
| **b. Plateau tournant indexé** | Un moteur pas à pas ou servo positionne un plateau (ou la goulotte) sur N bacs | Beaucoup de bacs (par exemple 12 valeurs) avec un seul actionneur | Précision du positionnement, vitesse |
| **c. Convoyeur + poussoirs** | Un poussoir par bac le long du tapis | Bonne cadence. Se retrouve dans un projet [3] | Plus d'actionneurs, synchronisation |
| **d. Bras pick-and-place** | Un bras dépose la pièce | Flexible | Complexe, coûteux |

**Piste recommandée** : **b** (plateau ou goulotte indexée) si l'objectif est un grand nombre de valeurs, ou **a** pour une v1 avec peu de bacs. Le nombre de bacs retenu est une décision de cadrage (voir contexte).

### 2.5 Variante F : vérification électrique
Ajouter une mesure de résistance (contacts + convertisseur analogique-numérique de l'ESP32) pour contrôler la valeur lue. Cela rend le tri plus fiable et fournit une vérité terrain pour valider l'algorithme de vision. Non nécessaire en v1, mais précieux pour les **tests**.

## 3. Matrice de décision (provisoire)

Notes de 1 (mauvais) à 5 (très bon). Poids : importance du critère, à fixer en équipe. Total = somme (note × poids).

### 3.1 Architecture de calcul
| Critère | Poids | A. PC + ESP32 | B. Raspberry Pi | C. ESP32-CAM seul | D. Carte hybride |
|---------|-------|---------------|-----------------|-------------------|------------------|
| Puissance / précision de reconnaissance | 3 | 5 | 3 | 1 | 3 |
| Facilité de développement | 3 | 5 | 4 | 2 | 3 |
| Autonomie de la machine | 2 | 1 | 5 | 5 | 5 |
| Coût | 2 | 4 | 3 | 5 | 3 |
| Risque technique | 3 | 4 | 3 | 1 | 2 |
| **Total** | | **52** | **46** | **32** | **40** |

### 3.2 Reconnaissance
| Critère | Poids | Vision classique | CNN seul | YOLO + décodage | Hybride |
|---------|-------|------------------|----------|-----------------|---------|
| Précision attendue | 3 | 3 | 3 | 4 | 4 |
| Besoin de données | 2 | 5 | 2 | 2 | 3 |
| Temps de développement | 3 | 4 | 3 | 2 | 2 |
| Explicabilité dans le rapport | 2 | 5 | 2 | 3 | 4 |
| Robustesse à la lumière | 3 | 2 | 2 | 4 | 4 |
| **Total** | | **47** | **32** | **40** | **44** |

### 3.3 Tri mécanique
| Critère | Poids | a. Volet | b. Plateau indexé | c. Convoyeur | d. Bras |
|---------|-------|----------|-------------------|--------------|---------|
| Nombre de valeurs triables | 3 | 2 | 5 | 4 | 5 |
| Simplicité de réalisation | 3 | 5 | 3 | 2 | 1 |
| Coût | 2 | 5 | 4 | 3 | 2 |
| Fiabilité mécanique | 3 | 4 | 3 | 3 | 2 |
| Cadence | 1 | 3 | 2 | 4 | 2 |
| **Total** | | **46** | **43** | **37** | **30** |

> Ces matrices servent de modèle : les poids dépendent des objectifs que l'équipe fixera (par exemple « beaucoup de valeurs » favorise la solution b).

## 4. Architecture de référence proposée

```
 [Trémie / dépôt manuel] -> [Échappement 1 pièce] -> [Zone d'imagerie : boîte fermée, LED diffuse, fond uni, caméra USB]
        |                                                         |
        |                                                  (USB) v
        |                                          PC : OpenCV -> valeur (Ω, tolérance)
        |                                                         |
        |                                                  (USB série) v
        +----------------------------- ESP32 : servos / moteur pas à pas ---------------> [Plateau indexé -> bacs]
```

## 5. Risques et questions techniques à traiter tôt

1. Quelle résolution et quelle optique permettent de lire des bandes de 1 mm ? (caméra macro, distance)
2. Comment éviter le chevauchement et l'orientation aléatoire de la pièce (la résistance est symétrique : le sens de lecture doit être déterminé) ?
3. Comment synchroniser le moment de la décision avec le moment du tri (délai de chute) ?
4. Combien de temps par pièce, et quelle cadence vise-t-on ?
5. Comment constituer le jeu d'images de test (valeurs, tolérances, marques, corps clairs et sombres, 4 / 5 bandes) ?
