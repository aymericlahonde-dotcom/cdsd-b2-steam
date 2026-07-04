# Préparation soutenance — Bloc 2 Steam (Big Data / PySpark)

> Certification Jedha CDSD — RNCP35288 — Bloc 2 : *Exploration, analyse et représentation des données* (Big Data distribué)

## 1. Pitch (30 secondes)

L'équipe data de Steam veut une analyse macro de son catalogue (55 691 jeux). Le dataset est un
**JSON imbriqué et volumineux**, donc je le traite en **PySpark** (Databricks ou Spark local).
Je nettoie et j'aplatis les champs imbriqués (genres, langues, plateformes), je construis des tables
d'agrégation qui alimentent un **dashboard Databricks**, et j'ajoute deux modèles simples pour
prédire le succès d'un jeu.

## 2. Chiffres à connaître par cœur

| Sujet | Résultat |
|---|---|
| Volume | **55 691** jeux |
| Top éditeur | Big Fish Games (422 jeux) — longue traîne |
| Pic de sorties | **2021 : 8 805 jeux** (explosion indé dès 2014) |
| Top genres | Indie 39,7k · Action 23,8k · Casual 22,1k · Adventure 21,4k |
| Prix | 14 % gratuits · médiane **4,99 €** · moyenne payante **8,99 €** |
| Plateformes | Windows **100 %** · Mac 22,9 % · Linux 15,2 % |
| Hits vs plateformes | 6,9 % (1 OS) → 11,5 % (2) → **17,9 % (3 OS)** |
| Hits vs prix | 3,5 % (<5 €) → **29,8 % (20-40 €)** |
| Modèle MLlib | Régression logistique **AUC ≈ 0,72** ; `nb_platforms` = 1ᵉʳ facteur |

*Définition d'un « hit » (`success_high`) : `n_reviews >= 500` **et** `positive_rate >= 0.80`.*

## 3. Compétences du bloc démontrées

- **Big Data distribué** : justification de Spark (JSON imbriqué volumineux, `explode` parallélisé,
  schéma variable). Traitement out-of-core, `spark.read.json(multiLine)`.
- **Nettoyage / feature engineering** : helpers de tolérance de schéma (`has_path`, `col_or_null`),
  prix en centimes → euros, parsing de dates `yyyy/MM/dd` (mode LEGACY), `popularity_score =
  log1p(n_reviews) × positive_rate`, `nb_platforms`, `success_high`.
- **Agrégations** : `groupBy` / `countDistinct` / `avg`, `explode` sur genres et langues.
- **Représentation** : dashboard Databricks + 7 graphiques matplotlib exportés.
- **Modélisation** : baseline « règle métier » (matrice de confusion) + régression logistique MLlib
  (`VectorAssembler`, `LogisticRegression`, `BinaryClassificationEvaluator`).

## 4. Questions probables du jury + réponses

**Pourquoi Spark et pas pandas ?**
Le JSON est imbriqué, le schéma varie d'un jeu à l'autre (des centaines de tags par jeu), et
l'`explode` des genres/langues multiplie les lignes. Spark gère ça de façon distribuée et out-of-core ;
le même code passe à l'échelle sur un vrai cluster. Sur ce snapshot (63 Mo) pandas suffirait, mais
la démarche cible un pipeline scalable.

**Comment gérez-vous le schéma variable du JSON ?**
Avec `has_path(schema, "data.platforms.windows")` qui vérifie l'existence d'un chemin imbriqué avant
de le lire, et `col_or_null` qui renvoie `NULL` si le champ manque. Aucune colonne codée en dur.

**Le prix, comment est-il traité ?**
Les champs `price`/`initialprice` sont en **centimes** (999 = 9,99 €) → division par 100. Piège que
j'ai corrigé après inspection du schéma.

**Comment définissez-vous le succès et n'y a-t-il pas de fuite de données ?**
`success_high` = jeu avec ≥ 500 reviews et ≥ 80 % d'avis positifs. Pour la régression logistique
je n'utilise que des variables **connues avant les reviews** (prix, remise, nb de plateformes,
année) pour éviter la fuite. L'AUC de 0,72 montre que ces signaux expliquent partiellement le succès,
mais le vrai moteur reste la qualité du jeu (non disponible à la sortie).

**Pourquoi la matrice de confusion de la baseline n'est pas parfaite ?**
La baseline prédit « hit si multi-plateforme (≥ 2 OS) ». C'est un signal faible mais réel
(precision 0,15 / recall 0,42) : les gros studios portent leurs hits sur plusieurs OS, mais beaucoup
de jeux multi-OS ne sont pas des hits.

**Le dashboard n'est pas accessible : comment montrez-vous les résultats ?**
J'ai exporté 7 graphiques matplotlib (`assets/*.png`) et les 6 tables du dashboard en CSV
(`assets/tables/`), tous versionnés dans le repo et repris dans la présentation.

**Limites du projet ?**
Snapshot statique (pas d'historisation), cluster community limité, pas de scheduling. En prod :
Delta Lake, Databricks Jobs, tests unitaires sur les fonctions de cleaning.

## 5. Déroulé de la démo (si demandée)

1. Ouvrir `01_steam_eda.ipynb` — montrer le schéma imbriqué, la table `games`, les agrégations et
   les graphiques déjà exécutés.
2. Ouvrir `02_steam_stat_models.ipynb` — matrice de confusion baseline + AUC de la régression.
3. Montrer le dashboard (`dashboards/STEAM_Project_Dashboard.lvdash.json`) et les CSV correspondants.
