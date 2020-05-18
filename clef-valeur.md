# Clef-Valeur

- [Clef-Valeur](#clef-valeur)
  - [Hash table](#hash-table)
    - [load factor](#load-factor)
    - [hash function](#hash-function)
    - [collisions](#collisions)
    - [résoudre les collisions](#r%c3%a9soudre-les-collisions)
      - [separate chaining with linked lists](#separate-chaining-with-linked-lists)
      - [separate chaining with list head cells](#separate-chaining-with-list-head-cells)
      - [open addressing](#open-addressing)
  - [Redis](#redis)
    - [Types](#types)
    - [Syntaxe](#syntaxe)
      - [Classique](#classique)
      - [Increment/Decrement](#incrementdecrement)
      - [TTL](#ttl)
      - [Array operations](#array-operations)
      - [Set operations](#set-operations)
      - [Hash operations](#hash-operations)
  - [Swiss army knife queries](#swiss-army-knife-queries)

## Hash table

Une **hash table** est une structure de donnéees qui implémente le type abstrait de donnée **associative array**, un tableau qui associe à chaque clef une valeur unique.

Une hash table utilise une **fonction de hashing** qui assure la liaison entre la clef et l'index du tableau où la valeur est stockée.

La performance récupération d'un élément est indépendante du nombre d'éléments stockés et se fait en temps constant (`O(1)`).

### load factor

Le **load factor** est défini comme le quotient du nombre d'entrées dans la table par le nombre de slots disponibles.

```text
load factor = n/k

n = table entries
k = available slots
```

La **hash table** est moins performante quand le ratio est grand. Il faut garder ce **load factor** sous une certaine limite.

### hash function

L'idée de la **fonction de hashing** est de distribuer les entrées (clef-valeur) au sein d'un tableau de slots indexés.

La fonction de hashing utilise la clef pour calculer un indice où sera stockée la valeur.

`index = f(key, array_size)`

où `f` est ressemble typiquement à ceci:

```python
def f(key, array_size):
    hash = hash_function(key)
    index = hash % array_size
    return index
```

où `hash_function` est indépendant de la taille du tableau et l'index réduit entre `0` et `array_size - 1` grâce au modulo.

Il est important pour une fonction de hashing de distribuer les clef de manière uniforme sur les indices.
Une distribution non-uniforme augmente le nombre de collisions.

- distribution uniforme des indices
- pas de clustering

Une **fonction de hashing parfaite** est une fonction dont les résultats sont dépourvus de collisions pour un ensemble de clefs de `S` (fonction **_injective_**)

Une **fonction de hashing minimale** est une fonction de hashing parfaite qui associe `n` clefs de `S` sur un ensemble de `n` entiers consécutifs (fonction **_bijective_**)

### collisions

Idéalement la fonction de hashing donne des résultats uniques pour chaque entrée unique. Dans la pratique il est possible que deux clef génère le même hash, ce qui crée une **collision**.

Les collisions sont inévitables.

### résoudre les collisions

Avec les chaînages, on peut se retrouver avec une utilisation en espace complètement folle dans la mesure où il n'est pas possible d'anticiper le nombre d'éléments présents dans un slot avant de les compter.

Avec la méthode _open addressing_, bien qu'elle puisse donner lieu à des collisions qui n'existeraient pas avec les méthode de chainage, elle a au moins l'avantage de maitriser totallement la taille qu'elle fera sur le disque.

#### separate chaining with linked lists

Au lieu de stocker la valeur dans le slot, on va y stocker un pointeur vers un tuple `(clef, valeur, pointeur)`.
Si ce slot est récupéré, on compte le nombre d'élément dans la liste chainée. S'il n'y en a qu'un seul il est retourné.

Si plusieurs éléments sont présents, il va falloir parcourir la liste (`O(n)`) pour récupérer la valeur qui correspond à la clef qui est rentrée en collision.

#### separate chaining with list head cells

Même technique que pour les _linked lists_ mais il est possible de stocker un tuple `(clef, valeur, pointeur)` directement dans le slot.

#### open addressing

Il n'y a pas de pointeurs en jeu. Les slots sont prévus pour recevoir un tuple `(clef, valeur)`.

Si il y a des collisions, on va incrémenter l'index et comparer la clef donnée en entrée avec celle enregistrée dans le slot; jusqu'à soit trouver un slot vide pour une insertion, soit trouver la même clef pour une lecture.

## Redis

- Open-source
- In-memory
- Distribué

### Types

- String
- List
- Hash
- Set
- Ordered Set

### Syntaxe

#### Classique

```redis
SET key value
SETNX key value // 1 if key was set, 0 if key was not set
GET key
DEL key [key...]
```

```redis
KEYS patter // return all keys matching pattern
```

Attention la commande `KEYS` possède une complexité de l'ordre de `O(n)`. Il ne faut pas l'utiliser en production à moins de n'avoir aucune autre solution.

#### Increment/Decrement

```redis
INCR key
DECR key
```

Les fonctions `INCR` et `DECR` ont été implémentées pour éviter des problèmes d'accès concurents (deux applications récupèrent la valeur de la clef `key`, lui ajoutent 1 et envoyent la commande pour mettre à jour cette clef)

#### TTL

```redis
EXPIRE key seconds // 1 if timeout is set, 0 if key does not exist
TTL key // -2 if key does not exist, -1 if key exists but no expire associated
```

#### Array operations

```redis
RPUSH key value [value...] // insert at tail
LPUSH key value [value...] // insert at head
LRANGE key start stop // zero based index
```

Une erreur est renvoyée lorsque l'on essaye de pousser une liste pour une clef qui n'en est pas déjà une.

```redis
LLEN key // get list length
```

#### Set operations

```redis
SADD
SREM
SISMEMBER
SMEMBERS
SUNION
```

#### Hash operations

```redis
HSET
HGET
HGETALL
```

## Swiss army knife queries

```redis
// work in progress
```
