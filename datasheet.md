# Datasheet — Adult Income enrichi (Athéna RH v1.0.0)

> Document accompagnant le dataset livré à Athéna RH.
> **Modèle Gebru et al. (2018), 7 sections, 2 pages max.**
> Signée binôme.

**Auteurs** : Jeremy, Nawelle
**Date** : 16/07/2026
**Version** : v1.0.0

## 1. Motivation

> Pourquoi ce dataset existe ? Qui l'a créé ?

- Ce dataset est le dataset RH de la boîte Athéna RH alimenté dans le but d'entraîner un modèle qui prédit le niveau de revenu d'une personne (>50K vs ≤50K) à partir de son profil professionnel, pour alimenter un outil d'aide à la décision RH.

## 2. Composition

> Combien d'observations, quelles colonnes, types, distribution cible,
> **variables sensibles signalées explicitement**, + le résumé du
> verdict éthique (DI les plus problématiques).

| Aspect | Valeur |
|---|---|
| Nombre de lignes | ... |
| Nombre de colonnes | 16 (14 features UCI + cible `income` + `manager_comments` synthétique) |
| Cible | `income` : `<=50K` / `>50K` |
| Distribution cible | ... |
| Variables sensibles | `sex`, `race`, `native_country`, `marital_status` |

**Schéma des colonnes** :

| Colonne | Type | Note |
|---|---|---|
| `age` | int | 17 — 90 |
| `workclass` | str | Statut de travail (9 modalités) |
| `education` | str | Diplôme (16 modalités) |
| `marital_status` | str | ⚠️ Sensible |
| `occupation` | str | Profession (15 modalités) |
| `relationship` | str | Position familiale (6 modalités) |
| `race` | str | ⚠️ Sensible (5 modalités) |
| `sex` | str | ⚠️ Sensible binaire |
| `capital_gain` / `capital_loss` | int | Très asymétriques (médiane 0) |
| `hours_per_week` | int | 1 — 99 |
| `native_country` | str | ⚠️ Sensible (40+ modalités) |
| `income` (cible) | str | `<=50K`, `>50K` |
| `manager_comments` | str | Texte libre **avec PII** — à anonymiser en async |

**Résumé verdict éthique** :
- DI le plus problématique : race (0.347) / sex (0.358)
- Intersectionnalités notables : Race X sex (cf schema dans le jupyter)
  Nous avons détecté un biais raciste en faveur des asiatiques / blancs (avec un DI de 0.347 (mais grosse disparité)) et un biais sexiste en faveur des hommes (avec un DI de 0.358). Les valeurs de DI sont particulièrement extrèmes la borne limite inférieure acceptable étant de 0.80.
  Nous avons aussi détecté que la variable marital_status est un proxy de la variable sex et que la variable race est un proxy de native_country.


## 3. Processus de collecte

> Origine UCI Adult Census 1994 + enrichissement Athéna RH 2026.

- ...

## 4. Preprocessing appliqué

> Ce que **votre binôme** a fait dans la phase sync.

- **Audit qualité express** (`df.info()` + `isna().sum()`) sur les 32 561 lignes × 16 colonnes : pas de doublons, pas de valeurs manquantes hors 3 colonnes catégorielles.

| Colonne | Manquants | % |
|---|---|---|
| `occupation` | 1 843 | 5,66 % |
| `workclass` | 1 836 | 5,64 % |
| `native_country` | 583 | 1,79 % |

- **Manquants — décision** : aucune imputation ni suppression de ligne appliquée à ce stade. Les `?` d'origine UCI (recodés en `NaN`) recoupent largement les mêmes individus pour `workclass`/`occupation` (profils hors emploi ou non déclarés) : les supprimer biaiserait le dataset vers les actifs déclarés, et les imputer (mode/valeur la plus fréquente) créerait un signal artificiel sur une variable corrélée au revenu. Décision : conserver les lignes telles quelles et documenter le manquant comme catégorie explicite (`"Unknown"`) à la charge du modélisateur en aval. Pour `native_country`, même logique — supprimer ces lignes réduirait justement le sous-groupe `non-USA` déjà mesuré comme désavantagé dans l'audit DI (§2), ce qui fausserait la suite.
- **Colonnes dérivées d'audit** (`native_us` : USA/non-USA, `sex_race` : intersection sexe×race) : créées uniquement pour le calcul des DI et des visualisations, **non conservées** dans le jeu de données livré.
- **`manager_comments`** : PII identifiées (noms, dates, éléments nominatifs) lors du survol express (§5 du notebook), mais **aucune anonymisation appliquée ici** — traitement reporté à la phase async dédiée (spaCy NER / Presidio), conformément au périmètre de ce notebook sync.
- **Colonnes retirées du jeu de features pour la modélisation** — conservées dans le fichier brut livré (traçabilité), mais à exclure des features d'un futur modèle :

| Colonne | Raison du retrait |
|---|---|
| `sex` | ⚠️ Variable sensible directe (DI = 0,358) |
| `race` | ⚠️ Variable sensible directe (DI = 0,347) |
| `marital_status` | ⚠️ Proxy identifié de `sex` (cf §2) |
| `native_country` | ⚠️ Variable sensible + proxy de `race` (cf §2) |
| `relationship` | Fortement corrélée à `marital_status`/`sex` (position familiale) |
| `capital_gain` / `capital_loss` | Très asymétriques (médiane 0, <10 % de valeurs non nulles) et corrélées aux inégalités de patrimoine déjà captées par `race`/`sex` |
| `education` | Redondante avec `education_num` (même information, encodage ordinal) |
| `fnlwgt` | Poids d'échantillonnage du recensement US, sans valeur prédictive au niveau individu |
| `manager_comments` | Texte libre avec PII non anonymisées à ce stade |

  Features conservées pour un futur modèle : `age`, `workclass`, `occupation`, `education_num`, `hours_per_week` (+ `income` comme cible).

## 5. Usages prévus / à éviter

**Usages prévus** :
- ...

**Usages à éviter** :
- ...

## 6. Distribution

- Destinataire : Athéna RH (Laurence Béthencourt, DPO)
- Format : Parquet snappy
- Conditions : ...

## 7. Maintenance

- Mainteneur·euses : Jeremy, Nawelle
- Version : v1.0.0 — 16/07/2026
- Signaler un problème : ...

---

*Datasheet produite en binôme dans le cadre du brief M2-B2 ATOS.*