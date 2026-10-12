# État de l'art

> **Version 1, à enrichir.** Les références [1] à [8] viennent d'une première recherche web et sont à relire en entier avant d'être citées dans le rapport.
> Là où une source n'a été vue que partiellement, c'est indiqué. Format calqué sur le rapport de l'an dernier : solutions existantes, synthèse, législation, puis choix technologiques (voir `03-architecture-et-alternatives.md`).

## 1. Solutions existantes

### 1.1 Trieurs de résistances par vision (projets)
| Projet | Architecture | Remarques |
|--------|--------------|-----------|
| Trieur sur **Arduino UNO Q** (Arduino, février 2026) [2][5] | Caméra USB, OpenCV sans apprentissage : prétraitement, changement d'espace couleur, détection de contours, décodage de la séquence de couleurs | L'auteur signale que reflets et mélange de corps clairs et sombres compliquent l'analyse. Le traitement est assez rapide pour identifier partiellement la pièce avant qu'elle soit entièrement dans le champ |
| **Pi Resistor Sorter**, Cornell ECE 5725 (2017) [3] | Raspberry Pi + caméra Pi, OpenCV-Python, servos pour convoyeur et plateau de tri | Résistances à 4 bandes. Les auteurs soulignent que la vision donne aussi la tolérance, ce qu'une mesure électrique ne donne pas |
| Projets de lecture de résistance par application | Smartphone ou PC | Non étudiés en détail : à compléter |

### 1.2 Reconnaissance par apprentissage
- **Caoile (JETSUR, 2026)** [6] : CNN entraîné sur environ 1 200 images annotées de résistances à 4 et 5 bandes. Précision d'environ 92,3 % à l'entraînement et 90,1 % en validation en conditions contrôlées, mais environ 72 % en mauvais éclairage ou mal aligné. Propose une architecture hybride YOLO + CNN (localisation des bandes puis classification), présentée comme travail futur.
- **Jeux de données annotés publics** [7] : projets Roboflow Universe (détection de bandes de résistances, par exemple un jeu de 4 008 images annoté pour YOLOv11). Licence et qualité à vérifier avant usage.
- **Liu et al., JEIT (2020)** [8] : détection et localisation de résistances à anneaux de couleur sur carte, par réseau convolutionnel. Seules les métadonnées ont été vues : à lire.
- **HiRes (arXiv)** : méthode hiérarchique en cascade d'identification de valeur de résistance. Seule la liste de références a été vue : à lire en entier avant de la citer.

### 1.3 Architecture générale d'un trieur par vision
Un tutoriel de robot de tri par couleur découpe le système en sous-système de vision (Pi + OpenCV), sous-système de commande (Arduino) et diviseur mécanique, et recommande l'espace couleur HSV pour sa robustesse à l'éclairage [1]. À retrouver et citer précisément.

### 1.4 Trieurs industriels et testeurs de composants
Non encore documenté. À chercher : trieurs de composants CMS, testeurs LCR automatiques, équipements de contrôle en ligne. Objectif : comparer prix, cadence, principe (électrique ou optique).

## 2. Synthèse des technologies existantes

- Les projets existants combinent presque toujours **caméra à éclairage maîtrisé + OpenCV + microcontrôleur + actionneur simple** [2][3].
- L'apprentissage profond progresse mais exige des données, et la robustesse à l'éclairage reste un sujet [6].
- Aucun projet vu ne mesure électriquement pour contrôler la vision : une piste d'originalité et un moyen de valider les résultats (voir variante F dans `03`).
- Le cœur de la difficulté se trouve rarement dans l'algorithme seul : alimentation d'une pièce à la fois, éclairage et synchronisation avec le tri comptent autant.

## 3. Bases techniques

### 3.1 Code couleur et valeurs normalisées
- Le **marquage des résistances** (code couleur, valeur, tolérance) est défini par la norme **CEI / IEC 60062** (édition 2016, amendement 2019) [9]. L'édition 2016 a ajouté la couleur rose pour le multiplicateur 10⁻³.
- Les **séries de valeurs** (E3, E6, E12, E24, E48, E96, E192) sont définies dans **IEC 60063** (édition 2015), et non dans 60062 [9][10]. Les tolérances usuelles par série : E12 à 10 %, E24 à 5 %, E96 à 1 % [10].
- Les fabricants ont parfois leurs propres définitions : se référer aux fiches techniques.

### 3.2 Notions de traitement d'image à étudier
Espaces couleur (RGB, HSV, Lab), balance des blancs, segmentation, détection de contours, transformation de Hough, morphologie mathématique, calibrage colorimétrique, évaluation (matrice de confusion, précision et rappel).

### 3.3 Notions d'électronique et de commande
Pilotage de servomoteurs ou de moteurs pas à pas, communication série USB, alimentation et découplage, ADC de l'ESP32 si mesure électrique. Les bases de dimensionnement et de simulation (LTSpice) vues l'an dernier sont réutilisables.

## 4. Législation et normes applicables (à compléter)

Le volet alimentaire de l'an dernier ne s'applique plus. À examiner :
- **Sécurité des machines** : Directive 2006/42/CE (à vérifier pour un prototype pédagogique, que le rapport de l'an dernier citait).
- **Sécurité électrique / basse tension** et **compatibilité électromagnétique** : identifier les textes pertinents.
- **Sécurité des personnes** : pièces mobiles (servos, plateaux), éclairage LED.
- **Normes de composants** : IEC 60062 et IEC 60063 (voir 3.1).
- Protection des données : sans doute hors sujet (pas d'images de personnes).

## 5. Alternatives et choix technologiques

Voir `03-architecture-et-alternatives.md` : architecture, algorithme, alimentation, tri, matrices de décision.

## 6. À faire

- [ ] Lire en entier [6], [8] et HiRes ; compléter les données chiffrées
- [ ] Trouver ≥ 5 projets / articles supplémentaires (dont solutions industrielles)
- [ ] Vérifier les licences des jeux de données publics
- [ ] Récupérer les fiches techniques des caméras et ESP32 candidats
- [ ] Rédiger la partie normes et sécurité
- [ ] Mesurer nous-mêmes : mêmes images, algorithmes comparés (vérité terrain)

## Références (à normaliser au format du rapport)

1. Tutoriel de robot de tri par couleur (architecture vision / commande / mécanique, HSV) : résultat de recherche web, URL exacte à retrouver.
2. Arduino Blog, « Arduino UNO Q solves the classic resistor pile issue », 19/02/2026 : https://blog.arduino.cc/2026/02/19/arduino-uno-q-solves-the-classic-resistor-pile-issue/
3. Liu S., Tan R., « Pi Resistor Sorter », Cornell ECE 5725, automne 2017 : https://courses.ece.cornell.edu/ece5990/ECE5725_Fall2017_projects/Lab_sl2974_rt446_Sichen_Liu_Ruichen_Tan
4. Mesure de latence et de cadence en streaming de l'ESP32-CAM (QVGA 18 images/s, 142 ms ; VGA 10 images/s, 238 ms) : https://www.irejournals.com/formatedpaper/1718151.pdf
5. CircuitDigest, « Automate Resistor Identification and Sorting using Arduino UNO Q » : https://circuitdigest.com/news/automate-resistor-identification-and-sorting-using-arduino-uno-q
6. Caoile, « Automatic Resistor Value Detection Using Computer Vision and Machine Learning », JETSUR, 2026 : https://journals.usm.edu.ph/jetsur/index.php/online/article/view/46
7. Roboflow Universe, jeux de données « Band Detection » (ResistorAI, CC BY 4.0) et autres : https://universe.roboflow.com/resistorai/band-detection
8. Liu et al., « Method for Color-ring Resistor Detection and Localization in Printed Circuit Board Based on Convolutional Neural Network », Journal of Electronics & Information Technology, 2020 : https://jeit.ac.cn/en/article/doi/10.11999/JEIT190608
9. IEC 60062:2016, « Marking codes for resistors and capacitors » : https://webstore.iec.ch/publication/25395
10. IEC 60063:2015, « Preferred number series for resistors and capacitors » (voir aussi https://eepower.com/resistor-guide/resistor-standards-and-codes et https://www.allaboutcircuits.com/textbook/reference/chpt-2/resistor-color-codes/)
