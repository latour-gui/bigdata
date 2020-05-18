# Introduction

- [Introduction](#introduction)
  - [Avènement du Big Data](#av%c3%a8nement-du-big-data)
  - [3 V du Big Data](#3-v-du-big-data)
    - [Volume](#volume)
    - [Variété](#vari%c3%a9t%c3%a9)
    - [Vélocité](#v%c3%a9locit%c3%a9)
    - [mentions](#mentions)
  - [Outils du Big Data](#outils-du-big-data)
  - [SQL](#sql)
  - [No SQL](#no-sql)
    - [Moteurs clef-valeur](#moteurs-clef-valeur)
    - [Moteurs orientés documents](#moteurs-orient%c3%a9s-documents)
    - [Moteurs orientés colonnes](#moteurs-orient%c3%a9s-colonnes)
    - [Moteurs orientés graphes](#moteurs-orient%c3%a9s-graphes)
  - [SQL vs NoSQL](#sql-vs-nosql)

## Avènement du Big Data

Origine de l'explosion de la génération des données

- technologique
  - internet
  - iot
- economique
  - diminution du coût

## 3 V du Big Data

### Volume

Beaucoup de données

### Variété

Données structurées (DB), semi-structurées (xml, json, ...) ou non-structurées (image, son, document, ...)

### Vélocité

C’est la vitesse de circulation de données, la vitesse à laquelle les données sont générées, capturées et partagées

ex: géolocalisation en temps réel, trading haute fréquence

### mentions

- Volume
- Velocity
- Variety
- Veracity
- Value
- Variability
- Visualisation
- Volatility
- Vulnerability

## Outils du Big Data

outils de

- stockage
- prédiction
- visualisation
- gouvernance

Théorème de CAP (Consistency, Availability, Partition tolerance)

- Consistence : chaque requête reçoit la dernière écriture (ou une erreur)
- Disponibilité: chaque requête reçoit une réponse
- Tolérance au partitionnement : chaque noeud doit pouvoir fonctionner de manière autonome

## SQL

- relationnel
- éviter la redondance
- transactions ACID
  - atomicité
  - cohérence
  - isolation
  - durabilité

## No SQL

> Not Only SQL

- DB sur autre mécanisme que relationnel
- réponse aux limitations des SGBD relationnels vis-à-vis du théorème CAP

### Moteurs clef-valeur

- les + simples
- manipulent des paires `(clef, valeur)` ou des tableaux de hachage
- fonctionnalités simplifiées
- excellentes performances

ex: Redis (mémoire), Dynamo (distribué)

### Moteurs orientés documents

- stocke des documents dans des formats standards (xml, json, ...)
- structure hiérarchique (bien pour données structurées ou semi-structurées)

ex : MongoDB, CouchDB

### Moteurs orientés colonnes

- structure proche de la table
  - données représentées en lignes et séparées en collonnes
  - chaque ligne identifiée par une clef

ex : Google BigTable, Cassandra, Apache HBase

### Moteurs orientés graphes

- structure autre que relationnel (graphe)

ex : Neo4j

## SQL vs NoSQL

| _SQL_                                        | _NoSQL_                                    |
| :------------------------------------------- | :----------------------------------------- |
| Structure forte (normalisation, typage, ...) | Structure faible                           |
| utilisé pour n'importe quelle application    | choix du moteur dépend de la situation     |
| modélisation rigide                          | modélisation souple                        |
| absence de valeur gérée via `NULL`           | absence de valeur nativement gérée         |
| support des transactions                     | transactions pas nécessairement supportées |
| méthodologie _waterfall_                     | propice aux méthodes agiles                | <!-- Warning : bullshit> |
