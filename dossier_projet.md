# Dossier projet — Steam Big Data (Bloc 2, PySpark)

> Certification Jedha CDSD — RNCP35288
> Bloc 2 : Exploration, analyse et représentation des données (Big Data distribué)
> Auteur : Aymeric Lahonde — promotion CDSD 2026

## 1. Contexte et objectif

L'équipe data de Steam (Valve) souhaite une **analyse macro de son catalogue de jeux**. Les données
sont fournies sous forme d'un fichier **JSON imbriqué et volumineux** (55 691 jeux), ce qui justifie
un traitement **Big Data distribué avec PySpark** plutôt qu'un simple script pandas.

Trois axes d'analyse demandés :
1. **Macro** — éditeurs, sorties par année, prix, remises, langues supportées.
2. **Genres** — volume et performance moyenne par segment.
3. **Plateformes** — parts Windows / Mac / Linux et lien multi-plateforme ↔ succès.

## 2. Données

- Source : `s3://full-stack-bigdata-datasets/Big_Data/Project_Steam/steam_game_output.json`
  (bucket public Jedha, ~63 Mo, 55 691 enregistrements).
- Format : **tableau JSON** (`multiLine=True`) ; chaque élément a un `id` et un champ imbriqué `data`
  (nom, éditeur, genre, langues, prix en centimes, remise, `platforms.{windows,mac,linux}` booléens,
  `positive`/`negative`, `release_date` au format `yyyy/MM/dd`, `categories`, plusieurs centaines de tags).

## 3. Pipeline (notebook `01_steam_eda.ipynb`)

1. **Lecture** distribuée du JSON imbriqué (`spark.read.option("multiLine", True).json(...)`).
2. **Helpers de robustesse** : `has_path` (existence d'un chemin imbriqué) et `col_or_null`
   (colonne ou `NULL`) — le schéma variant d'un jeu à l'autre.
3. **Table propre `games`** (gold) avec corrections métier :
   - prix en centimes → euros (`/100`) ;
   - dates `yyyy/MM/dd` (jour parfois sur 1 chiffre) parsées en mode `LEGACY` ;
   - plateformes booléennes → entiers 0/1 ;
   - variables dérivées : `n_reviews`, `positive_rate`, `popularity_score = log1p(n_reviews) × positive_rate`,
     `nb_platforms`, et la cible `success_high`.
4. **Normalisation** des champs multi-valués : `explode` sur genres et langues.
5. **Tables d'agrégation** pour le dashboard (schéma `default` sur Databricks, CSV en local) :
   `publisher_most_games`, `releases_by_year`, `top_genres`, `genre_performance`,
   `price_bucket_vs_success`, `platforms_vs_success` (+ `platform_share`, `price_stats`, `top_languages`).
6. **Exports visuels** : 7 graphiques matplotlib (`assets/*.png`).

## 4. Modélisation (notebook `02_steam_stat_models.ipynb`)

- **Baseline « règle métier »** : prédire un hit si le jeu est multi-plateforme (≥ 2 OS).
  Matrice de confusion → accuracy 0,73 · precision 0,15 · recall 0,42.
- **Régression logistique MLlib** (`VectorAssembler` + `LogisticRegression`) sur des variables
  disponibles à la sortie (prix, remise, nb de plateformes, année) → **AUC ≈ 0,72**.
  Le coefficient dominant est `nb_platforms`.

## 5. Résultats principaux

| Indicateur | Valeur |
|---|---|
| Jeux analysés | 55 691 |
| Éditeur le plus prolifique | Big Fish Games (422) |
| Année record de sorties | 2021 (8 805) |
| Genres majeurs | Indie, Action, Casual, Adventure |
| Prix médian / moyen payant | 4,99 € / 8,99 € |
| Support plateformes | Windows 100 % · Mac 22,9 % · Linux 15,2 % |
| Taux de hits par nb d'OS | 6,9 % → 11,5 % → 17,9 % |
| Taux de hits par tranche de prix | 3,5 % (<5 €) → 29,8 % (20-40 €) |

**Lecture métier** : le marché Steam est dominé par une longue traîne de petits éditeurs et une vague
indé depuis 2014 ; le prix médian est bas (4,99 €) ; le support multi-plateforme et un positionnement
prix premium (20-40 €) sont associés à des taux de succès nettement plus élevés.

## 6. Reproductibilité

Les notebooks détectent automatiquement l'environnement : sur Databricks ils utilisent `spark` et
`display` natifs et écrivent les tables dans le schéma `default` (dashboard) ; en local ils créent un
`SparkSession local[*]`, rendent les tables en pandas et exportent les CSV dans `assets/tables/`.
Voir le `README.md` pour les commandes exactes.

## 7. Livrables

- `notebooks/01_steam_eda.ipynb`, `notebooks/02_steam_stat_models.ipynb` (exécutés, sorties visibles)
- `dashboards/STEAM_Project_Dashboard.lvdash.json`
- `assets/*.png` (7 graphiques) + `assets/tables/*.csv` (6 tables)
- `Presentation_Steam.pptx`, `PREPARE_JURY.md`, ce dossier.
