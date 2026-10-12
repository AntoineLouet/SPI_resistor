# Journal des décisions

Une ligne par décision importante : date, décision, options comparées, justification, auteurs. Ces lignes alimentent directement le rapport (« Choix technologiques »).

| Date | # | Décision | Statut | Options étudiées | Justification | Où |
|------|---|----------|--------|------------------|---------------|-----|
| 2026-10-12 | D1 | Valeurs de la série E12 | **Prise** | Autres séries (E24, E96) non étudiées | Moins de valeurs, correction d'erreurs possible au décodage | `05` §1.1 |
| 2026-10-12 | D2 | Caméra dans une boîte noire à lumière diffuse | **Prise** | Éclairage ambiant écarté | Réduire les reflets et rendre l'image reproductible | `05` §2 |
| 2026-10-12 | D3 | ESP32 classique pour la commande | **Prise** | Autres cartes non comparées | Suffisant pour servos / pas à pas et liaison série | `05` §3.4 |
| | D4 | Architecture de calcul (PC + ESP32 / Raspberry Pi / autre) | Ouverte | Voir `03` §2.1 | | `03` |
| | D5 | Algorithme de reconnaissance | Ouverte | Voir `03` §2.2 | | `03` |
| | D6 | Solution d'éclairage | Ouverte | L1 à L6, protocole de test `05` §2.5 | | `05` |
| | D7 | Mécanique de tri | Ouverte (M1 pressentie) | M1 à M8, `05` §3.3 | | `05` |
| | D8 | Alimentation d'une pièce à la fois | Ouverte (F1 pour la v1) | F1 à F5 | | `05` |
| | D9 | Nombre de bacs | Ouverte | `05` §4 | | `05` |
| | D10 | Liaison PC ↔ ESP32 | Pressentie : USB série | USB, Wi-Fi, Bluetooth, MQTT | Fiabilité et simplicité (`06` §2) | `06` |
| | D11 | Moteur de sélection | Pressentie : pas à pas (28BYJ-48, puis NEMA 17 si besoin) + capteur de référence | 5 moteurs comparés | `06` §4 | `06` |
| | D12 | Mécanisme de lâcher et contrôle de chute | Pressentie : servo + volet, barrière infrarouge | `06` §5 et §6 | | `06` |
| | D13 | Langage de programmation de l'ESP32 | Pressentie : Arduino-ESP32 | Arduino, ESP-IDF, MicroPython (compétences de l'équipe à confirmer) | `06` §8 | `06` |
