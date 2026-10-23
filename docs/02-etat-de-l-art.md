# État de l'art — Trieur automatique de résistances

> Partie « État de l'art » du rapport, sur le plan du rapport de l'an dernier : solutions existantes, synthèse comparative, législation et normes. Les choix technologiques détaillés sont dans [`03-architecture-et-alternatives.md`](03-architecture-et-alternatives.md).
> Rédigé le 12/10/2026.

## 1. Introduction

Il n'existe pas aujourd'hui de trieur de résistances par code couleur accessible à un atelier ou un fablab : l'industrie trie par mesure électrique sur des machines coûteuses, et les solutions par vision restent des prototypes de makers ou d'étudiants. C'est l'espace que vise notre projet SPI.

**Problématique.** Dans un labo d'électronique, les résistances traversantes s'accumulent en vrac. Les trier à la main oblige à lire 4 à 6 anneaux de couleur, souvent minuscules et ternes : c'est lent et source d'erreurs (marron/rouge/orange, violet/gris se confondent facilement).

**Objectif du projet.** Concevoir un système qui identifie automatiquement la valeur d'une résistance traversante par caméra (lecture du code couleur) et la dépose dans le bac correspondant grâce à une partie mécanique.

**Périmètre de cet état de l'art.**

- le principe du code couleur et des séries normalisées ;
- les solutions existantes (industrielles, commerciales, DIY, académiques) ;
- les technologies d'identification de la valeur (mesure électrique vs vision) ;
- les technologies d'alimentation et de tri mécanique ;
- la synthèse et le positionnement de notre trieur.

## 2. Bases techniques : le code couleur

Le marquage des résistances traversantes est normalisé par la [CEI 60062:2016](https://scc-ccn.ca/standardsdb/standards/2026746) : 3 à 6 anneaux, couleurs noir (0) à blanc (9), plus or, argent et rose pour les multiplicateurs inférieurs à 1. C'est cette norme que notre logiciel de vision devra décoder.

| Nb d'anneaux | Composition | Tolérance typique | Usage |
|---|---|---|---|
| 3 | 2 chiffres + multiplicateur | ±20 % (pas d'anneau) | Ancien, rare |
| 4 | 2 chiffres + multiplicateur + tolérance | ±5 % (or), ±10 % (argent) | Le plus courant, série E12/E24 |
| 5 | 3 chiffres + multiplicateur + tolérance | ±1 % (marron), ±2 % (rouge) | Précision, série E96 |
| 6 | 5 anneaux + coefficient de température (ppm/K) | ±1 % ou mieux | Haute précision |

*Tableau 1 : composition du code couleur selon le nombre d'anneaux*

Sources : [EEPower](https://eepower.com/resistor-guide/resistor-standards-and-codes/resistor-color-code/), [TE Connectivity](https://te.com/en/products/passive-components/resistors/intersection/resistor-color-codes.html).

**Points utiles pour la lecture automatique :**

- **Sens de lecture.** La norme place le premier anneau au plus près d'une extrémité, et l'anneau de tolérance doit être 1,5 à 2 fois plus large que les autres ([DIN EN 60062](https://www.dinmedia.de/en/standard/din-en-60062/267498948)). L'écart entre multiplicateur et tolérance est aussi un peu plus grand. Ces indices permettent à l'algorithme de trouver le sens.
- **Couleurs voisines.** Marron/rouge/orange, violet/gris et or/jaune sont proches en teinte : c'est la principale source d'erreur, d'autant que la couleur du corps (beige, bleu) varie selon les fabricants.
- **Séries normalisées.** Les valeurs suivent les séries E12 (12 valeurs par décade), E24 et E96. Un décodage qui tombe hors série signale une erreur de lecture : c'est un contrôle de cohérence gratuit.
- **Hors périmètre.** Les CMS (composants montés en surface) sont marqués par chiffres, pas par couleurs : notre trieur ne vise que les résistances traversantes.

## 3. Solutions existantes

Aucun trieur de résistances en vrac n'est vendu au grand public : l'industrie trie par mesure électrique en fin de production, et tous les autres systèmes trouvés sont des prototypes universitaires ou de makers. Une équipe de l'université de l'Illinois le présente d'ailleurs comme un produit absent du marché ([UIC](https://engineeringexpo.uic.edu/news-stories/resistor-sorter)).

### 3.1 Industrie

Les fabricants mesurent chaque résistance sur un disque rotatif à poste de mesure, puis l'aiguillent vers un bac selon son écart à la valeur nominale. Le brevet [US 6750416](https://patents.google.com/patent/US6750416) décrit ce type de machine, avec mesure à température contrôlée. Ces lignes trient des lots d'une seule valeur par tolérance, pas un vrac de valeurs mélangées.

### 3.2 Prototypes

Deux familles coexistent : mesure électrique et vision par caméra.

| Projet | Origine | Identification | Mécanique de tri | Résultat publié |
|---|---|---|---|---|
| [Trieur Arduino UNO Q (Z. Hipps)](https://blog.arduino.cc/2026/02/19/arduino-uno-q-solves-the-classic-resistor-pile-issue/) | Maker, 2026 | Vision : microscope USB + OpenCV, sans IA | Aucune, tri manuel guidé | Lecture en temps réel |
| [Resistor Sorter (Queen's)](https://qspace.library.queensu.ca/items/8aadfc76-ea6d-4126-8359-9e7fd034a73f) | Université, 2023 | Vision : caméra + code couleur | Gravité + moteurs, 12 résistances redressées à la main | 60 % de lectures justes, 40 % de tris justes |
| [Automated Resistor Sorter (RIT)](https://www.rit.edu/imagine/exhibits/automated-resistor-sorter) | Université (capstone) | Non précisé | Bacs par valeur | Cahier des charges : 95 % de précision, moins de 30 s par résistance |
| [Resistor sorter (I. Adarsh)](https://www.hackster.io/news/low-cost-machine-sorts-resistors-at-a-high-speed-0f63e695dee6) | Maker | Mesure : pont diviseur, Arduino Mega | Entonnoir + servos, tri en 2 voies | 3 s par résistance |
| [Automated Resistor Sorter with GUI (Cornell)](https://people.ece.cornell.edu/land/courses/ece4760/FinalProjects/f2016/anp56_bwg38_nol5/anp56_bwg38_nol5/anp56_bwg38_nol5/index.html) | Université, 2016 | Mesure : source de courant, R = U/I (PIC32) | Pièces PLA imprimées 3D, servo, pas-à-pas, solénoïde | Tri vers bacs prédéfinis |
| [Pi Resistor Sorter (Cornell)](https://courses.ece.cornell.edu/ece5990/ECE5725_Fall2017_projects/Lab_sl2974_rt446_Sichen_Liu_Ruichen_Tan) | Université, 2017 | Vision : Raspberry Pi + OpenCV, 4 anneaux | Tri automatique | Fonctionne en partie |
| [Resistor Sorter (Trinity)](https://digitalcommons.trinity.edu/engine_designreports/90) | Université, 2013 | Mesure : ohmmètre « intelligent » | Redresseur de pattes + plateau rotatif à bacs | Tous les modules fonctionnels |
| [Resistor Sorter (Swarthmore)](https://www.swarthmore.edu/sites/default/files/assets/documents/engineering/dg_cs_FinalReport.pdf) | Université, 2004 | Mesure : pont diviseur, 1 Ω à 1 MΩ | Pyramide de tubes + solénoïdes (tri binaire) | 96 valeurs classées |
| [Resistance Is Futile](https://github.com/jvanburen/rif) | Hackathon | Vision : Raspberry Pi, classifieur bayésien en espace CIE Lab | Aucune, lecture seule | Boîtier à éclairage uniforme |

*Tableau 2 : prototypes de trieurs de résistances recensés*

### 3.3 Recherche

Les travaux portent sur la seule lecture d'image : un réseau LeNet-5 atteint 99,6 % sur 240 valeurs de 4 anneaux en conditions maîtrisées ([Puno et al., 2022](https://animorepository.dlsu.edu.ph/faculty_research/5749)), mais un CNN testé en temps réel par webcam chute d'environ 90 % à 72 % avec un mauvais éclairage ou un mauvais alignement ([JETSUR](https://journals.usm.edu.ph/jetsur/index.php/online/article/view/46)).

### 3.4 Machines voisines : les trieuses de vis et de petites pièces

Notre machine ressemble beaucoup plus à une **trieuse de vis** qu'à un testeur de résistances. Elle enchaîne les quatre mêmes fonctions : isoler une pièce dans un vrac, la présenter devant un capteur, l'identifier, puis l'envoyer vers le bon bac. La seule vraie différence est le critère de tri : une vis se trie par sa **forme et ses dimensions**, une résistance par la **couleur** de ses anneaux. Les trieuses de vis sont un domaine bien plus mûr. Il est donc utile de regarder comment elles résolvent les problèmes que nous allons rencontrer.

| Famille | Exemple | Alimentation | Identification | Tri | Cadence | Coût |
|---|---|---|---|---|---|---|
| Trieuse optique industrielle à plateau de verre | [Qualimatest QMTInspect-R](https://www.saphir.fr/pdf/Products/QMTInspect-R_Datasheet_F.pdf) | Automatique, sur un plateau de verre tournant | Jusqu'à 5 postes de 5 caméras, optiques télécentriques, mesures répétables au micron | Éjection pneumatique, jusqu'à 8 sorties (bonnes et mauvaises pièces) | Jusqu'à 36 000 pièces/h (10 par seconde) | À partir de 130 000 € (document d'environ 2012) |
| Trieuse optique industrielle à bol vibrant | Vester Sensorcheck VSC ([annonce d'occasion](https://www.used-machines.com/vester+sortierautomat-sensorcheck+vsc-10c/gm-212-22857)), machines AOI asiatiques ([Made-in-China](https://fr.made-in-china.com/category23_Manufacturing-Processing-Machinery/Optical-Sorting-Machine_109.html)) | Bol vibrant + séparation en file | Caméras CCD/CMOS, objectifs télécentriques | Éjection pneumatique bonne pièce / rebut | 100 à 250 pièces/min | Environ 10 000 à 85 000 $ (prix catalogue) ; bol vibrant seul : environ 500 à 10 000 $ |
| Trieuse mécanique par fentes | Modèles imprimés en 3D ([MakerWorld](https://makerworld.com/models/765163), [J. Alves](https://jpralves.net/post/2018/11/08/automatic-screw-sorter.html)) | Manuelle, la vis roule sur une rampe | Aucune : la vis tombe dans la première fente assez longue | Gravité, un bac par fente | Rapide à la main | Quelques euros de filament |
| Trieuse amateur par vision et bras | [Trieur de vis de « Aad » (Hackaday, 2026)](https://hackaday.com/2026/08/08/robotic-screw-and-bolt-sorter-seeks-a-new-challenge/), [trieur de pièces sur table lumineuse (Hackaday, 2019)](https://hackaday.com/2019/02/12/) | Convoyeur puis goulotte vers une table éclairée par-dessous ; pièces simplement étalées en 2019 | Caméra au-dessus (ESP32-CAM à objectif M12 en 2026, webcam + OpenCV en 2019) | Portique ou bras qui saisit la pièce et la dépose dans un bac | Lente (une pièce à la fois) | Imprimante 3D recyclée + cartes ESP32 + PC |
| Trieuse amateur sur plateau tournant | [Fab Academy Oulu (2026)](https://fabacademy.org/2026/labs/oulu/Machine_projects/Team1/) | Pièce posée à la main sur un plateau | Caméra ESP32-S3 | Plateau tourné par un moteur pas à pas, un servo pousse la pièce hors du plateau | Non publiée | Projet étudiant |
| Trieuse de LEGO par IA | [Universal LEGO Sorter, D. West (2019)](https://www.raspberrypi.com/news/raspberry-pi-lego-sorter/) | Bac + plaques vibrantes qui isolent les pièces | Pi Camera + réseau ResNet-50 sur un PC portable, 2 927 types de pièces | Tapis et volets vers 18 bacs | 1 pièce toutes les 2 s | Plus de 2 ans de développement |

*Tableau 3 : trieuses de vis et de petites pièces, machines voisines de notre trieur*

**Ce que les trieuses de vis nous apprennent :**

- **Isoler les pièces est le vrai problème mécanique.** Toutes les machines industrielles commencent par un bol vibrant, qui coûte à lui seul plus que tout notre budget. Les prototypes amateurs le contournent par une dépose à la main (Fab Academy) ou par un bras qui sépare les pièces collées (Aad). Un groupe d'étudiants de NTU note que les machines professionnelles coûtent jusqu'à 10 000 $ et pèsent plus de 600 kg ([NTU, 2022](https://blogs.ntu.edu.sg/ps5888-2022-g07/?p=6)).
- **L'éclairage est maîtrisé partout.** On trouve un plateau de verre éclairé par-dessous et des optiques télécentriques en industrie, une table lumineuse chez les amateurs. Les étudiants d'Oulu ont constaté que l'éclairage dégradait leur détection et recommandent un éclairage contrôlé.
- **Le plateau tournant indexé par moteur pas à pas est une solution classique.** Un brevet de trieur à bacs rotatifs note que le pas à pas sert à la fois de moteur et de repère de position ([US 6354590](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/6354590)). C'est exactement notre choix (décision D7).
- **Une vis se lit en ombre chinoise, une résistance non.** Une silhouette suffit pour mesurer une vis : un éclairage par-dessous et une caméra noir et blanc conviennent. Les anneaux d'une résistance ne se voient qu'en couleur, éclairés par-dessus, sur une pièce cylindrique et vernie qui fait des reflets. **Notre problème de vision est donc plus difficile** que celui d'une trieuse de vis, et l'éclairage de la boîte noire est notre point le plus critique.
- **Le nombre de catégories pèse sur la mécanique.** Une trieuse industrielle n'a que 2 à 8 sorties (bonne pièce ou rebut). Une trieuse de rangement comme la nôtre ou celle de West a besoin de 13 à 18 bacs. Elle doit donc soit déplacer les bacs (plateau tournant), soit multiplier les volets.

### 3.5 Ce qu'on en retient

Les projets par vision butent sur la fiabilité de lecture, ceux par mesure sur l'alimentation pièce à pièce. Aucun ne combine lecture fiable de la couleur et tri vers de nombreux bacs. Les trieuses de vis montrent comment organiser la machine (poste de lecture éclairé, aiguillage indexé, contrôle de sortie), mais leur problème de vision est plus simple que le nôtre.

## 4. Technologies d'identification de la valeur

La mesure électrique est plus simple et plus fiable, mais elle donne la valeur réelle, pas la valeur marquée ni la tolérance ; la vision lit le marquage complet, au prix d'un éclairage et d'un cadrage très maîtrisés.

### 4.1 Mesure électrique

On fait passer un courant connu ou on forme un pont diviseur avec une résistance de référence, puis un CAN lit la tension et on calcule R. Pour couvrir 1 Ω à 1 MΩ, on commute plusieurs résistances de référence selon la gamme (projets Swarthmore et [Resistorganizer](https://hackaday.com/blog/page/1441/?cat=PHL)). Il faut ensuite arrondir à la valeur normalisée E12/E24 la plus proche.

### 4.2 Vision par traitement d'image classique

C'est la chaîne utilisée par le projet Arduino UNO Q et la plupart des prototypes :

1. acquisition (webcam ou microscope USB) sous éclairage diffus ;
1. localisation de la résistance, redressement à l'horizontale ;
1. passage dans un espace couleur robuste à la luminosité (HSV ou CIE Lab) ;
1. segmentation des anneaux : soustraction de la couleur du corps, seuillage, contours ([brevet US 9959616](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/9959616)) ;
1. classification de chaque anneau par seuils ou classifieur entraîné ;
1. détermination du sens de lecture, décodage et contrôle par la série E.

Outils courants : OpenCV (Python ou C++), sur Raspberry Pi ou Arduino UNO Q.

### 4.3 Vision par apprentissage profond

Deux approches : classer l'image entière (CNN type LeNet-5) ou détecter d'abord chaque anneau (YOLO) puis classer sa couleur. La seconde est jugée plus robuste et plus facile à diagnostiquer ([JETSUR](https://journals.usm.edu.ph/jetsur/index.php/online/article/view/46)). Elle demande un jeu d'images annoté ; des jeux publics existent, comme [Band Detection](https://universe.roboflow.com/resistorai/band-detection) (944 images).

| Critère | Mesure électrique | Vision classique (OpenCV) | Vision deep learning |
|---|---|---|---|
| Information obtenue | Valeur réelle | Valeur marquée + tolérance | Valeur marquée + tolérance |
| Fiabilité | Élevée | Moyenne, sensible à l'éclairage | Élevée en labo, chute en conditions réelles |
| Contraintes mécaniques | Contact sur les deux pattes | Résistance immobile, bien cadrée | Idem |
| Matériel | CAN + résistances de référence | Caméra + éclairage diffus + calculateur | Idem + entraînement du modèle |
| Complexité logicielle | Faible | Moyenne | Élevée (jeu de données) |
| Résistances abîmées ou décolorées | Lues correctement | Erreurs | Erreurs |

*Tableau 4 : comparaison des méthodes d'identification de la valeur*

**Le verrou principal de la vision est la couleur perçue** : éclairage, reflets sur le vernis, couleur du corps et anneaux voisins faussent la teinte. Toutes les sources convergent vers un éclairage diffus fermé et un fond uniforme comme condition nécessaire.

## 5. Technologies de manutention et de tri

La partie la plus difficile n'est pas le tri final mais l'alimentation : isoler une seule résistance, pattes droites, depuis un vrac emmêlé. Presque tous les prototypes contournent le problème en imposant un chargement manuel pièce par pièce.

### 5.1 Alimentation (séparer une pièce à la fois)

| Solution | Principe | Utilisée par | Limite |
|---|---|---|---|
| Chargement manuel | L'utilisateur pose les résistances une à une, pattes redressées | Queen's (12 à la fois) | Pas d'automatisation du vrac |
| Entonnoir + volet servo | La pièce descend et est bloquée au poste de lecture | I. Adarsh | Se coince si les pattes sont tordues |
| Redresseur de pattes | Mouvement linéaire qui aligne les pattes sur l'axe | Trinity | Mécanique complexe |
| Bol vibrant | Vibration qui aligne les pièces en file | Industrie (composants vrac) | Coûteux, encombrant |
| Tapis convoyeur | Défilement continu devant la caméra | Queen's | Résistances à espacer avant |

*Tableau 5 : solutions d'alimentation pièce à pièce*

### 5.2 Actionneurs de tri (envoyer vers le bon bac)

| Solution | Nb de bacs | Utilisée par | Commentaire |
|---|---|---|---|
| Volet servo gauche/droite | 2 | I. Adarsh | Simple mais limité |
| Pyramide de tubes + solénoïdes | Jusqu'à 96 (tri binaire en cascade) | Swarthmore | Rapide, beaucoup d'actionneurs |
| Plateau rotatif à bacs | 8 à 13 environ | Trinity, Cornell 2016, notre trieur (13 bacs) | Un seul moteur pas-à-pas, compact |
| Chariot sur axe linéaire | Une rangée de bacs | UIC | Simple à piloter |
| Soufflage pneumatique | 2 à 8 sorties | Trieuses de vis industrielles (QMTInspect-R, Vester) | Très rapide, mais demande de l'air comprimé |
| Portique XY (type imprimante 3D) | Tiroirs d'un meuble de rangement | [Bradley University](https://vip.bradley.edu/resistor-sorter/) (NEMA 17) | Le plus flexible, le plus long à construire |

*Tableau 6 : actionneurs de tri*

### 5.3 Unité de commande

| Carte | Atout | Limite pour notre projet |
|---|---|---|
| Arduino Uno / Mega | Pilotage direct des servos et pas-à-pas, simple | Ne peut pas faire tourner OpenCV |
| Raspberry Pi | Linux + OpenCV + caméra officielle | Moins adaptée aux signaux temps réel (moteurs) |
| Arduino UNO Q | Microprocesseur Linux + microcontrôleur sur une même carte, déjà prouvé sur ce cas ([Arduino](https://blog.arduino.cc/2026/02/19/arduino-uno-q-solves-the-classic-resistor-pile-issue/)) | Carte récente, moins de documentation |
| ESP32 (+ ESP32-CAM) | Très bon marché, Wi-Fi | Puissance de calcul juste pour la vision |

*Tableau 7 : cartes de commande envisageables*

Une architecture courante sépare les rôles : un ordinateur embarqué (Raspberry Pi) pour la vision, un microcontrôleur (Arduino) pour les moteurs, reliés par liaison série. Le PIC32 de Cornell et nos cours sur PIC suivent la même logique côté commande.

## 6. Synthèse : comparaison avec les solutions existantes et positionnement

Notre trieur se place sur la voie la moins aboutie mais la plus riche : lire le code couleur à la caméra, comme un humain, puis trier automatiquement. Les prototypes existants montrent que c'est faisable, mais aucun n'atteint une fiabilité suffisante en conditions réelles.

### 6.1 Notre solution en bref

Rappel des choix déjà pris ou pressentis (voir `decisions.md`) :

- **Alimentation** : la résistance est déposée à la main dans une gouttière en V, une à la fois (F1), avec une trémie automatique en amélioration.
- **Lecture** : caméra USB à objectif M12 dans une boîte noire à dôme diffusant et bande LED. Un PC analyse l'image avec OpenCV et décode la valeur, contrôlée par la série E12.
- **Tri** : l'ESP32 reçoit le numéro de bac par liaison USB série. Il tourne un plateau de 13 bacs (12 valeurs E12 et un bac de rejet) avec un moteur NEMA 17 et un capteur de position. Il ouvre ensuite une trappe à servo, et une barrière infrarouge confirme la chute de la pièce.
- **Coût** : environ 170 € de matériel, options comprises jusqu'à environ 200 € (`07-liste-composants.md`).

### 6.2 Comparaison globale

| Critère | **Notre trieur** | Trieuse de vis industrielle | Trieuse de vis amateur par vision | Trieurs de résistances par vision | Trieurs de résistances par mesure |
|---|---|---|---|---|---|
| Exemples | — | QMTInspect-R, Vester VSC | Aad 2026, Fab Academy Oulu | Queen's, Arduino UNO Q, Cornell 2017 | Adarsh, Cornell 2016, Trinity, Swarthmore |
| Alimentation | Manuelle, une pièce à la fois | Bol vibrant, entièrement automatique | Convoyeur et bras, ou à la main | À la main (12 pièces redressées chez Queen's) | Entonnoir, redresseur de pattes |
| Ce qui est lu | Couleur des anneaux | Forme et dimensions | Forme et dimensions | Couleur des anneaux | Valeur électrique |
| Éclairage | Boîte noire, dôme diffusant | Par-dessous et télécentrique | Table lumineuse | Variable, souvent peu maîtrisé | Sans objet |
| Nombre de sorties | 13 bacs | 2 à 8 (bonne pièce ou rebut) | Plusieurs bacs | 0 à quelques bacs | 2 à 96 |
| Actionneur de tri | Plateau tournant pas à pas + trappe servo | Soufflage pneumatique | Portique et pince, ou poussoir servo | Moteurs, ou tri à la main | Servos, solénoïdes, plateau rotatif |
| Contrôle de la chute | Barrière infrarouge | Capteurs de sortie | Capteur piézo (Aad) | Non décrit | Non décrit |
| Erreur de lecture | Bac de rejet + contrôle E12 | Éjection en rebut | Non décrit | Non décrit | Arrondi à la valeur E la plus proche |
| Cadence | À mesurer (quelques secondes par pièce visées) | 100 à 600 pièces/min | Lente | Jusqu'à 30 s par pièce (cible RIT) | Environ 3 s par pièce (Adarsh) |
| Fiabilité publiée | À mesurer | Très élevée (contrôle à 100 %) | Non publiée | 60 % de tris justes (Queen's) | Élevée (mesure directe) |
| Coût | Environ 170 à 200 € | 10 000 à plus de 130 000 € | Quelques centaines d'euros | Quelques centaines d'euros | Quelques dizaines à centaines d'euros |

*Tableau 8 : comparaison de notre trieur avec les solutions existantes*

### 6.3 Comparaison fonction par fonction avec une trieuse de vis

| Fonction | Trieuse de vis | Notre trieur | Conséquence pour nous |
|---|---|---|---|
| Isoler une pièce | Bol vibrant et rail de séparation | Dépose à la main dans une gouttière | Nous renonçons au vrac automatique en v1 : c'est hors budget et c'est le point le plus coûteux d'une trieuse |
| Présenter la pièce | Plateau de verre tournant, pièce stable sur une face | Gouttière en V : la résistance est couchée et centrée | La forme cylindrique se prête bien à une gouttière. En revanche, il faut que les anneaux soient visibles, d'où un léger roulement possible à prévoir |
| Éclairer | Par-dessous : la silhouette suffit | Par-dessus, diffus, dans une boîte noire | C'est notre point le plus critique : les reflets et les couleurs voisines faussent la lecture |
| Identifier | Mesures géométriques en noir et blanc | Couleur de 4 anneaux, sens de lecture, décodage | Problème plus difficile, mais le contrôle par la série E12 détecte une partie des erreurs |
| Décider | Bonne pièce ou rebut | Une valeur parmi 12, ou rejet | Plus de catégories, donc plus de bacs |
| Aiguiller | Soufflage d'air comprimé, très rapide | Plateau tournant + trappe | Pas d'air comprimé dans un atelier étudiant ; le plateau donne 13 bacs avec un seul moteur |
| Vérifier | Capteurs de sortie | Barrière infrarouge sous la trappe | Équivalent, à notre échelle |

*Tableau 9 : fonctions d'une trieuse de vis comparées à celles de notre trieur*

### 6.4 Ce qui nous distingue

- **Le seul trieur de résistances par couleur qui trie physiquement vers plus de 10 bacs** parmi ceux recensés : le projet Arduino UNO Q ne trie pas, celui de Queen's trie mal, et les trieurs multi-bacs mesurent tous électriquement.
- **Une architecture de trieuse de vis appliquée à la lecture de couleur** : poste de lecture fixe et éclairé, plateau de bacs indexé, contrôle de chute.
- **Un coût environ 100 fois plus faible qu'une trieuse industrielle** (environ 200 € contre plus de 10 000 €), en contrepartie d'une alimentation manuelle et d'une cadence bien plus basse.
- **Une gestion explicite des erreurs** (bac de rejet, contrôle E12, barrière de chute), que les prototypes de résistances recensés ne décrivent pas.

### 6.5 Verrous techniques identifiés

1. **Fiabilité de la lecture couleur** : 60 % de réussite seulement pour Queen's, et une chute à 72 % hors laboratoire pour un CNN.
1. **Alimentation en pièce unique** : pattes tordues, résistances emmêlées, blocages dans l'entonnoir. Chez les trieuses de vis, c'est la partie la plus coûteuse.
1. **Sens de lecture** : savoir quel anneau est le premier.
1. **Synchronisation vision / mécanique** : la résistance doit être immobile et bien cadrée au moment de la capture.

### 6.6 Choix proposés, justifiés par l'état de l'art

| Verrou | Choix proposé | Justification |
|---|---|---|
| Lecture couleur | OpenCV classique (HSV/Lab + seuils), sans deep learning au départ | Prouvé par le projet Arduino UNO Q ; pas de jeu de données à constituer |
| Éclairage | Boîte noire, dôme diffusant, LED blanches, fond uniforme | Condition commune à tous les projets qui fonctionnent, vis comprises |
| Erreurs de lecture | Contrôle par la série E12 + bac de rejet | Une valeur hors série signale une erreur ; on ne trie pas au hasard |
| Alimentation | Dépose manuelle dans une gouttière (F1), trémie en amélioration | Le bol vibrant des trieuses de vis est hors budget |
| Tri | Plateau de 13 bacs sur moteur pas à pas NEMA 17, sous une trappe fixe | Un seul actionneur pour 13 bacs, solution classique des trieurs à bacs rotatifs |
| Commande | PC (vision) + ESP32 (moteurs), reliés en USB série | Séparation des rôles éprouvée (Aad, Cornell, Raspberry Pi + Arduino) |
| Périmètre | Résistances à 4 anneaux, une décade E12 | Le cas le plus répandu ; 5 anneaux en évolution |

*Tableau 10 : choix proposés pour notre trieur*

**Option d'amélioration** : ajouter un contact de mesure au poste de lecture. Mesure et vision se vérifieraient mutuellement, ce qu'aucun projet recensé ne fait.

### 6.7 Questions ouvertes pour l'équipe

- Quelle cadence cible (secondes par résistance) ? Le RIT visait moins de 30 s.
- Quel taux de lecture correcte fixe-t-on comme critère de réussite ?
- Faut-il viser une alimentation automatique (trémie F2 ou roue F3) dès la v1 ?

## 7. Législation et normes applicables

Le volet alimentaire du projet de l'an dernier ne s'applique plus. Pour un trieur de résistances, les textes à examiner sont les suivants (à confirmer avec le tuteur pour un prototype pédagogique) :

| Domaine | Texte | Ce qui nous concerne |
|---|---|---|
| Marquage des composants | IEC 60062:2016 (amendement 2019) | Code couleur que le logiciel décode (voir partie 2) |
| Valeurs normalisées | IEC 60063:2015 | Séries E12, E24, E96 utilisées pour le contrôle de cohérence et le choix des bacs |
| Sécurité des machines | Directive 2006/42/CE (remplacée par le règlement (UE) 2023/1230 à partir de 2027) | Pièces mobiles (plateau rotatif, trappe, servos) : carter, arrêt, absence de points de pincement accessibles |
| Basse tension | Directive 2014/35/UE | Ne s'applique qu'au-delà de 50 V alternatif ou 75 V continu : notre alimentation 12 V reste en dessous, seul le bloc secteur doit être marqué CE |
| Compatibilité électromagnétique | Directive 2014/30/UE | Moteur pas à pas et alimentation à découpage : découplage, câblage court |
| Sécurité photobiologique | IEC 62471 | Éclairage LED de la chambre de prise de vue : LED courantes, groupe de risque à vérifier sur la fiche technique |
| Déchets électroniques | Directive DEEE 2012/19/UE, RoHS 2011/65/UE | Composants conformes RoHS, résistances rejetées et prototype en fin de vie à recycler |

*Tableau 11 : textes réglementaires et normes applicables*

La protection des données (RGPD) est hors sujet : les images ne contiennent que des composants.

### À compléter

- [ ] Vérifier avec le tuteur quels textes le rapport doit traiter pour un prototype non commercialisé
- [ ] Récupérer les fiches techniques (bloc d'alimentation, LED, moteur) pour justifier la conformité

## 8. Sources

**Normes et code couleur**

- [IEC 60062:2016 — Marking codes for resistors and capacitors](https://scc-ccn.ca/standardsdb/standards/2026746)
- [DIN EN 60062 (version allemande)](https://www.dinmedia.de/en/standard/din-en-60062/267498948)
- [EEPower — Resistor color code](https://eepower.com/resistor-guide/resistor-standards-and-codes/resistor-color-code/)
- [TE Connectivity — Resistor color codes](https://te.com/en/products/passive-components/resistors/intersection/resistor-color-codes.html)

**Industrie et brevets**

- [US 6750416 — Instrument for measuring and sorting resistors](https://patents.google.com/patent/US6750416)
- [US 9959616 — Device for reading resistor value with computer vision](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/9959616)

**Prototypes**

- [Arduino Blog — Arduino UNO Q solves the classic resistor pile issue (2026)](https://blog.arduino.cc/2026/02/19/arduino-uno-q-solves-the-classic-resistor-pile-issue/)
- [CircuitDigest — Automate resistor identification with Arduino UNO Q](https://circuitdigest.com/news/automate-resistor-identification-and-sorting-using-arduino-uno-q)
- [Queen's University — Resistor Sorter (2023)](https://qspace.library.queensu.ca/items/8aadfc76-ea6d-4126-8359-9e7fd034a73f)
- [RIT — Automated Resistor Sorter](https://www.rit.edu/imagine/exhibits/automated-resistor-sorter)
- [UIC Engineering Expo — Resistor Sorter](https://engineeringexpo.uic.edu/news-stories/resistor-sorter)
- [Hackster — Low-cost machine sorts resistors at a high speed](https://www.hackster.io/news/low-cost-machine-sorts-resistors-at-a-high-speed-0f63e695dee6)
- [Cornell ECE 4760 — Automated Resistor Sorter with GUI (2016)](https://people.ece.cornell.edu/land/courses/ece4760/FinalProjects/f2016/anp56_bwg38_nol5/anp56_bwg38_nol5/anp56_bwg38_nol5/index.html)
- [Cornell ECE 5725 — Pi Resistor Sorter (2017)](https://courses.ece.cornell.edu/ece5990/ECE5725_Fall2017_projects/Lab_sl2974_rt446_Sichen_Liu_Ruichen_Tan)
- [Trinity University — Senior Design Project: Resistor Sorter (2013)](https://digitalcommons.trinity.edu/engine_designreports/90)
- [Swarthmore College — Resistor Sorter, rapport final (2004)](https://www.swarthmore.edu/sites/default/files/assets/documents/engineering/dg_cs_FinalReport.pdf)
- [Bradley University — Resistor Sorter](https://vip.bradley.edu/resistor-sorter/)
- [GitHub — Resistance Is Futile](https://github.com/jvanburen/rif)
- [Hackaday — Resistorganizer](https://hackaday.com/blog/page/1441/?cat=PHL)

**Trieuses de vis et de petites pièces**

- [Qualimatest QMTInspect-R — fiche technique (Saphir)](https://www.saphir.fr/pdf/Products/QMTInspect-R_Datasheet_F.pdf)
- [Qualimatest QMTInspect-R — documentation détaillée](https://www.saphir.fr/pdf/Details_products_documentations/QMTInspect-R_documentation_detaille_F.pdf)
- [Vester Sensorcheck VSC-10C — annonce de machine d'occasion](https://www.used-machines.com/vester+sortierautomat-sensorcheck+vsc-10c/gm-212-22857)
- [Made-in-China — machines de tri de vis par inspection CCD](https://fr.made-in-china.com/category23_Manufacturing-Processing-Machinery/Optical-Sorting-Machine_109.html)
- [US 7669707 — Material handling apparatus with integrated part sorter (bol vibrant + caméra)](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/7669707)
- [US 6354590 — Rotary bin sorter](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/6354590)
- [Hackaday — Robotic Screw And Bolt Sorter Seeks A New Challenge (2026)](https://hackaday.com/2026/08/08/robotic-screw-and-bolt-sorter-seeks-a-new-challenge/)
- [Hackaday — This Light-Up Sorter Is A Bright Idea (2019)](https://hackaday.com/2019/02/12/)
- [Fab Academy Oulu — Intelligent sorter (2026)](https://fabacademy.org/2026/labs/oulu/Machine_projects/Team1/)
- [NTU — projet de trieuse de vis PS5888 (2022)](https://blogs.ntu.edu.sg/ps5888-2022-g07/?p=6)
- [J. Alves — Automatic screw sorter (2018)](https://jpralves.net/post/2018/11/08/automatic-screw-sorter.html)
- [MakerWorld — trieuse de vis par longueur imprimée en 3D](https://makerworld.com/models/765163)
- [Raspberry Pi — Universal LEGO sorter de D. West (2019)](https://www.raspberrypi.com/news/raspberry-pi-lego-sorter/)

**Recherche**

- [Puno et al. (2022) — 4-band resistor recognition using LeNet-5](https://animorepository.dlsu.edu.ph/faculty_research/5749)
- [JETSUR — Automatic Resistor Value Detection Using Computer Vision and Machine Learning](https://journals.usm.edu.ph/jetsur/index.php/online/article/view/46)
- [Bansmann (2024) — Automatische Klassifikation von Schichtwiderständen, HAW Hamburg](https://reposit.haw-hamburg.de/bitstream/20.500.12738/18023/1/MA_%20Automatische%20Klassifikation%20von%20Shichtwiderst%c3%a4nden%20mittels%20Bildbearbeitung%20und%20Deep%20Learning_geschw%c3%a4rzt.pdf)
- [Roboflow — Band Detection dataset](https://universe.roboflow.com/resistorai/band-detection)
