---
layout: post
title: direnv pour booster votre shell
created: '2022-06-17'
language: fr
categories: devops
tags: shell
---

# direnv pour booster votre shell
## Le problème
Je suis le genre de développeur qui travaille avec un terminal ouvert sur le côté, en plus de mon IDE.
Je lance souvent des commandes `mvn` pour m'assurer que mon projet compile et que mes tests s'exécutent correctement. C'est un vieux réflexe qui date de l'époque où les IDE n'avaient qu'un support limité de maven, mais lancer ces commandes hors-IDE m'aident souvent à valider que tout fonctionnera bien dans un environnement de CI par exemple.
J'ai donc parfois besoin de changer de version de *Java* en fonction du projet dans lequel je me trouve.
*Maven* utilise la variable d'environnement `JAVA_HOME` pour localiser l'installation de *Java* à utiliser. Donc être capable de charger des variables d'environnement différentes en fonction d'un projet peut s'avérer pratique.
Un autre usage courant consiste à venir charger des clé d'API ou des secrets d'accès cloud comme des variables `AWS_ACCESS_KEY` ou autres en variables de mes différents projets.
## direnv
`direnv`  ([lien](https://direnv.net/)) est un outil écrit en go qui permet de charger des variables d'environnement dans la session courante du terminal, lorsqu'on change de répertoire en effectuant un `cd` .
### Installation
`direnv` est disponible dans les dépots de nombreuses distribution, l'installation sur ubuntu se fait avec les commandes habituelles, à savoir `apt install direnv`.
### Configuration
Une fois installé, il faut demander au shell que vous utilisez d'appeler le binaire `direnv` à chaque changement de répertoire.
Le shell que j'utilise au quotidien est `zsh`. Pour `zsh`, la configuration de `direnv` consiste à venir modifier mon fichier `~/.zshrc` pour y ajouter la ligne suivante:
```shell
eval "$(direnv hook zsh)"
```
Les procédures de configuration pour les autres shells sont détaillées [sur le site de direnv](https://direnv.net/docs/hook.html), et elles sont du même ordre que la procédure ci-dessus.
## Utilisation basique
L'utilisation de direnv se fait au travers d'un fichier `.envrc` à positionner dans le répertoire souhaité.
Ce fichier peut contenir :
* des commandes `export` pour déclarer des variables d'environnement
* des appels de fonction de la stdlib `direnv` 
* des appels de fonction customisés
* du code shell (bash)

Un exemple de fichier `.envrc` déposé dans un répertoire `~/workspaces/demo-direnv`:
```shell
export JAVA_HOME=/opt/jdk-17.0.1+12
export MAVEN_HOME=/opt/apache-maven-3.8.4
```
À l'entrée dans le répertoire, `direnv` va tenter de charger le fichier. Les variables d'environnement exportées par le fichier seront alors chargées dans la session courante.
Au premier chargement d'un fichier, où après une modification, `direnv` demandera une validation explicite pour autoriser le fichier.
```shell
~/workspaces > cd demo-direnv 
direnv: error /home/jwittouck/workspaces/demo-direnv/.envrc is blocked. Run `direnv allow` to approve its content             
```
J'exécute donc `direnv allow` pour autoriser le chargement de mon fichier `.envrc`:
```shell
~/workspaces/demo-direnv > direnv allow
direnv: loading ~/workspaces/demo-direnv/.envrc
direnv: export +JAVA_HOME +MAVEN_HOME
~/workspaces/demo-direnv > 
```
`direnv` nous indique qu'il a chargé notre fichier, ainsi que nos deux variables d'environnement.
Nous pouvons maintenant les utiliser:
```shell
~/workspaces/demo-direnv > echo $JAVA_HOME
/opt/jdk-17.0.1+12
~/workspaces/demo-direnv > echo $MAVEN_HOME
/opt/apache-maven-3.8.4
```
Si on quitte le répertoire, les variables sont déchargées:
```shell
~/workspaces/demo-direnv > cd ..
direnv: unloading
~/workspaces > echo $JAVA_HOME
# rien ici !
```
## Modification du PATH
Pour nous simplifier la vie, `direnv` propose des fonctions qui permettent de manipuler le `PATH` facilement. La fonction `PATH_add` permet d'ajouter simplement une nouvelle valeur au `PATH`. En voici un exemple dans mon fichier `.envrc`:
```shell
export JAVA_HOME=/opt/jdk-17.0.1+12
export MAVEN_HOME=/opt/apache-maven-3.8.4

PATH_add $JAVA_HOME/bin
PATH_add $MAVEN_HOME/bin
```
Avec cette nouvelle version de mon fichier, j'ai donc ajouté mes versions de `java` et `maven` au `PATH`, et ces ajouts seront retirés à la sortie du répertoire:
```shell
~/workspaces/demo-direnv > java --version
openjdk 17.0.1 2021-10-19
OpenJDK Runtime Environment Temurin-17.0.1+12 (build 17.0.1+12)
OpenJDK 64-Bit Server VM Temurin-17.0.1+12 (build 17.0.1+12, mixed mode, sharing)
~/workspaces/demo-direnv > cd ..
direnv: unloading
~/workspaces > java --version
zsh: command not found: java
```
Avec `direnv`, je peux donc changer de version de `java` dans mon `PATH`, simplement en changeant de répertoire.

## Utilisation avancée
Pour aller plus loin, on peut s'intéresser aux fonctions disponibles dans la stdlib de  `direnv`. Ces fonctions sont listées et documentées dans la documentation de direnv [man/direnv-stdlib.1](https://direnv.net/man/direnv-stdlib.1.html).
Nous allons voir ici quelques unes de ces commandes que j'utilise régulièrement.
### [source_up](https://direnv.net/man/direnv-stdlib.1.html#codesourceup-ltfilenamegtcode)
Cette commande permet d'aller chercher et exécuter le premier fichier `.envrc` dans l'arborescence remontante des fichiers, ce qui permet de factoriser un peu le contenu de mes fichiers `.envrc`.
Un exemple concret :
* workspaces
	* .envrc
```shell
export MAVEN_HOME=/opt/apache-maven-3.8.4
PATH_add $MAVEN_HOME/bin
```
	* projet-java-11
		* .envrc
```shell
source_up
export JAVA_HOME=/opt/jdk-11.0.13+8
PATH_add $JAVA_HOME/bin
``` 
	* projet-java-17
		* .envrc
```shell
source_up
export JAVA_HOME=/opt/jdk-17.0.1+12
PATH_add $JAVA_HOME/bin
``` 

Avec cette utilisation, la variable `MAVEN_HOME` est disponible dans les sous-répertoires, et permet de factoriser ces configurations pour tous mes projets.
### [dotenv](https://direnv.net/man/direnv-stdlib.1.html#codedotenv-ltdotenvpathgtcode)
`direnv` peut aussi scruter les fichiers `.env`, qui doivent exposer les variables d'environnement sous forme de clé/valeur. C'est moins souple car aucun script ne peut être écrit dans ces fichiers, mais ça peut aider si des `.env` existent déjà sur un projet.
Voici un exemple de fichier `.env`:
```shell
AWS_ACCESS_KEY_ID=xxxx-xxx-xxx-xxxx
AWS_SECRET_ACCESS_KEY=xxxx-xxx-xxx-xxxx
```
Et un exemple de fichier `.envrc` qui charge le fichier `.env`:
```shell
dotenv
```
Dans le cas où beaucoup de projets utilisent des `.env`, on peut aussi configurer `direnv` pour charger ces fichiers automatiquement, en plus des `.envrc` (qui restent chargés en priorité)
Cette configuration se fait dans `~/.config/direnv/direnv.toml`:
```toml
[global]
load_dotenv = true
```
### [whitelist](https://direnv.net/man/direnv.toml.1.html#whitelist)
Par défaut, pour chaque fichier `.envrc`, nouveau ou modifié, `direnv` affichera ce message:
```text
direnv: error .envrc is blocked. Run `direnv allow` to approve its content
```
Il faut donc utiliser la commande `direnv allow` pour autoriser ces fichiers. Ça peut être assez rébarbatif, quand cela arrive sur de nombreux projets.
`direnv` fournit une configuration permettant de whitelister les fichiers, et les autoriser automatiquement. J'y ai ajouté mon répertoire de travail. Cette configuration se fait dans `~/.config/direnv/direnv.toml`:
```toml
[whitelist]
prefix = ["/home/jwittouck/workspaces"]
```
### Fonction customisées
Pour améliorer l'exemple plus haut concernant la gestion de la variable `JAVA_HOME`, il est aussi possible d'enrichir les fonctions disponibles de `direnv`.
Pour cela, il faut les ajouter au fichier `~/.config/direnv/direnvrc`.
Pour mon usage, j'ai créé cette fonction :
```shell
function use_java(){
    echo "Using Java version $1"
    export JAVA_HOME=$(find /opt -maxdepth 1 -type d -name "jdk-$1*")
    echo "Loading JAVA_HOME $JAVA_HOME"
    PATH_add $JAVA_HOME/bin
}
```
Tous mes JDK sont installés dans `/opt`, cette fonction permet de trouver le JDK qui correspond au premier paramètre de la fonction, et de positionner les variables `JAVA_HOME` et `PATH`.
Elle s'utilise dans un `.envrc` de cette manière:
```shell
use_java 17
# ou
# use_java 11
```
et voici ce que loggue `direnv` à l'entrée du répertoire :
```shell
~/workspaces > cd demo-direnv
direnv: loading ~/workspaces/demo-direnv/.envrc
direnv: using java 17
Using Java version 17
Loading JAVA_HOME /opt/jdk-17.0.1+12
direnv: export +JAVA_HOME +MAVEN_HOME ~PATH
```
## Conclusion
`direnv` est un outils très pratique pour manipuler les variables d'environnement.
Cet article a montre principalement les usages que j'en fait pour mes projets *Java*, mais `direnv` propose aussi des fonctions pour les projets `node`, `ruby`, `python` et d'autres.
C'est vite devenu un indispensable dans mes shells.
### Liens
* la page d'accueil du projet sur [direnv.net](https://direnv.net/)
* le repository Github du projet [direnv/direnv](https://github.com/direnv/direnv)
* la doc d'install [docs/installation](https://direnv.net/docs/installation.html)
* la doc de la stdlib [man/direnv-stdlib](https://direnv.net/man/direnv-stdlib.1.html)