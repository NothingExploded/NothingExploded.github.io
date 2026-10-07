# NotExp

Blog statique généré avec [Zola](https://www.getzola.org) et le thème
[Duckquill](https://duckquill.daudix.one), déployé automatiquement sur GitHub Pages.

👉 https://notexp.github.io

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

Créer un dossier dans `content/blog/` contenant un fichier `index.md` :

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
`[Contact](@/contact.md)`.

## Structure du dépôt

```
config.toml              # configuration du site (titre, URL, navigation, thème)
content/
  _index.md              # page d'accueil
  contact.md             # page « Contact »
  blog/
    _index.md            # configuration de la section blog
    <article>/index.md   # un article
i18n/
  fr.toml                # chaînes d'interface françaises (surcharge le thème)
templates/               # surcharges du thème (prioritaires sur themes/duckquill/templates/)
static/                  # fichiers copiés tels quels (favicon, .nojekyll, images)
themes/duckquill/        # thème versionné ici (pas de sous-module)
.github/workflows/deploy.yml  # CI : build Zola + déploiement GitHub Pages
```

## Intégration continue

Le workflow `.github/workflows/deploy.yml` se déclenche à chaque `push` sur `main`
(ou manuellement depuis l'onglet *Actions*) :

1. checkout du dépôt ;
2. téléchargement de Zola `0.23.6` et `zola build` ;
3. `actions/configure-pages`, puis upload de `./public` comme artefact Pages ;
4. déploiement sur GitHub Pages.

Pour activer le déploiement, dans les réglages du dépôt :
**Settings → Pages → Build and deployment → Source : GitHub Actions**.

## Personnalisation

- **Couleur d'accent** : modifier `accent_color` (thème clair) et `accent_color_dark`
  (thème sombre) dans la section `[extra]` de `config.toml`. Toute la déclinaison
  (liens, cartes, navigation, ombres) en découle.
- **Navigation, pied de page, réseaux sociaux** : `[extra.nav]` et `[extra.footer]`
  de `config.toml`.
- **Commentaires Mastodon** : renseigner `host` et `user` dans `[extra.comments]`
  de `config.toml`, puis renseigner la même chose dans l'en-tête `[extra.comments]`
  de l'article concerné.
- **Statistiques** : décommenter `[extra.goatcounter]` dans `config.toml`.
- **Traductions** : les chaînes de l'interface sont dans `i18n/fr.toml` ; pour une
  autre langue, changer `default_language` puis déposer `i18n/<langue>.toml`.
- **Mise à jour du thème** : remplacer le contenu de `themes/duckquill/` par la
  dernière version de <https://codeberg.org/daudix/duckquill> (attention : Zola
  doit rester ≥ 0.23.6), puis vérifier que les surcharges de `templates/` sont
  toujours compatibles.

## Licence

Le thème Duckquill est distribué sous licence MIT (`themes/duckquill/LICENSE.txt`).
