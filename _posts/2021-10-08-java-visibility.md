---
title: 'Retour aux bases: Les visibilités en Java'
created: '2021-10-08'
modified: '2022-04-15'
language: fr
published: true
---

> Ou "pourquoi j'aime le package-protected", et "pourquoi package-protected" est une bonne visibilité par défaut pour Java

## Le modificateurs de visibilité

Le language Java supporte quatre modificateurs de visibilité qui peuvent être appliqués aux classes, méthodes et attributs de classes :

* public
* private
* protected
* package-private


On retrouve souvent sur le web ce tableau comparatif des visibilités : 

| modificateur        | same package | other package | sub class <br/>in same package | sub class <br/>in other package |
|---------------------|--------------|---------------|---------------------------|----------------------------|
| public              | Y            | Y             | Y                         | Y                          |
| private             | N            | N             | N                         | N                          |
| protected           | Y            | N             | Y                         | Y                          |
| package-protected   | Y            | N             | Y                         | N                          |


Pour résumer, `public` est visbile de tout le monde, `private` uniquement à l'intérieur de la classe déclarante, `protected` à l'intérieur du package et aux sous-classes, `package-protected` uniquement à l'intérieur du package.

A noter que `package-protected` est la visibilité par défaut, lorsqu'aucun autre modificateur n'est déclaré.

Et cela est une bonne chose !

## Pourquoi pas "public" par défaut ?

On voit souvent dans beaucoup de codes, toutes les classes de packages déclarés en visibilité `public`. En partie parce que les développeurs doivent utiliser du code dans un autre module (fichier ou package) et donc ils posent cette visibilité en supposant que c'est la bonne solution.

Cependant, avoir ce réflexe de déclarer tous les objets `public` conduit à ouvrir l'intégralité du code d'un module, là où cela n'est pas nécessaire.

## Les architectures à couche "techniques"

On retrouve souvent ce type de découpage, avec des couches controller/service/repository.

![](/assets/java-visibility/diagram-1.png)

Puisque c'est pas beau de dépendre de chose concrètes, on a aussi souvent le fameux "impl" qui s'amène au niveau des services.

On le représente souvent de cette manière :

![](/assets/java-visibility/diagram-2.png)

Bien qu'en réalité, on a ça : 

![](/assets/java-visibility/diagram-3.png)

Un exemple concret, dans une architecture à "couches" controlleur/service/repository, pour un service Rest :

PokemonRepository.java
```java
package pokemon.repository;

@Repository  
public interface PokemonRepository extends MongoRepository<Pokemon, Integer> {  
  
}
```

PokemonService.java
```java
package pokemon.service;

public interface PokemonService {  
  
    Pokemon findPokemonById(int id);  
  
}
```

PokemonServiceImpl.java
```java
package pokemon.service.impl;

@Service  
public class PokemonServiceImpl implements PokemonService {  
      
    private PokemonRepository pokemonRepository;  
  
 public PokemonServiceImpl(PokemonRepository pokemonRepository) {  
        this.pokemonRepository \= pokemonRepository;  
 }  
  
    @Override  
 public Pokemon findPokemonById(int id) {  
        return pokemonRepository.findById(id)  
                .orElseThrow(PokemonNotFound::new);  
 }  
}
```

PokemonController.java
```java
package pokemon.controller;

@RestController("/pokemons")  
public class PokemonController {  
  
    private PokemonService pokemonService;  
  
 public PokemonController(PokemonService pokemonService) {  
        this.pokemonService \= pokemonService;  
 }  
  
    @GetMapping("/{id}")  
    public Pokemon findPokemonById(int id){  
        return this.pokemonService.findPokemonById(id);  
 }  
  
}
```

Ici, le package `pokemon` expose en visibilité publique l'intégralité de ses classes/interfaces. Cela implique que depuis n'import où ailleurs dans le code, on peut importer `pokemon.PokemonRepository`, alors que cette interface ne devrait être pas pouvoir être consommée par autre chose que le `PokemonService`. Du code externe a également accès à l'implémentation `PokemonServiceImpl` et pourrait utiliser directement la classe, au lieu d'utiliser l'interface.

Bien que l'architecture en couches soit respectée, le module n'impose aucunement le respect de cette architecture, et n'expose pas d'interface claire.
