---
title: Talk - Self-Service infrastructure pour GCP avec Terraform et Gitlab
language: fr
article: false
date: 2022-06-10
events:
  - DevFest Lille 2022
type: Talk
tags: 
  - DevOps
  - Gitlab
  - GCP
  - Terraform
abstract: |+
  Ce REX présente comment nous utilisons Gitlab, Gitlab-CI et Terraform pour construire une infrastructure GCP en self-service pour nos utilisateurs (squads/projets).

  Chez Kiabi, dans le cadre de la migration sur le cloud GCP (depuis l'été 2021), nous mettons à disposition de nos développeurs une project-factory, en self-service, pour de l'infrastructure cloud (VM/Databases/Buckets etc...).
  Le but est d'accélérer les phases de démarrage des projets, en rendant autonomes au maximum les développeurs sur le provisionning de l'infrastructure essentielle à leurs développements.

  Nous allons voir dans ce REX comment nous avons assemblé le module Terraform google-project-factory avec Gitlab et Gitlab-CI pour:
  * créer des projets sur GCP pour plusieurs environnements en quelques minutes
  * fournir des templates de code Terraform prêts à l'emploi sur Gitlab à nos développeurs
  * utiliser des pipelines Gitlab-CI pour exécuter le code Terraform et provisionner l'infrastructure de nos projets

  Tout cela accompagné d'une démo:
  * lancement de la project-factory (code terraform + pipeline) pour créer un projet GCP + Gitlab
  * création de ressources (code terraform) dans le projet GCP nouvellement créé

  Ce REX est destiné aux développeurs curieux de l'infrastructure as code, quelques notions de Terraform, Gitlab et GCP sont les bienvenues pour bien le comprendre.
slides:
  - event: BBL
    slides: /talks/devfest2022-talk-self-service-infrastructure-gcp-terraform-gitlab.pdf
youtube: DxC8gRvqvpA
---
