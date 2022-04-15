---
title: Implémentation d'un CLI pour la Elgato KeyLight
created: '2021-04-06'
modified: '2022-04-15'
language: fr
published: true
---

## Adresse IP/URL du KeyLight

La KeyLight se connecte à votre réseau WiFi.
La première étape consiste à récupérer son adresse IP.

La keylight répond aux requêtes mDNS (multicast dns).

C'est d'ailleurs indiqué dans leur documentation:

* [What Communication Protocol Is Used by Elgato Wi-Fi Products?](https://help.elgato.com/hc/en-us/articles/360060048331-What-Communication-Protocol-Is-Used-by-Elgato-Wi-Fi-Products)
* [mDNS Service Strings for Elgato Devices](https://help.elgato.com/hc/en-us/articles/4413403384845-mDNS-Service-Strings-for-Elgato-Devices)

Pour obtenir l'IP du keylight, il suffit donc d'emettre une requête DNS:

```shell
dig -p 5353 PTR _elg._tcp.local @224.0.0.251

; <<>> DiG 9.16.15-Ubuntu <<>> -p 5353 PTR _elg._tcp.local @224.0.0.251
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18533
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 4

;; QUESTION SECTION:
;_elg._tcp.local.		IN	PTR

;; ANSWER SECTION:
_elg._tcp.local.	10	IN	PTR	Elgato\032Key\032Light\032Air\032F3DF._elg._tcp.local.

;; ADDITIONAL SECTION:
Elgato\032Key\032Light\032Air\032F3DF._elg._tcp.local. 10 IN SRV 0 0 9123 elgato-key-light-air-f3df.local.
Elgato\032Key\032Light\032Air\032F3DF._elg._tcp.local. 10 IN TXT "mf=Elgato" "dt=200" "id=3C:6A:9D:16:12:45" "md=Elgato Key Light Air 20LAB9901" "pv=1.0"
elgato-key-light-air-f3df.local. 10 IN	A	192.168.1.11
elgato-key-light-air-f3df.local. 10 IN	AAAA	fe80::3e6a:9dff:fe16:1245

;; Query time: 120 msec
;; SERVER: 192.168.1.11#5353(224.0.0.251)
;; WHEN: Fri Apr 15 13:55:25 CEST 2022
;; MSG SIZE  rcvd: 254
```

On obtient l'URL d'accès au Keylight `Elgato\032Key\032Light\032Air\032F3DF._elg._tcp.local.`, son IP `192.168.1.11` ainsi que son port d'écoute `9123`

## Découverte de l'API

La deuxième étape est de comprendre comment fonctionne l'API du KeyLight.

Elle très simple.

Un `GET /elgato/lights` permet de récupérer l'état de la KeyLight:

```
GET /elgato/lights

{
  "numberOfLights": 1,
  "lights": [
    {
      "on": 1,
      "brightness": 100,
      "temperature": 167
    }
  ]
}
```

Nous avons donc 3 champs intéressants :
* `on` indique si la KeyLight est allumée (1) ou éteinte (0)
* `brightness` indique la puissance de la luminausité (de 0 à 100)
* `temperature` indique la température de la lumière (valeurs de 143 à 344) pour une couleur entre 2900 et 7000 kelvin

Une requête `PUT` permet de mettre à jour la KeyLight.

### Extinction de la KeyLight

```
PUT /elgato/lights

{
  "lights": [
    {
      "on": 0
    }
  ]
}
```

### Alumage de la KeyLight

```
PUT /elgato/lights

{
  "lights": [
    {
      "on": 1
    }
  ]
}
```

### Modification de la luminosité

```
PUT /elgato/lights

{
  "lights": [
    {
      "brightness": 20
    }
  ]
}
```

### Modification de la temperature

```
PUT /elgato/lights

{
  "lights": [
    {
      "temperature": 200
    }
  ]
}
```

## Développement d'un CLI en Kotlin

J'ai choisi pour m'amuser de développer un CLI en Kotlin, avec compilation native.

Ce n'est probablement pas le setup le plus pratique, mais cela permet de mettre en place la compilation native avec GraalVM.

Le CLI devra supporter les commandes suivantes :

```
keylight on
keylight off
keylight brightness <X>
keylight color <Y>
```

### Installation de GraalVM

L'installation de GraalVM est plutôt simple (comme un JDK habituel).

J'ai téléchargé le `.tar.gz` sur Github ici : https://github.com/graalvm/graalvm-ce-builds/releases/tag/vm-21.0.0.2

J'ai décompressé le tar dans mon répertoire `/opt`

J'ai aussi fait pointer ma variable `JAVA_HOME` au bon endroit :

```
~ > echo $JAVA_HOME
/opt/graalvm-ce-java11-21.0.0.2
```

### Création du projet

Dans IntelliJ, j'ai créé un nouveau projet Kotlin/Maven.
Je récupère donc un squelette de projet composé d'un `pom.xml`, et d'un `main.kt`.

Il n'y a plus qu'a combler les trous !

### Configuration de la compilation native GraalVM

Il faut tout d'abord installer l'outil `native-image` :

```
/opt/graalvm-ce-java11-21.0.0.2/bin > sudo ./gu install native-image
Downloading: Component catalog from www.graalvm.org
Processing Component: Native Image
Downloading: Component native-image: Native Image  from github.com
Installing new component: Native Image (org.graalvm.native-image, version 21.0.0.2)
```

(j'exécute cette commande avec sudo, car mon répertoire /opt est protégé)

Ensuite, on va utiliser le plugin maven `native-image-maven-plugin` pour produire l'image native de notre projet.
On ajoute le bloc de configuration dans notre `pom.xml`:

```xml
<!-- building native image with graal VM -->
<plugin>
    <groupId>org.graalvm.nativeimage</groupId>
    <artifactId>native-image-maven-plugin</artifactId>
    <version>${graalvm.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>native-image</goal>
            </goals>
            <phase>package</phase>
        </execution>
    </executions>
    <configuration>
        <imageName>keyligh</imageName>
        <mainClass>MainKt</mainClass>
        <buildArgs>
            --no-fallback
            --enable-https
        </buildArgs>
    </configuration>
</plugin>
```

Les paramètres importants sont :
* imageName : le nom du binaire produit
* mainClass : le nom de la classe Java "point d'entrée" de notre application. Ici, notre fichier main.kt qui contient la fonction kotlin `main` sera transformé à la compilation en classe Java `MainKt`.
* --no-fallback : permet d'empêcher de produire un simple jar si la compilation native échoue
* --enable-https : permet d'activer le mode HTTPS (même si on ne l'utilise pas, le client Java HTTPClient instancie le mode par défaut)

La doc complète du plugin est ici https://www.graalvm.org/reference-manual/native-image/NativeImageMavenPlugin/

### Implémentation

L'implémentation du code est très basique, j'ai choisi d'utiliser la librairie `jmdns` pour exécuter les requêtes mDNS depuis le CLI, et le framework `picocli` pour implémenter la gestion des commandes et des paramètres.

Le code est disponible sur Github dans le repository [juwit/keylight-cli](https://github.com/juwit/keylight-cli/).

J'ai aussi configuré `picocli` pour la compilation native avec un l'annotation processor `picocli-codegen`, en m'inspirant de leur documentation [Picocli on GraalVM](https://picocli.info/picocli-on-graalvm.html) : 

```xml
 <plugin>
    <groupId>org.jetbrains.kotlin</groupId>
    <artifactId>kotlin-maven-plugin</artifactId>
    <version>1.4.31</version>
    <executions>
        <execution>
            <id>kapt</id>
            <goals>
                <goal>kapt</goal>
            </goals>
            <configuration>
                <sourceDirs>
                    <sourceDir>src/main/kotlin</sourceDir>
                </sourceDirs>
                <annotationProcessorPaths>
                    <annotationProcessorPath>
                        <groupId>info.picocli</groupId>
                        <artifactId>picocli-codegen</artifactId>
                        <version>${picocli.version}</version>
                    </annotationProcessorPath>
                </annotationProcessorPaths>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Conclusion

Ce petit projet m'a permis:

* d'outiller mon Keylight d'Elgato avec un CLI sous Linux (que je vais pouvoir cabler avec mon Streamdeck)
* de découvrir le protocole mDNS !
* et de tester `picocli` et la compilation native de `GraalVM`

Rien de très compliqué, mais de bout en bout, j'y ai bien passé 8 à 10 heures. Le résultat est plutôt cool, et je me sert du CLI presque tous les jours !