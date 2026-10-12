# État de l'art

> **Squelette de travail.** Les pistes ci-dessous viennent de connaissances générales. **Aucune n'est encore sourcée ni vérifiée** :
> chaque affirmation devra être appuyée par une référence avant d'entrer dans le rapport.
> Format calqué sur le rapport de l'an dernier : solutions existantes, synthèse, législation, puis choix technologiques.

## 1. Solutions existantes

### 1.1 Trieurs et testeurs de composants du commerce
- Testeurs LCR et multimètres automatisés (mesure électrique) ; trieurs de composants industriels (CMS en bande ou en vrac).
- À documenter : principe, prix, vitesse, limites. ❓ Références à chercher.

### 1.2 Lecture du code couleur par vision (projets et applications)
- Applications mobiles et projets open source de lecture de résistances par photo.
- Projets étudiants / makers : caméra + OpenCV, parfois Arduino ou Raspberry Pi.
- À documenter : précision annoncée, conditions d'éclairage, valeurs gérées. ❓ Références à chercher.

### 1.3 Machines de tri par vision, plus généralement
- Tri par caméra de pièces, de fruits, de pièces mécaniques : architecture type caméra, éclairage, traitement, actionneur.
- Utile pour justifier l'architecture.

## 2. Approches de reconnaissance (axe central)

| Approche | Idée | Avantages | Inconvénients |
|----------|------|-----------|---------------|
| **Vision classique** | Segmentation du corps de la résistance, extraction des bandes (HSV / Lab), classification des couleurs, décodage selon la norme | Léger, explicable, tourne sur microcontrôleur ou petit ordinateur embarqué | Sensible à l'éclairage, nécessite un calibrage |
| **Apprentissage profond** | Détection et classification des bandes avec un CNN (type YOLO / MobileNet) ou classification directe de la valeur | Robuste aux variations, évolutif | Il faut constituer un jeu de données annoté, plus de calcul |
| **Hybride** | Détection par réseau, puis lecture classique des couleurs | Compromis | Complexité |
| **OCR** | Lecture de marquages numériques (CMS) | Seule option pour les CMS | Hors du cœur du sujet si THT seulement |

❓ Direction à choisir après comparaison et premiers essais.

## 3. Acquisition d'image (choix matériel)

- Caméra : webcam USB, module caméra (type Raspberry Pi), ESP32-CAM… ❓ à comparer (résolution, optique, prix).
- **Éclairage maîtrisé** : boîte fermée, LED à lumière diffuse, fond uni. C'est probablement le facteur de réussite n°1.
- Calibrage des couleurs (mire, balance des blancs).

## 4. Traitement et calcul embarqué

- Options : PC / Raspberry Pi (OpenCV, Python) ou microcontrôleur seul (ex. STM32 comme l'an dernier, probablement trop limité pour de l'image).
- ❓ Choix à discuter selon la compétence de l'équipe et le budget.

## 5. Tri mécanique et commande

- Actionneurs : servomoteur, trappe, convoyeur, tapis, bras…
- Électronique de commande : drivers, alimentation, éventuel PCB maison (l'an dernier : MOSFET, transistors, amplificateur opérationnel, schémas et simulation LTSpice).
- ❓ Nombre de bacs et de valeurs triées.

## 6. Normes de référence sur les résistances

- **Code couleur** : norme CEI / IEC 60062.
- **Séries de valeurs** : E12, E24, E96… (utile pour contraindre la classification aux valeurs existantes).
- À vérifier avant de les citer.

## 7. Législation et normes applicables

À reprendre et adapter de l'exemple de l'an dernier. L'alimentaire ne s'applique plus ici.
- Sécurité des machines : Directive Machines 2006/42/CE. ❓ Vérifier sa pertinence pour un prototype.
- Sécurité électrique et basse tension, compatibilité électromagnétique. ❓ À identifier.
- Si données, images ou personnes : RGPD probablement hors sujet. ❓
- Normes sur les résistances (voir 6).

## 8. Synthèse et choix (à remplir à la fin)

| Question | Options | Choix | Justification |
|----------|---------|-------|---------------|
| Approche de reconnaissance | classique / deep / hybride | ❓ | ❓ |
| Calcul | MCU / Raspberry Pi / PC | ❓ | ❓ |
| Caméra | ❓ | ❓ | ❓ |
| Mécanisme de tri | ❓ | ❓ | ❓ |

## 9. Sources à collecter

- [ ] Articles et projets de lecture de résistances par vision (≥ 5)
- [ ] Fiches techniques des caméras candidates
- [ ] Norme IEC 60062 et séries E
- [ ] Solutions commerciales de tri de composants
