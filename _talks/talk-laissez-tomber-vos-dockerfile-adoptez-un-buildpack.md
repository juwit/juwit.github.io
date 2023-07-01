---
title: Talk - Laissez tomber vos Dockerfile, adoptez un buildpack !
language: fr
article: false
date: 2023-06-29
events:
  - Suny Tech 2023
type: Talk
tags: 
  - DevOps
  - Docker
  - Buildpacks
abstract: |+
  Depuis plusieurs années maintenant, Docker est utilisé par toute l'industrie de l'IT pour packager et déployer des applications.

  Bien que l'écriture d'un Dockerfile soit facile, la construction d'images OCI/Docker reste un exercice compliqué:
  * optimisation des layers de l'image
  * bonne gestion des processus Linux
  * séparation des phases de build et de run des images
  * bonnes pratiques de sécurité

  Pire, lorsqu'une faille de sécurité est détecté dans une layer basse (distribution ou runtime) d'une image applicative, il faut alors potentiellement reconstruire plusieurs dizaines ou centaines d'images pour y intégrer les version patchées.

  Dans ce talk, nous apprendrons comment les buildpacks permettent de construire des images OCI/Docker sans _Dockerfile_ et bénéficier des bonnes pratiques issues de la communauté open-source.

  Nous verrons :
  * ce qu'est une image OCI, une layer, et comment Docker les construit
  * comment analyser le contenu des layers d'une image OCI, et ce qui ne va pas dans les images que nous construisons au quotidien
  * ce qu'est un buildpack et comment un buildpack construit une image OCI
  * avec une démo, comment utiliser un buildpack proposé par la communauté open-source pour contruire une image OCI contenant une application NodeJS ou Java optimisée
  * enfin, nous verrons comment les buildpacks proposent de _rebaser_ des image, et nous permettre de patcher en masse des images applicatives pour corriger des failles de sécurité, sans rebuilder complètement nos images !

  Ce talk est donc à destination des Ops et des Devs qui manipulent Docker au quotidien.

  À la sortie de ce talk, je devrai vous avoir convaincu d'abandonner vos Dockerfile et d'expérimenter les buildpacks !
---
