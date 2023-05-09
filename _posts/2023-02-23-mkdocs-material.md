---
title: Créer un site web de documentation statique avec MkDocs
created: '2023-02-23'
modified: '2023-03-09'
language: fr
tags:
  - Git
  - Tools
  - Docs
---

# Créer un site web de documentation statique avec MkDocs

Que ce soit pour un projet d'entreprise ou un projet open-source, la documentation utilisateur et technique est cruciale.
Dans une documentation d'usage, les utilisateurs doivent pouvoir retrouver les instructions leur permettant d'accomplir les gestes métier de tous les jours.
Pour la documentation technique, les administrateurs, opérateurs et développeurs doivent pouvoir retrouver les opérations d'installation, de mise à jour, ou encore de développement du produit.

La documentation peut prendre plusieurs formes:

* un document bureautique, de type LibreOffice Writer ou Microsoft Word
* des pages dans un référentiel documentaire de type _wiki_, ou _Atlassian Confluence_
* un site web dédié

Les documents bureautique sont bien adaptés à la documentation de procédures, ou de spécifications. Cependant, ils souffrent de nombreux défauts comme le manque d'archivage des modifications, une recherche compliquée, un fort liant entre le format et le contenu et sont souvent portés par des formats propriétaires (le fameux `.docx`).

Les référentiels de type _wiki_ répondent au problèmes de recherche et d'archivage des historiques de modifications. Cependant, ils nécessitent souvent une infrastructure avec une base de données, et ont donc un coût d'hébergement et de maintenance. 

Le cas du site web statique dédié est le plus adapté à un projet open-source. Le site sera exposé sur internet, aidant à mettre en visibilité le projet. Un site web statique nécessite un hébergement minimum, sans base de données, et sera donc plus simple à mettre en place.

# _documentation-as-Code_

Pour encourager les développeurs à rédiger plus de documentation, il est intéressant de considérer l'approche *documentation-as-code*.
Cette approche s'appuie sur les process et outils de développement pour proposer aux développeurs un moyen moderne d'écrire et publier leur documentation.

Dans un processus de _documentation-as-code_, les développeurs rédigent et archivent la documentation au même endroit que le code source, dans un repository _git_.
Cela permet de fournir une documentation plus précise, et plus à jour. Les mises à jour de documentation sont effectuées en même temps que les mises à jour du code source.

Un site web de documentation pourra alors être généré à partir des documents rédigés par les développeurs. Ce site pourra être publié avec des chaînes d'intégration continue.

## Les formats _asciidoc_ et _markdown_

En _documentation-as-code_, la documentation sera écrite dans un format simple, déjà maitrisé par les développeurs. Un des avantages principaux est que ces formats peuvent être édités dans les IDE que les développeurs utilisent déjà.

Les deux formats les plus utilisés sont l'_asciidoc_ et le _markdown_.

L'intérêt d'utiliser ces formats est multiple:

* le développeur se concentre sur la rédation du contenu plutôt que sur le formattage des données
* la documentation peut être archivée comme du code source, voire dans le même repository que le projet
* ces formats sont simples d'utilisation, même pour des personnes ayant peu ou pas de connaissances techniques

## Les générateurs de sites web statiques

Des fichiers _markdown_ ou _asciidoc_ sur un repository _git_ peuvent parfois être suffisant. _Gitlab_ et _Github_ sont capables d'afficher des fichiers _markdown_ ou _asciidoc_ directement dans leur interface. C'est le cas par exemple des fameux `README.md` qu'on retrouve sur une majorité de projets.
Cet usage peut suffire pour documenter une librairie ou un projet simple. 
D'autres projets, plus compliqués, auront plus de pages de documentation, et des besoins de personnalisation, c'est là où un générateur de site web statique sera utile.

### _asciidoc_ et _asciidoctor_

La suite de cet article ne décrira pas comment mettre en place _asciidoc_ et _asciidoctor_, mais il était important de mentionner ces outils, qui sont utilisés par de nombreux projets.

_asciidoc_, couplé au générateur *asciidoctor* permet directement de générer des pages _HTML_ statiques à partir de un ou plusieurs fichiers de documentation source.
Les pages _HTML_ générées peuvent alors être publiées directement.
C'est ce qui est utilisé par le framework Java *Spring* par exemple.

Voici à titre d'exemple la [documentation générée pour le framework Spring](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/){:target="_blank"} , à partir des documents *asciidoc* disponible sur _Github_: 

![](/assets/2023-02-23-mkdocs-material/mkdocs-spring.png)


Les sources sont visibles sur le [_Github_ de _Spring_](https://github.com/spring-projects/spring-boot/tree/main/spring-boot-project/spring-boot-docs/src/docs/asciidoc){:target="_blank"}.

*Spring* utilise donc *asciidoctor* pour générer leur documentation dans 3 formats différents:

* un pdf complet
* une page HTML complète (autonome)
* un ensemble de pages HTMLs liées entre elles

### _markdown_ et _MkDocs_

Bien qu'*asciidoctor* propose de générer plusieurs formats, d'autres générateurs compatibles Markdown existent:

* [_Jekyll_](https://jekyllrb.com/){:target="_blank"}, est plutôt orienté blogging
* [_Hugo_](https://gohugo.io/){:target="_blank"}, est orienté site web généraliste
* [_MkDocs_](https://www.mkdocs.org/){:target="_blank"}, est orienté documentation

Ces générateurs amènent une structure de site web classique, avec une navigation à plusieurs niveaux de menus, et souvent des champs de recherche.

Parmis ces 3 solutions, _MkDocs_ est le plus orienté documentation, c'est cet outil que nous allons mettre en place dans la suite de cet article.

# Mise en place de _MkDocs_

Nous allons voir dans la suite de cet article:

* l'installation et la configuration de _MkDocs_
* l'utilisation et la configuration d'un thème et sa customisation (logo, couleurs)
* la rédaction de quelques pages
* la publication en site statique avec _Github Actions_ & _Github Pages_

# Création du projet de documentation

La documentation peut être créée comme un repository Git dédié, ou comme un répertoire d'un repository existant, dans le cas d'une approche _mono-repo_ par exemple.

C'est cette deuxième solution que nous allons suivre.
Dans un projet existant, on commence par créer un répertoire `docs`:

```bash
mkdir docs/
```

La création du squelette de documentation peut se faire avec le _CLI_ (pour Command Line Interface) de _MkDocs_ et la commande [`mkdocs new`](https://www.mkdocs.org/user-guide/cli/#mkdocs-new), ou simplement en initialisant quelques fichiers à la main.

Nous allons créer l'arborescence suivante:

```text
.
├─ docs/
│  └─ index.md
└─ mkdocs.yml
```

Le contenu de notre fichier `mkdocs.yml` minimal est le suivant:

```yaml
site_name: Galactic Empire
```

Pour référence, la [documentation de _MkDocs_](https://www.mkdocs.org/user-guide/configuration/){:target="_blank"} liste les paramètres de configuration qui peuvent être modifiés dans le fichier `mkdocs.yml`.

Un fichier `docs/index.md` permet de servir de page d'accueil:

```markdown
# Documentation

Bienvenue sur la documentation de l'Empire Galactique
```

# Démarrage de _MkDocs_ en mode preview

_MkDocs_ est écrit en python, et peut être installé en quelques commandes. La [procédure d'installation](https://www.mkdocs.org/user-guide/installation/#installing-mkdocs){:target="_blank"} est simple:

```bash
pip install mkdocs
```

Il existe également des images _Docker_ dédiées, qui seront utiles pour l'automatisation avec _Github Actions_ ou _Gitlab CI_ pour la publication, bien que nous allons nous en passer.

Le démarrage du site, en mode _preview_, se fait en une commande:

```bash
$ mkdocs serve
INFO     -  Building documentation...
INFO     -  Cleaning site directory
INFO     -  Documentation built in 0.15 seconds
INFO     -  [09:32:45] Watching paths for changes: 'docs', 'mkdocs.yml'
INFO     -  [09:32:45] Serving on http://127.0.0.1:8000/
```

Le site web est construit, et disponible en local:

![](/assets/2023-02-23-mkdocs-material/create-docs-website-preview.png)

# Ajout d'un thème & customisation

Le thème par défaut n'est pas très élégant.
Plusieurs thèmes existent, et il est possible de créer le sien, à condition de savoir un peu développer en HTML/CSS.

Néanmoins, un thème populaire est [_Material for MkDocs_](https://squidfunk.github.io/mkdocs-material){:target="_blank"} (13k stars sur _Github_). Ce thème propose une interface épurée, personnalisable, et le support d'extensions à _markdown_.

L'installation du thème se fait en une ligne de commande:

```bash
pip install mkdocs-material
```

Pour utiliser notre thème, il faut le préciser dans le fichier `mkdocs.yml`:

```yaml
site_name: Galactic Empire

# configuration du thème
theme:
  name: material
```

Nous allons maintenant un peu customiser notre thème.
Toutes les instructions sont disponible dans la [documentation de _Material for Mkdocs_](https://squidfunk.github.io/mkdocs-material/setup/changing-the-colors/){:target="_blank"}: 

## Le logo

Pour changer le logo, il faut indiquer quel fichier utiliser dans la configuration.
Pour notre exemple, je suis allé chercher un logo libre de droits sur le site web [https://pngrepo.com](https://pngrepo.com){:target="_blank"}, et je l'ai positionné dans un répertoire `docs/assets` que j'ai créé pour l'occasion:

```
.
├── docs
│   ├── assets
│   │   └── death-star.png
│   └── index.md
└── mkdocs.yml
```

Le logo est ensuite référencé dans le fichier de configuration:

```yaml
site_name: Galactic Empire project

# configuration du thème
theme:
  name: material
  logo: assets/death-star.png
```

À noter que le répertoire _assets_ déclaré dans la configuration est bien relatif au répertoire _docs_.

## Les couleurs
La configuration des couleurs passe par une déclaration CSS, si on veut sortir des couleurs proposées dans la palette de couleurs par défaut.
Un fichier css peut surcharger l'ensemble des variables de couleurs prédéfinies. La liste des variables est disponible dans le [fichier de définition des couleurs](https://github.com/squidfunk/mkdocs-material/blob/master/src/assets/stylesheets/main/_colors.scss){:target="_blank"} du code source de _Material for MkDocs_.

Pour surcharger les couleurs principales, il suffit donc de les re-déclarer dans un fichier `.css`, de cette manière:

```css
:root {
  --md-primary-fg-color: #394A59;
  --md-accent-fg-color:  #556567;
}
```

Le fichier `.css` doit être déposé où l'on souhaite. Ici, nous avons créé un répertoire `docs/stylesheets`:

```
.
├── docs
│   ├── assets
│   │   └── death-star.png
│   ├── index.md
│   └── stylesheets
│       └── galactic-empire.css
└── mkdocs.yml
```

Enfin, le fichier `.css` doit être référencé dans la configuration de notre site:

```yaml
site_name: Galactic Empire project

# configuration du thème
theme:
  name: material
  logo: assets/death-star.png

extra_css:
  - stylesheets/galactic-empire.css
```

Et voici notre site avec le thème et sa customisation.

![](/assets/2023-02-23-mkdocs-material/create-docs-website-theme.png)

# L'ajout de contenu
L'ajout de contenu passe maintenant simplement par l'ajout de nouveaux fichiers _markdown_ dans le répertoire de documentation.

Voici un exemple de page de contenu supplémentaire:

```markdown
---
title: Organisation
author: Darth Vader
---

# Organisation de l'Empire Galactique

L'Empire Galactique est dirigé par l'Empereur Palpatine.

Son bras droit est le seigneur Vador.

Le Grand Moff Tarkin est le gouverneur régional de la Bordure Extérieur.
Il est à la tête du projet "Étoile de la mort", et son commandant.
```

L'en-tête du fichier est au format _Front Matter_, qui a été popularisé par [_Jekyll_](https://jekyllrb.com/docs/front-matter/){:target="_blank"}.

Le rendu est le suivant:

![](/assets/2023-02-23-mkdocs-material/create-docs-website-content-page.png)

Notez que :

* le titre de la page est celui du premier header Markdown `#`
* dans la navigation de gauche, le titre utilisé est celui du bloc de description de la page *Front Matter*.
* le nom de l'auteur apparaît dans les méta-données de la page générée:

```html
<head>
  ...
  <meta name="author" content="Darth Vader">
  ...
</head      
```

# Publication avec _Github Actions_ & _Github Pages_

Maintenant que notre site web est fonctionnel, nous allons le mettre à disposition sur le web avec _Github Pages_.

La création d'un repository _Github_ et les bases de _Github Actions_ sont en dehors du périmètre de cet article. 

La construction du site web se fait avec la commande `mkdocs build`.
L'exécution de cette commande produit un nouveau répertoire `site` qui contient le site web généré:

```bash
$ mkdocs build 
INFO     -  Cleaning site directory
INFO     -  Building documentation to directory: ekit3/mkdocs-website-sample/site
INFO     -  Documentation built in 0.16 seconds
```
Nous allons donc utiliser cette commmande pour générer notre site web à publier.

Voici un workflow _Github Actions_, qui installe le langage _Python_, _MkDocs_ et _Material for MkDocs_, et construit notre site:

```yaml
name: Build & Publish site

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v3
    - name: Install dependencies
      run: |
        python -m pip install mkdocs
        python -m pip install mkdocs-material
    - name: Build site
      run: |
        mkdocs build
```

L'étape suivante consiste à publier le site construit sur _Github Pages_, ce qui se fait avec 2 actions dédiées.

L'action `upload-pages-artifact` permet de sélectionner un répertoire à envoyer à Github Pages, en l'occurence, notre répertoire `site`, qui contient notre site web.

```yaml
- name: Upload Pages Artifact
  uses: actions/upload-pages-artifact@v1
  with:
    path: ./site
```

Cette action peut être ajoutée au job _build_ décrit plus haut.

Enfin, l'action `deploy-pages` permet d'exécuter le déploiement, nous l'ajoutons dans un job séparé:

```yaml
jobs:
  build:
    [...]
  deploy:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v1
```

Le fichier final complet est disponible sur [le repository _Github_](https://github.com/ekit3/mkdocs-website-sample/blob/main/.github/workflows/publish.yml){:target="_blank"} contenant le code source de cet article.

À noter que le déploiement avec _Github Pages_ doit être activé dans le repository _Github_, et que le repository doit être public :

![](/assets/2023-02-23-mkdocs-material/create-docs-website-github-pages.png)

Une fois le fichier Github Actions publié, le workflow pourra s'exécuter, et publier le site web:

![](/assets/2023-02-23-mkdocs-material/create-docs-website-workflow.png)

Le résultat final est visible [via ce lien](https://ekit3.github.io/mkdocs-website-sample/){:target="_blank"}.

Le code source de cet article est disponible dans notre repository _Github_ [mkdocs-website-sample](https://github.com/ekit3/mkdocs-website-sample){:target="_blank"}

# Conclusion

En conclusion, nous avons vu les principes de documentation-as-code, et qu'il était très simple de publier une documentation sous la forme d'un site web. 
Nous avons également vu comment utiliser le format _markdown_, couplé à l'outil _MkDocs_ et le thème _Material for MkDocs_ pour générer un site web et le personnaliser.
Enfin, nous avons vu comment utiliser _Github Actions_ et _Github Pages_ pour publier notre site de documentation sur internet.

_MkDocs_, couplé à un hébergement avec _Gitlab Pages_ ou _Github Pages_, permet de générer un site web en quelques étapes rapides.
Cela en fait l'outil idéal pour documenter un projet open-source ou interne.

# Liens

* [Documentation](https://www.mkdocs.org/){:target="_blank"} de _MkDocs_
* [Documentation](https://squidfunk.github.io/mkdocs-material/){:target="_blank"} de _Material for MkDocs_
* [La customisation](https://squidfunk.github.io/mkdocs-material/setup/changing-the-colors/){:target="_blank"} de _Material for MkDocs_ (couleurs, fonts, icônes)
* L'icône _Death Star_ utilisée pour le site généré sur [pngrepo.com](https://www.pngrepo.com/svg/275952/death-star-star-wars){:target="_blank"}
* Les actions Github utilisées:
    * [upload-pages-artifact](https://github.com/actions/upload-pages-artifact){:target="_blank"}
    * [deploy-pages](https://github.com/actions/deploy-pages){:target="_blank"}
* Le [code source](https://github.com/ekit3/mkdocs-website-sample){:target="_blank"} associé à cet article