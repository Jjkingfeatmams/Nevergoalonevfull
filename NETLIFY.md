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
- le commentaire facultatif.

Ce sondage ne comporte ni nom, ni e-mail, ni identifiant de profil, ni donnée
personnelle. Il est stocké séparément sous `survey/` et n’est jamais lu par les
routes publiques qui affichent les groupes.

Les profils de groupe sont enregistrés séparément sous `interest/`. Seuls le
prénom, l’âge, la nationalité, le genre renseigné, l’activité, la description,
les passions et un avatar illustré peuvent apparaître publiquement après le
consentement requis. Le nom et l’e-mail ne sont pas stockés dans les profils
partagés.

Le profil est rempli une seule fois au début de la visite puis réutilisé lors
de l’ajout à un groupe. Il reste uniquement en mémoire dans le navigateur pour
cette visite : il n’est ni enregistré dans l’URL ni partagé avec une personne
qui ouvre le même lien dans une nouvelle session.

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

## Google Form dédié au test

Le parcours conserve son interface intégrée : aucun Google Form n’est affiché
dans l’application. Avec son consentement **facultatif et distinct**, la
personne autorise l’envoi de son e-mail et de ses quatre réponses de test vers le
Google Form dédié. L’URL par défaut pointe vers le formulaire fourni pour cette
version.

Une réponse « Non, ça ne m’intéresse pas » à l’étape de l’offre est aussi
enregistrée dans ce Google Form, même si la personne ne poursuit pas le
parcours. Comme le formulaire impose un champ e-mail et un choix de formule,
la Function utilise alors une adresse technique aléatoire, non délivrable et
se terminant par `@never-go-alone.invalid`, ainsi que l’option de formule
indiquant que la personne n’utiliserait pas l’application. Cette adresse ne
correspond jamais à un e-mail réel et ne permet pas d’identifier la personne.
Dans ce cas, aucun profil, nom, prénom, e-mail réel ou autre donnée personnelle
n’est envoyé à Google.

Dans les réponses Google Forms, le nombre de refus peut donc être obtenu en
filtrant la colonne e-mail sur `@never-go-alone.invalid`, puis la réponse
« Non, ça ne m’intéresse pas » à la question sur l’utilité du service. Une
même session de test, dans le même onglet, réutilise le même identifiant après
un retour ou une actualisation ; le bouton est aussi verrouillé pendant
l’envoi. Cela évite les doublons usuels. Si Google Forms est momentanément
indisponible, le refus reste enregistré anonymement dans Netlify et pourra être
réessayé lors d’un nouveau passage sur cette réponse. Les réponses avec un
e-mail réel restent exclusivement celles pour lesquelles la personne a donné
son consentement distinct.

L’envoi est effectué par la Function Netlify, et non par le navigateur, afin
d’éviter les blocages CORS de Google Forms. En cas d’indisponibilité de Google,
le profil est quand même ajouté au groupe ; aucun échec de formulaire externe
ne doit annuler cette action.

Pour remplacer le Google Form un jour, configure uniquement la variable
d’environnement `NEVER_GO_ALONE_CONTACT_FORM_RESPONSE_URL` avec son endpoint
`formResponse`, puis mets à jour les identifiants `entry.*` et les libellés de
choix dans `netlify/functions/live-interest.mjs`. L’ancienne variable
`GOOGLE_FORM_RESPONSE_URL` n’est volontairement pas utilisée.

Le Google Form actuel ne comporte pas de champs de profil. Les informations
publiées dans le groupe restent donc stockées séparément dans Netlify Blobs et
ne sont jamais transmises aux organisateurs.

Pour protéger l’accès au prototype, active la protection de site dans les
réglages Netlify plutôt que de placer un mot de passe dans le code source.
