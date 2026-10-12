# Éclairage, imagerie et mécanique : solutions étudiées

> Section du rapport : « Choix technologiques ». Même méthode que `03` : plusieurs solutions, une matrice, un choix justifié.
> Ce document vient de l'analyse de Claude à partir de connaissances générales d'ingénierie. **Aucune valeur n'est encore mesurée** : les protocoles de test de la section 2.5 servent à les confirmer.
> Les notes des matrices sont **provisoires** et à discuter en équipe.

## 1. Décisions déjà prises par l'équipe

| # | Décision | Conséquence |
|---|----------|-------------|
| D1 | Valeurs de la **série E12** | Voir 1.1 |
| D2 | Caméra dans une **boîte noire** avec lumière diffuse | Voir section 2 |
| D3 | **ESP32 classique** pour la commande | Voir 3.4 |

### 1.1 Ce que la série E12 apporte
- 12 valeurs par décade : 10, 12, 15, 18, 22, 27, 33, 39, 47, 56, 68, 82 (× 10ⁿ).
- **Correction d'erreurs gratuite pour la vision** : parmi les 90 combinaisons possibles de deux premiers chiffres, seules 12 sont valides. Si l'algorithme lit « 1 2 » ou « 1 5 », il confond peu ; s'il lit une paire invalide (par exemple 1 4), il peut l'écarter ou la corriger vers la valeur E12 la plus proche. À tester : cela peut augmenter nettement le taux de réussite.
- La tolérance habituelle de la série E12 est de 10 % (bande argent), mais on trouve aussi des E12 à 5 % (bande or). L'algorithme doit donc accepter or *et* argent, et le sens de lecture peut se déduire de cette bande.
- **Question de cadrage à trancher** : 7 décades de 1 Ω à 10 MΩ font environ 84 valeurs, ce qui est trop pour autant de bacs. Voir la section 4 : nombre de bacs.

## 2. Éclairage et prise de vue

### 2.1 Flash ou lumière continue ?
Mon avis : **lumière continue, pas de flash**, sauf si la résistance est en mouvement au moment de la photo.
- Si la pièce est **immobile** pendant la prise de vue (arrêtée dans un support), un flash n'apporte rien.
- Une webcam USB (UVC) n'a en général pas d'entrée de déclenchement matériel : le flash de l'ESP32 serait *asynchrone* avec l'exposition, donc des images de luminosité variable. C'est un risque à vérifier sur la caméra choisie.
- Une LED en PWM (variation par découpage) peut créer des bandes sur l'image (obturateur à balayage). Il faut alimenter les LED en **courant constant**, sans PWM, ou à une fréquence très élevée.
- Un flash devient utile si la résistance se déplace sur un tapis (pour figer le mouvement). Ce n'est pas le cas dans la plupart de nos architectures (section 3).

### 2.2 Solutions d'éclairage diffus
| # | Solution | Principe | Avantages | Inconvénients |
|---|----------|----------|-----------|---------------|
| L1 | **Anneau de LED** autour de l'objectif + diffuseur | Lumière frontale, alignée avec la caméra | Simple, disponible tout fait | Reflet circulaire sur les corps brillants |
| L2 | **Éclairage indirect** : LED cachées qui éclairent des parois blanches mates | Lumière renvoyée de tous côtés, ombres douces | Très diffus, peu cher | Boîte plus grande ; intérieur blanc (extérieur noir) |
| L3 | **Dôme diffusant (coupole)** : demi-sphère blanche, LED sur le bord dirigées vers le haut, caméra à travers un trou au sommet | Lumière « jour nuageux » venant de toutes directions | Souvent la meilleure solution pour limiter les reflets sur les objets cylindriques brillants | À fabriquer (impression 3D en PLA blanc ou bol translucide) |
| L4 | **Polarisation croisée** : film polarisant sur les LED, film polarisant tourné à 90° devant l'objectif | Supprime les reflets spéculaires | Très efficace contre les reflets de vernis | Perd de la lumière ; films à acheter ; à combiner avec L1 à L3 |
| L5 | **Deux bandes de LED latérales + diffuseurs** | Éclairage symétrique gauche / droite | Simple à construire | Reflets en lignes sur le corps |
| L6 | **Rétro-éclairage** (plan lumineux sous la pièce) | Silhouette nette | Excellent pour trouver le corps de la résistance et ses extrémités | Ne donne pas les couleurs : à combiner avec une lumière frontale |

### 2.3 Éléments qui comptent autant que la lampe
- **Intérieur** : mat, sans surface brillante. Fond mat de teinte neutre.
- **Lumière** : LED à bon indice de rendu des couleurs (IRC > 90), température de 5 000 à 6 500 K. Courant constant.
- **Caméra** : réglages **verrouillés** (exposition, gain, balance des blancs, mise au point). Sur une webcam, passer en manuel via les contrôles UVC.
- **Mire de calibrage** : un petit patch blanc ou gris et quelques couleurs de référence dans le champ, pour corriger automatiquement chaque image.
- **Support de la pièce** : une gouttière en V qui maintient la résistance couchée et alignée, avec les bagues visibles de dessus.
- **Résolution** : un corps de ¼ W mesure en gros 6 × 2,5 mm (valeur typique, à vérifier). Pour un champ d'environ 30 mm de large, une image de 640 pixels de large donne déjà environ 20 pixels par mm, donc des bagues de 10 pixels ou plus. Une webcam 720p ou 1080p suffit largement, à condition de pouvoir faire la mise au point à courte distance (un objectif macro ou une distance suffisante peuvent être nécessaires).

### 2.4 Pièges colorimétriques à anticiper
Marron / rouge / orange, or / jaune / marron clair, violet / bleu, gris / argent, et corps bleus qui masquent les bagues bleues. La correction E12 (1.1) et la mire de calibrage réduisent ces risques.

### 2.5 Protocole de comparaison (pour justifier le choix de façon mesurée)
1. Construire 3 maquettes bon marché (par exemple L1, L3, L3 + L4).
2. Photographier les **mêmes 30 résistances** (E12, 4 bandes, corps de couleurs variées), 3 fois chacune, dans chaque maquette.
3. Mesurer : part de pixels saturés (reflets), écart entre les groupes de couleurs (par exemple distance dans l'espace HSV / Lab), répétabilité d'une prise à l'autre, taux de bonne lecture par l'algorithme.
4. Choisir la maquette gagnante et consigner les chiffres dans le rapport.

## 3. Mécanique : solutions étudiées

Le système doit enchaîner **alimenter une pièce → l'immobiliser dans la boîte d'imagerie → l'envoyer dans le bon bac**.

### 3.1 Solutions de tri (vers les bacs)
| # | Solution | Principe | Atouts | Limites |
|---|----------|----------|--------|---------|
| M1 | **Toboggan / tube rotatif** | La pièce tombe dans un tube incliné monté sur un axe moteur ; le moteur oriente le tube vers l'un des bacs disposés en cercle. La pièce est lâchée une fois le tube en place | Un seul moteur pour le tri, très compact, pas de synchronisation délicate (on positionne *puis* on lâche) | Rebonds possibles, fils qui s'accrochent, positionnement angulaire à fiabiliser |
| M2 | **Roue à alvéoles + carrousel de bacs** | Une roue à logements amène la pièce de la zone d'alimentation à la boîte d'imagerie puis à une zone d'éjection ; un second plateau tourne pour présenter le bon bac | Pièce toujours bien guidée | Deux axes motorisés à synchroniser |
| M3 | **Arbre de volets** | Des volets motorisés (un par embranchement) dirigent la pièce, comme un arbre binaire | Aucun mouvement lent, cadence élevée | Il faut environ N−1 volets pour N bacs, très coûteux et encombrant dès qu'on a beaucoup de bacs |
| M4 | **Chariot linéaire** | Un entonnoir ou un tube se déplace sur un rail devant une rangée de bacs | Simple à comprendre | Longueur de la machine proportionnelle au nombre de bacs |
| M5 | **Portique XY (type traceur)** | Un tube se déplace au-dessus d'une grille de bacs | Beaucoup de bacs possibles | Deux axes, plus de mécanique, plus de réglages |
| M6 | **Bras articulé (pick-and-place)** | Un bras saisit la pièce (pince ou ventouse) et la dépose | Flexible, spectaculaire | Saisir un petit cylindre avec fils est difficile ; coût et complexité les plus élevés |
| M7 | **Convoyeur + poussoirs** | Les pièces défilent sous la caméra et un poussoir devant chaque bac les éjecte | Cadence élevée | Un poussoir par bac, synchronisation par position ou temps |
| M8 | **Jet d'air** | Une électrovanne souffle la pièce qui tombe vers un bac | Pas de pièce mobile au contact | Compresseur, bruit, pièces très légères donc dispersion ; non retenu en première approche |

### 3.2 Solutions d'alimentation (une pièce à la fois)
| # | Solution | Principe | Avantages | Inconvénients |
|---|----------|----------|-----------|---------------|
| F1 | **Dépose manuelle** dans une gouttière | L'utilisateur place une pièce à la fois | Très simple, parfait pour valider la vision | Peu automatique |
| F2 | **Trémie + échappement à volet** | Les pièces s'alignent en file dans une goulotte et un servo en libère une | Automatique, peu coûteux | Risque d'enchevêtrement et de pièces qui se couchent mal |
| F3 | **Roue à alvéoles** (doseur) | Chaque rotation prélève une pièce dans la trémie | Une pièce à la fois, bien orientée | Pièces à fils longs qui se croisent ; réglage délicat |
| F4 | **Rail vibrant / bol vibrant** | Les vibrations alignent les pièces | Automatique | Complexe, bruyant, hors périmètre pour une v1 |
| F5 | **Pièces sur bande** (conditionnement en ruban) | Les résistances vendues sur ruban sont avancées une par une | Orientation parfaite | Ne convient pas aux pièces en vrac |

**Pour la v1** : F1 (manuel), puis F2 ou F3 en amélioration si le temps le permet.

### 3.3 Matrice de décision : tri
Notes de 1 à 5. Total = somme (note × poids). Maximum possible : 75.

| Critère | Poids | M1 Tube rotatif | M2 Roue + carrousel | M3 Volets | M4 Chariot | M5 Portique XY | M6 Bras | M7 Convoyeur |
|---------|-------|-----------------|---------------------|-----------|------------|----------------|---------|--------------|
| Nombre de bacs atteignables | 3 | 4 | 4 | 2 | 3 | 5 | 5 | 3 |
| Simplicité de réalisation | 3 | 5 | 3 | 2 | 3 | 2 | 1 | 2 |
| Fiabilité (blocages, rebonds) | 3 | 4 | 4 | 3 | 4 | 3 | 2 | 3 |
| Coût | 2 | 5 | 4 | 1 | 3 | 2 | 1 | 2 |
| Compacité | 1 | 4 | 3 | 2 | 2 | 2 | 3 | 2 |
| Facilité de commande (ESP32) | 2 | 5 | 3 | 2 | 4 | 3 | 2 | 2 |
| Cadence | 1 | 3 | 4 | 5 | 3 | 3 | 2 | 5 |
| **Total** | | **66** | **54** | **34** | **49** | **45** | **35** | **39** |

**Solution pressentie : M1 (tube rotatif sur couronne de bacs)**. Elle repose sur un seul moteur pour le tri et ne nécessite pas de synchronisation fine : on positionne le tube, puis on lâche la pièce. Les solutions M2 et M4 constituent des replis raisonnables si M1 pose problème de fiabilité.

### 3.4 Détails de M1 et choix des actionneurs (ESP32 classique)
**Chaîne verticale compacte (proposition)** :

```
 [Dépôt manuel / trémie]
          |  (chute)
 [Gouttière en V + volet-trappe commandé par un servo]   <- pièce immobilisée
          |   boîte noire : caméra au-dessus, éclairage diffus (dôme)
          v   (le volet s'ouvre une fois le tube orienté)
 [Tube incliné monté sur l'axe d'un moteur pas à pas]  ---> oriente la pièce
          v
 [Couronne de bacs (N ouvertures) + bac de rejet]
```

| Choix | Options | Remarque |
|-------|---------|----------|
| Moteur du tube | Servo 180° : simple mais angle limité et précision moyenne ; **moteur pas à pas** (28BYJ-48 bon marché mais lent, ou NEMA17 + driver) : 360° et position répétable | Le pas à pas avec un **capteur de référence** (fin de course, optique ou à effet Hall) paraît le meilleur compromis |
| Volet / trappe | Servo | Plus simple qu'un électro-aimant |
| ESP32 classique | Suffisant pour ces commandes | Pas d'USB natif : le port série passe par la puce USB-série de la carte de développement, ce qui convient pour dialoguer avec le PC |
| Alimentations | Moteur et servos sur alimentation 5 V ou 12 V séparée de la logique 3,3 V de l'ESP32 | Masse commune ; condensateurs de découplage |
| Éclairage | LED à courant constant | Commutable par l'ESP32 par un transistor (pas de PWM visible dans l'image) |

**Estimation de cadence (à mesurer)** : alimentation et immobilisation (~0,5 s) + prise de vue et analyse (~0,5 s) + rotation du tube (jusqu'à ~1 s) + chute (~0,3 s), soit environ 2 à 3 s par pièce, c'est-à-dire de l'ordre de 20 à 30 pièces par minute. Valeur à confirmer par des essais.

## 4. Nombre de bacs : décision de cadrage à prendre

Avec M1 on peut raisonnablement viser quelques dizaines de millimètres d'ouverture par bac sur une couronne de 15 à 20 cm de diamètre, soit de l'ordre de **12 à 16 bacs**. À trancher :

| Option | Bacs | Intérêt |
|--------|------|---------|
| (a) 12 valeurs E12 d'une même décade + 1 bac de rejet | 13 | Démonstration claire, très réaliste pour une v1 |
| (b) Valeurs de 100 Ω à 100 kΩ (4 décades) | 48 + rejet | Utile en pratique, mais incompatible avec un seul tube rotatif simple |
| (c) Bacs configurables par logiciel : l'utilisateur affecte une valeur à chaque bac, le reste va au bac « autre » | 12 à 16 | Souple, bon compromis |
| (d) Tri en deux passes (d'abord la décade, puis la valeur) | 2 × 12 | Étend la capacité sans plus de mécanique, mais demande de repasser les pièces |

**Recommandation : (c)**, avec (a) comme configuration de démonstration. Le choix final dépend des objectifs de l'équipe.

## 5. Architecture globale pressentie (à valider)

1. Dépôt manuel d'une résistance dans la gouttière en V (F1).
2. Caméra USB dans une boîte noire, éclairage diffus continu en dôme avec polarisation si nécessaire (L3, éventuellement + L4).
3. PC : OpenCV, décodage de la valeur avec correction E12, envoi du numéro de bac par USB série.
4. ESP32 : oriente le tube (M1), ouvre la trappe, confirme la fin du mouvement.
5. Tests : jeu d'images de référence, mesures de précision et de cadence.

## 6. Prochaines actions

- [ ] Choisir l'option de nombre de bacs (section 4)
- [ ] Corriger les notes de la matrice (3.3) avec les priorités de l'équipe
- [ ] Choisir et commander une caméra (mise au point à courte distance, contrôles manuels)
- [ ] Construire les maquettes d'éclairage et photographier le jeu de test (2.5)
- [ ] Prototyper le tube rotatif en carton ou impression 3D rapide pour vérifier les rebonds
- [ ] Réunir 30 à 50 résistances E12 de couleurs de corps variées pour les tests
