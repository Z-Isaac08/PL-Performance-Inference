# ⚽ Premier League Performance Inference (2019-2025)

## Objectif

L'objectif de ce projet est de réaliser une **analyse inférentielle** des facteurs déterminant le résultat d'un match de Premier League. Contrairement à une simple prédiction, nous cherchons ici à comprendre et quantifier l'influence de différents indicateurs techniques (Efficacité, Niveau théorique, Discipline, Forme) sur la probabilité de victoire, de nul ou de défaite.

## Structure du Projet
```text
.
├── data/               # Dossier pour les jeux de données (ex: Matches.csv)
│   └── .gitkeep        # Préserve le dossier dans Git
├── notebooks/          # Notebooks Jupyter/Colab d'analyse
│   └── PL_Performance_Inference_Logit.ipynb
├── output/             # Résultats générés (graphiques, résumés)
│   ├── .gitkeep
│   ├── coefficients_chart.png
│   └── model_summary.txt
├── venv/               # Environnement virtuel Python (local)
├── .gitignore          # Fichiers et dossiers à ignorer par Git
├── README.md           # Documentation du projet
└── requirements.txt    # Liste des dépendances Python
```

## Dataset

### Source

Le jeu de données provient de Kaggle : [Club Football Match Data (2000-2025)](https://www.kaggle.com/datasets/adamgbor/club-football-match-data-2000-2025). L'étude se concentre sur la période allant d'août 2019 à mai 2025, couvrant 3 308 matchs de Premier League.

### Description des features

Nous utilisons des **différentiels** (Domicile - Extérieur) pour chaque match :

- `Diff_Elo` : Écart de niveau théorique entre les deux équipes.
- `Diff_Target` : Écart de tirs cadrés durant le match.
- `Diff_Shots` : Écart du volume total de tirs.
- `Diff_Form5` : Différence de forme sur les 5 derniers matchs.
- `Diff_RedCards` : Écart de cartons rouges reçus.
- `Diff_Corners` : Écart de corners obtenus.

**Cible (Target) :** `Result_Code` (H: 2, D: 1, A: 0).

## Installation et Configuration

Pour reproduire cette analyse sur votre machine locale :

1.  **Cloner le dépôt :**

    ```bash
    git clone https://github.com/votre-pseudo/PL-Performance-Inference.git
    cd PL-Performance-Inference
    ```

2.  **Créer l'environnement virtuel :**

    ```bash
    python -m venv venv
    ```

3.  **Activer l'environnement :**
    - **Windows :** `.\venv\Scripts\activate`
    - **Linux/macOS :** `source venv/bin/activate`

4.  **Installer les dépendances :**
    ```bash
    pip install -r requirements.txt
    ```

## Utilisation et Environnements

Ce projet peut être exécuté dans plusieurs environnements selon vos préférences :

### 1. VS Code (Recommandé pour Windows)

- Ouvrez le dossier du projet dans **VS Code**.
- Installez l'extension **Jupyter** (si ce n'est pas déjà fait).
- Ouvrez le notebook `notebooks/PL_Performance_Inference_Logit.ipynb`.
- En haut à droite, cliquez sur **"Select Kernel"** et choisissez l'environnement virtuel situé dans `venv/`.

### 2. Jupyter Notebook / Lab

- Assurez-vous que votre environnement virtuel est activé.
- Lancez Jupyter :
  ```bash
  jupyter notebook
  ```
- Naviguez jusqu'au notebook et exécutez les cellules.

### 3. Google Colab

- Importez le notebook dans [Google Colab](https://colab.research.google.com/).
- **Données** : Téléchargez le fichier `Matches.csv` dans l'espace de stockage temporaire de Colab.
- **Correction de version** : Colab utilise souvent des versions récentes de `scikit-learn`. Si vous obtenez une erreur sur `multi_class='multinomial'`, retirez simplement cet argument de l'initialisation de `LogisticRegression` comme indiqué dans la méthodologie.

## Méthodologie

### Exploration

Analyse des corrélations et distribution des différentiels pour vérifier la pertinence des variables choisies.

### Nettoyage

- Filtrage des données sur la Premier League et les saisons récentes.
- Suppression des valeurs manquantes (`NaN`) sur les colonnes statistiques vitales.
- Mappage des résultats (Win/Draw/Loss) en codes numériques.

### Modèle choisi et justification

Nous avons choisi la **Régression Logistique Multinomiale**.

- **Pourquoi ?** Elle est idéale pour une cible à trois classes (Victoire, Nul, Défaite) et offre une excellente **explicabilité** via l'analyse des coefficients (Betas).
- **Approche :** Utilisation de `scikit-learn` pour l'entraînement et `statsmodels` pour obtenir des statistiques détaillées (P-values, Intervalles de confiance).

## Glossaire et Mathématiques

### Concepts clés

- **Standardisation (Z-score) :** $z = \frac{x - \mu}{\sigma}$. Permet de comparer des coefficients sur une même échelle (ex: comparer l'impact d'un point d'Elo avec celui d'un tir cadré).
- **Régression Logistique Multinomiale :** Le modèle estime la probabilité d'appartenance à chaque classe via une fonction Softmax.
- **Coefficient (Beta) :** Représente la direction et la force de l'influence d'une variable.
- **Odds Ratio (OR) :** $OR = e^{\beta}$. Il indique de combien les "chances" de victoire sont multipliées pour chaque augmentation d'une unité (Ecart-type) de la variable.
- **P-value :** Mesure la probabilité que l'effet observé soit dû au hasard. Si $P < 0.05$, l'impact est considéré comme statistiquement significatif.

## Résultats

### Métriques

- **Précision globale (Accuracy) :** ~57.85% (Test set).
- **Significativité :** La quasi-totalité des variables sont significatives ($P < 0.001$), sauf la forme récente.

### Visualisations

L'analyse des coefficients montre que :

1.  **Diff_Target (Efficacité) :** Coefficient le plus élevé (~1.08). Un écart positif de tirs cadrés multiplie les chances de victoire par environ **2.94**.
2.  **Diff_Elo (Niveau) :** Impact solide mais secondaire par rapport à l'efficacité directe sur le terrain ($OR \approx 1.30$).
3.  **Diff_Form5 (Mythe de la forme) :** $P = 0.838$. La forme sur 5 matchs n'a **aucune influence réelle** sur le résultat final du match étudié une fois les autres facteurs pris en compte.

## Conclusion

L'étude démontre que l'**efficacité immédiate** (tirs cadrés) est le prédicteur de succès le plus puissant, surpassant largement le prestige ou le niveau théorique (`Elo`). Elle tord également le cou à l'idée reçue de la "dynamique de forme", qui s'avère négligeable statistiquement face à la performance brute du jour J.

## Pistes d'amélioration

- **Expected Goals (xG) :** Intégrer la qualité des tirs plutôt que leur simple nombre cadré.
- **Données de tracking :** Distance parcourue, intensité du pressing (PPDA).
- **Modèles non-linéaires :** Explorer des modèles comme XGBoost tout en utilisant SHAP pour conserver l'explicabilité.
