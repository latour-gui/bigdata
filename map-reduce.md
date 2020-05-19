# Map-Reduce

- [Map-Reduce](#map-reduce)
  - [HDFS](#hdfs)

## HDFS

> Hadoop Distributed File System

- distribué
- très *fault-tolerant*
  - "Hardware failure is the norm rather than the exception"
  - détection des fautes et *recovery* rapide et automatique
- conçu pour être déployé sur du hardware bon marché
- haut débit
- conçu pour grands datasets
- streaming
  - pas conçu pour être intereactif
  - conçu pour envoyer des données vers un traitement
  - priorité donnée au débit plutôt qu'au temps de réponse
- moins cher de déplacer les calculs que les données
  - minimiser la congestion réseau
- "write once, read many"
  - insert ok, update nok
- haute portabilité
- architecture maitre-esclave
  - `NameNode`
    - maitre
    - gère les namespaces du file system
      - ouverture
      - fermeture
      - renommage
    - régule l'accès aux fichiers
    - détermine le *mapping* des blocks aux *DataNode*
    - les données ne circulent jamais à travers le *NameNode*
    - reçoit un **heartbeat** des *DataNode* afin de s'assurer que le Node est fonctionnel
    - reçoit un **blockreport** des *DataNode* qui contient une liste des blocks du *DataNode*
  - `DataNode`
    - esclave
    - 1 par node
    - gère le stockage des données
    - read/write request
- le client voit un dossier et peut y stocker des fichiers
  - les fichiers sont divisés en blocks
  - les blocks sont stockés dans un set de DataNodes
  - les blocks sont répliqués pour la tolérance à l'erreur
  - tous les blocks sont de la même taille (sauf le dernier)
  - la taille des blocks et le facteur de réplication sont configurables
- Java
- GNU/Linux
