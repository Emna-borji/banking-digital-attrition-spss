# banking-digital-attrition-spss

## Objectif
Analyse prédictive de l'**attrition digitale** de la clientèle d'une banque : identifier
les clients susceptibles de se désengager des canaux digitaux (mobile banking, paiement
mobile, cartes) afin d'orienter des actions de rétention.

## Données
Dataset **`banque_digitale_tunisie_100k.csv`** — **100 000 observations** de clients
bancaires (format `Client_ID` : `CL000001`, `CL000002`, …), montants en TND (Dinar Tunisien).

**Nature : données synthétiques** — dataset de cours généré pour un projet académique
(Mastère DSB). Pas de clients réels, aucune donnée personnelle sensible.

Fichiers inclus dans le repo :
- `banque_digitale_tunisie_100k.csv` — dataset brut (25 variables)
- `nettoye.csv` — dataset après feature engineering / sélection (23 variables, incluant
  les scores dérivés et versions `_transformed`)

Variables :
- Démographie / profil : `Client_ID`, `Age`, `Profession`, `Type_Compte`,
  `Anciennete_Bancaire_Annees`
- Produits : `Nb_Produits_Bancaires`, `Carte_Bancaire`, `Credit_Actif`
- Finances : `Revenu_Mensuel_TND`, `Solde_Moyen_TND`, `Montant_Transactions_Digitales_TND`
- Usage digital : `Activation_Mobile_Banking`, `Utilisation_Paiement_Mobile`,
  `Nb_Connexions_Web`, `Nb_Connexions_Mobile`, `Nb_Operations_Online`,
  `Temps_Depuis_Derniere_Connexion_Jours`, `Nb_Visites_Agence`
- Relation client : `Score_Satisfaction`, `Nb_Reclamations`, `Incidents_Connexion`
- Cible : **`Attrition_Digitale`** (24,6 % positifs, 75,4 % négatifs — classes déséquilibrées)

Valeurs manquantes principales : `Revenu_Mensuel_TND` (1,22 %), `Score_Satisfaction` (1,73 %),
`Montant_Transactions_Digitales_TND` (0,98 %), `Profession` (610 non renseignés).

## Méthode
Flux **IBM SPSS Modeler** (`Flux1.str`) en 11 étapes :
1. **Audit** du dataset (Data Audit, distributions, skewness/kurtosis).
2. **Traitement des outliers** par capping (Revenu, Solde, Transactions, Réclamations, Incidents).
3. **Nettoyage** : vérification des doublons (nœud Distinct, aucun doublon), imputation
   des valeurs manquantes.
4. **Variables dérivées** :
   - `Score_Engagement_Digital = Nb_Connexions_Web + Nb_Connexions_Mobile + Nb_Operations_Online`
   - `Score_Problemes = Nb_Reclamations + Incidents_Connexion`
   - `Intensite_Digitale = Montant_Transactions_Digitales_TND / (Nb_Operations_Online + 1)`
   - `Score_Relation_Banque = Nb_Produits_Bancaires + Anciennete_Bancaire_Annees`
5. **Feature Selection** : Chi² (catégorielles), ANOVA (continues), corrélations de Pearson.
6. **Segmentation** : clustering K-Means (clients premium en rupture, autonomes/satisfaits,
   traditionnels de proximité).
7. **Détection d'anomalies** (~1 % de profils atypiques).
8. **Split** 70 % / 30 % (train / test), classes conservées dans le test.
9. **Modélisation** (6 algorithmes comparés) :
   - **C5.0**, CHAID, QUEST, Régression Logistique, Réseau de Neurones, Réseau Bayésien
10. **Évaluation** : Accuracy, AUC, Recall, Interprétabilité.
11. **Croisement scoring × segments** et recommandations stratégiques.

## Stack
- **IBM SPSS Modeler** (fichier `.str`)
- Dataset tabulaire CSV
- Rapport PDF détaillé (12 parties) joint

## Résultats
| Modèle | Accuracy | AUC |
|---|---|---|
| **C5.0** ⭐ | **83,82 %** | **0,893** |
| CHAID | 81,56 % | - |
| Régression Logistique | 80,55 % | - |
| Réseau de Neurones | 80,51 % | - |
| QUEST | 79,90 % | - |
| Réseau Bayésien | 79,56 % | - |

**Modèle retenu : C5.0** — meilleur compromis performance / interprétabilité (règles de
décision exploitables côté métier).

**Variables les plus influentes** (toutes versions `_transformed`) :
`Score_Satisfaction` > `Temps_Depuis_Derniere_Connexion_Jours` > `Score_Engagement_Digital`
> `Score_Problemes`.

## Comment lancer
1. Ouvrir `Flux1.str` avec **IBM SPSS Modeler**.
2. Vérifier le chemin du fichier source `nettoye` (CSV) dans le nœud *Source*.
3. Exécuter le flux de bout en bout — chaque partie du rapport correspond à une branche
   du flux.
