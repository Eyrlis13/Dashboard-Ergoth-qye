# Myria — tableau de bord Ergothèque®

Tableau de bord d'analyse pour le dispositif Ergothèque® de Merci Julie, adossé
à la thèse de Syrielle Zouakh. Il présente 13 063 bénéficiaires sur 32
ergothèques, période 2021–2026.

## Règle non négociable : affichage uniquement

**Ne jamais introduire ni modifier un calcul dans le code de rendu.** Les
chiffres viennent du pipeline de données en amont ; une statistique dérivée côté
page ne correspondrait pas à ce que la thèse publie. Cela vaut même pour remplir
une colonne que le code réclame déjà, et même quand la page calcule la même
chose ailleurs.

Quand une fonction de dessin lit un champ absent des données, afficher un
placeholder (`—`) et signaler le manque — ne pas dériver la valeur. Le choix
d'une méthode (approximation normale, bootstrap, vraisemblance profilée pour un
intervalle de confiance) est une décision méthodologique qui appartient à
l'amont.

## Structure

Site statique, **aucune étape de build**, aucune dépendance à installer.

| Fichier | Rôle |
| --- | --- |
| `index.html` | ~1,4 Mo : le balisage des 7 pages, puis un `<script>` unique contenant les données et tout le code |
| `styles.css` | Toute la présentation, extraite du HTML |
| `logo-*`, `favicon*`, `apple-touch-icon.png` | Identité visuelle (mascotte fourmi) |

La mascotte forme une famille `logo-myria-*` : `logo-myria-source.jpeg` est
l'original avec fond, `logo-myria-full.png` le master détouré en 1000 px,
`logo-myria-mark.png` la seule déclinaison affichée (sidebar, 56 px CSS rendus
en 200 px pour les écrans denses). Les deux premières servent à regénérer les
autres. Les favicons gardent en revanche leurs noms conventionnels, attendus par
les navigateurs et les outils — ne pas les renommer.

Deux dépendances externes, chargées par CDN : Chart.js 4.4.1 et les polices
Google (Playfair Display, DM Sans). Une machine hors ligne rendra la page sans
graphiques ni typographie.

### Dans `index.html`

- Les données sont des littéraux JS en tête de script : `DATA` (agrégats),
  `STATS` (résultats statistiques), plus `AT_PRICES`, `CAT_PRICES`,
  `ERGO_MAP_MARKERS`, et les constantes du calcul SROI (`REDUC_MIN`,
  `AGE_FALL_INCIDENCE`, `GIR_FALL_WEIGHT`…). Elles sont produites en amont —
  les éditer à la main est presque toujours une erreur.
- Une page = un `<div id="p-{vue}" class="page">`. `go(view)` bascule la classe
  `active` et délègue le rendu à `drawPage(view)`, qui appelle `drawVue()`,
  `drawPersonnes()`, `drawAnalyses()`… Plusieurs pages ont des sous-onglets
  (`switchVueTab`, `switchEqTab`, `switchResearchTab`).
- Environ la moitié des tableaux sont injectés en `innerHTML` depuis ces
  fonctions. Une règle CSS visant un conteneur ajouté au balisage ne les
  atteindra pas — viser l'élément lui-même.

### Pièges connus

- `go(view)` lit la variable globale `event`. Le piloter en cliquant un
  `.nav-item`, jamais en l'appelant directement.
- `closeNav()` s'exécute **avant** `drawPage()`, volontairement : la navigation
  ne doit pas dépendre de la réussite du rendu. Ne pas inverser.
- Chart.js anime par défaut, ce qui rend toute capture non déterministe. Poser
  `Chart.defaults.animation = false` avant de comparer des rendus.

## Vérifier une modification

Servir le dossier et l'ouvrir :

```sh
python3 -m http.server 8899   # puis http://localhost:8899/index.html
```

Il n'y a pas de tests. Pour une modification de présentation, la vérification
attendue est un **diff contre la prod** : ajouter un arbre de travail sur
`origin/main`, servir les deux, et comparer avec Chrome headless page par page,
onglet par onglet, filtre par filtre. Des ajouts sont acceptables ; **une valeur
affichée qui change ne l'est pas**.

Balayer au minimum 320 / 375 / 768 / 1280 px, tiroir ouvert et fermé. Un
débordement se détecte en comparant `scrollWidth` à `clientWidth` — en ignorant
les descendants d'un conteneur à `overflow-x:auto` (les tableaux larges défilent
volontairement) et les éléments en `position:absolute` (les flèches décoratives
de `.stage-flow` dépassent par construction).

## CSS

Mobile d'abord : chaque règle part du petit écran, les media queries
`min-width` ajoutent. **Ne pas introduire de `max-width`.** Points de rupture :
**640 / 900 / 1200**. En dessous de 900 px la sidebar est un tiroir hors écran ;
au-delà, une colonne fixe.

Palette et rayons en variables CSS dans `:root` (`--sage`, `--coral`,
`--charcoal`, `--r`, `--r2`…). Ne pas écrire une couleur en dur.

Un style inline ne peut pas être surchargé par une media query. Pour un gabarit
qui doit changer selon la largeur, mettre la valeur dans une propriété
personnalisée et la consommer depuis une classe — voir `.g-split` et `--cols`.

## Conventions

- **Interface et commentaires de code en français.** Messages de commit en
  anglais, préfixés d'un gitmoji (`💄` présentation, `📱` responsive, `🐛`
  correction, `🎨` structure, `✏️` texte, `🍱` ressources). La branche
  `backup-avant-gitmoji` conserve l'historique sans emoji : la convention est
  un choix délibéré.
- Le corps du message explique *pourquoi*, pas *quoi* — le diff dit déjà quoi.

## Déploiement

GitHub Pages sert `origin/main` de `Eyrlis13/Dashboard-Ergoth-qye` sur
<https://eyrlis13.github.io/Dashboard-Ergoth-qye/>. Pousser sur `main` publie.
Travailler sur une branche.

## Dette connue

`drawAnalyses()` s'interrompt sur deux formes de données qui ne correspondent
pas au code, ce qui laisse cinq graphiques vides en bas de la page Analyses :

- `by_at_category.rate_by_cat` est une table plate catégorie → volume, là où le
  bloc attend des objets de régression (`beta`, `se`, `p`).
- `corr.top` est `undefined`.

Corriger demande de la donnée, pas de l'affichage.
