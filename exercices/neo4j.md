# Neo4J Exercices

- [Neo4J Exercices](#neo4j-exercices)
  - [Movies](#movies)
    - [Lister les acteurs, leur nombre de film et les titres des films par ordre décroissant du nombre de films](#lister-les-acteurs-leur-nombre-de-film-et-les-titres-des-films-par-ordre-d%c3%a9croissant-du-nombre-de-films)
    - [Lister les réalisateurs qui ont fait jouer "Hugo Weaving" et pour chacun d'eux, la liste des films en question](#lister-les-r%c3%a9alisateurs-qui-ont-fait-jouer-%22hugo-weaving%22-et-pour-chacun-deux-la-liste-des-films-en-question)
    - [Lister les films de genre "Sci-Fi" des réalisateurs avec qui "Morgan Freeman" a joué](#lister-les-films-de-genre-%22sci-fi%22-des-r%c3%a9alisateurs-avec-qui-%22morgan-freeman%22-a-jou%c3%a9)
    - [Lister le nombre de fois qu'un réalisateur a fait jouer un acteur particulier dans ses films](#lister-le-nombre-de-fois-quun-r%c3%a9alisateur-a-fait-jouer-un-acteur-particulier-dans-ses-films)
    - [Quel est le chemin de relations le plus court si "Keanu Reeves" souhaitait prendre contact avec "Francis Ford Coppola"](#quel-est-le-chemin-de-relations-le-plus-court-si-%22keanu-reeves%22-souhaitait-prendre-contact-avec-%22francis-ford-coppola%22)
    - [Retourner la liste des films pour lesquels DES acteurs sont nés avant 1950](#retourner-la-liste-des-films-pour-lesquels-des-acteurs-sont-n%c3%a9s-avant-1950)
    - [Retourner la liste des films pour lesquels TOUS LES acteurs sont nés avant 1950](#retourner-la-liste-des-films-pour-lesquels-tous-les-acteurs-sont-n%c3%a9s-avant-1950)
    - [Lister les acteurs qui ont joué plusieurs fois dans un film avec "Keanu Reeves"](#lister-les-acteurs-qui-ont-jou%c3%a9-plusieurs-fois-dans-un-film-avec-%22keanu-reeves%22)
    - [Lister les films sorti après 1990 dans lesquels Tom Hanks a joué](#lister-les-films-sorti-apr%c3%a8s-1990-dans-lesquels-tom-hanks-a-jou%c3%a9)
    - [Pour chaque réalisateur définir son acteur fétiche](#pour-chaque-r%c3%a9alisateur-d%c3%a9finir-son-acteur-f%c3%a9tiche)
    - [Déterminer l'acteur qui a joué dans le plus de film](#d%c3%a9terminer-lacteur-qui-a-jou%c3%a9-dans-le-plus-de-film)
    - [Trouver les acteurs qui sont aussi réalisateurs](#trouver-les-acteurs-qui-sont-aussi-r%c3%a9alisateurs)
    - [Trouver les personnes qui ont et joué dans et réalisé un même film](#trouver-les-personnes-qui-ont-et-jou%c3%a9-dans-et-r%c3%a9alis%c3%a9-un-m%c3%aame-film)

## Movies

En partant des informations suivantes

Noeuds de types

- Genre
- Keyword
- Movie
- Person

Liens de types

- HAS_GENRE
- HAS_KEYWORD
- WRITER_OF
- PRODUCED
- DIRECTED
- ACTED_IN

Il est demandé de répondre aux différentes questions suivantes

### Lister les acteurs, leur nombre de film et les titres des films par ordre décroissant du nombre de films

```cypher
MATCH (actor:Person)-[:ACTED_IN]->(movie:Movie)
RETURN actor.name, count(movie) as nbr_movies, collect(DISTINCT movie.title)
ORDER BY count(movie) DESC
```

### Lister les réalisateurs qui ont fait jouer "Hugo Weaving" et pour chacun d'eux, la liste des films en question

```cyper
MATCH (realisator:Person)-[:DIRECTED]->(movie:Movie)<-[:ACTED_IN]- (:Person {name: 'Hugo Weaving'})
RETURN realisator.name, collect(DISTINCT movie.title)
```

### Lister les films de genre "Sci-Fi" des réalisateurs avec qui "Morgan Freeman" a joué

```cypher
MATCH (:Person {name: 'Morgan Freeman'}) -[:ACTED_IN]-> (:Movie) <-[:DIRECTED]- (realisator:Person) -[:DIRECTED]-> (movie:Movie) -[:HAS_GENRE]-> (:Genre {name:'Sci-Fi'})
RETURN realisator.name, collect(DISTINCT movie.title)
```

### Lister le nombre de fois qu'un réalisateur a fait jouer un acteur particulier dans ses films

```cypher
MATCH (realisator:Person) -[:DIRECTED]-> (:Movie) <-[:ACTED_IN]- (actor:Person)
RETURN DISTINCT realisator.name, actor.name, count(realisator.name = realisator.name and actor.name = actor.name) as number_of_collaboration
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

### Retourner la liste des films pour lesquels DES acteurs sont nés avant 1950

```cypher
MATCH (actor:Person) -[:ACTED_IN]-> (movie:Movie)
WHERE actor.born < 1950
RETURN DISTINCT movie.title
```

### Retourner la liste des films pour lesquels TOUS LES acteurs sont nés avant 1950

```cypher
MATCH (actor:Person) -[:ACTED_IN]-> (movie:Movie)
WITH movie.title as title, collect(actor.born) as dates
WHERE ALL(d in dates WHERE d < 1950)
RETURN DISTINCT title
```

### Lister les acteurs qui ont joué plusieurs fois dans un film avec "Keanu Reeves"

```cypher
MATCH (actor:Person)-[:ACTED_IN]->(movie:Movie)<-[:ACTED_IN]-(:Person {name: 'Keanu Reeves'})
WITH  collect(movie.title) as movies, actor.name as actor, count(movie) as nbr
WHERE nbr > 1
RETURN actor, movies
```

### Lister les films sorti après 1990 dans lesquels Tom Hanks a joué

```cypher
// /!\ There is no `movie.date` in our given database !
MATCH (actor:Person {name:'Tom Hanks'}) -[:ACTED_IN]-> (movie:Movie)
WHERE movie.date > 1990
RETURN DISTINCT movie.title
```

### Pour chaque réalisateur définir son acteur fétiche

```cypher
// wallah chsuis dvenu fou sur cette query
WHERE nbr_collaboration > 2
WITH distinct(director) as d, max(nbr_collaboration) as n
MATCH (d) -[:DIRECTED]-> (m:Movie) <-[:ACTED_IN]- (collab:Person)
WITH d.name as director_name, collab.name as actor_name, n, count(m) as countm
WHERE countm = n
RETURN DISTINCT(director_name), actor_name, n
```

### Déterminer l'acteur qui a joué dans le plus de film

```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WITH a.name as actor, count(m) as nbr
RETURN actor, nbr
ORDER BY nbr DESC
LIMIT 1
```

### Trouver les acteurs qui sont aussi réalisateurs

(bonus lister les films en tant qu'acteur et les films en tant que réalisateurs)

```cypher
MATCH (multi:Person)-[:ACTED_IN]->(m1:Movie)
MATCH (multi) -[:DIRECTED]-> (m2:Movie)
RETURN
    multi.name as what_a_guy,
    collect(distinct m1.title) as actor_film,
    collect(distinct m2.title) as director_film
```

### Trouver les personnes qui ont et joué dans et réalisé un même film

```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
MATCH (a) -[:DIRECTED]-> (m)
RETURN a.name, m.title
```
