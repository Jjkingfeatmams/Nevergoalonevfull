# Publier la V5 abonnement sur Netlify

Cette V5 conserve les profils intéressés live et ajoute un test d'abonnement
natif : après avoir ajouté son profil pour une sortie, la personne indique si
elle préférerait 2,99 € par mois, 7,99 € par trimestre, un paiement par
événement ou aucun abonnement. Il n'y a ni carte bancaire ni paiement réel.

## Publication

1. Créer un **nouveau projet Netlify** depuis le même dépôt Git (ne pas
   déposer seulement `index.html`, car les Functions ne seraient pas
   publiées). Ainsi, la version existante reste en ligne.
2. Dans la configuration du nouveau projet, choisir `v5-abonnement` comme
   **Base directory**. Netlify utilisera alors `v5-abonnement/netlify.toml`, la commande
   `npm run build` et le dossier publié `dist`.
   Le `netlify.toml` situé à la racine du dépôt reste volontairement réglé
   sur une version antérieure ; ne pas modifier le projet existant si
   l'objectif est de conserver les deux liens.
3. Déclencher le déploiement. La commande est `npm run build` et le dossier
   publié est `dist`.
4. Dans **Site configuration → Environment variables**, ajouter de préférence
   `INTEREST_HASH_SECRET` avec une longue valeur aléatoire privée. Ne pas
   modifier cette valeur après les premières inscriptions, car elle protège
   les liens de retrait des profils.

Les routes `/api/*` sont traitées par `netlify/functions/live-interest.mjs`.
Elles enregistrent seulement le prénom, l'âge, la nationalité, l'activité,
la courte description, les passions et un avatar illustré qui peuvent être
affichés publiquement.
Le genre choisi peut aussi être affiché publiquement après consentement.
L'e-mail et le nom ne sont pas conservés dans le stockage partagé.

La préférence d'abonnement et le commentaire facultatif sont enregistrés dans
le même dossier de profil mais restent **privés** : ils ne sont jamais renvoyés
par les routes qui affichent les membres du groupe. Pour que ce test fonctionne,
publier impérativement le dossier complet, y compris `netlify/functions/` ;
publier uniquement `dist/` ne suffit pas.

Pour protéger l'accès au prototype, activer la protection du site dans les
réglages Netlify plutôt que de mettre un mot de passe dans le code source.
