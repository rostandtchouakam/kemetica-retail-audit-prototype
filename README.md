# Kémética — Retail Audit · Prototype de dashboard

Prototype cliquable du dashboard de consultation des indicateurs de **Retail Audit**
(analyse de marché boissons) développé par **Kémética**.

> ⚠️ **Prototype à données simulées.** Toutes les valeurs affichées sont générées de façon
> déterministe à des fins de démonstration. Aucune donnée de production, aucun secret,
> aucune connexion à une base réelle.

## 🌐 Démo en ligne

**https://kemetica-prototype.vercel.app**

> Déploiement automatique : chaque `git push` sur `main` redéploie la démo via Vercel.

## ✨ Fonctionnalités

- **Navigation Panel → Type de produit → Mois** (2 panels : boissons alcoolisées / non alcoolisées).
- **Vue d'ensemble** : cartes KPI (valeur du mois + variation + sparkline), évolution, classement par entreprise.
- **Détail d'un indicateur en drill-down** : de l'Entreprise vers Type → Marque → Variante/Saveur → SKU,
  **la hiérarchie produit reste toujours enracinée sur l'entreprise** (les sociétés ne sont jamais
  mélangées dans un même groupe).
- **Filtres géographiques** (Région / Ville / Type de PDV) appliqués à toutes les vues.
- **Explorateur (tableau croisé)** : granularité produit en lignes × géographie en colonnes, avec heatmap.
- Indicateurs : parts de marché (volume/valeur), distribution (ND, WD, efficacité), ruptures & stocks
  (OOS, couverture), prix moyen, etc.

## 🚀 Utilisation

Le prototype est **un seul fichier HTML autonome**. Ouvre simplement `index.html` dans un navigateur
(aucune installation, aucun serveur requis).

## 🎨 Stack

- HTML + CSS + JavaScript (vanilla), sans dépendance runtime.
- Graphiques dessinés en SVG.
- Polices : Bricolage Grotesque + Hanken Grotesk.
- Charte Kémética : bleu `#005b97`, turquoise `#1abc9c`.

## 📄 Design

La spécification de conception est dans [`docs/dashboard-prototype-design.md`](docs/dashboard-prototype-design.md).

---

© Kémética — Prototype de démonstration.
