# Choix et justification des composants

> Sections du rapport : partie 6 « Présentation des composants » (paragraphes **Principe**) et partie 7 « Choix et justification des composants » (tout le reste). Les choix de *type* de solution (pas à pas ou servo, plateau ou tube…) sont dans la partie 4, documents `03`, `05` et `06`.
> Les caractéristiques viennent des fiches techniques et des pages citées [n]. Pour plusieurs composants bon marché, les valeurs **varient selon le vendeur pour une même référence** : relever celles du composant reçu. Les prix sont les estimations de `07`. Les calculs sont des ordres de grandeur, **à confirmer par la mesure**.
> Les références entre parenthèses (C2, V1…) renvoient à la liste d'achat `07-liste-composants.md`.

## Méthode

Pour chaque composant : **principe** de fonctionnement, **besoin** chiffré, **candidats** comparés (2 à 4 références réelles), **choix** justifié, et **dimensionnement** quand il faut prouver que le composant convient.

Ne sont pas comparés : vis, câbles, plaques d'essai, connecteurs. Ils n'impliquent pas de choix technique.

## Synthèse

| # | Composant | Retenu | Écartés | Raison principale |
|---|-----------|--------|---------|-------------------|
| 1 | Carte de commande | ESP32 DevKit V1 (WROOM-32) | ESP32-S3, ESP32-C3 | Assez de broches, meilleur support de la bibliothèque du moteur, documentation la plus abondante |
| 2 | Moteur du plateau | NEMA 17 17HS4401 | 17HS8401, 17HS4023, 28BYJ-48 | Couple ≈ 3 fois le besoin calculé, sans surpoids |
| 3 | Driver du moteur | TMC2209 | A4988, DRV8825, TMC2208 | Silencieux, réduit son courant à l'arrêt, détecte un blocage |
| 4 | Servo de la trappe | MG90S | SG90, MG996R | Pignons métal pour le même encombrement ; le couple n'est pas le critère |
| 5 | Caméra | Module ELP à capteur OV2710 + objectif M12 de 8 ou 12 mm | Logitech C920, microscope USB | Mise au point fixe et objectif adapté au champ voulu |
| 6 | Référence du plateau | Fourche optique (ITR9608 + LM393) | Effet Hall A3144, fin de course | Sans contact, basculement net, alimentable en 3,3 V |
| 7 | Contrôle de chute | Barrière infrarouge 5 mm | TCRT5000, laser | Portée suffisante, réponse < 2 ms, sans danger |
| 8 | Éclairage | Bande LED COB IRC ≥ 90 | Bande LED à points, anneau WS2812 | Lumière continue, sans points ni scintillement |
| 9 | Commutation des LED | Module D4184 (AOD4184A) | IRF520, relais | Conduit avec les 3,3 V de l'ESP32 |
| 10 | Conversion 12 V → 5 V | XL4015 | 7805, LM2596, MP1584 | Rendement, et marge en courant |
| 11 | Alimentation secteur | Bloc 12 V 3 A marqué CE | 12 V 2 A, alimentation nue, alimentation de laboratoire | 40 % de marge, aucune borne 230 V accessible |
| 12 | Guidage du plateau | Roulement de plateau tournant | Roulement 608 + axe, arbre du moteur seul | Porte le poids sur un grand diamètre |
| 13 | Filament | PLA | PETG, ABS | Le plus simple à imprimer ; seul le support du moteur risque de chauffer |

---

## 1. Carte de commande (C1)

**Principe.** Un microcontrôleur exécute en boucle un programme stocké dans sa mémoire flash. Ses broches d'entrée-sortie (GPIO) lisent les capteurs et commandent les actionneurs. Des périphériques matériels intégrés génèrent les signaux sans occuper le processeur : liaison série (UART), PWM pour le servo, générateurs d'impulsions (MCPWM, RMT) pour le moteur pas à pas.

**Besoin.**
- Une liaison série avec le PC par USB (`06` §2).
- Environ 8 à 12 GPIO : STEP, DIR et EN du driver, liaison UART du driver (option), servo, fourche optique, barrière infrarouge, commande des LED, surveillance de l'arrêt d'urgence et voyant (options).
- Un générateur d'impulsions pour le moteur, compatible avec la bibliothèque FastAccelStepper (`06` §4.1).

**Candidats.**

| | ESP32 DevKit V1 (ESP32-WROOM-32) | ESP32-S3-DevKitC-1 | Carte ESP32-C3 |
|---|---|---|---|
| Processeur | 2 cœurs Xtensa LX6, 240 MHz [1] | 2 cœurs Xtensa LX7, 240 MHz [2] | 1 cœur RISC-V, 160 MHz [3] |
| SRAM | 520 Ko [1] | 512 Ko [2] | 400 Ko [3] |
| GPIO | 34, dont 6 en entrée seule [1] | 45 [2] | 22 [3] |
| USB | Par la puce USB-série de la carte | USB natif (OTG) [2] | USB série / JTAG intégré, sans OTG [3] |
| Générateurs d'impulsions utilisables par FastAccelStepper | 6 MCPWM/PCNT + 8 RMT [4] | 4 MCPWM/PCNT + 4 RMT ; le pilote MCPWM dépend de la version d'ESP-IDF [4] | 2 RMT seulement [4] |
| Bluetooth | Classique + BLE 4.2 [1] | BLE 5 [3] | BLE 5 [3] |
| Prix estimé | ≈ 4 € | ≈ 7 € | ≈ 3 € |
| Verdict | **Retenu** | Surdimensionné pour le besoin | Fonctionnerait, avec moins de marge |

**Choix : ESP32 DevKit V1.** Les trois cartes conviennent. La classique offre largement assez de broches et le plus de générateurs d'impulsions. C'est aussi la plus documentée, ce qui accélère la mise au point. L'USB natif de l'ESP32-S3 n'apporte rien ici : la puce USB-série de la carte suffit pour dialoguer avec le PC. L'ESP32-C3 n'a qu'un cœur et deux générateurs RMT, ce qui laisse peu de marge.

**Limites à citer dans le rapport.**
- Espressif marque certaines variantes de la puce ESP32 « non recommandées pour de nouvelles conceptions » [1]. C'est sans conséquence pour un prototype ; pour un produit, l'ESP32-S3 serait le choix naturel.
- FastAccelStepper signale des incompatibilités avec certaines versions du noyau Arduino-ESP32 (`06` §4.1) : figer la version utilisée.

---

## 2. Moteur du plateau (C2)

**Principe.** Un moteur pas à pas hybride a un rotor aimanté et denté, et deux bobines (phases). Le driver alimente les bobines dans un ordre précis : à chaque changement, le rotor avance d'un pas de 1,8°, soit 200 pas par tour. En dosant le courant dans les deux bobines (micro-pas), on obtient des positions intermédiaires. Le moteur fonctionne en boucle ouverte : s'il doit fournir plus que son couple, il saute des pas sans le savoir, d'où le capteur de référence (§6).

**Besoin.**
- Rotation sur 360°, par le chemin le plus court (un demi-tour au maximum).
- Position répétable bien en dessous de l'écart entre deux bacs (27,7° pour 13 bacs).
- Demi-tour en 0,5 s dans le pire cas (objectif de cadence, `06` §10).

**Candidats.**

| | 17HS4401 | 17HS8401 | 17HS4023 (moteur « galette ») | 28BYJ-48 (pour mémoire) |
|---|---|---|---|---|
| Longueur du corps | 40 mm [5] | 48 mm [6] | 23 mm [7] | — |
| Couple de maintien | 40 N·cm [5] | 52 N·cm [6] | 13 à 14 N·cm [7] | Faible, avec réducteur [8] |
| Courant par phase | 1,7 A [5] | 1,7 A [6] | 0,7 A [7] | Unipolaire 5 V |
| Résistance, inductance | 1,5 Ω ; 2,8 mH [5] | 1,8 Ω ; 3,2 mH [6] | ≈ 4 Ω [7] | — |
| Prix estimé | ≈ 9 € | ≈ 11 € | ≈ 8 € | ≈ 3 € |
| Verdict | **Retenu** | Surdimensionné, plus lourd | Marge insuffisante (voir calcul) | Trop lent (`06` §4) |

**Dimensionnement.** Hypothèses : plateau en PLA de Ø 200 mm et ≈ 50 g, 13 bacs de ≈ 10 g contenant chacun ≈ 50 résistances de ≈ 0,3 g, soit ≈ 320 g à ≈ 80 mm de l'axe. Ces masses sont à peser.

| Grandeur | Calcul | Résultat |
|----------|--------|----------|
| Inertie du plateau (disque) | ½ × 0,050 kg × (0,100 m)² | 2,5 × 10⁻⁴ kg·m² |
| Inertie des bacs pleins (masse à 80 mm) | 0,320 kg × (0,080 m)² | 2,05 × 10⁻³ kg·m² |
| Inertie totale J | (le rotor du moteur est négligeable) | ≈ 2,3 × 10⁻³ kg·m² |
| Accélération angulaire α | Demi-tour θ = π rad en t = 0,5 s, moitié accélération, moitié freinage : α = θ / (t/2)² | ≈ 50 rad/s² |
| Couple d'accélération | C = J × α | ≈ 0,12 N·m = 12 N·cm |
| Couple avec frottements | + ≈ 2 N·cm (roulement, estimation) | **≈ 14 N·cm** |
| Vitesse maximale | α × t/2 | ≈ 12,6 rad/s ≈ 120 tr/min |
| Fréquence de pas | 2 tr/s × 3 200 micro-pas par tour (1/16) | 6 400 pas/s, très en dessous des 200 000 pas/s de FastAccelStepper [4] |
| Résolution | 360° / 3 200 | 0,11° par micro-pas, soit ≈ 250 micro-pas entre deux bacs |

**Choix : 17HS4401.** Son couple de maintien (40 N·cm) vaut près de 3 fois le besoin. Même avec le courant réduit à ≈ 1 A pour limiter l'échauffement, on garde ≈ 23 N·cm en première approximation (couple proportionnel au courant), donc une marge d'environ 1,7. Le couple baisse avec la vitesse : vérifier la marge à 120 tr/min sur la courbe couple-vitesse du vendeur, ou par un essai. Le 17HS4023 n'a presque aucune marge au pire cas, et le 17HS8401 ajoute du poids et du coût sans nécessité.

---

## 3. Driver du moteur (C3)

**Principe.** Le driver contient un pont en H par bobine. Il régule le courant dans chaque bobine par découpage (hacheur) et fait avancer le moteur d'un pas, ou d'un micro-pas, à chaque impulsion reçue sur l'entrée STEP. L'entrée DIR fixe le sens de rotation.

**Besoin.** Moteur de 1,7 A par phase sous 12 V, logique en 3,3 V, positionnement fin, bruit faible pour une démonstration en salle.

**Candidats.**

| | A4988 | DRV8825 | TMC2208 | TMC2209 |
|---|---|---|---|---|
| Tension du moteur | 8 à 35 V [9] | 8,2 à 45 V [10] | 4,75 à 36 V [11] | 4,75 à 29 V [12] |
| Courant | ± 2 A, avec dissipateur [9] | 2,5 A crête, avec dissipateur, à 24 V [10] | ≈ 1,4 A RMS, 2 A crête [11] | 2 A RMS, 2,8 A crête [12] |
| Micro-pas | Jusqu'à 1/16 [9] | Jusqu'à 1/32 [10] | Réglage par broches, interpolation à 1/256 [11] | Réglage par broches, interpolation à 1/256 [12] |
| Bruit | Sifflement audible | Sifflement audible | Silencieux (StealthChop2) [11] | Silencieux (StealthChop2) [12] |
| Réduction du courant à l'arrêt | Non | Non | Oui, automatique [11] | Oui, automatique [12] |
| Détection de blocage | Non | Non | Non | Oui (StallGuard4) [12] |
| Configuration avancée | Non | Non | UART [11] | UART [12] |
| Prix estimé du module | ≈ 1 € | ≈ 1,5 € | ≈ 3 € | ≈ 4 € |
| Verdict | Écarté | Écarté | Juste en courant | **Retenu** |

**Choix : TMC2209.** Il couvre les 1,7 A du moteur avec de la marge, contrairement au TMC2208. Ses deux avantages décisifs ici :
- La **réduction automatique du courant à l'arrêt**. Le plateau reste immobile la plupart du temps ; le moteur chauffe moins, ce qui protège son support en PLA (§13).
- La **détection de blocage**. Elle peut signaler un plateau coincé (`ERR JAM`, `06` §9).

Le surcoût par rapport à un A4988 est d'environ 3 €.

**Réglage.** Le courant se règle par le potentiomètre du module (tension de référence) ou par UART. La formule dépend du module : suivre la documentation du fabricant [12].

---

## 4. Servo de la trappe (C5)

**Principe.** Un servomoteur de modélisme contient un petit moteur à courant continu, un réducteur, un potentiomètre qui mesure l'angle de sortie et une électronique d'asservissement. La position est commandée par la largeur d'une impulsion répétée toutes les 20 ms (environ 1 à 2 ms selon l'angle).

**Besoin.** Ouvrir puis refermer un volet léger en moins de 0,3 s, des milliers de fois, sans casser si la trappe se bloque.

**Candidats.**

| | SG90 | MG90S | MG996R |
|---|---|---|---|
| Couple | ≈ 1,8 kgf·cm à 4,8 V [13] | 1,8 kgf·cm à 4,8 V ; 2,2 kgf·cm à 6 V [14] | 9,4 kgf·cm à 4,8 V ; 11 kgf·cm à 6 V [15] |
| Pignons | Plastique | Métal [14] | Métal [15] |
| Masse | ≈ 9 g | 13,4 g [14] | 55 g [15] |
| Vitesse | — | 0,10 s pour 60° à 4,8 V [14] | — |
| Courant de blocage | — | — | 2,5 A à 6 V [15] |
| Prix estimé | ≈ 1,5 € | ≈ 2,5 € | ≈ 5 € |
| Verdict | Fragile si la trappe se bloque | **Retenu** | Surdimensionné, trop gourmand |

**Dimensionnement.** Volet en PLA d'environ 40 × 15 × 2 mm (≈ 1,5 g) plus une résistance (≈ 0,3 g), avec un bras de levier de 2 cm : couple ≈ 0,002 kg × 2 cm ≈ 0,004 kgf·cm. Le couple de n'importe quel candidat est donc plus de 400 fois suffisant : **le couple n'est pas le critère**. Une ouverture de 90° prend ≈ 0,15 s avec le MG90S, ce qui tient dans le budget de temps de `06` §10.

**Choix : MG90S.** Il a le même encombrement que le SG90, mais des pignons en métal qui résistent mieux si une patte de résistance coince le volet. Le MG996R tirerait jusqu'à 2,5 A en blocage, ce qui surchargerait le 5 V (`07` §6).

---

## 5. Caméra et objectif (V1, V4)

**Principe.** L'objectif forme l'image de la pièce sur un capteur CMOS, une matrice de photosites couverte d'un filtre coloré (rouge, vert, bleu). Un **filtre IR-cut** bloque l'infrarouge, que le capteur voit mais pas l'œil, et qui fausserait les couleurs. La caméra respecte la norme **UVC** (classe vidéo USB standard) : elle fonctionne sans pilote et OpenCV peut régler l'exposition et la balance des blancs.

**Besoin.**
- Un champ d'environ 40 mm de large : le corps de la résistance (≈ 6 mm), une partie des pattes et une petite mire de couleurs.
- Au moins 10 pixels par bague (bagues d'environ 0,5 à 1 mm, à mesurer).
- Réglages bloqués (mise au point, exposition, balance des blancs) pour des images reproductibles (`05` §2.3).
- Assez de distance entre la caméra et la pièce pour loger le dôme d'éclairage.

**Candidats.**

| | Module ELP USBFHD01M (capteur OV2710) | Webcam Logitech C920 | Microscope USB générique |
|---|---|---|---|
| Capteur | OV2710 1/2,7", 1 920 × 1 080, pixels de 3 µm [16] | 1080p, capteur de 3 Mpx [18] | Caractéristiques rarement documentées |
| Cadence | 30 i/s en 1080p, 60 i/s en 720p, 100 à 120 i/s en 640 × 480 selon les versions [17] | 30 i/s en 1080p [18] | — |
| Mise au point | Manuelle, objectif M12 vissé et interchangeable ; version à zoom 2,8–12 mm disponible [17] | Autofocus [18], à bloquer par logiciel | Molette manuelle |
| Champ | Selon l'objectif choisi | 78° en diagonale, fixe [18] | Très petit (macro) |
| Lumière intégrée | Non (éviter les versions à LED infrarouges) | Non | Anneau de LED blanches |
| Format | Carte de 38 × 38 mm [17] | Boîtier avec pince | Tube sur pied |
| Prix estimé | ≈ 30 € + objectif | ≈ 50 à 70 € | ≈ 20 à 30 € |
| Verdict | **Retenu** | Champ trop large (voir calcul) | Reflets de l'anneau de LED, qualité non maîtrisée |

**Dimensionnement de l'objectif.** Largeur active du capteur w = 1 920 × 3 µm = 5,76 mm. Pour un champ W = 40 mm, la distance objet d vaut environ d = f × W / w (approximation valable quand d est grand devant f).

| Focale f | Distance caméra-pièce d | Commentaire |
|----------|-------------------------|-------------|
| 2,1 mm (objectif souvent fourni) | ≈ 15 mm | Impossible de loger l'éclairage |
| 3,6 mm | ≈ 25 mm | Trop près, image déformée sur les bords |
| **8 mm** | **≈ 56 mm** | Convient |
| **12 mm** | **≈ 83 mm** | Convient, moins de déformation |

Résolution : 1 920 pixels pour 40 mm donnent 48 pixels par mm, soit 24 à 48 pixels par bague. Même en 640 × 480, on garde 16 pixels par mm, donc au moins 8 pixels par bague, ce qui reste proche du seuil.

Pour la Logitech C920, un champ diagonal de 78° donne ≈ 1,41 × d de largeur en 16/9 : pour 40 mm, il faudrait la placer à ≈ 28 mm de la pièce, ce qui gêne l'éclairage et sort probablement de sa plage de mise au point (à vérifier).

**Choix : module ELP à capteur OV2710 avec un objectif de 8 ou 12 mm.** Le lot d'objectifs (V4) devient indispensable, ou bien prendre directement la version à zoom 2,8–12 mm [17]. Points à vérifier à la commande : version **avec filtre IR-cut**, sans LED infrarouges. Pour l'analyse des couleurs, tester aussi le format non compressé YUY2 : la compression MJPEG peut dégrader les couleurs fines.

---

## 6. Capteur de référence du plateau (C6)

**Principe.** Une **fourche optique** est une LED infrarouge placée face à un phototransistor, de part et d'autre d'une fente. Un drapeau opaque fixé sur le plateau coupe le faisceau ; un comparateur (LM393) transforme le signal en sortie numérique franche.

**Besoin.** Retrouver la même position angulaire à chaque recalage (`06` §4.2), sans contact, avec un signal lisible par une broche 3,3 V.

**Candidats.**

| | Fourche optique ITR9608 + module LM393 | Capteur à effet Hall A3144 + aimant | Fin de course (micro-interrupteur) |
|---|---|---|---|
| Principe | Un drapeau coupe le faisceau dans une fente de 5 mm [19] | L'aimant fait basculer la sortie au-dessus d'un seuil de champ | Le plateau appuie sur un levier |
| Contact | Non | Non | Oui : frottement, usure, force sur le plateau |
| Alimentation | Module annoncé en 3,3 à 5 V (à vérifier) | 4,5 à 24 V [20] : pas en 3,3 V | Aucune |
| Répétabilité | Basculement net, fixé par l'arête du drapeau | Seuils variables selon le fabricant (60 à 100 G pour basculer, 40 à 80 G pour revenir [20]) et hystérésis : position de basculement moins précise | Dépend du jeu mécanique et des rebonds du contact |
| Prix estimé | ≈ 0,4 € | ≈ 0,5 € + aimants | ≈ 0,3 € |
| Verdict | **Retenu** | Écarté | Écarté |

**Choix : fourche optique.** Elle est sans contact et son point de basculement dépend seulement de la position de l'arête du drapeau. Le drapeau, imprimé sur le bord du plateau, doit faire 1 à 2 mm d'épaisseur pour passer dans la fente de 5 mm. Pour une meilleure précision, l'ESP32 peut s'arrêter sur le front du signal en approchant toujours dans le même sens.

---

## 7. Contrôle de chute (C7)

**Principe.** Une **barrière infrarouge** est une LED émettrice face à un récepteur. Quand une pièce coupe le faisceau, la sortie du récepteur change d'état. La sortie est à collecteur ouvert : une résistance de rappel vers 3,3 V (interne à l'ESP32) suffit.

**Besoin.** Détecter à coup sûr le passage d'une résistance qui tombe de la trappe vers le bac (`06` §6).

**Candidats.**

| | Barrière infrarouge 5 mm (modèle de référence : Adafruit 2168) | Capteur réflectif TCRT5000 | Module laser + récepteur |
|---|---|---|---|
| Principe | Émetteur et récepteur face à face | Émetteur et récepteur côte à côte, détecte la lumière renvoyée | Faisceau laser vers un récepteur |
| Portée | ≈ 50 cm [21] | 0,2 à 15 mm, optimum à 2,5 mm [22] | Plusieurs mètres |
| Temps de réponse | < 2 ms [21] | — | — |
| Alimentation et sortie | 3,3 à 5,5 V, collecteur ouvert [21] | Phototransistor | — |
| Limite | Faisceau à aligner sur le corps de la résistance | La pièce doit passer à quelques mm du capteur : incompatible avec une chute libre | Risque pour les yeux, puissance rarement garantie sur les modules bon marché |
| Prix estimé | ≈ 1,5 € la paire | ≈ 1 € | ≈ 2 € |
| Verdict | **Retenu** | Écarté | Écarté (sécurité) |

**Dimensionnement.** Une pièce lâchée 3 cm au-dessus du faisceau le traverse à v = √(2 g h) = √(2 × 9,81 × 0,03) ≈ 0,77 m/s. Le corps (Ø ≈ 2,4 mm) et un faisceau d'environ 3 mm donnent une coupure d'environ 5,4 mm / 0,77 m/s ≈ **7 ms**. Le récepteur répond en moins de 2 ms [21] : la détection est possible, mais l'ESP32 doit la capter par **interruption**, pas en lisant la broche dans une boucle ralentie par des attentes. Une patte seule (Ø ≈ 0,6 mm) ne coupe pas assez le faisceau : il faut viser le corps, dont la position est connue grâce à la gouttière en V.

**Choix : barrière infrarouge 5 mm.** Les paires vendues sur AliExpress reprennent le format du modèle Adafruit ; vérifier la tension et le type de sortie du modèle reçu.

---

## 8. Éclairage (V2)

**Principe.** Une **bande LED COB** (« chip on board ») porte des puces LED serrées en ligne continue, couvertes d'une couche de phosphore qui produit une lumière blanche à spectre large. L'**indice de rendu des couleurs** (IRC, 100 = lumière de référence) mesure la fidélité des couleurs éclairées : au-dessus de 90, rouge, orange et marron restent distincts.

**Besoin.** Lumière blanche à fort rendu des couleurs (IRC ≥ 90, 5 000 à 6 500 K), uniforme sous le dôme, **sans scintillement** (pas de PWM visible par l'obturateur déroulant, `05` §2.1).

**Candidats.**

| | Bande COB IRC ≥ 90 | Bande LED à points (SMD 2835) IRC ≥ 90 | Anneau de LED adressables (WS2812) |
|---|---|---|---|
| Rendu des couleurs | IRC ≥ 90 disponible | IRC ≥ 90 disponible | Blanc obtenu en mélangeant rouge, vert et bleu : spectre à trois pics, rendu des couleurs médiocre |
| Uniformité | Ligne continue, sans points | Points visibles : il faut un diffuseur épais | Points |
| Scintillement | Aucun en courant continu | Aucun en courant continu | PWM interne non réglable, dont la fréquence n'est pas garantie : risque de bandes sur l'image [23] |
| Commande | Tout ou rien par un MOSFET (§9) | Idem | Une broche de données |
| Prix estimé | ≈ 5 € | ≈ 4 € | ≈ 3 à 5 € |
| Verdict | **Retenu** | Repli si la COB IRC ≥ 90 est introuvable | Écarté |

**Choix : bande COB IRC ≥ 90**, environ 50 cm autour de la base du dôme. Si elle n'existe qu'en 12 V, la prendre en 12 V : elle se branche alors directement sur le 12 V, avec le même MOSFET.

---

## 9. Commutation des LED (V3)

**Principe.** Un **MOSFET** à canal N est un interrupteur commandé en tension : au-delà de la tension de seuil V_GS(th) entre grille et source, le canal conduit, avec une résistance R_DS(on) d'autant plus faible que la tension de grille est élevée. Un MOSFET « logique » est pleinement passant avec quelques volts seulement.

**Besoin.** Allumer environ 1 A de LED avec une sortie de 3,3 V de l'ESP32, sans PWM.

**Candidats.**

| | Module D4184 (AOD4184A) | Module IRF520 | Module relais 5 V |
|---|---|---|---|
| Seuil V_GS(th) | 1,7 à 2,6 V [24] | 2 à 4 V [25] | Bobine à commander en 5 V |
| Résistance à l'état passant | < 9,5 mΩ à V_GS = 4,5 V [24] | ≤ 0,27 Ω, mais garantie seulement à V_GS = 10 V [25] | Contact mécanique |
| Avec 3,3 V de commande | Passant (valeur non garantie à 3,3 V) | Peu ou pas passant selon l'exemplaire | Pas directement |
| Remarque | — | Composant marqué obsolète chez des distributeurs [25] | Bruyant, lent, s'use |
| Prix estimé | ≈ 1 € | ≈ 1 € | ≈ 1 € |
| Verdict | **Retenu** | Écarté | Écarté |

**Dimensionnement.** La fiche du AOD4184A donne R_DS(on) à 4,5 V et 10 V, pas à 3,3 V. Même si elle était dix fois plus grande à 3,3 V (≈ 0,1 Ω), la puissance dissipée à 1 A vaudrait P = R × I² ≈ 0,1 W, ce qui est négligeable pour ce boîtier. **À vérifier** en mesurant la chute de tension entre drain et source, LED allumées.

**Choix : module D4184.** C'est le seul candidat garanti conducteur avec 3,3 V de commande. Le module IRF520, très répandu dans les kits, est un bon contre-exemple à citer : son seuil peut dépasser 3,3 V.

---

## 10. Conversion 12 V → 5 V (A3)

**Principe.** Un **convertisseur abaisseur à découpage** (« buck ») hache la tension d'entrée à haute fréquence ; une inductance et un condensateur lissent le résultat. Il ne dissipe qu'une petite partie de la puissance, contrairement à un **régulateur linéaire**, qui transforme en chaleur toute la différence de tension.

**Besoin.** 5 V pour le servo et les LED : jusqu'à ≈ 1,7 A au pire (`07` §6), à partir du 12 V.

**Candidats.**

| | Régulateur linéaire 7805 | Module LM2596 | Module XL4015 | Module MP1584 |
|---|---|---|---|---|
| Courant maximal | 1,5 A [26] | 3 A [27] | 5 A [28] | 3 A [29] |
| Tension d'entrée | — | 4,5 à 40 V [27] | 8 à 36 V [28] | 4,5 à 28 V [29] |
| Fréquence de découpage | — (linéaire) | 150 kHz [27] | 180 kHz [28] | Jusqu'à 1,5 MHz [29] |
| Rendement | 5/12 ≈ 42 % | — | Jusqu'à 96 % [28] | — |
| Taille du module | Petit, mais gros dissipateur nécessaire | Moyenne | Grande, avec dissipateur | Très petite |
| Prix estimé | ≈ 0,5 € | ≈ 1,5 € | ≈ 2,5 € | ≈ 1 € |
| Verdict | Écarté | Possible | **Retenu** | Petit module, peu de surface pour évacuer la chaleur |

**Dimensionnement.**
- 7805 : pertes = (12 − 5) V × 1,7 A ≈ **12 W**, au-delà de son courant maximal de 1,5 A [26]. Écarté.
- Convertisseur à découpage avec un rendement de 85 % (hypothèse) : puissance de sortie 5 V × 1,7 A = 8,5 W, puissance d'entrée ≈ 10 W, donc ≈ **1,5 W** de pertes.

**Choix : XL4015.** Il fonctionne à environ un tiers de son courant maximal, ce qui limite l'échauffement pour 1 € de plus que le LM2596. Régler la sortie à 5,0 V au multimètre avant tout branchement.

---

## 11. Alimentation secteur (A1)

**Principe.** Un bloc secteur à découpage convertit le 230 V alternatif en 12 V continu, isolé du secteur. Le marquage CE atteste la conformité aux directives européennes, notamment basse tension (voir `02` §7).

**Besoin.** ≈ 1,8 A au pire sur le 12 V (`07` §6), sans aucune borne 230 V accessible pour l'équipe ou le public.

| | Bloc 12 V 2 A | **Bloc 12 V 3 A** | Alimentation nue 12 V 5 A (boîtier grillagé) | Alimentation de laboratoire du FabLab |
|---|---|---|---|---|
| Charge au pire | 90 % : aucune marge | 60 % : ≈ 40 % de marge | 36 % | Selon le modèle |
| Sécurité | Bloc fermé | Bloc fermé | Bornes 230 V à câbler soi-même : risque électrique | Bonne |
| Remarque | — | — | — | Reste au FabLab : la machine n'est pas autonome |
| Prix estimé | ≈ 6 € | ≈ 8 € | ≈ 10 € | 0 € |
| Verdict | Écarté | **Retenu** | Écarté (sécurité) | Pour les essais seulement |

**Choix : bloc 12 V 3 A** à prise européenne et marqué CE. L'alimentation de laboratoire reste utile pendant les essais pour limiter le courant en cas d'erreur de câblage.

---

## 12. Guidage du plateau (M1, M2)

**Principe.** Un **roulement de plateau tournant** (« lazy susan ») est formé de deux plaques annulaires séparées par des billes. Il supporte une charge verticale répartie sur un grand diamètre et empêche le plateau de basculer.

**Besoin.** Porter ≈ 400 g de plateau et de bacs pleins sans voile (le plateau ne doit pas osciller verticalement sous la trappe), en laissant le moteur seulement entraîner.

| | **Roulement de plateau tournant Ø 100 à 150 mm** | Roulement à billes 608 (8 × 22 × 7 mm) + axe | Arbre du moteur seul |
|---|---|---|---|
| Charge portée par | Le roulement, sur un grand diamètre | Le roulement, mais sur 22 mm de diamètre : le plateau peut basculer, il en faut deux | Les roulements internes du moteur, qui ne sont pas prévus pour ça |
| Voile du plateau | Faible | Moyen | Élevé |
| Pièces supplémentaires | Aucune | Axe et logement à fabriquer | Aucune |
| Prix estimé | ≈ 4 € | ≈ 2 € les deux | 0 € |
| Verdict | **Retenu** | Écarté | Écarté |

**Choix : roulement de plateau tournant.** Le moteur entraîne le plateau par un moyeu à bride de 5 mm (M2). Laisser un peu de souplesse dans la liaison imprimée entre le moyeu et le plateau, pour qu'un léger désalignement entre le moteur et le roulement ne force pas.

---

## 13. Filament (K1, K2)

**Principe.** L'impression 3D par dépôt de fil fond un filament thermoplastique dans une buse chauffée et le dépose couche par couche.

**Besoin.** Pièces rigides et précises (plateau, bacs, boîte, dôme), faciles à imprimer au FabLab, pour une machine sans source de chaleur importante, sauf le moteur.

| | **PLA** | PETG | ABS |
|---|---|---|---|
| Température de buse | 215 °C puis 210 °C [30] | 230 °C puis 240 °C [31] | 230 à 255 °C [32] |
| Plateau chauffant | 60 °C [30] | 85 °C puis 90 °C [31] | Chaud, enceinte conseillée [32] |
| Tenue en température | Ramollit au-dessus de 60 °C [30] | Jusqu'à ≈ 80 °C [31] | Bonne [32] |
| Déformation à l'impression | Faible | Faible | Forte [32] |
| Fumées | Faibles | Faibles | Styrène : pièce ventilée [32] |
| Prix estimé | ≈ 20 €/kg | ≈ 20 à 25 €/kg | ≈ 20 à 25 €/kg |
| Verdict | **Retenu** | Pour le support du moteur si besoin | Écarté |

**Choix : PLA.** C'est le plus simple et le plus sûr à imprimer. Seul point de vigilance : un moteur pas à pas sous tension chauffe, même à l'arrêt, et son boîtier peut dépasser 60 °C, la température où le PLA ramollit. Parades : réduction automatique du courant à l'arrêt par le TMC2209 (§3), et **mesure de la température du moteur** après 30 minutes de fonctionnement. Si elle dépasse ≈ 50 °C, imprimer le support du moteur en PETG ou intercaler une entretoise métallique.

---

## Références

1. Espressif, *ESP32 Series Datasheet* : https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf
2. Espressif, *ESP32-S3 Series Datasheet* : https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf
3. Espressif, *ESP32-C3 Series Datasheet* : https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_en.pdf ; comparaison C3 / S3 : https://jlcpcb.com/blog/esp32-c3-vs-s3
4. FastAccelStepper, README (version 2.0.0) : https://components.espressif.com/components/gin66/fastaccelstepper/versions/2.0.0/readme
5. 17HS4401, fiche MotionKing (copie) : https://www.datasheetcafe.com/17hs4401-datasheet-stepper-motor/
6. 17HS8401B : https://www.bitsandparts.nl/en/stepper-motor-nema-17-17hs8401b-p1907185
7. 17HS4023 : https://www.bitsandparts.nl/en/stepper-motor-nema-17-17hs4023-i.a.-titan-extruders-p1910823 ; https://www.datasheetcafe.com/17hs4023-datasheet-stepper-motor/
8. 28BYJ-48 : https://docs.circuit.rocks/motors/28byj-48/
9. Allegro A4988, fiche technique (copie) : https://www.radiolocman.com/datasheet/data.html?di=687711
10. Texas Instruments DRV8825 : https://www.ti.com/product/DRV8825
11. Analog Devices (Trinamic) TMC2208 : https://www.analog.com/en/products/tmc2208.html
12. Analog Devices (Trinamic) TMC2209, fiche technique rév. 1.08 : https://www.analog.com/media/en/technical-documentation/data-sheets/TMC2209_datasheet_rev1.08.pdf ; module BIGTREETECH : https://bttwiki.com/TMC2209.html
13. SG90 : https://www.waveshare.com/SG90-Servo.htm
14. TowerPro MG90S : https://www.getfpv.com/towerpro-mg90s-digital-servo.html ; https://components101.com/motors/mg90s-metal-gear-servo-motor
15. TowerPro MG996R, fiche technique (copie) : https://people.ece.cornell.edu/land/courses/ece4760/FinalProjects/f2017/kmv48_rp625/kmv48_rp625/kmv48_rp625/MG996RTowerPro.pdf ; courant de blocage : https://thepihut.com/products/towerpro-servo-motor-mg996r-metal-gear-180
16. OmniVision OV2710 : https://www.ovt.com/products/ov2710/
17. ELP USBFHD01M : http://www.elpcctv.com/elp-full-hd-1080p-usb-camera-module-cmos-ov2710-sensor-2mp-with-no-distortion-lens-100-degree-p-626.html ; version 2,1 mm : https://www.svpro.cc/product/elp-high-speed-120fps-usb-camera-module-with-2-1mm-wide-angle-lens-full-hd-1080p-ov2710-embedded-webcam-module-elp-usbfhd01m-l21/ ; version à zoom 2,8–12 mm : https://www.svpro.cc/product/2-0megapixel-2-8-12mm-varifocal-lens-usb-camera-module-hd-cmos-ov2710-cctv-mini-usb-camera-board-for-android-linux-windows-mac-osusbfhd01m-fv/
18. Logitech C920 : https://www.logitech.com/en-gb/product/hd-pro-webcam-c920
19. Everlight ITR9608 : https://datasheet.lcsc.com/lcsc/Everlight-Elec-ITR9608_C52833.pdf
20. A3144, fiche technique (copie) : https://cdn.shopify.com/s/files/1/1560/1473/files/a3144eul.pdf ; seuils d'un distributeur : https://makerselectronics.com/product/3144-to-92s-hall-switch/
21. Adafruit, *IR Break Beam Sensor – 5mm LEDs* (2168) : https://www.adafruit.com/product/2168
22. Vishay TCRT5000 : https://www.vishay.com/en/product/83760/
23. ENTTEC, *Why do LEDs flicker on camera* : https://support.enttec.com/pixel/pixel-general-knowledge/why-do-leds-flicker-on-camera
24. Alpha & Omega Semiconductor AOD4184A : https://aosmd.com/res/datasheets/AOD4184A.pdf
25. Vishay IRF520 : https://www.vishay.com/docs/91017/irf520.pdf ; statut obsolète : https://www.digikey.it/en/products/detail/vishay-siliconix/IRF520/11566
26. STMicroelectronics L78 (7805) : https://www.st.com/resource/en/datasheet/l78.pdf
27. Texas Instruments LM2596 : https://www.ti.com/product/LM2596
28. XLSEMI XL4015, fiche technique (copie) : https://www.radiolocman.com/datasheet/data.html?di=685191
29. Monolithic Power Systems MP1584 : https://www.monolithicpower.com/pub/media/document/MP1584_r1.1.pdf
30. Prusa Research, PLA : https://help.prusa3d.com/article/pla_2062
31. Prusa Research, PETG : https://help.prusa3d.com/article/petg_2059
32. Prusa Research, ABS : https://help.prusa3d.com/article/abs_2058
