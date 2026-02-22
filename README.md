# XML_PROJECT


# Construction Automatique de Graphes de Connaissances à partir de Données Tabulaires

> **Projet de Master — Ingénierie des Systèmes Intelligents**  
> Faculté des Sciences de Kénitra — Université Ibn Tofail  
> **Auteurs :** Basma Essabri & Alaa Eddine Haddout  
> **Année :** 2025–2026

---

## Table des Matières

1. [Description du Projet](#description-du-projet)
2. [Architecture du Système](#architecture-du-système)
3. [Prérequis et Installation](#prérequis-et-installation)
4. [Structure des Fichiers](#structure-des-fichiers)
5. [Étapes d'Exécution](#étapes-dexécution)
6. [Description Détaillée du Code](#description-détaillée-du-code)
7. [Tests Effectués](#tests-effectués)
8. [Résultats Obtenus](#résultats-obtenus)
9. [Fichiers Générés](#fichiers-générés)
10. [Requêtes SPARQL et Résultats](#requêtes-sparql-et-résultats)
11. [Comparaison avec R2RML Classique](#comparaison-avec-r2rml-classique)
12. [Publication LOD](#publication-lod)
13. [Dépannage](#dépannage)

---

## Description du Projet

Ce projet implémente un **pipeline complet et automatisé** pour transformer un jeu de données tabulaires (CSV) en un **graphe de connaissances RDF enrichi**. L'approche dépasse le simple mapping R2RML en intégrant des algorithmes d'apprentissage automatique (similarité cosinus, K-Means, DBSCAN) pour **découvrir automatiquement des relations implicites et latentes** entre les entités.

### Jeu de Données

- **Fichier :** `students.csv`
- **Taille :** 2 392 étudiants × 15 attributs
- **Colonnes :**

| Colonne | Type | Description |
|---|---|---|
| `StudentID` | int | Identifiant unique (1001–3392) |
| `Age` | int | Âge de l'étudiant (15–18 ans) |
| `Gender` | int | Genre (0=Féminin, 1=Masculin) |
| `Ethnicity` | int | Groupe ethnique (0–3) |
| `ParentalEducation` | int | Niveau d'éducation parental (0–4) |
| `StudyTimeWeekly` | float | Heures d'étude hebdomadaires (0–20h) |
| `Absences` | int | Nombre d'absences (0–30) |
| `Tutoring` | int | Soutien scolaire (0/1) |
| `ParentalSupport` | int | Soutien parental (0–4) |
| `Extracurricular` | int | Activités parascolaires (0/1) |
| `Sports` | int | Pratique sportive (0/1) |
| `Music` | int | Pratique musicale (0/1) |
| `Volunteering` | int | Bénévolat (0/1) |
| `GPA` | float | Moyenne académique (0.0–4.0) |
| `GradeClass` | float | Classe de note (0=A, 4=F) |

---

## Architecture du Système

```
students.csv
     │
     ▼
┌─────────────────────────────────────────────┐
│         COUCHE DONNÉES                      │
│   Pandas DataFrame (2392 × 15)              │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│         COUCHE TRAITEMENT                   │
│   StandardScaler → X_scaled (2392 × 13)     │
└──────┬────────────────┬─────────────────────┘
       │                │
       ▼                ▼
┌──────────────┐  ┌─────────────────────────────┐
│  COUCHE ML   │  │    COUCHE ONTOLOGIQUE        │
│              │  │                              │
│ • cosine_    │  │  FOAF + Schema.org +         │
│   similarity │  │  Dublin Core + OWL           │
│ • K-Means    │  │  (équivalences OWL)          │
│   (K=4)      │  └────────────┬─────────────────┘
│ • DBSCAN     │               │
│   (ε=0.5)    │               │
└──────┬───────┘               │
       │                       │
       └───────────┬───────────┘
                   ▼
┌─────────────────────────────────────────────┐
│         COUCHE RDF (rdflib Graph)           │
│   1 715 146 triplets RDF                    │
└──────────────────┬──────────────────────────┘
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
  Turtle (.ttl)  N-Triples  JSON-LD
                   │
                   ▼
           VoID (LOD metadata)
```

---

## Prérequis et Installation

### Python

```bash
Python >= 3.8
```

### Installation des dépendances

```bash
pip install pandas numpy rdflib scikit-learn matplotlib networkx seaborn
```

Ou via conda :

```bash
conda install pandas numpy scikit-learn matplotlib networkx seaborn
pip install rdflib
```

### Versions utilisées (environnement de développement)

| Bibliothèque | Version |
|---|---|
| Python | 3.12.7 |
| pandas | 2.x |
| numpy | 1.x / 2.x |
| rdflib | 7.x |
| scikit-learn | 1.x |
| matplotlib | 3.x |
| networkx | 3.x |
| seaborn | 0.x |

---

## 📁 Structure des Fichiers

```
projet/
│
├── xxmmll.ipynb                  ← Notebook principal (code complet)
├── students.csv                  ← Jeu de données source (REQUIS)
│
├── knowledge_graph.ttl           ← Graphe RDF (format Turtle)     [GÉNÉRÉ]
├── knowledge_graph.nt            ← Graphe RDF (format N-Triples)  [GÉNÉRÉ]
├── knowledge_graph_sample.jsonld ← Sous-graphe JSON-LD (50 étudiants) [GÉNÉRÉ]
├── r2rml_classic.ttl             ← Mapping R2RML classique         [GÉNÉRÉ]
├── void_metadata.ttl             ← Métadonnées VoID pour LOD       [GÉNÉRÉ]
├── publication_lod_instructions.txt ← Instructions SPARQL endpoint [GÉNÉRÉ]
│
├── knowledge_graph_with_implicit_relations.png  ← Visualisation graphe [GÉNÉRÉ]
│
└── README.md                     ← Ce fichier
```

> ⚠️ **Important :** Le fichier `students.csv` doit être placé dans le même répertoire que le notebook avant toute exécution.

---

##  Étapes d'Exécution

### Méthode 1 : Exécution via Jupyter Notebook (recommandée)

```bash
# 1. Lancer Jupyter
jupyter notebook

# 2. Ouvrir xxmmll.ipynb

# 3. Exécuter toutes les cellules dans l'ordre :
#    Kernel → Restart & Run All
```

### Méthode 2 : Exécution cellule par cellule

Exécuter les cellules **dans l'ordre numérique** suivant :

---

### Cellule 1 — Imports

```python
import pandas as pd
import numpy as np
from rdflib import Graph, URIRef, Literal, Namespace, RDF, RDFS, XSD
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans, DBSCAN
from sklearn.metrics.pairwise import cosine_similarity
import matplotlib.pyplot as plt
import networkx as nx
import seaborn as sns
```

**✅ Sortie attendue :**
```
Imports réussis
```

---

### Cellule 2 — Chargement des données

```python
file_path = "students.csv"
df = pd.read_csv(file_path)
```

**✅ Sortie attendue :**
```
Dataset chargé: 2392 étudiants

Colonnes disponibles: ['StudentID', 'Age', 'Gender', 'Ethnicity',
'ParentalEducation', 'StudyTimeWeekly', 'Absences', 'Tutoring',
'ParentalSupport', 'Extracurricular', 'Sports', 'Music',
'Volunteering', 'GPA', 'GradeClass']
```

**⏱Temps estimé :** < 1 seconde

---

### Cellule 3 — Initialisation du graphe RDF et alignement ontologique

Initialise le graphe `rdflib.Graph()` avec 7 namespaces et déclare les alignements OWL (`owl:equivalentClass`, `rdfs:subClassOf`, `owl:equivalentProperty`).

**✅ Sortie attendue :**
```
Graphe RDF initialisé avec namespaces enrichis et alignements ontologiques
   Ontologies alignées: FOAF, Schema.org, Dublin Core, OWL
```

---

### Cellule 4 — Mapping de base CSV → RDF

Convertit chaque ligne CSV en triplets RDF. Chaque étudiant reçoit 3 types (`schema:Person`, `foaf:Person`, `edu:Student`) et 15+ propriétés.

**✅ Sortie attendue :**
```
Mapping de base terminé: 2392 étudiants convertis en RDF
   Nombre de triplets RDF de base: 45 457
```

**⏱Temps estimé :** 5–15 secondes

---

### Cellule 5 — Normalisation des features

StandardScaler sur 13 colonnes numériques pour préparer les données ML.

**✅ Sortie attendue :**
```
Features normalisées pour l'analyse de similarité
```

---

### Cellule 6 — Relation 1 : Similarité de profil (cosine similarity)

Calcule la matrice de similarité cosinus (2392×2392), détermine le seuil au 97ème percentile (τ = 0.569), crée les triplets `rel:hasSimilarProfile`.

**✅ Sortie attendue :**
```
=== Découverte de relations de similarité ===
 85 790 relations de similarité détectées (seuil: 0.569)
```

**⏱Temps estimé :** 30–90 secondes (calcul matriciel intensif)

---

### Cellule 7 — Relation 2 : Clustering de performance (K-Means)

K-Means avec K=4 sur {GPA, StudyTimeWeekly, Absences, Tutoring}. Les clusters sont triés par GPA moyen et étiquetés sémantiquement.

**✅ Sortie attendue :**
```
=== Clustering par performance académique ===
 4 groupes de performance créés (nommage corrigé)
  - Groupe 'NeedsSupport': 844 étudiants (GPA moyen: 1.06)
  - Groupe 'Average':      374 étudiants (GPA moyen: 1.38)
  - Groupe 'Good':         827 étudiants (GPA moyen: 2.59)
  - Groupe 'Excellent':    347 étudiants (GPA moyen: 2.90)
```

---

### Cellule 8 — Relation 3 : Groupes d'activités (DBSCAN)

DBSCAN avec ε=0.5 et MinPts=5 sur les 4 features binaires d'activités. Les clusters sont nommés selon le profil dominant (Athletes, Musicians, Volunteers, ActiveStudents).

**✅ Sortie attendue :**
```
=== Découverte de groupes d'activités ===
 16 groupes d'activités identifiés
```

---

### Cellule 9 — Relation 4 : Mentorat potentiel

Croise les étudiants à GPA ≥ P80 (479 mentors potentiels) avec les étudiants à GPA ≤ P20 (479 étudiants à soutenir), filtrés par compatibilité d'âge et de contexte familial.

**✅ Sortie attendue :**
```
=== Détection de relations de mentorat potentiel ===
 104 071 relations de mentorat potentiel identifiées
   - 479 mentors potentiels (GPA >= 2.77)
   - 479 étudiants à soutenir (GPA <= 1.01)
```

**⏱Temps estimé :** 1–5 minutes (double boucle 479×479)

---

### Cellule 10 — Relation 5 : Corrélations causales

Calcule la matrice de corrélation de Pearson et encode les facteurs avec |r| > 0.3.

**✅ Sortie attendue :**
```
=== Analyse des corrélations causales ===
Facteurs fortement corrélés au GPA (|r| > 0.3):
  - Absences: -0.919

 1 relations causales identifiées
```

---

### Cellule 11 — Relation 6 : Influence entre pairs

Détecte les paires d'étudiants partageant les mêmes activités et support parental similaire (|ΔPS| ≤ 1) avec un écart de GPA > 1.0.

**✅ Sortie attendue :**
```
=== Détection d'influence entre pairs ===
 311 000 relations d'influence entre pairs détectées
```

**⏱Temps estimé :** 5–20 minutes (boucle 2392×2392)

---

### Cellule 12 — Statistiques finales du graphe

**✅ Sortie attendue :**
```
============================================================
STATISTIQUES DU GRAPHE DE CONNAISSANCES ENRICHI
============================================================

Nombre total de triplets RDF: 1 715 146

Types de relations découvertes (top 15):
  sharesActivitiesWith:       866 556
  hasSimilarProfile:          171 580
  mayPositivelyInfluence:     170 350
  mayBeInfluencedBy:          170 350
  couldMentor:                104 071
  couldBeMentoredBy:          104 071
  similarityScore:             76 947
  22-rdf-syntax-ns#type:        8 146
  identifier:                   4 784
  tutoring:                     2 392
  absences:                     2 392
  gradeClass:                   2 392
  parentalSupport:              2 392
  music:                        2 392
  belongsToActivityGroup:       2 392

 Relations explicites (mapping direct): 123 383
 Relations implicites (découvertes):  1 591 763
 Enrichissement du graphe:              +1 290.1%
```

---

### Cellule 13 — Export RDF multi-formats

**✅ Sortie attendue :**
```
=== Export du Graphe RDF ===
 Graphe exporté en Turtle     → knowledge_graph.ttl
 Graphe exporté en N-Triples  → knowledge_graph.nt
Graphe exporté en JSON-LD    → knowledge_graph_sample.jsonld
   (sous-graphe de 50 étudiants, 39 246 triplets)
```

**⏱Temps estimé :** 2–5 minutes (fichier Turtle > 100 Mo)

---

### Cellule 14 — Requêtes SPARQL (4 requêtes)

Exécution de 4 requêtes SPARQL sur le graphe (voir section dédiée ci-dessous).

---

### Cellule 15 — Visualisation du graphe (NetworkX)

Génère une visualisation du sous-graphe des 100 premiers étudiants avec coloration des arêtes par type de relation.

**✅ Sortie attendue :**
```
Création de la visualisation du graphe...
 Visualisation sauvegardée: knowledge_graph_with_implicit_relations.png
   Nœuds: 100, Arêtes: 1 014
```

---

### Cellule 16 — Heatmap des corrélations

```
Heatmap des corrélations sauvegardée
```

---

### Cellule 17 — Comparaison R2RML vs Notre approche

---

### Cellule 18 — Publication LOD (VoID)

**✅ Sortie attendue :**
```
=== Publication LOD (Linked Open Data) ===
 Métadonnées VoID générées     → void_metadata.ttl

 Instructions de publication LOD → publication_lod_instructions.txt

 RÉCAPITULATIF DES FICHIERS GÉNÉRÉS:
   knowledge_graph.ttl           - Graphe RDF principal (Turtle)
   knowledge_graph.nt            - Graphe RDF (N-Triples)
   knowledge_graph.jsonld        - Graphe RDF (JSON-LD pour APIs web)
   r2rml_classic.ttl             - Mapping R2RML classique (comparaison)
   void_metadata.ttl             - Métadonnées VoID pour LOD
   publication_lod_instructions.txt - Instructions SPARQL endpoint
```

---

## Tests Effectués

### Test 1 — Intégrité du chargement des données

| Vérification | Attendu | Obtenu | Statut |
|---|---|---|---|
| Nombre de lignes | 2 392 | 2 392 | ✅ |
| Nombre de colonnes | 15 | 15 | ✅ |
| Valeurs manquantes | 0 | 0 | ✅ |
| Type `StudentID` | int | int64 | ✅ |
| Plage GPA | [0.0, 4.0] | [0.0, 4.0] | ✅ |

---

### Test 2 — Mapping RDF de base

| Vérification | Attendu | Obtenu | Statut |
|---|---|---|---|
| Triplets après mapping base | > 40 000 | 45 457 | ✅ |
| Chaque étudiant a 3 types RDF | Oui | Oui | ✅ |
| Propriétés FOAF correctement liées | Oui | Oui | ✅ |
| Alignement `owl:equivalentClass` présent | Oui | Oui | ✅ |

---

### Test 3 — Similarité cosinus

| Vérification | Attendu | Obtenu | Statut |
|---|---|---|---|
| Dimension matrice similarité | 2392×2392 | 2392×2392 | ✅ |
| Seuil τ (97ème percentile) | ~0.5–0.7 | 0.569 | ✅ |
| Nombre de paires similaires | > 0 | 85 790 | ✅ |
| Symétrie des relations | Oui | Oui | ✅ |

---

### Test 4 — Clustering K-Means (K=4)

| Groupe | Étudiants | GPA moyen | Absences moyennes | Statut |
|---|---|---|---|---|
| NeedsSupport | 844 | 1.06 | ~24.3 | ✅ |
| Average | 374 | 1.38 | ~18.7 | ✅ |
| Good | 827 | 2.59 | ~8.2 | ✅ |
| Excellent | 347 | 2.90 | ~4.1 | ✅ |
| **Total** | **2 392** | — | — | ✅ |

**Validation :** La somme des étudiants dans les 4 clusters = 2 392 ✅ (aucun étudiant non assigné, K-Means ne génère pas de bruit).

---

### Test 5 — Clustering DBSCAN

| Paramètre | Valeur |
|---|---|
| ε (epsilon) | 0.5 |
| min_samples | 5 |
| Nombre de clusters trouvés | 16 |
| Points classés comme bruit (-1) | Exclus du graphe |
| Profils nommés détectés | Athletes, Musicians, Volunteers, ActiveStudents |

---

### Test 6 — Relations de mentorat

| Vérification | Résultat | Statut |
|---|---|---|
| Mentors potentiels (GPA ≥ P80) | 479 étudiants | ✅ |
| Étudiants à soutenir (GPA ≤ P20) | 479 étudiants | ✅ |
| Seuil P80 | GPA ≥ 2.77 | ✅ |
| Seuil P20 | GPA ≤ 1.01 | ✅ |
| Paires mentor-mentoré créées | 104 071 | ✅ |
| Critère âge respecté (|Δ| ≤ 1) | Oui | ✅ |
| Critère PE respecté (|Δ| ≤ 1) | Oui | ✅ |

---

### Test 7 — Corrélations causales

| Facteur | Corrélation de Pearson | Direction | Encodé dans KG | Statut |
|---|---|---|---|---|
| Absences | **−0.919** | Négative | `rel:negativelyInfluences` | ✅ |
| StudyTimeWeekly | +0.18 | — | Non (< 0.3) | ✅ |
| ParentalSupport | +0.20 | — | Non (< 0.3) | ✅ |

**Interprétation :** Les absences sont le **seul prédicteur fort** (|r| > 0.3) du GPA dans ce jeu de données, avec une corrélation négative quasi-linéaire très forte (r = −0.919).

---

### Test 8 — Intégrité du graphe final

| Métrique | Valeur | Statut |
|---|---|---|
| Triplets totaux | 1 715 146 | ✅ |
| Triplets explicites | 123 383 | ✅ |
| Triplets implicites (ML) | 1 591 763 | ✅ |
| Ratio implicites/explicites | 12.9× | ✅ |
| Graphe sérialisable en Turtle | Oui | ✅ |
| Graphe sérialisable en N-Triples | Oui | ✅ |
| Graphe sérialisable en JSON-LD | Oui | ✅ |

---

### Test 9 — Requêtes SPARQL

Les 4 requêtes SPARQL ont été exécutées sur le graphe et ont retourné des résultats cohérents (voir section suivante).

---

### Test 10 — Visualisation NetworkX

| Vérification | Résultat |
|---|---|
| Sous-graphe (100 étudiants) construit | ✅ |
| Nombre de nœuds | 100 |
| Nombre d'arêtes | 1 014 |
| Coloration par type de relation | ✅ Bleu/Vert/Rouge/Orange |
| Image PNG générée | ✅ |

---

## 📊 Résultats Obtenus

### Résultat Principal : Enrichissement du Graphe

```
┌─────────────────────────────────────────────────────────────┐
│           BILAN DU GRAPHE DE CONNAISSANCES                 │
├─────────────────────────────┬───────────────────────────────┤
│ Entités (étudiants)         │                         2 392 │
│ Triplets totaux RDF         │                     1 715 146 │
│ Relations explicites        │                       123 383 │
│ Relations implicites (ML)   │                     1 591 763 │
│ Enrichissement vs R2RML     │                    +1 290.1 % │
└─────────────────────────────┴───────────────────────────────┘
```

### Distribution des Relations

| Type de Relation | Prédicat RDF | Nombre | % du total |
|---|---|---|---|
| Partage d'activités | `rel:sharesActivitiesWith` | 866 556 | 50.5% |
| Profil similaire | `rel:hasSimilarProfile` | 171 580 | 10.0% |
| Influence positive | `rel:mayPositivelyInfluence` | 170 350 | 9.9% |
| Peut être influencé | `rel:mayBeInfluencedBy` | 170 350 | 9.9% |
| Peut mentorer | `rel:couldMentor` | 104 071 | 6.1% |
| Peut être mentoré | `rel:couldBeMentoredBy` | 104 071 | 6.1% |
| Score de similarité | `rel:similarityScore` | 76 947 | 4.5% |
| Types RDF | `rdf:type` | 8 146 | 0.5% |
| Propriétés factuelles | (divers) | ~43 075 | 2.5% |

### Groupes de Performance (K-Means)

```
NeedsSupport  ████████████████████████████  844 étudiants  GPA: 1.06
Average       ██████████████               374 étudiants  GPA: 1.38
Good          ████████████████████████████ 827 étudiants  GPA: 2.59
Excellent     █████████████                347 étudiants  GPA: 2.90
```

### Corrélation GPA vs Absences

```
r = -0.919  ← Corrélation forte négative

GPA  4.0 │  ●
         │  ● ●
     3.0 │    ● ●●
         │      ● ● ●
     2.0 │         ● ●●
         │            ●●
     1.0 │              ● ●
         │                ● ●
     0.0 └──────────────────────── Absences
         0    5    10   15   20   25   30
```

---

## Requêtes SPARQL et Résultats

### Requête 1 — Mentors avec GPA > 3.0 pour étudiants avec GPA < 1.5

```sparql
PREFIX edu: <http://example.org/education/>
PREFIX rel: <http://example.org/relations/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?mentor ?mentee ?mentorGPA ?menteeGPA
WHERE {
    ?mentor rel:couldMentor ?mentee .
    ?mentor edu:gpa ?mentorGPA .
    ?mentee edu:gpa ?menteeGPA .
    FILTER(?mentorGPA > 3.0 && ?menteeGPA < 1.5)
}
ORDER BY DESC(?mentorGPA) ASC(?menteeGPA)
LIMIT 10
```

**Résultats (10 premiers) :**

| Mentor ID | Mentee ID | GPA Mentor | GPA Mentee |
|---|---|---|---|
| 1045 | 1475 | 4.000 | 0.000 |
| 2706 | 1475 | 4.000 | 0.000 |
| 3320 | 1475 | 4.000 | 0.000 |
| 1443 | 2287 | 4.000 | 0.000 |
| 2279 | 2287 | 4.000 | 0.000 |
| 3320 | 2287 | 4.000 | 0.000 |
| 1045 | 2501 | 4.000 | 0.000 |
| 2706 | 2501 | 4.000 | 0.000 |
| 3029 | 2501 | 4.000 | 0.000 |
| 3320 | 2501 | 4.000 | 0.000 |

---

### Requête 2 — Étudiants "Excellent" avec plus de 5 absences

```sparql
PREFIX edu: <http://example.org/education/>

SELECT ?student ?gpa ?absences ?studyTime
WHERE {
    ?student edu:belongsToPerformanceGroup
             edu:PerformanceGroup/Excellent .
    ?student edu:gpa ?gpa .
    ?student edu:absences ?absences .
    ?student edu:studyTimeWeekly ?studyTime .
    FILTER(?absences > 5)
}
ORDER BY DESC(?absences)
LIMIT 10
```

**Résultats (10 premiers) :**

| Étudiant | GPA | Absences | Temps Étude (h) |
|---|---|---|---|
| 1156 | 2.631 | 16 | 17.23 |
| 1651 | 2.424 | 16 | 18.88 |
| 2407 | 2.435 | 16 | 17.70 |
| 3328 | 2.288 | 16 | 18.58 |
| 1167 | 2.538 | 15 | 13.88 |
| 1487 | 2.222 | 15 | 15.56 |
| 1514 | 2.354 | 15 | 16.76 |
| 1518 | 2.644 | 15 | 11.86 |
| 1910 | 2.247 | 15 | 13.21 |
| 2110 | 2.385 | 15 | 16.95 |

**Interprétation :** Ces étudiants classifiés "Excellent" (malgré des absences élevées) compensent par un temps d'étude hebdomadaire très important (11–19h/semaine), ce qui illustre la richesse sémantique du graphe.

---

### Requête 3 — Distribution par groupe de performance

```sparql
PREFIX edu: <http://example.org/education/>

SELECT ?groupe (COUNT(?student) AS ?nb)
WHERE {
    ?student edu:belongsToPerformanceGroup ?groupe .
}
GROUP BY ?groupe
ORDER BY DESC(?nb)
```

**Résultats :**

| Groupe | Nb Étudiants |
|---|---|
| NeedsSupport | 844 |
| Good | 827 |
| Average | 374 |
| Excellent | 347 |

---

### Requête 4 — Facteurs influençant négativement le GPA

```sparql
PREFIX rel: <http://example.org/relations/>
PREFIX edu: <http://example.org/education/>

SELECT ?facteur ?force
WHERE {
    ?facteur rel:negativelyInfluences edu:GPA .
    ?facteur rel:correlationStrength ?force .
}
ORDER BY DESC(?force)
```

**Résultats :**

| Facteur | Force de corrélation |
|---|---|
| Absences | 0.9193 |

---

## 🔄 Comparaison avec R2RML Classique

| Critère | R2RML Classique | Notre Approche |
|---|---|---|
| **Nombre de triplets RDF** | 38 272 | **1 715 146** |
| **Relations inter-étudiants** | Aucune | 500 861+ |
| **Clusters détectés** | Aucun | 4 perf. + 16 activités |
| **Relations de mentorat** | Non | 104 071 paires |
| **Influence entre pairs** | Non | 311 000 relations |
| **Alignement ontologies** | Partiel (manuel) | FOAF + Schema.org + DC + OWL |
| **Requêtes SPARQL enrichies** | Basiques uniquement | 4 types complexes |
| **Découverte automatique** | Non (mapping manuel) | Oui (cosine, KMeans, DBSCAN) |
| **Machine Learning intégré** | Non | Oui (sklearn) |
| **Formats d'export** | Turtle, N-Triples | Turtle, N-Triples, JSON-LD |
| **Publication LOD** | Non | VoID + SPARQL endpoint |
| **Ratio d'enrichissement** | — | **+1 290.1%** |

---

## Publication LOD

Le graphe est prêt pour publication sur le **Web des Données Liées** :

### Option 1 — Apache Jena Fuseki (endpoint local)

```bash
# Télécharger : https://jena.apache.org/
./fuseki-server --update --mem /students

# Charger le graphe
curl -X POST --data-binary @knowledge_graph.ttl \
     --header 'Content-Type: text/turtle' \
     http://localhost:3030/students/data

# Endpoint SPARQL disponible sur :
# http://localhost:3030/students/sparql
```

### Option 2 — Virtuoso Open Source

```sql
-- Via isql :
LOAD 'knowledge_graph.ttl' into <http://example.org/students>
```

### Option 3 — GraphDB (Ontotext)

1. Télécharger GraphDB Free : https://www.ontotext.com/products/graphdb/
2. Créer un repository `students`
3. Importer `knowledge_graph.ttl` via l'interface web

### Fichiers à publier

| Fichier | Usage |
|---|---|
| `knowledge_graph.ttl` | Données principales (Turtle) |
| `knowledge_graph.jsonld` | APIs REST / JSON consumers |
| `void_metadata.ttl` | Métadonnées dataset pour LOD |

---

## Dépannage

### ❌ `FileNotFoundError: students.csv`

**Solution :** Placer `students.csv` dans le même répertoire que le notebook, ou modifier la variable :
```python
file_path = "/chemin/complet/vers/students.csv"
```

### ❌ `ModuleNotFoundError: No module named 'rdflib'`

```bash
pip install rdflib
```

### ⚠️ Exécution très lente (cellule similarité ou influence)

- La cellule de similarité cosinus prend 30–90s : normale (matrice 2392×2392)
- La cellule d'influence entre pairs prend 5–20 min : normale (double boucle n²)
- Réduire `df` à un sous-ensemble pour des tests rapides :
  ```python
  df = df.head(200)  # test sur 200 étudiants
  ```

### ❌ `MemoryError` lors de l'export Turtle

Le fichier Turtle généré peut dépasser 150 Mo. S'assurer d'avoir au moins 2 Go de RAM libre.

### ⚠️ Résultats SPARQL vides

S'assurer que toutes les cellules ont été exécutées **dans l'ordre** avant d'exécuter les requêtes SPARQL. Le graphe `g` doit contenir 1 715 146 triplets.

---

## Temps d'Exécution Estimés

| Cellule | Opération | Temps estimé |
|---|---|---|
| 1–3 | Imports + chargement + init | < 5 s |
| 4 | Mapping RDF de base | 5–15 s |
| 5 | Normalisation | < 1 s |
| 6 | Similarité cosinus | 30–90 s |
| 7 | K-Means | < 5 s |
| 8 | DBSCAN | < 5 s |
| 9 | Relations de mentorat | 1–5 min |
| 10 | Corrélations | < 1 s |
| 11 | Influence entre pairs | 5–20 min |
| 12–13 | Stats + export RDF | 2–5 min |
| 14 | Requêtes SPARQL | < 30 s |
| 15–18 | Visu + comparaison + VoID | < 2 min |
| **Total** | | **~30–45 min** |

---

## 👥 Auteurs

| Nom | Formation | Établissement |
|---|---|---|
| **Basma Essabri** | Master ISI | Faculté des Sciences, Université Ibn Tofail, Kénitra |
| **Alaa Eddine Haddout** | Master ISI | Faculté des Sciences, Université Ibn Tofail, Kénitra |

---

## 📄 Licence

Ce projet est réalisé dans un cadre académique. Le jeu de données `students.csv` est utilisé à des fins de recherche et d'enseignement uniquement.

---

*README faite le 18 Février 2026*
