---
title: Anatomie d'une requête HTTP
created: '2023-05-25'
modified: '2023-07-01'
language: fr
tags:
  - Basics
---

# Anatomie d'une requête HTTP

![](/assets/2023-05-25-anatomie-requete-http/http-anatomy.jpg)

HTTP, pour _Hypertext Transfer Protocol_, est le protocole principal pour les échanges internet. Il est utilisé aussi bien par le navigateur que vous utilisez pour lire cet article, que pour faire communiquer des applications.
Il s'appuie sur un éhange de requête et réponse, entre un client et un serveur, au format texte. L'avantage du format texte est qu'il est facile à implémenter dans tous les langages de programmation.
Le protocole HTTP est spécifié par la [RFC 2616](https://www.rfc-editor.org/rfc/rfc2616), protocole dont la toute première version d'HTTP date de 1990.

Cet article traite de la version 1.1 du protocole, qui a été normalisée en 1997. Une version 2.0 a été normalisée en 2015, et étend la version 1.1 avec d'autres capacités. Les bases de la version 1.1 ne sont donc pas obsolètes&nbsp;!

![](/assets/2023-05-25-anatomie-requete-http/client-server.png)

Dans cet article, nous allons étudier la structure d'une requête et d'une réponse HTTP.

## La structure d'une requête

Une requête est structurée en 3 parties&nbsp;:

* la ligne de requête, obligatoire, en première ligne
* les entêtes optionnels
* le corps de requête, lui aussi optionnel

```
REQUEST-LINE
[HEADERS]

[BODY]
```

Exemple de requête HTTP _GET_ sans corps de requête&nbsp;:

```
GET /articles/http-anatomy?version=1#headers HTTP/1.1
Host: ekite.tech
Accept: text/plain
Accept-Charset: utf-8
Accept-Encoding: gzip
Accept-Language: fr-fr
```

Exemple comportant un corps de requête&nbsp;:

```
POST /articles/ HTTP/1.1
Host: ekite.tech
Content-Type: application/json
Content-Length: 31
Authorization: Basic UmljayBBc3RsZXk6TmV2ZXIgR29ubmEgR2l2ZSBZb3UgVXAK

{"title":"mon nouvel article"}
```

### La _Request Line_

Les _HEADERS_ et _BODY_ sont facultatifs.

La _REQUEST-LINE_ est définie de cette manière&nbsp;:

```
METHOD REQUEST-URI HTTP-VERSION
```

_METHOD_ est le verbe HTTP. Il peut être `GET`, `POST`, `PUT`, `DELETE`, qui sont les verbes HTTP les plus courants.
La RFC 2616 définit aussi des méthodes supplémentaires, comme `HEAD`, `OPTIONS`, `TRACE`, `CONNECT`.

_REQUEST-URI_ est la ressource à appeler. Elle peut être une URI absolue, comme `https://ekite.tech/articles/http-anatomy`, ou une URI relative, comme `/articles/http-anatomy`. En pratique, ce sont plutôt des URI relatives qui sont utilisées, en combinaison avec un _Header_ `Host`.

Le champ `HTTP-VERSION` vaudra `HTTP/1.1`.

Une requête HTTP minimale est&nbsp;:

```
GET /articles/http-anatomy HTTP/1.1
```

### _URL_ et _URI_

Une _URI_, pour _Uniform Resource Identifier_, définit une ressource internet. Cette ressource peut être une page Web, un document au format PDF, ou encore une vidéo.

Une _URL_, pour _Uniform Resource Locator_, est une _URI_ particulière, qui contient les informations réseau pour accéder à une ressource.
Les _URI_ sont définies par un format texte composé de caractères _ASCII_.

La [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986) spécifie les formats des _URI_.

La structure d'une _URI_ absolue est&nbsp;:

```
SCHEME://HOST/PATH[?QUERY][#FRAGMENT]
```
Le _SCHEME_ est souvent associé au protocole, _http_, _https_, ou encore _file_ quand des fichiers locaux sont désignés.
Le _HOST_ est le nom du serveur hébergeant la ressource.
Le _PATH_ identifie le chemin vers la ressource.
La _QUERY_, aussi souvent appelé _Query Strings_, définit des paramètres supplémentaires que la ressource peut traiter, sous la forme `name=value`.
Le _FRAGMENT_ définit la partie de la ressource consultée.
_QUERY_ et _FRAGMENT_ sont optionnels.

Une _URI_ simple pour notre article est&nbsp;:

```
https://ekite.tech/articles/http-anatomy
\___/   \________/ \___________________/
  |          |              |
scheme      host           path
```

Une _URI_ comportant des éléments supplémentaires&nbsp;:

```
https://ekite.tech/articles/http-anatomy?version=1#headers
\___/   \________/ \___________________/ \_______/ \_____/
  |          |              |                |        |
scheme      host           path            query    fragment
```

Une _URI_ peut également être relative, et ne comporter qu'une partie du _PATH_.
Par exemple&nbsp;:

```
/http-anatomy?version=1#headers
```

### Les _Headers_ de requête

Les _Headers_ définissent des informations supplémentaires à propos de la requête, comme le format de données attendu en réponse, ou des informations concernant les données actuellement en cache côté client.

Cette structure définit les _Headers_&nbsp;:

```
NAME: VALUE
```

La [RFC 2616](https://www.rfc-editor.org/rfc/rfc2616#page-38), définit des _Headers_ standards, mais les applications peuvent également définir leurs propres _Headers_.
Les principaux _Headers_ standards sont `Accept`, `Accept-Charset`, `Accept-Encoding` et `Accept-Language` pour la négociation de contenu.
Cette RFC définit aussi le _Header_ `Authorization` pour l'authentification des clients. Le _Header_ `Host` précise l'hôte auquel la requête est envoyée s'il n'est pas précisé dans l'URI de la requête.

Une requête HTTP comportant des _Headers_ est&nbsp;:

```
GET /articles/http-anatomy HTTP/1.1
Host: ekite.tech
Accept: text/plain
Accept-Charset: utf-8
Accept-Encoding: gzip
Accept-Language: fr-fr
```

### Le _Body_

Lorsqu'une requête comporte un corps, les _Headers_ `Content-Type` et `Content-Length` doivent être précisés et définir son type et sa taille en octets.
Un saut de ligne doit également séparer les _Headers_ du _Body_.

Exemple de requête avec un _Body_ au format `JSON`&nbsp;:

```
POST /articles/ HTTP/1.1
Host: ekite.tech
Content-Type: application/json
Content-Length: 31

{"title":"mon nouvel article"}
```
## La structure d'une réponse

Les réponses HTTP ont un format assez similaire à celui des requêtes.
```
STATUS-LINE
[HEADERS]

[BODY]
```

### La _Status Line_

La _Status Line_ représente la réponse codifiée du serveur à la requête.

_STATUS-LINE_ est définie de cette manière&nbsp;:

```
HTTP-VERSION CODE PHRASE 
```

Le premier élément de la réponse est la version HTTP `HTTP/1.1`

Les codes utilisés sont représentés par 3 chiffres.

* Les codes `1xx` sont informationnels et rarement utilisés en pratique.
* Les codes `2xx` représentent une opération en succès.
* Les codes `3xx` indiquent une redirection.
* Les codes `4xx` indiquent une erreur du client (requête mal formée, ressource non existante).
* Les codes `5xx` représentent une erreur du serveur dans le traitement de la requête.

Les codes sont suivis d'une _Phrase_ descriptive.

Les codes les plus courants et des phrases associées sont définis dans la [RFC 2616](https://www.rfc-editor.org/rfc/rfc2616#section-6.1.1).

Exemple de réponse en erreur&nbsp;:

```
HTTP/1.1 404 Not Found
```

Exemple de réponse en succès&nbsp;:

```
HTTP/1.1 200 OK
```

### Les _Headers_ de réponse

Au même titre qu'une requête, une réponse peut contenir des _Headers_. Ils définissent le type et la taille du _Body_ de réponse, ou des paramètres pour indiquer au client que la réponse peut être stockée en cache. D'autres _Headers_ définissent la ressource à appeler en cas de redirection (code `3xx`).

Les _Headers_ de réponse sont définis de la même manière que les requêtes&nbsp;:

```
NAME: VALUE
```

Exemple de réponse HTTP comportant un _Header_ `Location` pour une redirection vers une page `index.html`&nbsp;:

```
HTTP/1.1 301 Moved Permanently
Location: index.html
```

### Le _Body_ de réponse

Le _Body_ est définit de la même manière que les requêtes, et impose également les mêmes règle de présence des _Headers_ `Content-Type` et `Content-Length`.

Exemple de réponse en succès comportant le contenu de notre article&nbsp;:

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 31

{"title":"mon nouvel article"}
```

Exemple d'une erreur serveur, avec une page HTML la représentant&nbsp;:

```
HTTP/1.1 500 Internal Server Error
Content-Type: text/html
Content-Length: 56

<html><body><h1>Erreur de traitement</h1></body></html>
```
## Conclusion

Le protocole HTTP décrit un format texte, sous la forme de requête et réponse, facile à implémenter dans tous les langages de programmation.
Sa simplicité et son expressivité sont la clé de son succès depuis les années 90, jusqu'aux nouvelles versions qui continuent de s'appuyer sur le même format.

## Références

* [RFC 2616](https://www.rfc-editor.org/rfc/rfc2616)&nbsp;: Hypertext Transfer Protocol -- HTTP/1.1
* [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986)&nbsp;: Uniform Resource Identifier (URI) : Generic Syntax
* [RFC 7540](https://www.rfc-editor.org/rfc/rfc7540)&nbsp;: Hypertext Transfer Protocol Version 2 (HTTP/2)