# Publier le prototype Never Go Alone sur Netlify

Cette version garde les profils de groupes en direct avec Netlify Blobs et
ajoute un sondage tarifaire **anonyme**, distinct des profils de groupe.
Les événements, tarifs et données affichés sont fictifs et servent uniquement
au prototype.

## Simulation tarifaire

Les prix de l’interface proviennent d’une configuration unique dans
`index.html` :

- remise groupe : 10 % du plein tarif ;
- frais sans abonnement : 50 % de cette remise, arrondis au centime ;
- abonnement mensuel : 5 € ;
- abonnement trimestriel : 12 €.

Ainsi, dès qu’il existe une remise, le total sans abonnement reste strictement
inférieur au plein tarif. Avec l’abonnement, les frais de service sont nuls.
Le détail du tarif groupe, des frais et du total est affiché avant que la
personne manifeste son intérêt.

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
- l’avis sur le prix mensuel de 5 €.

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
Google Form dédié. L’URL par défaut pointe vers le [formulaire de test à 5 €](https://forms.gle/NoDpMuuNKEQeeRvd7)
fourni pour cette version.

Une réponse « Non, ça ne m’intéresse pas » à l’étape de l’offre est aussi
enregistrée dans ce Google Form, même si la personne ne poursuit pas le
parcours. Comme le formulaire impose un champ e-mail et un choix de formule,
la Function utilise alors une adresse technique aléatoire, non délivrable et
se terminant par `@never-go-alone.invalid`, ainsi que l’option indiquant que
la personne n’utiliserait pas l’application avec ce modèle. Cette adresse ne
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

Le formulaire de ce test est fixé directement dans
`netlify/functions/live-interest.mjs` afin qu’une ancienne variable Netlify ne
redirige jamais les réponses vers un autre formulaire. Pour le remplacer un
jour, mets à jour à la fois son endpoint `formResponse`, les identifiants
`entry.*` et les libellés de choix dans ce fichier, puis redéploie la Function.

Le Google Form dédié fourni à cette version contient les quatre choix suivants
pour la formule :

- « 5,00 € par mois. Un accès aux tarifs de groupe sans frais de service » ;
- « 12,00 € par trimestre. Environ 4,00 € par mois, sans frais de service, avec un engagement de trois mois » ;
- « Payer seulement à l’événement. Tu paies le tarif de groupe plus 50% de la remise de frais de service à chaque sortie » ;
- « Je n’utiliserais pas l’application avec ce modèle. Cette proposition ne me conviendrait pas. »

Il comporte aussi une question distincte sur le prix mensuel de 5 €, avec les
réponses « 5 € par mois, c'est un prix qui me convient », « 5 € par mois,
c'est trop cher, mais je serais prêt(e) à payer 2,99 € par mois » et « Même
2,99 € par mois, ce serait trop cher pour moi et je privilégierais la formule
sans abonnement ».

Le Google Form actuel ne comporte pas de champs de profil. Les informations
publiées dans le groupe restent donc stockées séparément dans Netlify Blobs et
ne sont jamais transmises aux organisateurs.

Pour protéger l’accès au prototype, active la protection de site dans les
réglages Netlify plutôt que de placer un mot de passe dans le code source.
