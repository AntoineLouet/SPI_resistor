# Bon de commande (tout passe par le CESI)

> Remplace la répartition CESI / AliExpress du `07` §2 (décision D16). **Plafond : 200 € TTC**, frais de port compris.
> Les prix sont des **estimations chez un distributeur français** (Gotronic, Conrad, Amazon…), plus chers qu'AliExpress. **À vérifier** auprès du fournisseur choisi par le responsable de formation.
> Les spécifications détaillées et la justification de chaque composant sont dans `07` §3 et §4.

## 1. Priorité 1 : indispensable (≈ 181 €)

| Réf. | Article | Spécification à respecter | Qté | Prix estimé (€) |
|------|---------|---------------------------|-----|-----------------|
| K1 | Filament PLA noir mat | **1,75 mm** (2,85 mm si imprimante Ultimaker), 1 kg | 1 | 22,00 |
| K3 | Assortiment de résistances E12, film carbone 5 % | ¼ W, 4 bandes, corps beige, toutes les valeurs E12 de 10 Ω à 1 MΩ, ≥ 10 pièces par valeur (vérifier 1,2 k, 3,9 k, 8,2 k) | 1 | 12,00 |
| C1 | Carte ESP32 DevKit V1 | ESP32-WROOM-32, 38 broches, USB-C | 1 | 10,00 |
| C2 | Moteur pas à pas NEMA 17 | 17HS4401, 1,8°, 1,7 A, arbre 5 mm, câble fourni | 1 | 14,00 |
| C3 | Driver TMC2209 | Format StepStick | 1 | 8,00 |
| C4 | Condensateur électrolytique | 100 µF, ≥ 25 V | 2 | 1,00 |
| C5 | Servo MG90S | Pignons métal, 4,8 à 6 V | 1 | 5,00 |
| C6 | Capteur optique à fourche | Module LM393, fente ≈ 5 mm, sortie numérique | 1 | 3,00 |
| C7 | Barrière infrarouge | Émetteur + récepteur 5 mm, portée ≥ 10 cm | 1 paire | 5,00 |
| A1 | Bloc secteur 12 V 3 A | **Prise européenne, marquage CE**, jack 5,5 × 2,1 mm | 1 | 12,00 |
| A2 | Embase jack femelle vers bornier | 5,5 × 2,1 mm | 1 | 2,00 |
| A3 | Convertisseur abaisseur réglable | XL4015 (ou LM2596), sortie réglée à 5 V | 1 | 5,00 |
| A4 | Bouton d'arrêt d'urgence | Coup de poing 22 mm, contact NC, avec boîtier | 1 | 10,00 |
| A5 | Porte-fusible + fusibles | 5 × 20 mm, fusibles 3 A | 1 lot | 3,00 |
| V1 | Module caméra USB 1080p | UVC, objectif M12 à mise au point manuelle, **avec filtre IR (« IR-cut »)**, exposition et balance des blancs réglables | 1 | 40,00 |
| V2 | Bande LED COB | **IRC ≥ 90**, 5 000 à 6 500 K, 5 V (ou 12 V), 1 m | 1 | 10,00 |
| V3 | Module MOSFET | D4184, commandable en 3,3 V | 1 | 3,00 |
| M1 | Roulement de plateau tournant | Ø 100 à 150 mm | 1 | 6,00 |
| M2 | Moyeu à bride rigide | Alésage 5 mm | 1 | 4,00 |
| P2 | Plaque à pastilles + connecteurs | 7 × 9 cm, barrettes femelles 2,54 mm, borniers à vis 5,08 mm | 1 lot | 6,00 |
| | **Total priorité 1** | | | **181,00** |

## 2. Priorité 2 : seulement si le FabLab ou un membre ne l'a pas déjà

À demander **avant** d'envoyer la commande. Il reste ≈ 19 € de marge (moins les frais de port).

| Réf. | Article | Spécification | Qté | Prix estimé (€) |
|------|---------|---------------|-----|-----------------|
| K2 | Filament PLA blanc | Même diamètre que K1, 250 ou 500 g (≈ 75 g utilisés : le dôme) | 1 | 15,00 |
| M3 | Vis et écrous M3 | Longueurs 6 à 20 mm | 1 boîte | 8,00 |
| P1 | Plaque d'essai + câbles Dupont | 830 points, M-M, M-F, F-F | 1 kit | 8,00 |
| P3 | Fil de câblage | 22 AWG, plusieurs couleurs | 1 lot | 6,00 |
| P4 | Câble USB-C de **données** | 1 m | 1 | 4,00 |
| K5 | Étain 0,8 mm + gaine thermorétractable | | 1 lot | 8,00 |

## 3. Si le budget le permet (dans cet ordre)

| Réf. | Article | Intérêt | Prix estimé (€) |
|------|---------|---------|-----------------|
| R2 | Driver TMC2209 de rechange | Le composant le plus fragile (détruit si on débranche le moteur sous tension) | 8,00 |
| O2 | Objectif M12 8 ou 12 mm | Si l'objectif fourni avec la caméra est trop grand angle | 8,00 |
| K4 | Assortiment de résistances film métal 1 % (corps bleu) | Tester la vision sur des corps bleus | 8,00 |
| R1 | Carte ESP32 de rechange | | 10,00 |

Abandonnés par rapport au `07` : options O1, O3, O4 et rechange R3 (trop chères hors AliExpress pour leur intérêt).

## 4. Avant d'envoyer

1. Vérifier le **diamètre du filament** selon l'imprimante du FabLab.
2. Recenser ce que le FabLab et les membres ont déjà (§2) et rayer ces lignes.
3. Demander au responsable chez quel fournisseur il commande et les **frais de port** : s'ils dépassent ≈ 15 €, retirer des lignes du §2 ou §3.
4. Demander le **délai de livraison** et si les pièces imprimées peuvent rester sur la machine.
