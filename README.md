# Nuage Retail – Pilotage de la performance du réseau

Rapport Power BI interactif d'aide à la décision pour une enseigne retail de **40 magasins** en France (formats Hyper, Super et Drive, 6 régions). Il couvre **2024–2025**.

> Projet réalisé dans le cadre d'une évaluation Power BI (M1 MIADS, IAE Lille).
> Les données ne sont pas incluses dans ce dépôt : il s'agit d'un jeu de données pédagogique fourni par l'école.

![Page de couverture](assets/cover_05.png)

📄 Le rapport complet est disponible en PDF : [`docs/Sales_Performance_Dashboard.pdf`](docs/Sales_Performance_Dashboard.pdf)

---

## Objectifs

La Direction Réseau veut :
- suivre la performance des magasins sur 2 ans, par magasin, format et région ;
- comparer chaque période à N-1 ;
- identifier les magasins en Top / Flop selon le CA, le nombre de clients ou la marge ;
- mesurer l'impact des promotions sur les unités vendues, le CA et la marge.

## Pages du rapport

| Page | Question | Contenu principal |
|---|---|---|
| **Synthèse** | Comment se porte le réseau ? | KPI vs N-1, CA mensuel vs N-1, CA par format, croissance par région |
| **Top / Flop** | Où ça marche, où ça décroche ? | Top 10 / Flop 10 magasins avec bascule CA / Clients / Marge (paramètre de champs), tableau détaillé |
| **Tendances & N-1** | D'où vient la croissance ? | Tendance sur 24 mois (CA + taux de marge), cascade de l'écart de CA par région, heatmap région × format |
| **Promotions** | Les promos sont-elles rentables ? | Effet d'une journée de promo par catégorie (volume, CA, marge), taux de marge en promo et hors promo, synthèse |

## Modèle de données

La source est une **table plate de ventes** (≈117 000 lignes, grain jour × magasin × catégorie). Elle est restructurée en **schéma en constellation** :

- **Tables de faits** : `FactVentes` (grain journalier) et `FactObjectifs` (grain mensuel)
- **Dimensions** : `DimDate` (créée en DAX, marquée comme table de dates), `DimMagasin`, `DimCategorie` et `DimPromo`
- Relations 1-à-plusieurs, filtrage unidirectionnel

### Transformations Power Query
- Suppression des espaces parasites (`Text.Trim`) sur les clés
- Typage explicite de toutes les colonnes
- Attributs magasin (région, format, surface) retirés de la table de faits et pris dans `DimMagasin`. La table plate contenait des incohérences de casse (`NORD` / `Nord`), des espaces en trop et des surfaces manquantes.
- `NbTransactions` supprimée, car identique à `NbClients` sur 100 % des lignes
- `Promo` (Oui/Non) convertie en clé `PromoFlag` (1/0)
- Colonnes d'aide au tri et à l'analyse : ordre des formats, tranche de surface

## Mesures DAX principales

```dax
CA N-1 = CALCULATE([CA], SAMEPERIODLASTYEAR(DimDate[Date]))
Var CA % = DIVIDE([CA] - [CA N-1], [CA N-1])

Uplift unités promo % =
DIVIDE(
    CALCULATE([Unités par jour], DimPromo[PromoFlag] = 1),
    CALCULATE([Unités par jour], DimPromo[PromoFlag] = 0)
) - 1

KPI Sélectionné =
SWITCH(SELECTEDVALUE('KPI'[KPI Order], 0),
    0, [CA], 1, [Nb Clients], 2, [Marge])
```

Les mesures sont rangées dans une table dédiée (`_Mesures`), par dossiers : KPI, N-1, Promo et Top-Flop. Les titres des pages et des graphiques sont **dynamiques** : ils suivent les filtres sélectionnés.

## Principaux enseignements (2025 vs 2024)

- **CA +3,8 %** et **marge +3,7 %**, à taux de marge stable (≈24,8 %). Le **nombre de clients est stable** : la croissance vient du **panier moyen**.
- Le **Sud-Ouest** et l'**Ouest** apportent près de la moitié de la croissance. Le **Nord** progresse le moins (+3,0 %), avec un point faible sur le Drive (+2,7 %).
- La **promotion achète du volume, pas de la valeur** : +27 % d'unités par jour de promo, mais seulement +2,4 % de CA et −4,7 % de marge. Le taux de marge passe de 25,0 % hors promo à 23,3 % en promo.
- La remise est identique partout (≈18 %), mais le **Frais** perd le plus de marge en promo (−7,4 %), le **DPH** le moins (−3,6 %).

## Ouvrir le projet

1. Ouvrir `Sales Performance Dashboard.pbip` avec **Power BI Desktop**. Le format PBIP (Power BI Project) est enregistré sous forme de fichiers texte, versionnables avec Git.
2. Les données n'étant pas incluses, les visuels seront vides. Pour les recharger avec le fichier source : **Transformer les données → Modifier les paramètres**, renseigner le chemin du fichier Excel, puis **Actualiser**.

Le thème personnalisé se trouve dans [`assets/Theme_Nuage_Retail.json`](assets/Theme_Nuage_Retail.json). Pour l'appliquer : **Affichage → Thèmes → Parcourir les thèmes**.

## Outils

Power BI Desktop · Power Query (M) · DAX · format PBIP · Git / GitHub

---

**Auteur :** Phichet Phuangkaeo · M1 MIADS, IAE Lille · [Portfolio](https://phuangkaeo.vercel.app)
