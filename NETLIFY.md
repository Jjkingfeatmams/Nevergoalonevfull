# Publier le prototype Never Go Alone sur Netlify

Cette version garde les profils de groupes en direct avec Netlify Blobs et
ajoute un sondage tarifaire **anonyme**, distinct des profils de groupe.
Les événements, tarifs et données affichés sont fictifs et servent uniquement
au prototype.

## Publication

1. Crée un projet Netlify depuis ce dossier Git complet. Ne publie pas
   seulement `index.html` : les Functions doivent être incluses.
2. Laisse le **Base directory** vide si ce dossier est à la racine du dépôt.
   Netlify utilisera `netlify.toml`, exécutera `npm run build` et publiera
   `dist`.
3. Déclenche le déploiement. Le dossier `dist` est généré par la build et ne
   doit pas être ajouté à Git.
4. Ajoute dans **Site configuration → Environment variables** une valeur
   longue et privée pour `INTEREST_HASH_SECRET`. Ne la modifie plus après les
   premières inscriptions, car elle protège les liens de retrait de profil.

## Données et confidentialité

Le parcours tarifaire enregistre, avec un identifiant de session aléatoire :

- la situation initiale ;
- l’indicateur interne `cible`, dérivé côté serveur ;
- l’intérêt pour l’offre ;
- la formule choisie ;
- le niveau de frais de service acceptable ;
- le commentaire facultatif.

Ce sondage ne comporte ni nom, ni e-mail, ni identifiant de profil, ni donnée
personnelle. Il est stocké séparément sous `survey/` et n’est jamais lu par les
routes publiques qui affichent les groupes.

Les profils de groupe sont enregistrés séparément sous `interest/`. Seuls le
prénom, l’âge, la nationalité, le genre renseigné, l’activité, la description,
les passions et un avatar illustré peuvent apparaître publiquement après le
consentement requis. Le nom et l’e-mail ne sont pas stockés dans les profils
partagés.

Aucune donnée personnelle n’est transmise aux organisateurs. Les éventuels
éléments partagés avec eux doivent rester des statistiques agrégées et
anonymisées — intérêt par sortie, préférences de formule ou volume potentiel
de participants — sans nom, e-mail ou profil individuel.

## Rapport anonyme interne

Pour consulter uniquement les statistiques agrégées, ajoute également une
variable d’environnement privée `SURVEY_REPORT_TOKEN`. Puis appelle :

```text
GET /api/internal/survey-summary
x-survey-report-token: <la valeur de SURVEY_REPORT_TOKEN>
```

La réponse répartit les résultats entre `cible` et `non-cible`, sans jamais
renvoyer les réponses brutes ni les commentaires.

## Contact facultatif

Le profil peut proposer une inscription facultative aux nouvelles de Never Go
Alone. Cette demande est séparée du sondage tarifaire. Si le projet conserve
un Google Form pour ce contact, configure au besoin
`GOOGLE_FORM_RESPONSE_URL`; aucune réponse du sondage tarifaire ne doit lui
être envoyée.

Pour protéger l’accès au prototype, active la protection de site dans les
réglages Netlify plutôt que de placer un mot de passe dans le code source.
