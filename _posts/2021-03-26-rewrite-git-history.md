---
title: Réécrire une branche git
created: '2021-03-26'
modified: '2022-04-22'
language: fr
canonical_url: https://codeka.io/2021/03/26/rewrite-git-history/
tags:
  - DevOps
  - Git
---

Je suis tombé sur un cas où un fichier a été ajouté dans git (commité), puis modifié par plusieurs commits successifs.

Malheureusement, ce fichier contient des credentials.

On va donc devoir supprimer ce fichier de tout l'historique git (oui ça implique une réécriture de la sainte branche `main` 😇).

## Trouver dans quel commit le fichier a été ajouté

Le fichier que je recherche s'appelle `config.json`.

Je vais faire un git log, pour trouver le commit qui a ajouté ce fichier.

On peut passer à la commande `git log` le nom du fichier à filtrer, ce qui nous permet d'avoir l'historique précis de ce fichier parmi tous les commits:

```shell
$ git log
d33736456c70ac9859da1e1f899c964db6f98cff (HEAD -> main, origin/main) 🔧 : update configuration
31157b8ce772c4b058d042fffef331dca87dadf5 ✨ : render button on http request
92587cbadee1063b9a9692cee014e362d99c1452 ✨ : add switch to page
de63402ee023daedb4f18cc916344ac170dcfdfd ♻ : use xdotool to write text instead of robotgo
15df7fd6e19c392dd29529b1b3b46c40961710bd ⚗️ : experiment with config loading
7f5f2abdfd67c588bb9c52b8c9c8dd4a6468a833 🙈 : add binary to .gitignore
dad0c5ec3d4b7334986b6aca4c483b1351cfa56a 📝 : add dependencies & build instructions
605e850fa4dffb49a14937400166c76b07ba817c 🙈 : add .gitignore
9a9e99a3bfd43b891bef3e2b16661d7b56dcf844 📝 : add README.md

```

```shell
$ git log config.json
d33736456c70ac9859da1e1f899c964db6f98cff (HEAD -> main) 🔧 : update configuration
15df7fd6e19c392dd29529b1b3b46c40961710bd ⚗️ : experiment with config loading
```

Ça filtre sur mon fichier, cool !

L'option `--diff-filter` permet de filtrer le log sur certaines opérations, pour l'ajout (`A`), modification (`M`) ou suppression (`D`) par exemple (voir la [doc](https://git-scm.com/docs/git-log#Documentation/git-log.txt---diff-filterACDMRTUXB82308203))

```shell
$ git log --diff-filter=A config.json
15df7fd6e19c392dd29529b1b3b46c40961710bd ⚗️ : experiment with config loading
```

C'est mon commit `15df7f` qui a créé le fichier !

## Remonter le temps

Maintenant que nous avons l'hisorique de notre fichier, l'idée est remonter le temps, et de réécrire l'ensemble des commits qui ont touché à ce fichier.

C'est une opération destructrice, qui va mettre à jour probablement pas mal de commits (seulement 2 dans notre exemple), donc à manipuler avec précaution.

Pour ré-écrire des branches git, on utilise la commande `git filter-branch` (voir la [doc](https://git-scm.com/docs/git-filter-branch)).

`git filter-branch` permet de ré-écrire de contenus ou des messages de commit.

`git filter-branch` prend en paramètre un nom de branche, ou un ensemble de révisions pour lister les commits à parcourir, voici quelques exemples:

* `HEAD` référence la branche courante (toute la branche sera parcourue)
* `main` permet de jouer le filtre sur la branche main.
* `HEAD~10..HEAD` permet de jouer le filtre sur les 10 derniers commits

Une option m'intéresse pour mon cas: `--tree-filter`.
Cette option permet de jouer une commande pour chaque commit de la branche, et re-commiter le résultat automatiquement, pratique !

Voici donc une commande qui permet de supprimer un fichier sur l'ensemble des commits de la branche courante:
```
$ git filter-branch --tree-filter 'rm -f config.json' HEAD
```

il est aussi possible de jouer sur toutes les branches avec l'option `--all`

```
$ git filter-branch --tree-filter 'rm -f config.json' -- --all
```

L'exécution prend un peu de temps, et affiche les commits modifiés : 

```
Rewrite 15df7fd6e19c392dd29529b1b3b46c40961710bd (9/13) (0 seconds passed, remaining 0 predicted)    rm 'config.json'
Rewrite de63402ee023daedb4f18cc916344ac170dcfdfd (10/13) (0 seconds passed, remaining 0 predicted)    rm 'config.json'
Rewrite 92587cbadee1063b9a9692cee014e362d99c1452 (11/13) (0 seconds passed, remaining 0 predicted)    rm 'config.json'
Rewrite 31157b8ce772c4b058d042fffef331dca87dadf5 (12/13) (0 seconds passed, remaining 0 predicted)    rm 'config.json'
Rewrite d33736456c70ac9859da1e1f899c964db6f98cff (13/13) (0 seconds passed, remaining 0 predicted)    rm 'config.json'

Ref 'refs/heads/main' was rewritten
Ref 'refs/remotes/origin/main' was rewritten
```

On a plus qu'a force push tout ça maintenant 💪
