# Graphe

- [Graphe](#graphe)
  - [Définition](#d%c3%a9finition)
    - [Labeled property graph](#labeled-property-graph)
    - [OLTP vs OLAP](#oltp-vs-olap)
  - [Propriétés des SGBD orientés graphe](#propri%c3%a9t%c3%a9s-des-sgbd-orient%c3%a9s-graphe)
    - [Performances](#performances)
  - [Neo4j & Cypher](#neo4j--cypher)
    - [Syntaxe](#syntaxe)
    - [Type des données stockées](#type-des-donn%c3%a9es-stock%c3%a9es)
    - [Create](#create)
    - [Match](#match)
  - [Swiss army knife queries](#swiss-army-knife-queries)
    - [récupérer les cousins](#r%c3%a9cup%c3%a9rer-les-cousins)
    - [Quel est le chemin de relations le plus court si "Keanu Reeves" souhaitait prendre contact avec "Francis Ford Coppola"](#quel-est-le-chemin-de-relations-le-plus-court-si-%22keanu-reeves%22-souhaitait-prendre-contact-avec-%22francis-ford-coppola%22)

## Définition

Un graphe est un ensemble de noeuds et de relations entre ces noeuds.

entité = noeud et lien = relation

### Labeled property graph

Un graphe est dit _labeled-property graph_ quand il dispose des caractéristiques suivantes :

- les noeuds contiennent des propriétés (clef-valeur)
- les noeuds peuvent être caractérisés par différents labels
- c'est un graphe orienté (les relations sont dirigées)
- les relations sont nommées
- les relations peuvent contenir des propriétés

### OLTP vs OLAP <!-- test -->

OLTP : Online Transactional Processing, utilisés pour être manipulé directement par les applications

OLAP : Online Analytical Processing, utilisés pour être manipulé lors d'analyses de graphe

## Propriétés des SGBD orientés graphe

- moteur de stockage natif en graphe ou non (sérialisation dans une technologie relationnelle ou orienté objet)
- moteur d'exécution
  - _index-free adjancy_ : Les noeuds pointent directement (physiquement) les autres. Plus performant mais difficilement distribuable
  - non-_index-free adjancy_: Les noeuds références les autres via des clefs

### Performances

Généralement les performances sont meilleures avec un moteur orienté graphe car :

- dans un graphe, la performance dépend surtout de la proportion de graphe impliquée par la requête
- dans un RSGDB, la performance dépend surtout de la taille totale du graphe (`JOIN`)

## Neo4j & Cypher

Voici un [super tutoriel](https://github.com/adambard/learnxinyminutes-docs/blob/master/cypher.html.markdown) disponible sur github. J'ai quand même résumé les slides ci-après.

Il existe quelques fonctions natives qui valent le coup d'être connues

- `avg()`
- `collect()`
- `count()`
- `min()`
- `max()`

La documentation de cypher se trouve [ici](https://neo4j.com/docs/cypher-manual/current/) et [cette section](https://neo4j.com/docs/cypher-manual/current/functions/) parle des fonctions.

### Syntaxe

- `( )` noeud
- `(i)` identifiant
- `(:label)` label
- `({prop1:valeur, prop2:valeur})` filtre sur propriétés
- `--` relation
- `-->` et `<--` relation orientée
- `-[:label]-` label
- `-[{prop1:valeur, prop2:valeur}]-` filtre sur les propriétés
- `(a) -[*]- (b)` nombre de relations indéterminé entre `a` et `b`
- `(a) -[*3]- (b)` nombre de relations entre `a` et `b` égale exactement 3
- `(a) -[*<min>..<max>]- (b)` nombre de relations entre `a` et `b` est compris entre `<min>` et `<max>`, les deux sont optionnels

### Type des données stockées

- Property Types
  - Number (float, integer)
  - String
  - Boolean
- Structural Types
  - Node
  - Relationship
  - Path
- Composite Types
  - List (ordered)
  - Maps (unordered, key/value)

### Create

```cypher
CREATE
(a:Person {name:'Jean'}),
(b:Person {name:'Pauline'}),
(a)-[:KNOWS]->(b)
```

L'idée ici c'est de voir qu'on peut créer des noeuds et des relations en une seule requête, en séparant les éléments avec une virgule.

On peut spécifier le label d'un noeud et des propriétés.

### Match

```cypher
MATCH (a:Person {name:'Jean'})-[:KNOWS]-(b)
RETURN b
```

est équivalent à

```cypher
MATCH (a:Person) -[:KNOWS]-(b)
WHERE a.name = 'Jean'
RETURN b
```

et va retourner tous les noeuds qui sont en relation `KNOWS` avec Jean.

## Swiss army knife queries

### récupérer les cousins

```cypher
MATCH
  (me:Person) -[:ISCHILD]-> (parent:Person) -[:ISCHILD]-> (grandparent:Person) <-[:ISCHILD]- (uncle:Person)<-[:ISCHILD]- (cousin:Person)
WHERE parent <> uncle
RETURN me, collect(distinct cousin)
```

### Quel est le chemin de relations le plus court si "Keanu Reeves" souhaitait prendre contact avec "Francis Ford Coppola"

```cypher
MATCH
  (the_one:Person {name:'Keanu Reeves'}),
  (godfather:Person  {name:'Francis Ford Coppola'}),
  p = shortestPath((the_one)-[*]-(godfather))
WHERE ALL(r in relationships(p) WHERE type(r)<> "HAS_GENRE" AND type(r) <> "HAS_KEYWORD")
RETURN p
```
