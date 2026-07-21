# Design — Prototype cliquable du Dashboard Retail Audit (Kémética)

**Date :** 2026-06-16
**Auteur :** Rostand Tchouakam (Kémética) + assistant
**Statut :** Validé pour construction

## 1. Objectif

Produire un **prototype cliquable** (un seul fichier HTML autonome, sans back-end) du dashboard
de consultation des indicateurs de Retail Audit, destiné au **client final** (ex. service
marketing des Brasseries du Cameroun). Le prototype sert à valider le *look & feel*, la
navigation et la structure des écrans avant que l'équipe (Ovide & co.) construise la vraie
application connectée aux données.

Hors périmètre du prototype : connexion à la base SQL Server, calcul réel des KPI,
authentification / multi-tenant, export Excel réel, collecte terrain.

## 2. Utilisateur & job-to-be-done

- **Utilisateur :** analyste marketing du client (non-technique), consultation seule.
- **Besoin :** voir les indicateurs à jour du mois, comparer dans le temps (série temporelle,
  variation vs mois précédent), et se comparer aux concurrents, le plus intuitivement possible.

## 3. Logique de navigation (validée)

**Panel → Catégorie d'abord.** L'analyse se fait toujours dans un panel donné, pour une
catégorie donnée ; on descend ensuite vers différentes granularités.

- Panels (source `Structure.xlsx`) : `Boissons alcoolisées` (Panel 1), `Boissons non alcoolisées` (Panel 2).
- **Type de produit** (sélecteur n°2, ex-« catégorie ») :
  - Panel 1 : `Bière`, `Alcool Mixte`
  - Panel 2 : `Boisson Gazeuse`, `Eau Minérale`, `Boisson Énergisante`, `Boisson Maltée`, `Jus de Fruit`
- **Hiérarchie de granularité produit (décroissante), TOUJOURS enracinée sur l'Entreprise** —
  les entreprises ne sont **jamais mélangées** dans un même groupe :
  - Panel 1 : `Entreprise` ▸ `Type` ▸ `Marque` ▸ `Variante` ▸ `SKU`
  - Panel 2 : `Entreprise` ▸ `Marque` ▸ `Saveur` ▸ `SKU`
- **Géographie = axe SÉPARÉ** (pas une granularité produit) : `Région`, `Ville`, `Canal`,
  `Type de PDV`. Utilisée comme **filtres globaux** + dimension dans l'Explorateur.
- Le set de produits/entreprises peut changer d'un mois à l'autre (apparition/disparition) —
  l'affichage doit le tolérer.

### Alignement avec le modèle SGBD existant (`Codes_SGBD/`)

La hiérarchie correspond exactement au `GROUP BY` / pivot de `Codes_Indicateurs_15_12_2025.ipynb` :
`Reporting_Item_Man` (= Entreprise, racine) ▸ `typeb_name` (Type) ▸ `brand_name` (Marque) ▸
`variante_name` (Variante) — avec `Barometre_Region` en colonnes (géographie). Pour la vraie
app : utiliser **`Reporting_Item_Man`** comme label d'entreprise (pas `manufacturer_name` brut),
et filtrer sur `use_flag = 1` et `CLIENT_PREFERRED_CAT` (= le type de produit).

## 4. Direction de layout : Hybride A+B (+ Explorateur)

- **Barre de filtres (toujours visible) :** `Panel ▾` · `Type de produit ▾` · `Mois ▾`.
- **Barre géographique (sous-barre) :** filtres `Région` · `Ville` · `Canal` · `Type de PDV`
  (défaut « Toutes ») + indicateur de périmètre + bouton « Réinitialiser ».
- **Sidebar gauche :** familles de KPI + entrée « Explorateur ».
- **Charte graphique :**
  - Bleu pro `#005b97` · Bleu clair `#e5f2fa` · Turquoise (accent/dynamisme) `#1abc9c`
  - Neutre medium `#dee2e6` · Texte `#212121` · Fond `#f7f9fb`

### Écrans

1. **Vue d'ensemble (card-first), niveau Type de produit.** 4 grandes cartes KPI (valeur du mois
   + variation vs mois précédent + sparkline) au niveau du client (SABC), + évolution + classement
   par entreprise. Un clic sur une carte ouvre le détail.
2. **Détail d'un KPI — drill-down progressif.** On part de la liste des **Entreprises** ; un clic
   sur une ligne descend au niveau suivant (Type → Marque → Variante/Saveur → SKU) **à l'intérieur
   de cette entreprise**. Un fil d'Ariane (breadcrumb) permet de remonter. À chaque niveau : graphe
   d'évolution des top membres + tableau classé + entité client (SABC) surlignée « VOUS ». Un
   bandeau rappelle que les entreprises ne sont jamais mélangées. Bascule Volume/Valeur sur les PDM.
   Les filtres géographiques restreignent les valeurs affichées.
3. **Explorateur (tableau croisé / pivot).** Mesure (KPI) × Lignes × Colonnes, où les dimensions
   sont soit une granularité produit (rattachée à son entreprise), soit une dimension géographique.
   Heatmap + bouton « Exporter Excel » (factice dans le prototype).

## 5. Familles & indicateurs (source : `Indicateurs & Formules.xlsx`)

- **Ventes & Parts de marché** : Market %Share Vol, Market %Share Val, Volume (HL),
  Valeur (unité monétaire), Share in Shop Handling %.
- **Distribution** : Numeric Distribution %, Numeric Distribution in Shop Handling %,
  Weighted Distribution %, Distribution Efficiency.
- **Ruptures & Stocks** : Numeric %OOS, Weighted OOS %, Stock Cover Day, Stock %Share.
- **Prix** : Average Price.

Cartes mises en avant sur l'accueil : Part de marché (vol.), Distribution numérique,
Rupture (OOS), Prix moyen.

## 6. Données d'exemple

Structure fidèle à `Structure.xlsx` : pour chaque type de produit, un arbre
`Entreprise → Marque(/Type) → Variante/Saveur → SKU` (ex. SABC ▸ LAGER ▸ 33 Export ▸ Original ▸
« 33 Export Original 65CL GRB »). Entreprises réelles (SABC, Guinness Cameroun, UCB, SOBRAGA,
Nigeria Breweries, Source du Pays, SEMC…). Géographie : Régions (Centre, Littoral, Nord, Ouest),
Villes, Canaux, Types de PDV. 6 mois de 2024 (série temporelle).

Les valeurs sont **simulées de façon déterministe** (PRNG seedé) : parts de marché additives
(les enfants d'un nœud somment au parent ; au niveau racine, les entreprises somment à 100 %),
cohérentes d'un mois à l'autre. Le prototype ne calcule rien depuis la base. Client « vous » =
SABC (Brasseries du Cameroun), surligné « VOUS ». Bandeau « Données d'exemple » affiché partout.

## 7. Technique

- **Un seul fichier `.html` autonome** : HTML + CSS + JS vanilla, données injectées en JSON
  inline, graphiques dessinés en SVG (pas de dépendance réseau). Ouvrable par double-clic,
  partageable à l'équipe.
- **Interactions cliquables :** changer panel / type de produit / mois ; filtres géographiques
  (Région/Ville/Canal/Type PDV) + réinitialiser ; naviguer vue d'ensemble ↔ détail ;
  **drill-down** par clic sur une ligne + remontée via breadcrumb ; bascule Volume/Valeur ;
  Explorateur (mesure × lignes × colonnes, produit ou géographie).
- **Responsive léger** : utilisable au navigateur ; pas d'app mobile native (le SaaS pourra
  l'envisager plus tard).

## 8. Critères de succès

- L'équipe et le client peuvent ouvrir le fichier et naviguer sans explication.
- La navigation Panel→Catégorie→KPI→granularité est fluide et intuitive.
- Le rendu respecte la charte graphique Kémética.
- Sert de référence visuelle claire pour l'implémentation de la vraie application.
