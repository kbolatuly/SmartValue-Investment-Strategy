# Smart Value — Stratégie Value-HMM Europe

## Le problème

Vous êtes gérant de portefeuille en 2007. Vous êtes convaincu que la prime value existe ; elle est robuste, documentée, répliquée sur un siècle de données. Vous investissez. Treize ans plus tard, vous sous-performez encore. C'est la **Value Lost Decade** : taux ultra-bas qui favorisent la croissance, montée des méga-caps tech, ratio Book-to-Market de moins en moins pertinent dans une économie d'actifs incorporels.

La prime value n'est pas morte. Mais l'approche passive a montré ses limites.

## La stratégie

Un processus systématique qui capture la prime value tout en évitant ses pièges.

---

## Architecture de la stratégie

### 1. Signal Value — 4 ratios, ranking intra-secteur
```python
# composite_score = 0.35 * vscore + 0.15 * vscore_sector + 0.25 * fscore + 0.15 * gp_at + 0.10 * accruals
# vscore = moyenne(P/B, EV/EBIT, P/FCF, P/Sales) — ranking intra-secteur GICS
```
Ne pas se contenter du B/M de Fama-French. Quatre ratios complémentaires, comparés au sein du même secteur pour ne pas mélanger une banque avec une tech.

### 2. Filtre Qualité — F-Score de Piotroski (9 critères)
```python
# Seuil : fscore_min = 7 pour le core, 5 pour le satellite
# Vente forcée : fscore_sell = 5
```
Le F-Score élimine les value traps ; entreprises décotées parce qu'elles méritent de l'être. Rentabilité, levier, efficacité opérationnelle : 9 critères binaires, seuil à 7/9. Kodak, Nokia, Sears, et d'autres... auraient été vendus avant l'effondrement.

### 3. Filtre Anti-Momentum
```python
# mom_trap_floor = -0.25 ; exclut les titres ayant chuté de plus de 25% sur 12-2 mois
```
On n'attrape pas des couteaux qui tombent. Tout titre en chute libre de plus de 25% est exclu de l'univers.

### 4. Pondération Inverse-Volatilité
```python
# inverse_vol_weight = True
# vol_12m = rolling(12).std() sur les rendements mensuels
# w_i = (1/vol_i) / sum(1/vol_j)
```
Les titres les moins volatils pèsent davantage. Réduit mécaniquement la volatilité du portefeuille sans pour autant trop sacrifier le rendement.

### 5. Détection de Régimes — HMM (Hidden Markov Model)
```python
# 3 états : BULLISH, TRANSITION, BEARISH
# Features : Mkt-RF, HML, RMW, Mkt_vol_6m (FF5 Europe)
# Walk-forward expanding window — aucun look-ahead
# Double-lock : 2 mois consécutifs + probabilité >= 70%
```

| Régime | Core | Satellite | Cash minimum |
|--------|------|-----------|-------------|
| BULLISH | 85% | 12% | 3% |
| TRANSITION | 80% | 7% | 5% |
| BEARISH | 72% | 0% | 8% |

En BEARISH, le satellite est immédiatement liquidé (`bearish_cut = True`). Le cash est rémunéré au taux €STR.

### 6. Construction du portefeuille
```python
# n_core = 25 titres (F-Score >= 7), n_satellite = 5 titres (F-Score >= 5)
# max_weight_single = 6%, max_weight_sector = 25%
# Rebalancement : semestriel + drift > 3% + changement de régime
# PIT lag = 60 jours (IAS 34)
```

---


## Résultats (2007–2025, nets de coûts)

| Métrique | Smart Value | HML Passif | Marché Europe |
|----------|------------|------------|---------------|
| CAGR | **6.59%** | 0.26% | 4.34% |
| Volatilité | **5.87%** | 9.57% | 18.73% |
| Sharpe | **1.010** | 0.027 | 0.232 |
| MaxDD | **-22.11%** | -50.89% | -58.44% |
| Hit % | **73.57%** | 47.81% | 55.26% |

**OOS (2021–2025)** : CAGR +8.45%, Sharpe 2.164, MaxDD -2.22%

---

## Limites

- Univers restreint à la zone euro (150M–25Md EUR de capitalisation), là où la data Fama-French prend en compte les entreprises Européennes (et non eurozone)
- Performances OOS exceptionnellement bonnes — contexte 2021-2025 structurellement favorable au value de qualité (hausse des taux, surperformance énergie/bancaires, période courte)

---

## Univers & contraintes

- Zone euro uniquement, cotées en EUR
- Capitalisation : 150M – 25 000M EUR
- Volume minimum : 1M EUR/mois
- Prix minimum : 3€
- Période IS : 2007–2020 | Période OOS : 2021–2025

## Code, données & graphiques

- Code : Code.ipynb
- Données : Company_Info_EUR.csv.zip, EONIARATE.csv, Europe_5_Factors.txt, Market_Cap_EUR.csv.zip, STR.csv, Taux de change
- Graphiques : equity_curves.png, metrics_compariso_global.png
