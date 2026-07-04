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

## Résultats clés (dataset réel, 55 691 jeux)

- **Catalogue** : longue traîne d'éditeurs (top : Big Fish Games, 422 jeux) ; explosion des sorties à partir de 2014, pic à **8 805 jeux en 2021**.
- **Genres** : Indie (39,7k), Action (23,8k), Casual (22,1k), Adventure (21,4k).
- **Prix** : 14 % des jeux gratuits, prix médian **4,99 €**, prix moyen payant **8,99 €**.
- **Plateformes** : Windows 100 %, Mac 22,9 %, Linux 15,2 %.
- **Succès** : le taux de « hits » passe de 6,9 % (1 OS) à 17,9 % (3 OS) et de 3,5 % (<5 €) à 29,8 % (20-40 €). Une régression logistique MLlib atteint **AUC ≈ 0,72**, avec le nombre de plateformes comme 1ᵉʳ facteur.

## Livrable

- 2 notebooks PySpark **exécutés** (sorties + graphiques visibles) : EDA + modèles statistiques
- Un dashboard Databricks (`STEAM_Project_Dashboard.lvdash.json`) à importer dans Databricks
- Exports visuels (`assets/*.png`) et tables du dashboard en CSV (`assets/tables/*.csv`)

## Structure du projet

```
Bloc_2_Steam/
├── notebooks/
│   ├── 01_steam_eda.ipynb              (EDA macro + genres + plateformes + prix)
│   └── 02_steam_stat_models.ipynb      (baseline règle + régression logistique MLlib)
├── dashboards/
│   └── STEAM_Project_Dashboard.lvdash.json  (à importer dans Databricks)
├── assets/
│   ├── 01_top_publishers.png … 07_success_by_platforms.png  (graphiques)
│   └── tables/*.csv                    (données des 6 tables du dashboard)
├── Presentation_Steam.pptx
└── README.md
```

## Comment rejouer le projet

**Option recommandée — Databricks Community** (gratuit) :
1. Créer un compte sur <https://community.cloud.databricks.com/>
2. Créer un cluster gratuit (single-node)
3. Importer les notebooks via "Workspace → Import" puis le dashboard via "Dashboards → Import"
4. Run all (les tables sont écrites dans le schéma `default` qui alimente le dashboard)

**Option locale** (Java 17-21 + Spark) — les notebooks détectent automatiquement l'absence de Databricks :
```bash
pip install pyspark==3.5.3 pandas matplotlib nbconvert ipykernel
# télécharger le dataset une fois :
#   https://full-stack-bigdata-datasets.s3.eu-west-3.amazonaws.com/Big_Data/Project_Steam/steam_game_output.json
export STEAM_JSON=/chemin/vers/steam_game_output.json   # sinon fallback S3
jupyter notebook   # puis Run All
```
En local, un `SparkSession local[*]` est créé automatiquement, `display()` rend des tables pandas
et les tables du dashboard sont exportées en CSV dans `assets/tables/` (le métastore Hive n'étant
pas requis hors Databricks).

## Auteur

[Aymeric Lahonde](https://github.com/aymericlahonde-dotcom) — promotion Jedha CDSD 2026.
