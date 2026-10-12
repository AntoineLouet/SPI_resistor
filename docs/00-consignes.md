# Consignes du projet SPI

> **Statut : reconstituées.** Aucun énoncé officiel n'a été fourni. Ces consignes sont déduites du rapport
> (`Rapport_SPI`) et de la soutenance (`Machine_Cocktail_SPI`) du groupe de l'an dernier (projet « Machine à cocktail »).
> Les points marqués ⚠️ sont à confirmer auprès du tuteur ou dans l'énoncé officiel du CESI.

## 1. Cadre

- Projet SPI de 2ᵉ année, filière S3E / SEEE, CESI.
- Équipe de 4 à 6 étudiants. Le sujet doit être en accord avec la filière ; il peut être proposé par les étudiants ou par un partenaire.
- Objectif pédagogique : donner un aperçu du monde de l'entreprise et des missions d'un ingénieur junior.
- Un tuteur suit le projet (l'an dernier : M. Nicolas Van Wambeke).
- Le projet mobilise plusieurs disciplines. Pour la machine à cocktail : pneumatique, hydraulique, électronique, modélisation 3D et programmation embarquée.
  ⚠️ Cette année, le sujet est le **tri de résistances par reconnaissance à partir de photos**. Il faut donc choisir les disciplines (vision, électronique, mécanique, logiciel) et vérifier qu'elles restent conformes à la filière.
- Le CESI donne accès au FabLab (impression 3D, etc.) et passe les commandes de composants via le responsable de formation.

## 2. Livrables attendus

| # | Livrable | Contenu attendu (d'après l'an dernier) |
|---|----------|----------------------------------------|
| 1 | **Note de cadrage** | Périmètre, contraintes bloquantes (budget, temps, technique), SWOT |
| 2 | **Planning prévisionnel** | Phases du projet, mis à jour pendant le projet |
| 3 | **Matrice RACI** | Rôles et responsabilités par tâche |
| 4 | **Budget** | Chiffrage prévisionnel et réel (RH théoriques au coût moyen journalier, plus achats matériels) |
| 5 | **Grille d'analyse des risques** | Risques techniques, matériels et d'organisation |
| 6 | **Revue de projet** | Point d'avancement (annexe 5 du rapport de l'an dernier) |
| 7 | **Rapport / mémoire** | Voir le plan type ci-dessous |
| 8 | **Prototype fonctionnel** | Première version physique, testée |
| 9 | **Soutenance orale** | Environ 40 diapositives l'an dernier, même structure que le rapport |
| 10 | **Dépôt Git** | Code versionné avec un workflow Git documenté |

⚠️ Dates de rendu, durée de la soutenance, format et notation : **à obtenir**.

## 3. Plan type du rapport

1. Page de garde, table des matières, introduction, remerciements
2. **Gestion de projet** : organisation de l'équipe, note de cadrage (SWOT), planification (planning, RACI), budget, analyse des risques
3. **État de l'art** : solutions existantes, synthèse comparative, **législation et normes applicables**
4. **Choix technologiques** : choix argumentés et comparés (tableaux avantages / inconvénients)
5. **Phénomènes physiques / bases théoriques** utiles au sujet
6. **Présentation des composants** : principe de fonctionnement de chacun
7. **Choix et justification des composants** : référence précise, caractéristiques, pourquoi ce choix
8. **Réalisation** : carte électronique (schéma, dimensionnement, simulation), modélisation 3D, code embarqué, workflow Git
9. **Tests fonctionnels** : démarche progressive, composant par composant, puis système complet
10. Conclusion et perspectives, références, table des illustrations, annexes
11. Abstract en français et en anglais

## 4. Exigences de fond

- **Travailler en équipe** avec des rôles définis dès le début. Les grandes décisions se prennent en groupe.
- **Étudier l'existant avant de concevoir**, en dégageant les avantages et les inconvénients de chaque solution.
- **Justifier chaque choix technique** : comparer les options (tableaux), puis choisir selon les contraintes (coût, taille, performances, faisabilité).
- **Dimensionner et prouver** : calculs (par exemple le dimensionnement des résistances de commande) et simulation (LTSpice l'an dernier).
- **Tester progressivement** : élément seul, puis éléments couplés, puis système complet. Comparer le résultat aux hypothèses de départ.
- **Intégrer la législation et les normes** applicables, même pour un prototype pédagogique.
- **Citer ses sources** dans les références. Légender et numéroter figures et tableaux.
- **Rester dans le cadre** : la note de cadrage sert à ne pas partir dans tous les sens.
- Le prototype doit **fonctionner**, au moins en version 1. Les limites et les pistes d'amélioration sont à expliciter.

## 5. Critères de qualité visibles dans l'exemple

- Cohérence entre le rapport et la soutenance (mêmes parties, mêmes chiffres).
- Chaque membre a un rôle identifiable (ex. : chef de projet, électronique, logiciel, état de l'art et rédaction).
- Budget suivi (prévu vs réel). L'an dernier : environ 106 € de matériel et environ 26 jours-homme théoriques.
- Traces de la démarche : schémas, captures de simulation, photos de tests, versions successives des pièces (V1 → V2).

## 6. Questions ouvertes (à régler avec le tuteur)

- [ ] Énoncé officiel, grille de notation, dates des jalons
- [ ] Budget alloué et délai de livraison des commandes
- [ ] Taille de l'équipe et rôles de chacun
- [ ] Le tri est-il purement logiciel (reconnaissance) ou aussi mécanique (convoyeur, bacs, tri physique) ?
- [ ] Nombre minimum de disciplines exigées et lien avec la filière
