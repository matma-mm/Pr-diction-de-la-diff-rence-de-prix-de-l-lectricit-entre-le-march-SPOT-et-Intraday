# Prédiction de la différence de prix de l'électricité entre le marché SPOT et Intraday

## Description du projet

Ce projet vise à prédire la différence de prix de l'électricité entre le marché SPOT et le marché Intraday. Il utilise des données de séries temporelles couvrant la période de janvier 2022 à mars 2023.

## Objectif

Développer un modèle de régression capable de prédire le prix du marché SPOT en fonction de :
- La demande d'électricité prévue (load_forecast)
- La disponibilité des sources de production (charbon, gaz, nucléaire)
- Les prévisions de production d'énergie renouvelable (éolien, solaire)
- La variabilité des sources d'énergie renouvelable

## Données

### Variables disponibles

Le dataset contient 9 colonnes :

| Variable | Description |
|----------|-------------|
| load_forecast | Demande d'électricité prévue (MW) |
| coal_power_available | Puissance charbon disponible (MW) |
| gas_power_available | Puissance gaz disponible (MW) |
| nucelear_power_available | Puissance nucléaire disponible (MW) |
| wind_power_forecasts_average | Production éolienne moyenne prévue (MW) |
| solar_power_forecasts_average | Production solaire moyenne prévue (MW) |
| wind_power_forecasts_std | Écart-type de la prévision éolienne |
| solar_power_forecasts_std | Écart-type de la prévision solaire |
| predicted_spot_price | Prix SPOT (variable cible) |

### Période couverte

- Début : 1er janvier 2022
- Fin : 29 mars 2023
- Granularité : Données horaires (10 605 observations)
- Note : Les données manquantes (NaN) sont présentes notamment en début de période pour la variable predicted_spot_price

## Analyse exploratoire

### Corrélations

L'analyse de corrélation révèle que :

- Forte corrélation positive : load_forecast ↔ predicted_spot_price
- Corrélation modérée positive : nucelear_power_available ↔ predicted_spot_price
- Corrélation modérée positive : wind_power_forecasts_average ↔ predicted_spot_price

Les variables les plus pertinentes pour la prédiction du prix SPOT sont donc :

1. La demande prévue (load_forecast)
2. La disponibilité nucléaire
3. La production éolienne prévue

## Modèles de régression testés

### Régression linéaire

```
LinearRegression()
Score R² : 0.734 (73.4% de variance expliquée)
```

Performance : Performance de base, inadéquate pour capturer les relations complexes du système électrique.

### Random Forest Regressor

```
RandomForestRegressor(n_estimators=10)
Score R² : 0.961 (96.1% de variance expliquée)
```

Performance : Excellente, avec une amélioration massive par rapport à la régression linéaire. Le modèle capture efficacement les relations non-linéaires entre les variables.

### XGBoost (Gradient Boosting)

```
XGBRegressor()
Version : xgboost >= 2.1.3
```

Performance : À tester et comparer avec Random Forest. Ce modèle peut potentiellement offrir une meilleure généralization et une meilleure capture des interactions entre variables.

## Comparaison des modèles

| Modèle | Score R² | Avantages | Inconvénients |
|--------|----------|-----------|---------------|
| Régression linéaire | 0.734 | Simple, rapide | Peu de flexibilité |
| Random Forest | 0.961 | Excellent score, capture les non-linéarités | Risque de surapprentissage |
| XGBoost | À évaluer | Robustesse, régularisation intégrée | Plus complexe à tuner |

## Méthodologie

### Sélection des features

Basée sur l'analyse de corrélation, les 3 variables suivantes ont été sélectionnées :

- load_forecast
- nucelear_power_available
- wind_power_forecasts_average

### Préparation des données

1. Suppression des valeurs manquantes (NaN)
2. Conversion des types en float64
3. Filtrage des données à partir de 2023 (1 846 observations complètes)

### Entraînement du modèle

- Les modèles ont été entraînés sur 1 846 observations complètes (à partir de 2023)
- Les prédictions ont été générées pour l'ensemble du dataset (10 605 observations)

## Structure du projet

```
.
├── data_c.ipynb              # Notebook Jupyter avec l'analyse complète
├── X_train_Wwou3IE.csv       # Features d'entraînement (10 605 × 9)
├── y_train_jJtXgMX.csv       # Variable cible d'entraînement
├── X_test_GgyECq8.csv        # Features de test
└── README.md                 # Ce fichier
```

## Dépendances

```
pandas >= 2.2.2
numpy
scikit-learn
xgboost >= 2.1.3
matplotlib
seaborn
plotly
```

## Comment utiliser

### Charger les données

```python
import pandas as pd

df = pd.read_csv('X_train_Wwou3IE.csv', parse_dates=['DELIVERY_START'], index_col='DELIVERY_START')
y_train = pd.read_csv('y_train_jJtXgMX.csv', parse_dates=['DELIVERY_START'], index_col='DELIVERY_START')
X_test = pd.read_csv('X_test_GgyECq8.csv', parse_dates=['DELIVERY_START'], index_col='DELIVERY_START')
```

### Entraîner le modèle Random Forest

```python
from sklearn.ensemble import RandomForestRegressor

model = RandomForestRegressor(n_estimators=10)
X_features = df[['load_forecast', 'nucelear_power_available', 'wind_power_forecasts_average']]
model.fit(X_features, y_train)
```

### Entraîner le modèle XGBoost

```python
import xgboost as xgb

model_xgb = xgb.XGBRegressor(n_estimators=100, learning_rate=0.1, max_depth=5)
X_features = df[['load_forecast', 'nucelear_power_available', 'wind_power_forecasts_average']]
model_xgb.fit(X_features, y_train)
```

### Générer des prédictions

```python
# Random Forest
predictions_rf = model.predict(X_test[['load_forecast', 'nucelear_power_available', 'wind_power_forecasts_average']])

# XGBoost
predictions_xgb = model_xgb.predict(X_test[['load_forecast', 'nucelear_power_available', 'wind_power_forecasts_average']])
```

## Résultats et interprétations

- Le modèle Random Forest atteint un R² de 0.961, indiquant une excellente capacité de prédiction
- La demande prévue est le facteur le plus influent sur le prix SPOT
- Les sources d'énergie renouvelable (éolien, solaire) ont un impact secondaire mais significatif
- Les relations non-linéaires entre les variables justifient l'utilisation de modèles ensemble (Random Forest, XGBoost)

## Perspectives d'amélioration

- Ajouter des features temporelles (heure, jour de la semaine, saison)
- Appliquer l'hyperparamètre tuning sur XGBoost (learning_rate, max_depth, n_estimators)
- Utiliser des données plus récentes
- Implémenter une validation croisée pour évaluer la stabilité du modèle
- Analyser les résidus pour identifier les patterns manqués
- Développer des modèles spécifiques par saison
- Tester des modèles hybrides combinant Random Forest et XGBoost

## Notes

- Toutes les données sont en format UTC avec décalage horaire (+00:00)
- Les valeurs manquantes en début de période reflètent probablement un délai de disponibilité des données réelles
- Le modèle actuel ne tient pas compte de la saisonnalité explicite
- XGBoost offre généralement une meilleure régularisation mais nécessite un tuning des hyperparamètres

## Auteur

matma-mm

Dernière mise à jour : 14 juin 2026
