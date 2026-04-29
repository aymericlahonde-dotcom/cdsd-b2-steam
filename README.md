# Bloc 2 — Steam Big Data (PySpark)

> Projet de la **Certification Jedha — Concepteur Développeur en Science des Données (CDSD)**
> [RNCP35288 — France Compétences](https://www.francecompetences.fr/recherche/rncp/35288/)
> **Bloc 2** : Exploration, analyse et représentation des données — Big Data distribué

## Objectif du projet

L'équipe data de Steam (Valve) veut une analyse macro de leur catalogue de jeux. Le dataset est volumineux et imbriqué (JSON nested), donc on travaille avec **PySpark** sur **Databricks** pour avoir un environnement Big Data distribué.

Trois axes d'analyse :

1. **Macro analysis** — vue d'ensemble du gaming sur Steam : top éditeurs, distribution des sorties par année, prix et remises, langues les plus supportées.
2. **Genre / category analysis** — performance par segment : top genres par nombre de jeux, performance moyenne (positive_rate, popularity_score, prix moyen).
3. **Platform analysis** — Windows vs Mac vs Linux : part de marché et lien entre nombre de plateformes supportées et popularité.

## Données

Le dataset est sur S3 public Jedha :
```
s3://full-stack-bigdata-datasets/Big_Data/Project_Steam/steam_game_output.json
```

Pas besoin de credentials — bucket en accès public lecture pour les apprenants Jedha.

## Livrable

- 2 notebooks PySpark : EDA + analyses statistiques
- Un dashboard Databricks (`STEAM_Project_Dashboard.lvdash.json`) à importer dans Databricks pour visualiser les résultats

## Structure du projet

```
Bloc_2_Steam/
├── notebooks/
│   ├── 01_steam_eda.ipynb              (EDA macro + genres + plateformes)
│   └── 02_steam_stat_models.ipynb      (analyses statistiques approfondies)
├── dashboards/
│   └── STEAM_Project_Dashboard.lvdash.json  (à importer dans Databricks)
└── README.md
```

## Comment rejouer le projet

**Option recommandée — Databricks Community** (gratuit) :
1. Créer un compte sur <https://community.cloud.databricks.com/>
2. Créer un cluster gratuit (single-node, 15 GB RAM)
3. Importer les notebooks via "Workspace → Import"
4. Importer le dashboard via "Dashboards → Import"
5. Run all

**Option locale** (plus complexe — Java + Spark requis) :
```bash
pip install pyspark==3.5.*
```
Lancer un notebook Jupyter avec un kernel PySpark configuré.

## Auteur

[Aymeric Lahonde](https://github.com/aymericlahonde-dotcom) — promotion Jedha CDSD 2026.
