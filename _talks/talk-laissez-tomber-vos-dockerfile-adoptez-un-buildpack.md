---
title: Talk - Laissez tomber vos Dockerfile, adoptez un buildpack&nbsp;!
language: fr
layout: talk
article: true
date: 2023-06-29
events:
  - Sunny Tech 2023
  - Cloud Nord 2023
type: Talk
tags: 
  - DevOps
  - Docker
  - Buildpacks
abstract: |+
  Depuis plusieurs années maintenant, _Docker_ est utilisé par toute l'industrie de l'IT pour packager et déployer des applications.

  Bien que l'écriture d'un `Dockerfile` soit facile, la construction d'images _OCI_/_Docker_ reste un exercice compliqué:
  * optimisation des layers de l'image
  * bonne gestion des processus Linux
  * séparation des phases de build et de run des images
  * bonnes pratiques de sécurité

  Pire, lorsqu'une faille de sécurité est détecté dans une layer basse (distribution ou runtime) d'une image applicative, il faut alors potentiellement reconstruire plusieurs dizaines ou centaines d'images pour y intégrer les version patchées.
  
  Dans ce talk, nous apprendrons comment les **buildpacks** permettent de construire des images OCI/Docker sans _Dockerfile_ et bénéficier des bonnes pratiques issues de la communauté open-source.

  Nous verrons :
  * ce qu'est une image _OCI_, une layer, et comment _Docker_ les construit
  * comment analyser le contenu des layers d'une image _OCI_, et ce qui ne va pas dans les images que nous construisons au quotidien
  * ce qu'est un **buildpack** et comment un **buildpack** construit une image OCI
  * avec une démo, comment utiliser un **buildpack** proposé par la communauté open-source pour construire une image _OCI_ contenant une application _Java_ optimisée
  * enfin, nous verrons comment les **buildpacks** proposent de _rebaser_ des image, et nous permettre de patcher en masse des images applicatives pour corriger des failles de sécurité, sans reconstruire complètement nos images !

  Ce talk est donc à destination des _Ops_ et des _Devs_ qui manipulent _Docker_ au quotidien.

  À la sortie de ce talk, je devrai vous avoir convaincu d'abandonner vos Dockerfile et d'expérimenter les buildpacks !
slides: 
  - event: Sunny Tech 2023
    slides: /talks/sunnytech2023-talk-laissez-tomber-vos-dockerfile-adoptez-un-buildpack.pdf
  - event: Cloud Nord 2023
    slides: /talks/cloudnord2023-talk-laissez-tomber-vos-dockerfile-adoptez-un-buildpack.pdf
youtube: 2Zo34sXsMxU
---
