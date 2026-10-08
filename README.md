# Nothing Exploded

Blog statique généré avec [Zola](https://www.getzola.org) et le thème
[Duckquill](https://duckquill.daudix.one), déployé automatiquement sur GitHub Pages.

👉 https://nothingexploded.github.io

## Prérequis

- [Zola](https://www.getzola.org/documentation/getting-started/installation/) **≥ 0.23.6**
  (version utilisée en intégration continue : `0.23.6`)

> ⚠️ Zola 0.23.4 et antérieures ne suffisent pas : le thème Duckquill migré vers
> Tera 2 utilise la fonction `text_direction`, corrigée dans Zola 0.23.6.
> `zola --version` doit donc afficher `0.23.6` ou plus.

## Développement local

```sh
# Serveur de développement avec rechargement à chaud : http://127.0.0.1:1111
zola serve

# Génération du site dans ./public
zola build
```

## Écrire un article

Créer un dossier dans `content/` contenant un fichier `index.md` — chaque
article est un dossier, ce qui permet d'y placer ses images :

```toml
+++
title = "Mon article"
description = "Résumé affiché dans la liste des articles et les flux."
date = 2025-01-01
[taxonomies]
tags = ["Notes"]
+++

Contenu de l'article, en Markdown.
```

Les images d'un article se placent dans le même dossier et s'appellent avec
`![description](mon-image.png)`.

Pour lier une page du site, utiliser le format Markdown de Zola, par exemple
`[Contact](@/contact/_index.md)`.

## Structure du dépôt

```
config.toml              # configuration du site (titre, URL, navigation, thème)
content/
  _index.md              # page d'accueil : liste des articles
  contact/
    _index.md            # page « Contact »
  <article>/index.md     # un article
i18n/
  fr.toml                # chaînes d'interface françaises (surcharge le thème)
sass/
  custom.scss            # surcharges de couleurs (compile vers static/custom.css)
  mods.scss              # mods Duckquill activés (compile vers static/mods.css)
templates/               # surcharges du thème (prioritaires sur themes/duckquill/templates/)
static/                  # fichiers copiés tels quels (favicon, .nojekyll, images)
themes/duckquill/        # thème versionné ici (pas de sous-module)
.github/workflows/deploy.yml  # CI : build Zola + déploiement GitHub Pages
```

## Intégration continue

Le workflow `.github/workflows/deploy.yml` se déclenche à chaque `push` sur `main`
(ou manuellement depuis l'onglet *Actions*). Il tient en **un seul job** `deploy`
(le pattern officiel GitHub Pages) :

1. checkout du dépôt ;
2. téléchargement de Zola `0.23.6` (variable `env.ZOLA_VERSION`) et
   `zola build --output-dir public` ;
3. `actions/configure-pages`, puis upload de `./public` comme artefact Pages ;
4. déploiement sur GitHub Pages.

Le site est servi à la racine de <https://nothingexploded.github.io>. C'est
possible parce que le dépôt porte le nom **`<owner>.github.io`** — condition
pour obtenir un site « utilisateur » (une seule URL par compte). Le nom du
dépôt n'a rien à voir avec le titre du site (`title = "Nothing Exploded"`) : GitHub
exige littéralement le nom du compte. C'est `base_url` dans `config.toml` qui
doit correspondre à l'URL finale, sinon le CSS, les favicons et les flux
pointent dans le vide.

> ⚠️ **Prérequis, sinon la CI échoue à l'étape 3.** L'action `configure-pages`
> interroge l'API `GET /repos/{owner}/{repo}/pages` ; si Pages n'est pas activé,
> elle répond 404 et le run s'arrête sur
> « Get Pages site failed. Please verify that the repository has Pages enabled
> and configured to build using GitHub Actions ».
> Réglage à faire une fois : **Settings → Pages → Build and deployment →
> Source : GitHub Actions**.
> Piège : l'option `enablement: true` de `configure-pages` ne permet pas
> d'automatiser ce réglage, car l'action refuse explicitement `GITHUB_TOKEN`
> pour cette opération (il faut un PAT en secret de dépôt).

## Personnalisation

- **Couleur d'accent** : modifier `accent_color` (thème clair) et `accent_color_dark`
  (thème sombre) dans la section `[extra]` de `config.toml`. Toute la déclinaison
  (liens, cartes, navigation, ombres) en découle.
- **Couleurs de fond et surfaces** : `sass/custom.scss`. Duckquill v6 dérive son
  fond de `accent_color` (`color-mix(… 20%, white)`), ce qui donne un rose pâle
  avec le rouge d'accent. Ce fichier écrase `--bg-color`, `--glass-bg` et
  `--bg-overlay` dans les trois contextes que le thème connaît : thème clair,
  `[data-theme="dark"]` et `prefers-color-scheme: dark`. Valeurs actuelles :
  `#ffffff` en clair, `#5a5a5a` en sombre.
- **Polices** : `bundled_fonts = true` charge `fonts.css` (Inter Variable pour le
  texte, JetBrains Mono pour le code), embarquées dans le thème. Sans cette
  option, `fonts.css` n'est jamais demandé et le navigateur retombe sur sa
  police système — le rendu change complètement d'une machine à l'autre.
- **Mise en page** : `sass/mods.scss` importe les « mods » officiels de Duckquill
  depuis `themes/duckquill/sass/mods/`. Actif ici : `classic-nav`, qui remplace la
  pilule flottante de v6 par une barre pleine largeur collée en haut. Pour en
  ajouter un, l'importer dans ce fichier — la liste
  <https://duckquill.daudix.one/mods/> documente chacun (attention :
  `modern-headings` force une police système sur les titres, donc incompatible
  avec Inter).
- **Ajouter une feuille de style** : déposer un `.scss` dans `sass/`, puis
  ajouter le `.css` correspondant à `styles` dans `[extra]` de `config.toml`.
  Zola compile `sass/*.scss` vers `static/*.css` à chaque build ; les fichiers
  produits sont ignorés par git (`.gitignore`) pour éviter les doublons avec le
  thème. L'ordre de `styles` compte : les feuilles sont chargées après
  `style.css`, dans l'ordre de la liste.
- **Navigation, pied de page, réseaux sociaux** : `[extra.nav]` et `[extra.footer]`
  de `config.toml`. Un lien du pied de page peut porter un champ `icon` : c'est une
  extension locale (`templates/partials/footer.html`), le thème v6 n'utilisant ce
  champ que pour les `socials`. La valeur doit nommer une variable `--icon-<nom>`
  du thème, par exemple `home` pour `--icon-home` ; la liste complète est dans
  `themes/duckquill/sass/icons.scss`. Le lien devient alors une icône seule, avec
  le libellé conservé en `title` et en texte pour lecteurs d'écran.
- **Commentaires Mastodon** : renseigner `host` et `user` dans `[extra.comments]`
  de `config.toml`, puis renseigner la même chose dans l'en-tête `[extra.comments]`
  de l'article concerné.
- **Statistiques** : décommenter `[extra.goatcounter]` dans `config.toml`.
- **Traductions** : les chaînes de l'interface sont dans `i18n/fr.toml` ; pour une
  autre langue, changer `default_language` puis déposer `i18n/<langue>.toml`.
- **Mise à jour du thème** : remplacer le contenu de `themes/duckquill/` par la
  dernière version de <https://codeberg.org/daudix/duckquill> (attention : Zola
  doit rester ≥ 0.23.6), puis vérifier que les surcharges de `templates/` sont
  toujours compatibles et que les variables écrasées dans `sass/custom.scss`
  existent encore (noms `--bg-color`, `--glass-bg`, `--bg-overlay` et liste des
  mods dans `themes/duckquill/sass/mods/`).

## Licence

Le thème Duckquill est distribué sous licence MIT (`themes/duckquill/LICENSE.txt`).
