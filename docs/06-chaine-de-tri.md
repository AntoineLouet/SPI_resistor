# La chaîne de tri : de la valeur reçue par l'ESP32 jusqu'au bac

> Section du rapport : « Choix technologiques » (partie commande et actionneurs). Fait suite à `05-eclairage-et-mecanique.md` (solution mécanique pressentie : **tube rotatif M1**).
> Les notes des matrices sont **provisoires** et viennent de l'analyse de Claude. Les chiffres tirés de sources sont référencés ; tous les autres sont des ordres de grandeur **à mesurer**.

## 1. Vue d'ensemble : que se passe-t-il quand la valeur est connue ?

```
 PC (vision)                       ESP32                                   Mécanique
 ───────────                       ─────                                   ─────────
 valeur lue (ex. 4,7 kΩ)
   │ 1. table valeur -> n° de bac
   │ 2. envoie  "S 5"  ───────────►  3. vérifie la commande, répond "ACK 5"
                                     4. fait tourner le tube vers le bac 5 ──► moteur pas à pas
                                     5. contrôle la position (capteur)
                                     6. ouvre la trappe ───────────────────► servo : la pièce tombe
                                     7. attend le passage de la pièce ◄───── barrière infrarouge
   ◄──────  "DONE 5"  ──────────── 8. confirme, referme la trappe, se prépare
 9. compte le bac, affiche, passe à la pièce suivante
```

L'ESP32 joue le rôle d'**exécutant sûr** : il ne « réfléchit » pas à la vision, il exécute une séquence courte et la vérifie à chaque étape.

## 2. Liaison PC ↔ ESP32

| Option | Principe | Avantages | Inconvénients |
|--------|----------|-----------|---------------|
| **USB série (UART)** | Câble USB, puce USB-série de la carte ESP32 | Fiable, rapide, pas de configuration réseau, alimente la logique | Un câble |
| Wi-Fi (TCP / WebSocket) | Réseau local | Sans fil | Latence variable, coupures, configuration |
| Bluetooth | Liaison sans fil à courte portée | Sans fil | Moins simple, débit et portée limités |
| MQTT | Messagerie via un serveur | Extensible | Surdimensionné pour 2 appareils |

| Critère | Poids | USB série | Wi-Fi | Bluetooth | MQTT |
|---------|-------|-----------|-------|-----------|------|
| Fiabilité | 3 | 5 | 3 | 3 | 3 |
| Latence | 2 | 5 | 3 | 3 | 2 |
| Simplicité | 3 | 5 | 3 | 2 | 2 |
| Câblage | 1 | 3 | 5 | 5 | 5 |
| Sans fil | 1 | 1 | 5 | 5 | 5 |
| **Total (max 50)** | | **44** | **34** | **31** | **29** |

**Choix pressenti : USB série.**

### 2.1 Protocole de messages (proposition)
Texte lisible, une ligne par message, avec accusé de réception. Facile à déboguer avec un simple terminal.

| Sens | Message | Sens du message |
|------|---------|-----------------|
| ESP32 → PC | `PART` | une pièce est en place sous la caméra |
| PC → ESP32 | `S 5` | trier vers le bac 5 (0 = bac de rejet) |
| ESP32 → PC | `ACK 5` | commande comprise |
| ESP32 → PC | `DONE 5` | pièce tombée dans le bac 5 |
| ESP32 → PC | `ERR JAM` / `ERR TIMEOUT` / `ERR HOME` | erreur (pièce coincée, délai dépassé, position perdue) |
| PC → ESP32 | `H` | retour à la position de référence |
| PC → ESP32 | `L 80` | régler l'éclairage (si commandé par l'ESP32) |

Règles de sûreté : un message inconnu ou un numéro de bac hors plage est refusé (`ERR ARG`) ; si le PC ne répond plus pendant un délai donné, l'ESP32 passe en position sûre (trappe fermée).

## 3. Logique de commande de l'ESP32 : une machine à états

```
   IDLE ──(pièce détectée)──► IMAGING ──(S n reçu)──► MOVING ──(position atteinte)──► RELEASING
     ▲                          │                       │                                 │
     │                    (délai dépassé)         (délai dépassé)                  (passage détecté)
     │                          ▼                       ▼                                 ▼
     └──────────────────────── ERROR  ◄────────────────────────────────────────────── DONE
                                 (reprise : H, nouvel essai, ou arrêt et signalement au PC)
```

Pourquoi une machine à états : chaque état a une condition de sortie *vérifiée* et un délai maximal. On ne dépend pas d'attentes fixes (`delay`) qui masquent les pannes.

## 4. Choix du moteur de sélection (positionnement du tube)

Un ordre de grandeur utile : avec N bacs sur 360°, l'écart entre bacs vaut 360/N (par exemple 27,7° pour 13 bacs). Il faut donc un positionnement répétable à bien moins de 5°.

| Option | Principe | Caractéristiques à retenir |
|--------|----------|----------------------------|
| **Servo 180° (type SG90)** | Servomoteur de loisir, commande par impulsion | Course d'environ 180° seulement (1 à 2 ms). Couple annoncé autour de 1,8 kgf·cm. Zone morte et valeurs variables selon les fabricants et les copies [1] |
| **Moteur pas à pas 28BYJ-48 + ULN2003** | Petit moteur à réducteur | 2 048 pas par tour (2 038 mesurés sur certaines unités, donc à étalonner) [2]. Couple modeste, plutôt lent [2] |
| **Moteur pas à pas NEMA 17 + driver** (A4988, DRV8825, TMC2209) | Moteur standard d'imprimante 3D | Précis, couple et vitesse largement suffisants. Driver et alimentation 12 V à prévoir |
| Moteur à courant continu + codeur | Boucle fermée | Rapide, mais il faut régler un asservissement (PID) |
| Servo « intelligent » à liaison série | Servo avec retour de position | Précis et à retour d'état, mais plus cher et moins courant |

| Critère | Poids | SG90 | 28BYJ-48 | NEMA 17 | DC + codeur | Servo intelligent |
|---------|-------|------|----------|---------|-------------|-------------------|
| Précision / répétabilité | 3 | 3 | 3 | 5 | 4 | 5 |
| Couple suffisant | 2 | 3 | 3 | 5 | 4 | 4 |
| Coût | 2 | 5 | 5 | 3 | 3 | 2 |
| Simplicité de câblage et de code | 3 | 5 | 4 | 3 | 1 | 3 |
| Course de 360° | 2 | 1 | 5 | 5 | 5 | 4 |
| Vitesse | 1 | 4 | 2 | 5 | 5 | 4 |
| Disponibilité | 2 | 5 | 5 | 4 | 3 | 2 |
| **Total (max 75)** | | **56** | **59** | **63** | **50** | **52** |

**Lecture** : le servo 180° ne couvre pas 360° (ou il faut un engrenage). Le 28BYJ-48 est le plus économique pour un premier prototype. Le NEMA 17 est le plus solide si l'on veut de la vitesse et de la marge. **Proposition : commencer par un 28BYJ-48, passer au NEMA 17 si la vitesse ou la précision manquent.**

### 4.1 Commande du moteur par l'ESP32
- Une bibliothèque de commande avec **accélération** (trapèze de vitesse) évite de perdre des pas. FastAccelStepper cible les ESP32 et se décrit comme une alternative rapide à AccelStepper, avec jusqu'à 200 000 pas/s annoncés sur ESP32 [3]. Attention : le projet signale des incompatibilités avec certaines versions du noyau Arduino-ESP32 [3].
- **Chemin le plus court** : le tube tourne dans le sens qui minimise l'angle (jamais plus de 180°). Pour des bacs répartis uniformément, l'angle moyen à parcourir est d'environ 90° (calcul, à confirmer par la mesure).
- **Astuce de rangement** : placer les valeurs les plus fréquentes près de la position de repos pour raccourcir les trajets.
- **Étalonner** le nombre de pas par tour de la machine réelle plutôt que de se fier au nombre nominal [2].

### 4.2 Référence de position (« homing »)
Un moteur pas à pas fonctionne en boucle ouverte : il peut perdre des pas sans le savoir. Il faut une référence.

| Solution | Principe | Remarque |
|----------|----------|----------|
| Fin de course mécanique | Un contact | Simple, usure |
| **Capteur optique à fourche** | Un doigt solidaire du tube coupe un faisceau | Sans contact, fiable, bon marché |
| Capteur à effet Hall + aimant | Détection d'un aimant | Sans contact, insensible à la poussière |
| Codeur | Mesure continue | Plus complexe |

**Proposition : capteur optique ou à effet Hall**, avec repassage par la référence à chaque cycle ou toutes les N pièces pour recaler la position.

## 5. Choix du mécanisme de lâcher (trappe)

| Option | Principe | Remarque |
|--------|----------|----------|
| **Servo + volet** | Un petit servo ouvre une trappe | Simple, silencieux |
| Électro-aimant (solénoïde) | Tige poussée par une bobine | Rapide, mais nécessite un transistor de puissance et une diode de roue libre |
| Électrovanne à air | Jet qui pousse la pièce | Réutilise l'expérience de l'an dernier, mais lourd pour des pièces légères |
| Roue à alvéoles | La rotation libère la pièce | Change toute l'architecture (voir M2 dans `05`) |

| Critère | Poids | Servo + volet | Électro-aimant | Électrovanne à air | Roue à alvéoles |
|---------|-------|---------------|----------------|--------------------|-----------------|
| Simplicité | 3 | 5 | 3 | 2 | 2 |
| Fiabilité | 3 | 4 | 4 | 3 | 4 |
| Rapidité | 2 | 3 | 5 | 5 | 3 |
| Coût | 2 | 5 | 4 | 2 | 3 |
| Consommation | 1 | 4 | 2 | 1 | 3 |
| Bruit / chocs | 1 | 4 | 2 | 2 | 4 |
| **Total (max 60)** | | **51** | **43** | **32** | **37** |

**Choix pressenti : servo + volet.**

## 6. Contrôler que la pièce est bien tombée

| Option | Principe | Fiabilité | Coût | Simplicité | Information utile | Total (max 50) |
|--------|----------|-----------|------|------------|-------------------|----------------|
| Aucun contrôle (temporisation) | On attend un délai | 1 | 5 | 5 | 1 | 30 |
| **Barrière infrarouge** | Une pièce qui passe coupe le faisceau | 4 | 4 | 4 | 4 | **40** |
| Capteur infrarouge réflectif | Détecte la pièce qui passe devant | 3 | 5 | 4 | 4 | 39 |
| Seconde caméra | Revoit la pièce | 4 | 2 | 2 | 5 | 32 |
| Micro-interrupteur à lamelle | Une lamelle est actionnée | 1 | 5 | 4 | 3 | 31 |

Poids : fiabilité 3, coût 2, simplicité 3, information 2. **Choix pressenti : barrière infrarouge**, à la sortie du tube. Une pièce de ¼ W est très légère (quelques dixièmes de gramme, ordre de grandeur) : un contact mécanique risque de ne pas réagir.

La barrière permet aussi de **compter** les pièces par bac et de détecter un bac plein par comptage.

## 7. Électronique de puissance et alimentation

- Séparer l'alimentation de la logique (3,3 V de l'ESP32) de celle des actionneurs (servo en 5 V, moteur pas à pas en 12 V, selon le choix). Relier les masses. Prévoir un condensateur de découplage près du moteur et du servo.
- Commutation de charges inductives (bobine, électro-aimant) : transistor ou MOSFET de puissance + **diode de roue libre**. C'est le même savoir-faire que celui du projet de l'an dernier (MOSFET IRFZ44N et pilotage de grille), donc une base déjà documentée.
- Les sorties de l'ESP32 sont limitées en courant (et en 3,3 V) : ne jamais piloter un moteur directement.
- **Arrêt d'urgence** ou coupure d'alimentation des actionneurs, accessible à l'utilisateur.
- Câblage propre : connecteurs, repérage des fils, et au besoin une carte électronique dédiée (schéma + simulation, comme l'an dernier).

## 8. Programmation de l'ESP32

| Option | Atouts | Limites |
|--------|--------|---------|
| **Arduino-ESP32 (C++)** | Nombreuses bibliothèques (servo, pas à pas), prise en main rapide, accès aux tâches FreeRTOS | Abstraction qui cache certains détails |
| ESP-IDF | Contrôle complet, usage industriel | Courbe d'apprentissage plus longue |
| MicroPython | Écriture rapide | Génération d'impulsions moins précise, moins adapté au temps réel |

| Critère | Poids | Arduino | ESP-IDF | MicroPython |
|---------|-------|---------|---------|-------------|
| Simplicité | 3 | 5 | 2 | 5 |
| Temps réel | 2 | 4 | 5 | 2 |
| Bibliothèques | 3 | 5 | 4 | 3 |
| Compétence de l'équipe | 2 | 4 | 2 | 3 |
| Débogage | 1 | 3 | 4 | 4 |
| **Total (max 55)** | | **49** | **36** | **38** |

**Choix pressenti : Arduino-ESP32.** La note « compétence de l'équipe » est une hypothèse à corriger.

## 9. Défaillances prévues et parades

| Problème | Détection | Parade |
|----------|-----------|--------|
| Pièce coincée dans le tube | La barrière ne voit rien dans le délai | Nouveau cycle trappe / petite secousse du tube, puis `ERR JAM` |
| Deux pièces à la fois | La vision en compte plus d'une | Envoi au bac de rejet, alerte à l'utilisateur |
| Valeur douteuse (confiance faible) | Seuil de la vision | Bac de rejet « à vérifier à la main » |
| Moteur qui perd des pas | Capteur de référence qui ne se déclenche plus au bon endroit | Recalage (`H`), puis `ERR HOME` si échec |
| Bac plein | Comptage | Message à l'utilisateur |
| Liaison coupée | Aucun message dans le délai | Trappe fermée, mouvement arrêté |
| Chute d'alimentation | Tension basse | Redémarrage propre, recalage au démarrage |

## 10. Estimation du temps par pièce (à mesurer)

| Étape | Ordre de grandeur |
|-------|--------------------|
| Détection et stabilisation | 0,2 à 0,4 s |
| Capture et analyse | 0,3 à 0,8 s |
| Échange série | < 0,05 s |
| Rotation du tube | 0,3 à 1,5 s selon le moteur et l'angle |
| Ouverture, chute, confirmation | 0,3 à 0,5 s |
| **Total** | **environ 1,5 à 3 s** |

## 11. Synthèse des choix pressentis

| Bloc | Choix pressenti | Principale raison |
|------|-----------------|-------------------|
| Liaison PC ↔ ESP32 | USB série | Fiable, simple |
| Logique de commande | Machine à états | Chaque étape est vérifiée |
| Moteur de sélection | Pas à pas (28BYJ-48 puis NEMA 17 si besoin) + capteur de référence | 360°, répétabilité, coût |
| Trappe | Servo + volet | Simple, silencieux |
| Contrôle de chute | Barrière infrarouge | Fiable pour des pièces légères |
| Firmware | Arduino-ESP32 | Bibliothèques et rapidité de mise en œuvre |

## Références
1. Fiches techniques SG90 (valeurs variables selon les fabricants) : https://www.waveshare.com/SG90-Servo.htm ; https://docs.m5stack.com/en/accessory/SG90_Servo ; https://bit.nkust.edu.tw/~csshieh/teach/111A/mc/datasheet/SG90.pdf
2. 28BYJ-48 : https://docs.circuit.rocks/motors/28byj-48/ ; https://forum.arduino.cc/t/28byj-48-steps-per-revolution/876716
3. FastAccelStepper (wiki du projet) : https://github.com/gin66/FastAccelStepper/wiki
