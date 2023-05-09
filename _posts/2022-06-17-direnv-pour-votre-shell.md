---
layout: post
title: direnv pour booster votre shell
created: '2022-06-17'
modified: '2022-08-05'
language: fr
tags:
  - DevOps
  - Shell
---

# direnv pour booster votre shell
## Le problème
Je suis le genre de développeur qui travaille toujours avec un terminal ouvert sur le côté, en plus de mon IDE.
Je lance souvent des commandes `mvn` pour m'assurer que mon projet compile et que mes tests s'exécutent correctement. C'est un vieux réflexe qui date de l'époque où les IDE n'avaient qu'un support limité de *Maven*. Lancer ces commandes hors-IDE m'aide souvent à valider que tout fonctionnera bien dans un environnement de CI par exemple.
J'ai donc parfois besoin de changer de version de *Java* en fonction du projet dans lequel je me trouve.
*Maven* utilise la variable d'environnement `JAVA_HOME` pour localiser l'installation de *Java* à utiliser. Donc être capable de charger des variables d'environnement différentes en fonction d'un projet peut s'avérer pratique.
Un autre usage courant consiste à venir charger des clé d'API ou des secrets d'accès cloud comme des variables `AWS_ACCESS_KEY` ou autres en fonction de mes différents projets.
## direnv
`direnv`  ([lien](https://direnv.net/){:target="_blank"}) est un outil écrit en go qui permet de charger des variables d'environnement dans la session courante du terminal, lorsqu'on change de répertoire en effectuant un `cd` .
### Installation
`direnv` est disponible dans les dépots de nombreuses distributions Linux, l'installation sur Ubuntu se fait avec les commandes habituelles, à savoir `apt install direnv`.
L'installation pour d'autres distributions se fait à partir des dépôts, ou bien directement à partir de binaires pré-compilés à récupérer sur Github dans [les releases de direnv](https://github.com/direnv/direnv/releases){:target="_blank"}.
### Configuration
Une fois installé, il faut indiquer au shell d'appeler le binaire `direnv` à chaque changement de répertoire.
Le shell que j'utilise au quotidien est `zsh`. Pour `zsh`, la configuration de `direnv` consiste à venir modifier mon fichier `~/.zshrc` pour y ajouter la ligne suivante:
```shell
eval "$(direnv hook zsh)"
```
Les procédures de configuration pour les autres shells sont détaillées [sur le site de direnv](https://direnv.net/docs/hook.html){:target="_blank"} et sont du même ordre que la procédure ci-dessus.
## Utilisation basique
L'utilisation de `direnv` se fait au travers d'un fichier `.envrc` à positionner dans le répertoire souhaité.
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
Ce fichier déclare deux variables d'environnement qui me sont utiles dans un projet Java.
À l'entrée dans le répertoire contenant ce fichier, `direnv` va tenter de charger le fichier. Les variables d'environnement exportées par le fichier seront alors chargées dans la session shell courante.
Au premier chargement d'un fichier, ou après une modification, `direnv` demandera une validation explicite pour autoriser le fichier.
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
Pour nous simplifier la vie, `direnv` propose des fonctions qui permettent de manipuler le `PATH` facilement. La fonction `PATH_add` permet d'ajouter simplement une nouvelle valeur au `PATH`. En voici un exemple dans mon fichier `.envrc` précédent:
```shell
export JAVA_HOME=/opt/jdk-17.0.1+12
export MAVEN_HOME=/opt/apache-maven-3.8.4

PATH_add $JAVA_HOME/bin
PATH_add $MAVEN_HOME/bin
```
Avec cette nouvelle version de mon fichier, j'ai donc ajouté mes versions de `java` et `maven` au `PATH`.  Ces ajouts seront retirés à la sortie du répertoire:
```shell
# je rentre dans mon répertoire demo-direnv
~/workspaces > cd demo-direnv
direnv: loading ~/workspaces/demo-direnv/.envrc                                                                                                                                                                                                               
direnv: export +JAVA_HOME +MAVEN_HOME ~PATH

# le $PATH a été modifié
~/workspaces/demo-direnv > java --version
openjdk 17.0.1 2021-10-19
OpenJDK Runtime Environment Temurin-17.0.1+12 (build 17.0.1+12)
OpenJDK 64-Bit Server VM Temurin-17.0.1+12 (build 17.0.1+12, mixed mode, sharing)

# je sors du répertoire 
~/workspaces/demo-direnv > cd ..
direnv: unloading

# le $PATH ne contient plus la commande java
~/workspaces > java --version
zsh: command not found: java
```
Avec `direnv`, je peux donc changer de version de `java` dans mon `PATH`, simplement en changeant de répertoire.
## Utilisation avancée
Pour aller plus loin, `direnv` propose une librairie de fonctions qu'ils nomment *stdlib*. Ces fonctions sont listées et documentées dans la documentation de direnv [man/direnv-stdlib.1](https://direnv.net/man/direnv-stdlib.1.html){:target="_blank"}.
Voici quelques unes de ces commandes que j'utilise régulièrement.
### [source_up](https://direnv.net/man/direnv-stdlib.1.html#codesourceup-ltfilenamegtcode){:target="_blank"}
Cette commande permet d'aller chercher et exécuter le premier fichier `.envrc` dans l'arborescence remontante des fichiers, ce qui permet de factoriser un peu le contenu de mes fichiers `.envrc`.
Un exemple concret d'arborescence de fichiers:
```shell
.
└── workspaces
    ├── .envrc
    ├── projet-java-11
    │   └── .envrc
    └── projet-java-17
        └── .envrc
```
Le fichier `~/workspaces/.envrc` contient les variables communes à mes deux projets:
```shell
export MAVEN_HOME=/opt/apache-maven-3.8.4
PATH_add $MAVEN_HOME/bin
```
Le fichier  `~/workspaces/projet-java-11/.envrc` contient une configuration pour *Java* 11, ainsi que la commande `source_up` qui permet de charger le fichier `~/workspaces/.envrc`:
```shell
source_up
export JAVA_HOME=/opt/jdk-11.0.13+8
PATH_add $JAVA_HOME/bin
``` 
Le fichier  `~/workspaces/projet-java-17/.envrc` contient une configuration similaire pour *Java* 17:
```shell
source_up
export JAVA_HOME=/opt/jdk-17.0.1+12
PATH_add $JAVA_HOME/bin
``` 
Avec cette utilisation, la variable `MAVEN_HOME` est disponible dans les sous-répertoires. Cette configuration permet de factoriser mes fichiers `.envrc` pour tous mes projets.
### [dotenv](https://direnv.net/man/direnv-stdlib.1.html#codedotenv-ltdotenvpathgtcode){:target="_blank"}
`direnv` peut aussi scruter les fichiers `.env`, qui doivent exposer les variables d'environnement sous forme de clé/valeur. Les fichiers `.env` sont moins souple car aucun script ne peut y être écrit. La commande `dotenv` peut néanmoins aider si des `.env` existent déjà sur un projet.
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
### [whitelist](https://direnv.net/man/direnv.toml.1.html#whitelist){:target="_blank"}
Par défaut, pour chaque fichier `.envrc`, nouveau ou modifié, `direnv` affichera ce message:
```text
direnv: error .envrc is blocked. Run `direnv allow` to approve its content
```
Il faut donc utiliser la commande `direnv allow` pour autoriser ces fichiers. Cela peut être assez rébarbatif quand cela arrive sur de nombreux projets.
`direnv` fournit une configuration permettant de whitelister les fichiers et de les autoriser automatiquement. J'y ai ajouté mon répertoire de travail. Cette configuration se fait dans `~/.config/direnv/direnv.toml`:
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
Tous mes JDK sont installés dans `/opt`. Cette fonction permet de trouver le JDK qui correspond au premier paramètre de la fonction et de positionner les variables `JAVA_HOME` et `PATH`.
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
`direnv` est un outil très pratique pour manipuler les variables d'environnement. Il s'adresse avant tout à ceux qui passent du temps dans un shell, devs ou sysadmins.
Cet article a présenté principalement les usages que j'en ai pour mes projets *Java*, mais `direnv` propose aussi des fonctions pour les projets `node`, `ruby`, `python` et d'autres.
C'est vite devenu un indispensable dans mes shells.
### Liens
* la page d'accueil du projet sur [direnv.net](https://direnv.net/){:target="_blank"}
* le repository Github du projet [direnv/direnv](https://github.com/direnv/direnv){:target="_blank"}
* la documentation d'install [docs/installation](https://direnv.net/docs/installation.html){:target="_blank"}
* la documentation de la stdlib [man/direnv-stdlib](https://direnv.net/man/direnv-stdlib.1.html){:target="_blank"}