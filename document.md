# Document

- [Document](#document)
  - [MongoDB](#mongodb)
    - [Cursor](#cursor)
  - [Syntaxe](#syntaxe)
  - [Swiss army knife queries](#swiss-army-knife-queries)

## MongoDB

- orienté document
- bonnes performances (C++)
- stocke documents formatés BSON
  - Binary Serialized dOcument Notation
  - max 16 Mo
  - plus compact que JSON
- mêmes types qu'en javascript
- pas de contrainte
- chaque document d'une collection est identifié par une clef unique
  - `_id`
  - `UUID`
    - Universally Unique IDentifier
    - 12 octets
    - numéro du serveur (3 octets)
    - timestamp (4 octets)
    - numéro unique généré pour la collection (3 octets)
    - process id du démon MongoDB (2 octets)
- sécurité faible
  - possibilité de créer plusieurs utilisateurs
  - définition de l'accès en lecture/écriture par DB
- Répartition (sharding)
  - répartir une collection sur plusieurs serveurs via un ensemble clefs-valeurs
  - architecture maitre-esclave pour répartir la charge de travail entre les esclaves
- gestion manuelle des index
- DDL
  - Data Description Langage
  - langage permettant de définir les structures d'une collection, d'un document
  - paradigme "objet"
- DML
  - Data Manipulation Langage
  - récupération et modification des données
  - méthodes de récupération
    - l'**extraction simple** permet d'effectuer des requêtes simples de sélection et de projection des données
    - l'**aggrégation** permet d'effectuer des actions sur les données avant de les renvoyer
    - la méthode **distinct** permet d'éviter les renvois de doublons

Un document BSON en MongoDB est comparable à une ligne en RSGDB. Une collection en MongoDB est comparable à une table en RSGDB.

Des documents au sein d'une même collection ne doivent pas forcément avoir la même structure.

### Cursor

je c pas cor ce que c

## Syntaxe

omg ya blindé


## Swiss army knife queries

```DML
   // work in progress
```
