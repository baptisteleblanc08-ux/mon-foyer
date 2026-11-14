# Mon Foyer — Accueil V2

## Statut
Branche de test pour la page d'accueil. Ne fusionner avec `main` qu'après validation sur PC et smartphone.

## Fonctionnement
- Page d'accueil : `index.html` (HTML/CSS/JavaScript autonomes).
- Douze indicateurs, quatre panneaux de graphiques, alertes, actions, six liens applicatifs.
- Les indicateurs indisponibles affichent **—**. Aucune donnée privée ni secret dans le dépôt.
- Les seuils et le relevé manuel de fioul sont stockés dans le navigateur sous `mon-foyer-dashboard-settings-v2`.
- Lecture seule du module Eau via `mon-eau-v1`.
- Lecture seule des clés Budget `mon-foyer-budget-v01`, `mon-foyer-budget-bank-v06` et `mon-foyer-budget-solde-verifie-v15`.
- **Jamais de modification des clés de stockage des applications existantes.**

## Connecteur de données prévu
Pour l'instant, un objet JavaScript `window.MON_FOYER_BRIDGE` ou un fichier JSON importé **uniquement en mémoire** permet d'alimenter les autres tuiles. Le contenu peut être fourni par un connecteur privé ultérieur.

Champs pris en charge (aucun n'est obligatoire) :

- `electricity.daily[]`: `date` (AAAA-MM-JJ), `consumptionKwh`, `renewableUsedKwh`, `pvProductionKwh`, `pvUsedKwh`. `renewableUsedKwh` doit être une énergie renouvelable effectivement consommée, et non une production totale.
- `budget.forecastEndEUR` : prévision de solde en euros, établie séparément des opérations bancaires.
- `budget.forecastSeries[]`, `budget.actualSeries[]` : objets `{date, value}` (euros).
- `fuel` : `{litres, litresPerDay, date}`.
- `costs` : `{totalMonthEUR, targetEUR, categories:[{name,value}]}`. Les transferts internes et doublons doivent être exclus avant injection.
- `savings.totalMonthEUR` : économies nettes validées (pas de double comptage).
- `alerts[]` : `{title, detail, level}`, avec `level` valant `warn`, `bad` ou `good`.
- `actions[]` : `{date, title, detail}`.

Les séries électriques complètes sur sept jours déclenchent le calcul des KPI. Une comparaison avec la période précédente demande au moins quatorze jours consécutifs de données.

## Contrôles métier
- **Part de renouvelable** = photovoltaïque autoconsommé / photovoltaïque produit.
- **Autonomie électrique** = renouvelable utilisé / consommation totale, batterie PV incluse une seule fois.
- **Eau sur 7 jours** = index du jour − index d'il y a sept jours, par interpolation **seulement si les relevés encadrent ces dates**. Sinon la valeur reste indisponible.
- **Budget connu** : priorité au solde de référence et mouvements bancaires compatibles; les opérations futures ne sont jamais automatiquement considérées comme constatées.
- **Fioul** : autonomie indicative = volume / litres par jour. Précision et saisonnalité à confirmer avant affichage comme valeur fiable.
- **État global** : une absence de données ne signifie jamais « conforme ».

## Test manuel
1. Ouvrir l'URL Azure de prévisualisation sur PC et smartphone.
2. Vérifier les liens des six applications.
3. Ouvrir **Seuils**, changer une valeur, rafraîchir : la valeur doit être conservée sur le même navigateur.
4. Sans données, tous les KPI concernés doivent afficher **—** (jamais zéro inventé).
5. Quand des relevés Eau et Budget existent sur le même navigateur et sur le même domaine, vérifier qu'ils sont consultés sans modification des données source.
6. Ouvrir les graphiques : les états vides doivent être explicites.
7. Contrôler en priorité la lisibilité sur téléphone étroit.

## Limites
Les données des modules publiés sur d'autres domaines Azure ne sont pas lisibles directement par `localStorage` depuis Mon Foyer. Elles exigeront un connecteur privé (Home Assistant/API/OneDrive selon le domaine), avec authentification. Le déploiement de cette page ne réalise pas cette synchronisation.
