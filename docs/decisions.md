# Journal des décisions

Une ligne par décision importante : date, décision, options comparées, justification, auteurs. Ces lignes alimentent directement le rapport (« Choix technologiques »).

| Date | # | Décision | Statut | Options étudiées | Justification | Où |
|------|---|----------|--------|------------------|---------------|-----|
| 2026-10-12 | D1 | Valeurs de la série E12 | **Prise** | Autres séries (E24, E96) non étudiées | Moins de valeurs, correction d'erreurs possible au décodage | `05` §1.1 |
| 2026-10-12 | D2 | Caméra dans une boîte noire à lumière diffuse | **Prise** | Éclairage ambiant écarté | Réduire les reflets et rendre l'image reproductible | `05` §2 |
| 2026-10-12 | D3 | ESP32 classique pour la commande | **Prise** | Autres cartes non comparées | Suffisant pour servos / pas à pas et liaison série | `05` §3.4 |
| | D4 | Architecture de calcul (PC + ESP32 / Raspberry Pi / autre) | Ouverte | Voir `03` §2.1 | | `03` |
| | D5 | Algorithme de reconnaissance | Ouverte | Voir `03` §2.2 | | `03` |
| | D6 | Solution d'éclairage | Pressentie : dôme blanc + bande LED IRC ≥ 90 (L3), polarisation (L4) en option | L1 à L6, protocole de test `05` §2.5 | Matériel commandé pour comparer L3 et L3 + L4 | `05`, `07` |
| 2026-10-23 | D7 | Mécanique de tri : plateau de bacs tournant sous une trappe fixe | **Prise** | M1 à M8 (`05` §3.3), plateau tournant | Chute verticale courte, capteurs fixes, un seul moteur ; un roulement porte le poids des bacs | `07` §1.1 |
| | D8 | Alimentation d'une pièce à la fois | Ouverte (F1 pour la v1) | F1 à F5 | | `05` |
| | D9 | Nombre de bacs | Pressentie : 13 (12 valeurs E12 d'une décade + rejet) | `05` §4 | Démonstration claire pour une v1 | `05`, `07` |
| | D10 | Liaison PC ↔ ESP32 | Pressentie : USB série | USB, Wi-Fi, Bluetooth, MQTT | Fiabilité et simplicité (`06` §2) | `06` |
| | D11 | Moteur du plateau | Pressentie : NEMA 17 + TMC2209 + capteur optique à fourche | 5 moteurs comparés | Précision et couple pour un plateau chargé ; le 28BYJ-48 est lent et son réducteur a du jeu (`06` §4, `07` §1.1) | `06`, `07` |
| | D12 | Mécanisme de lâcher et contrôle de chute | Pressentie : servo + volet, barrière infrarouge | `06` §5 et §6 | | `06` |
| | D13 | Langage de programmation de l'ESP32 | Pressentie : Arduino-ESP32 | Arduino, ESP-IDF, MicroPython (compétences de l'équipe à confirmer) | `06` §8 | `06` |
| | D14 | Caméra | Pressentie : module USB 1080p à objectif M12, avec filtre IR | Webcam grand public, microscope USB | Mise au point manuelle fixe, compacte, reconnue sans pilote par OpenCV | `07` §1.1 |
| 2026-10-23 | D15 | Achats : consommables (filament, résistances de test) par le CESI, le reste sur AliExpress aux frais de l'équipe | **Prise** | | L'équipe garde la machine à la fin du projet | `07` §2 |
