# Organisation du projet

> Modèle calqué sur la gestion de projet de l'an dernier (note de cadrage, SWOT, planning, RACI, budget, risques).
> Les ❓ sont à remplir ensemble. Les dates sont à caler sur le calendrier officiel.

## 1. Équipe et rôles

L'an dernier, 5 membres avec un chef de projet, deux profils électronique, un profil pneumatique et un profil état de l'art / rédaction. Pour ce sujet, les axes sont :

| Axe | Contenu | Responsable | Suppléant |
|-----|---------|-------------|-----------|
| Gestion de projet | Planning, suivi, comptes rendus, budget, lien avec le tuteur | ❓ | ❓ |
| Vision / algorithme | Acquisition, traitement d'image, jeu de test, évaluation | ❓ | ❓ |
| Électronique et commande | ESP32, drivers, alimentation, schéma / PCB | ❓ | ❓ |
| Mécanique / CAO | Boîte d'imagerie, alimentation, tri, impression 3D | ❓ | ❓ |
| État de l'art et rédaction | Recherche bibliographique, normes, relecture du rapport | ❓ | ❓ |

Règles : un responsable *et* un suppléant par axe. Les grandes décisions se prennent en groupe (décision tracée dans `docs/decisions.md`). Chacun relit au moins une partie d'un autre.

## 2. Note de cadrage (à finaliser)

| Élément | Contenu |
|---------|---------|
| Objectif | Trieur de résistances par reconnaissance d'image (voir `01-contexte.md`) |
| Livrables | Voir `00-consignes.md` |
| Contraintes bloquantes | Budget ❓, délai ❓, compétences, disponibilité du FabLab |
| Hors périmètre (proposition) | Alimentation en vrac par bol vibrant, CMS, industrialisation |
| Critères de réussite (à chiffrer) | Taux de bonne reconnaissance ❓ %, temps par pièce ❓ s, nombre de bacs ❓ |

## 3. Analyse SWOT (premier jet)

| Forces | Faiblesses |
|--------|------------|
| Sujet concret, bien borné et démontrable. Existants et jeux de données publics pour s'inspirer | Compétences vision et mécanique à répartir. Dépendance au PC dans l'architecture actuelle ❓ |
| **Opportunités** | **Menaces** |
| Accès au FabLab (impression 3D). Beaucoup de projets comparables à étudier | Éclairage et reflets. Pièces qui se coincent. Délais de livraison. Dérive du périmètre (trop ambitieux) |

## 4. Planning prévisionnel (proposition en phases)

Les semaines sont relatives (S1 = début effectif du projet). ❓ À caler sur les dates réelles et les semaines de cours.

| Phase | Contenu | Durée | Jalon de sortie |
|-------|---------|-------|-----------------|
| P0 Démarrage | Consignes, contexte, rôles, outils (Git, dossier partagé) | S1 | Consignes validées par le tuteur |
| P1 État de l'art | Existants, algorithmes, normes, alternatives | S1 à S3 | `02` et `03` rédigés avec références |
| P2 Cadrage | Note de cadrage, SWOT, planning, RACI, budget, risques | S2 à S3 | Rendu de gestion de projet |
| P3 Choix et dimensionnement | Décision d'architecture (matrices), liste de composants, commande | S3 à S5 | **Commande passée** |
| P4 Preuves de concept | Image de test + lecture des bandes sur PC. Test du tri sur maquette | S4 à S8 | Démo vision + démo tri séparées |
| P5 Intégration | Boîte d'imagerie, alimentation, tri, communication PC ↔ ESP32 | S8 à S12 | Prototype complet v1 |
| P6 Tests et évaluation | Jeu d'images de test, taux de réussite, cadence, défauts | S10 à S13 | Résultats chiffrés |
| P7 Documentation | Rapport, soutenance, répétition | S11 à S14 | Rendus finaux |

Règle d'or : **les commandes de composants partent tôt** (P3), car les délais de livraison étaient un risque l'an dernier.

## 5. Matrice RACI (proposition)

R = réalise, A = approuve (un seul par tâche), C = consulté, I = informé.

| Tâche | Chef de projet | Vision | Électronique | Mécanique | État de l'art / rédaction |
|-------|----------------|--------|--------------|-----------|---------------------------|
| Note de cadrage, planning | A/R | C | C | C | C |
| État de l'art | I | C | C | C | A/R |
| Choix d'architecture | A | R | R | R | C |
| Algorithme de vision | I | A/R | C | C | I |
| Jeu d'images et évaluation | I | A/R | I | C | C |
| Carte électronique, ESP32 | I | C | A/R | C | I |
| Boîte d'imagerie, tri, CAO | I | C | C | A/R | I |
| Budget et commandes | A/R | C | C | C | I |
| Tests système | A | R | R | R | C |
| Rapport et soutenance | A | R | R | R | R |

## 6. Budget (modèle)

Le rapport de l'an dernier distinguait les coûts RH théoriques (coût moyen journalier de 83,16 €) et les achats réels, avec prévu vs actualisé. Même format :

| Poste | Option retenue | Prévu (€) | Réel (€) |
|-------|----------------|-----------|----------|
| Caméra + optique | ❓ | | |
| Éclairage (LED, diffuseur) | ❓ | | |
| ESP32 + driver | ❓ | | |
| Servos / moteur pas à pas | ❓ | | |
| Matériaux (impression 3D, visserie, bacs) | ❓ | | |
| Électronique (câbles, alimentation, PCB) | ❓ | | |
| Calcul (PC ou Raspberry Pi) | ❓ | | |
| **Total matériel** | | | |
| RH théorique (jours × CMJ) | | | |

## 7. Analyse des risques (premier jet)

Gravité (G) et probabilité (P) de 1 à 4, criticité = G × P.

| # | Risque | G | P | Crit. | Action préventive |
|---|--------|---|---|-------|-------------------|
| R1 | Reflets / éclairage instable, mauvaise lecture | 4 | 3 | 12 | Boîte fermée, LED diffuse, calibrage, jeu d'images dès le début |
| R2 | Pièce coincée, double alimentation, mauvaise orientation | 3 | 4 | 12 | Alimentation manuelle en v1, prototype mécanique simple et testé tôt |
| R3 | Précision insuffisante de l'algorithme classique | 3 | 3 | 9 | Plan B : YOLO / hybride, déterminé par une évaluation chiffrée |
| R4 | Délais de livraison / composant indisponible | 3 | 3 | 9 | Commander tôt, deux fournisseurs, alternatives identifiées |
| R5 | Dépendance au PC pour la démo | 3 | 2 | 6 | PC dédié, image système prête, plan B Raspberry Pi |
| R6 | Dérive du périmètre | 3 | 3 | 9 | Note de cadrage, objectifs « minimum / cible / extension » |
| R7 | Indisponibilité d'un membre / travail à distance | 2 | 3 | 6 | Suppléants, outils partagés, compte rendu à chaque réunion |
| R8 | Retard cumulé sur le rapport | 3 | 3 | 9 | Rédiger au fil de l'eau : une section écrite = un jalon |
| R9 | Désynchronisation vision ↔ tri | 3 | 3 | 9 | Protocole série simple, tests d'intégration, journal des temps |

## 8. Outils et règles de travail

- **Git** : branche `main` protégée, une branche par sujet, relecture avant fusion. Messages de commit clairs. Dossiers `docs/`, `vision/`, `firmware/`, `cao/`, `hardware/`.
- **Documents** : tout le rapport est rédigé en Markdown dans `docs/`, puis exporté en Word / PDF à la fin. Une section = un fichier numéroté.
- **Réunions** : un point hebdomadaire de 30 minutes avec ordre du jour, décisions et actions notés dans `docs/comptes-rendus/`. Point tuteur selon son rythme.
- **Décisions** : tout choix technique important (architecture, composant) est consigné avec options comparées et justification, et sera directement repris dans le rapport.
- **Définition de « terminé »** : testé, documenté, relu, intégré dans le rapport.
- **Jeu de données** : dossier `vision/dataset/` (images brutes + étiquettes), partagé hors Git si trop lourd.

## 9. Prochaines actions

1. Valider les consignes avec le tuteur (dates, notation, budget).
2. Répartir les rôles (section 1).
3. Chaque membre relit `03-architecture-et-alternatives.md` et corrige les notes des matrices.
4. Fixer les objectifs chiffrés et le nombre de bacs.
5. Lancer la recherche bibliographique approfondie (voir `02`).
6. Planifier les premières expériences : photos de test avec un éclairage simple et un script de lecture des bandes.
