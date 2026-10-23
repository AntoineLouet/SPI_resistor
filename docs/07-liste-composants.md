# Liste des composants et achats

> Sections du rapport : « Présentation des composants », « Choix et justification des composants » et « Budget ».
> Les prix sont des **estimations** (ordre de grandeur AliExpress et distributeurs français, octobre 2026), **à vérifier le jour de la commande**. Ils servent de « prévu » dans le budget (`04` §6) ; le « réel » sera relevé sur les factures.
> La comparaison des références candidates, avec fiches techniques et calculs, est dans `08-choix-des-composants.md`.

## 1. Ce que la liste suppose

| Bloc | Choix | Statut | Où c'est argumenté |
|------|-------|--------|--------------------|
| Mécanique de tri | **Plateau de bacs tournant** sous une trappe fixe | Pris (D7) | §1.1 |
| Nombre de bacs | 13 : les 12 valeurs E12 d'une décade + 1 bac de rejet | Pressenti (D9) | `05` §4 |
| Moteur du plateau | NEMA 17 + driver TMC2209 | Pressenti (D11) | §1.1, `06` §4 |
| Référence du plateau | Capteur optique à fourche + drapeau imprimé sur le plateau | Pressenti | `06` §4.2 |
| Trappe | Servo MG90S + volet | Pressenti (D12) | `06` §5 |
| Contrôle de chute | Barrière infrarouge sous la trappe | Pressenti (D12) | `06` §6 |
| Caméra | Module USB 1080p (capteur OV2710) avec objectif M12 de 8 ou 12 mm, reliée au PC | Pressenti (D14) | §1.1, `08` §5 |
| Éclairage | Bande LED à fort rendu des couleurs sous un dôme blanc imprimé (L3), polarisation (L4) en option | Pressenti (D6) | `05` §2 |
| Commande | ESP32 classique, liaison USB série avec le PC | Pris (D3) / pressenti (D10) | `06` §2 |
| Arrivée des pièces | Dépôt manuel, une pièce à la fois (F1) | Pressenti (D8) | `05` §3.2 |
| Achats | Consommables par le CESI, le reste sur AliExpress aux frais de l'équipe | Pris (D15) | §2 |

### 1.1 Justification des trois choix qui fixent la liste

**Plateau de bacs tournant** (à la place du tube rotatif M1 pressenti dans `05`) :
- La trappe est fixe et la pièce tombe droit dans le bac placé dessous. Le trajet est court et il n'y a pas de tube où elle peut rebondir ou s'accrocher par ses pattes.
- Tous les capteurs sont fixes (barrière sous la trappe, capteur de référence) : aucun fil ne passe sur une pièce qui tourne. Avec M1, la barrière « à la sortie du tube » tournait avec lui.
- Limite : le plateau s'alourdit à mesure que les bacs se remplissent. Un roulement de plateau tournant porte le poids ; le moteur ne fait qu'entraîner.
- La roue à alvéoles (M2, deux moteurs) reste une amélioration possible si l'on automatise l'arrivée des pièces.

**NEMA 17 + TMC2209** (plutôt que 28BYJ-48) : avec 13 bacs, deux bacs voisins sont séparés de 27,7°. Le 28BYJ-48 met environ 2 s pour un demi-tour et son réducteur a du jeu. Le NEMA 17 fait 200 pas par tour (3 200 en micro-pas 1/16) : il est précis, rapide et a de la marge pour un plateau chargé. Le TMC2209 le rend silencieux. Surcoût d'environ 10 €, plus une alimentation 12 V.

**Module caméra USB à objectif M12** : la mise au point est manuelle et reste fixe une fois l'objectif vissé, ce qui donne des images reproductibles. Le module est petit et se fixe facilement dans la boîte noire. Il est reconnu sans pilote (norme UVC) par OpenCV sur le PC, puis sur un Raspberry Pi si la machine devient autonome. Écartés : la webcam grand public (autofocus qui peut changer de réglage, encombrante) et le microscope USB (LED intégrées sources de reflets, rendu des couleurs médiocre, champ petit).

## 2. Qui paie quoi

- **Le CESI paie les consommables** : ce qui est consommé pendant le projet ou rendu à la fin (restes de filament, résistances de test). Commande passée par le responsable de formation.
- **L'équipe paie tout ce qui reste sur la machine**, pour pouvoir la garder après le projet. Commande sur **AliExpress**.

⚠️ À confirmer avec le responsable de formation :
1. Les pièces imprimées avec le filament du CESI peuvent rester sur notre machine.
2. Les fournisseurs chez qui il commande (les références du §3 sont données par caractéristiques pour être trouvées chez n'importe lequel).
3. Le FabLab fournit-il déjà l'étain et la gaine thermorétractable ? Sinon, peut-on les ajouter aux consommables (K5) ?

## 3. Consommables commandés par le CESI

| Réf. | Article | Spécification à respecter | Qté | Prix estimé (€) |
|------|---------|---------------------------|-----|-----------------|
| K1 | Filament PLA noir mat | **1,75 mm** ⚠️ (2,85 mm si l'imprimante du FabLab est une Ultimaker), bobine de 1 kg. Le noir mat limite les reflets dans la boîte | 1 | 22,00 |
| K2 | Filament PLA blanc | Même diamètre que K1. Bobine de 250 ou 500 g si le fournisseur en propose, sinon 1 kg | 1 | 22,00 |
| K3 | Assortiment de résistances E12, film carbone 5 % | ¼ W, **4 bandes**, corps beige. Toutes les valeurs E12 d'au moins 10 Ω à 1 MΩ, **au moins 10 pièces par valeur**. Vérifier la liste des valeurs : beaucoup de kits sautent 1,2 k, 3,9 k ou 8,2 k et ajoutent des valeurs E24 (2 k, 5,1 k) | 1 | 12,00 |
| K4 | Assortiment de résistances film métal 1 % | ¼ W, 5 bandes, corps bleu, au moins 10 pièces par valeur. Sert à tester la vision sur des corps bleus (bagues bleues masquées, `05` §2.4) | 1 | 8,00 |
| | **Total CESI** | | | **64,00** |
| K5 | *Option à valider* : étain 0,8 mm + gaine thermorétractable | Seulement si le FabLab n'en fournit pas | 1 lot | *8,00* |

### 3.1 Estimation du filament

Ordre de grandeur à recalculer avec le logiciel de tranchage une fois la CAO faite.

| Pièce | Couleur | Masse estimée |
|-------|---------|---------------|
| Plateau (Ø ≈ 200 mm) + 13 bacs amovibles | Noir | ≈ 170 g |
| Boîte noire (parois, couvercle, support de caméra) | Noir | ≈ 250 g |
| Trappe, gouttière en V, supports du moteur et des capteurs, bâti | Noir | ≈ 100 g |
| Dôme diffuseur | Blanc | ≈ 50 g |
| Versions successives et impressions ratées (≈ + 50 %) | Les deux | ≈ 285 g |
| **Total** | | **≈ 780 g de noir, ≈ 75 g de blanc** |

Une bobine de 1 kg de noir suffit. Pour économiser du filament et du temps d'impression, les parois de la boîte noire peuvent être faites en carton plume noir.

## 4. Matériel à la charge de l'équipe (AliExpress)

Les mots-clés sont en anglais : c'est ainsi que les annonces AliExpress sont le mieux trouvées. **Retirer de la liste ce qu'un membre possède déjà.**

### 4.1 Commande et capteurs

| Réf. | Composant | Rôle | Spécification à respecter | Qté | Prix estimé (€) | Mots-clés AliExpress |
|------|-----------|------|---------------------------|-----|-----------------|----------------------|
| C1 | Carte ESP32 DevKit V1 | Commande de la machine | ESP32-WROOM-32, 38 broches, port USB-C, puce USB-série CP2102 ou CH340 | 1 | 4,00 | `ESP32 DevKit 38 pin USB-C CP2102` |
| C2 | Moteur pas à pas NEMA 17 | Rotation du plateau | 17HS4401 : 1,8° par pas, 1,7 A, ≈ 40 N·cm, arbre de 5 mm, câble fourni | 1 | 9,00 | `17HS4401 nema 17 stepper motor` |
| C3 | Driver TMC2209 | Pilotage du moteur | Module au format StepStick (ex. BIGTREETECH V1.3) | 1 | 4,00 | `TMC2209 stepstick driver` |
| C4 | Assortiment de condensateurs électrolytiques | Découplage | Contient du 100 µF ≥ 25 V (aux bornes de l'alimentation du driver) | 1 lot | 2,00 | `electrolytic capacitor assortment kit` |
| C5 | Servo MG90S | Ouverture de la trappe | Pignons métal, 4,8 à 6 V | 1 | 2,50 | `MG90S servo metal gear` |
| C6 | Capteur optique à fourche | Position de référence du plateau | Module LM393, fente d'environ 5 mm, sortie numérique | lot de 5 | 2,00 | `slot type optical sensor module LM393` |
| C7 | Barrière infrarouge (émetteur + récepteur) | Vérifier que la pièce est tombée | Paire de LED 5 mm, portée ≥ 10 cm | 2 paires | 3,00 | `IR break beam sensor 5mm` |
| | **Sous-total** | | | | **26,50** | |

### 4.2 Alimentation et sécurité

| Réf. | Composant | Rôle | Spécification à respecter | Qté | Prix estimé (€) | Mots-clés AliExpress |
|------|-----------|------|---------------------------|-----|-----------------|----------------------|
| A1 | Bloc secteur 12 V 3 A | Alimentation des actionneurs | **Prise européenne, marquage CE**, sortie jack 5,5 × 2,1 mm | 1 | 8,00 | `12V 3A power adapter EU plug` |
| A2 | Embase jack femelle vers bornier | Raccorder le bloc secteur | Jack 5,5 × 2,1 mm | lot de 5 | 1,50 | `DC jack female 5.5x2.1 terminal` |
| A3 | Convertisseur abaisseur réglable | 12 V → 5 V pour le servo et les LED | XL4015, jusqu'à 5 A (plus de marge que le LM2596) | 1 | 2,50 | `XL4015 step down module` |
| A4 | Bouton d'arrêt d'urgence | Couper le 12 V des actionneurs | Coup de poing 22 mm, contact NC, avec boîtier | 1 | 4,00 | `emergency stop button 22mm box` |
| A5 | Porte-fusible en ligne + fusibles | Protéger contre un court-circuit | Fusibles 3 A | 1 lot | 1,50 | `inline fuse holder 5x20` |
| | **Sous-total** | | | | **17,50** | |

### 4.3 Vision et éclairage

| Réf. | Composant | Rôle | Spécification à respecter | Qté | Prix estimé (€) | Mots-clés AliExpress |
|------|-----------|------|---------------------------|-----|-----------------|----------------------|
| V1 | Module caméra USB 1080p | Prise de vue, reliée au PC | UVC (sans pilote), objectif M12 × 0,5 à mise au point manuelle, **avec filtre IR (« IR-cut »)**, réglage manuel de l'exposition et de la balance des blancs (ex. ELP à capteur OV2710 ou équivalent) | 1 | 30,00 | `ELP USB camera 1080P M12 lens OV2710` |
| V2 | Bande LED COB | Éclairage diffus sous le dôme | **IRC ≥ 90**, 5 000 à 6 500 K, 5 V, 1 m (≈ 50 cm utilisés). Si la version IRC ≥ 90 n'existe qu'en 12 V, la prendre en 12 V et l'alimenter directement sur le 12 V | 1 | 5,00 | `COB LED strip 5V CRI 90` |
| V3 | Module MOSFET | Allumer les LED depuis l'ESP32, sans PWM | D4184 (AOD4184), commandable en 3,3 V | 1 | 1,00 | `D4184 MOSFET module` |
| V4 | Lot d'objectifs M12 (6, 8 et 12 mm) | Champ adapté à la pièce | Filetage M12 × 0,5. L'objectif fourni (souvent 2,1 à 3,6 mm) oblige à placer la caméra à 1,5 à 2,5 cm de la pièce, sans place pour l'éclairage (`08` §5). Alternative : commander directement la version à zoom 2,8–12 mm de la caméra | 1 lot | 6,00 | `M12 lens set 8mm 12mm` |
| | **Sous-total** | | | | **42,00** | |

### 4.4 Mécanique

| Réf. | Composant | Rôle | Spécification à respecter | Qté | Prix estimé (€) | Mots-clés AliExpress |
|------|-----------|------|---------------------------|-----|-----------------|----------------------|
| M1 | Roulement de plateau tournant | Porter le poids du plateau et des bacs | Ø 100 à 150 mm | 1 | 4,00 | `lazy susan turntable bearing` |
| M2 | Moyeu à bride rigide | Fixer le plateau sur l'arbre du moteur | Alésage 5 mm, vis de serrage | 1 | 2,00 | `rigid flange coupling 5mm` |
| M3 | Assortiment de vis et écrous | Assemblage, fixation du moteur | M3, longueurs 6 à 20 mm | 1 boîte | 5,00 | `M3 screw nut assortment kit` |
| | **Sous-total** | | | | **11,00** | |

### 4.5 Câblage et prototypage

| Réf. | Composant | Rôle | Spécification à respecter | Qté | Prix estimé (€) | Mots-clés AliExpress |
|------|-----------|------|---------------------------|-----|-----------------|----------------------|
| P1 | Plaque d'essai + câbles Dupont | Essais avant la carte finale | 830 points, câbles M-M, M-F et F-F | 1 kit | 4,00 | `breadboard 830 dupont wire kit` |
| P2 | Plaque à pastilles + connecteurs | Carte finale | 7 × 9 cm, barrettes femelles 2,54 mm (pour enficher l'ESP32 et le driver), borniers à vis au pas de 5,08 mm | 1 lot | 4,00 | `perfboard 7x9 female header screw terminal` |
| P3 | Fil de câblage | Liaisons de la carte finale | Silicone 22 AWG, plusieurs couleurs | 1 lot | 4,00 | `22AWG silicone wire kit` |
| P4 | Câble USB-C | Liaison PC ↔ ESP32 | Câble de **données** (pas seulement de charge), 1 m | 1 | 1,50 | `USB C data cable 1m` |
| | **Sous-total** | | | | **13,50** | |

### 4.6 Options

| Réf. | Composant | Intérêt | Qté | Prix estimé (€) | Mots-clés AliExpress |
|------|-----------|---------|-----|-----------------|----------------------|
| O1 | Carte d'extension à borniers pour ESP32 38 broches | Câblage sans soudure ; utile car certaines cartes ESP32 sont trop larges pour une plaque d'essai. Vérifier la largeur de la carte C1 | 1 | 2,50 | `ESP32 38 pin breakout board terminal` |
| O2 | Film polarisant linéaire | Tester la polarisation croisée contre les reflets (L4, `05` §2.2) | 1 feuille | 4,00 | `linear polarizer film sheet` |
| O3 | Inserts filetés en laiton M3 à poser à chaud | Fixations plus solides et démontables que des vis directement dans le PLA | lot de 50 | 3,00 | `M3 heat set insert brass` |
| | **Sous-total options** | | | **9,50** | |

### 4.7 Pièces de rechange conseillées

Ces trois pièces se détruisent facilement sur une erreur de câblage. À 2 à 3 semaines de livraison, une rechange évite de bloquer le projet.

| Réf. | Composant | Qté | Prix estimé (€) |
|------|-----------|-----|-----------------|
| R1 | Carte ESP32 (comme C1) | 1 | 4,00 |
| R2 | Driver TMC2209 (comme C3) | 1 | 4,00 |
| R3 | Servo MG90S (comme C5) | 1 | 2,50 |
| | **Sous-total rechanges** | | **10,50** |

Les capteurs à fourche (lot de 5) et la deuxième barrière infrarouge servent déjà de rechange.

## 5. Récapitulatif des coûts

| Payé par | Indispensable (€) | Options (€) | Rechanges (€) | Maximum (€) |
|----------|-------------------|-------------|---------------|-------------|
| CESI (consommables) | 64,00 | 8,00 (K5, si validé) | — | 72,00 |
| Équipe (AliExpress) | 110,50 | 9,50 | 10,50 | 130,50 |
| **Total** | **174,50** | **17,50** | **10,50** | **202,50** |

Pour l'équipe, la partie indispensable représente environ 18 à 28 € par personne pour une équipe de 4 à 6. L'an dernier, le matériel avait coûté environ 106 €.

## 6. Bilan de puissance (estimation, à mesurer)

| Consommateur | Tension | Courant estimé |
|--------------|---------|----------------|
| NEMA 17 via TMC2209 (courant réglé à ≈ 1 A) | 12 V | ≈ 0,5 à 1 A |
| Bande LED COB, 50 cm (5 à 10 W par mètre selon le modèle) | 5 V | ≈ 0,5 à 1 A |
| Servo MG90S (pic au démarrage ou en blocage) | 5 V | jusqu'à ≈ 0,7 A |
| Capteurs (fourche, barrière) | 3,3 V (fourni par l'ESP32) | < 50 mA |
| ESP32 | 5 V (USB du PC) | ≈ 0,1 à 0,25 A |

Côté 5 V : au pire ≈ 1,7 A, soit ≈ 10 W, ce qui fait ≈ 0,85 A tiré sur le 12 V avec un rendement de 85 % pour le convertisseur. Au total, ≈ 1,8 A au pire sur le 12 V : le bloc de 3 A garde environ 40 % de marge.

## 7. Précautions à la commande et au montage

**À la commande**
1. **Caméra** : prendre la version **avec filtre IR** (« IR-cut », « day »). Les versions « no IR filter » ou « night vision » donnent des couleurs fausses, ce qui rend la lecture des bagues impossible.
2. **Bloc secteur** : prise européenne et marquage CE (sécurité électrique, voir `02` §7). Les produits sans marquage sont fréquents sur AliExpress.
3. **Résistances (CESI)** : vérifier que la décade de démonstration est complète, par exemple 1 k ; 1,2 k ; 1,5 k ; 1,8 k ; 2,2 k ; 2,7 k ; 3,3 k ; 3,9 k ; 4,7 k ; 5,6 k ; 6,8 k ; 8,2 k.
4. **Filament (CESI)** : confirmer le diamètre selon l'imprimante du FabLab avant que la commande parte.
5. **Délais** : compter 1 à 3 semaines pour AliExpress. Commander en premier la caméra et l'ESP32, et commencer les essais de vision avec une webcam ou un téléphone en attendant.
6. **Frais** : la TVA est payée sur AliExpress. Vérifier le montant final affiché au paiement (frais éventuels sur les petits colis venant de hors UE). Grouper les achats en une ou deux commandes.

**Au montage**
1. Les GPIO de l'ESP32 ne supportent pas le 5 V : **alimenter les capteurs en 3,3 V**.
2. Régler la sortie du XL4015 à **5,0 V au multimètre avant** de brancher le servo et les LED.
3. Régler le courant du TMC2209 avant de brancher le moteur. **Ne jamais débrancher le moteur sous tension** : cela détruit le driver.
4. L'ESP32 est alimenté **uniquement par l'USB du PC** (ne pas relier en même temps le 5 V du convertisseur à sa broche 5 V). Il reste allumé quand on appuie sur l'arrêt d'urgence et peut signaler l'arrêt au PC. Toutes les masses sont reliées.
5. Le plateau de Ø ≈ 200 mm tient de justesse sur une imprimante de 220 × 220 mm. Sinon, l'imprimer en deux moitiés.
6. Le roulement porte le poids, le moteur entraîne : prévoir dans le moyeu imprimé un peu de souplesse pour qu'un léger désalignement entre l'axe du moteur et le roulement ne force pas.

## 8. Ce qu'on n'achète pas

| Élément | Source |
|---------|--------|
| PC de traitement d'image (Python + OpenCV) | Ordinateur portable d'un membre de l'équipe |
| Imprimante 3D | FabLab du CESI |
| Fer à souder, multimètre, outillage | FabLab ou membres de l'équipe ❓ |
| Ce qu'un membre possède déjà (ESP32, plaque d'essai, câbles, alimentation 12 V…) | À recenser avant de commander ❓ |
