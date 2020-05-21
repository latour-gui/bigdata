# HDFS & Map-Reduce

- [HDFS & Map-Reduce](#hdfs--map-reduce)
  - [HDFS](#hdfs)
  - [The Hadoop Distributed File System: Architecture and Design](#the-hadoop-distributed-file-system-architecture-and-design)
    - [Optimisations](#optimisations)
    - [SafeMode](#safemode)
    - [Persistence of FS Metadata](#persistence-of-fs-metadata)
    - [Communication](#communication)
    - [Failures](#failures)
    - [Data Organization](#data-organization)
    - [Commands](#commands)

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
  - 64 MB par défaut
- Java
- GNU/Linux

## The Hadoop Distributed File System: Architecture and Design

### Optimisations

En plus de ce qui est noté ci dessus, on y apprend qu'une tâche compliquée qui est gérée par HDFS consiste à répartir les *replicas* à travers les différents *racks*.

Lors du démarage, le fs détermine sur quel *rack* se trouvent tel et tel *DataNode* pour optimiser la réplication de manière à améliorer la sécurité et le débit.

Une action consiste à mettre des répliques sur des racks différents au cas où tout un rack venait à faillir.

D'autres optimisations consistent à choisir le bon *replica* à partir duquel envoyer les données vers un client. (le + proche généralement)

### SafeMode

Lors du démarage du *NameNode*, pas d'écriture.
Le maitre attend les rapports des *heartbeat* et *blockreport* des esclaves avant de faire quoi que ce soit.

Une fois tous les rapports reçus, le *NameNode* va déterminer si tous les blocks sont considérés comme **safely replicated** (possèdent un nombre minimum de copie à travers les *DataNode*) et sort ensuite du *SafeMode* pour écrire les blocs qui ne respectent pas cette règle, s'il y en a.

### Persistence of FS Metadata

L'entièreté des allocations est *in-memory* et chaque opération fait l'objet d'une écriture dans un fichier de log stocké sur le disque.

Lors d'un démarrage, le fichier de log est reparcouru de manière à ce que l'image se recrée en mémoire vive.

### Communication

Deux protocoles sont utilisés pour communiquer avec le *NameNode* (**ClientProtocol**) et avec le *DataNode* (**DatanodeProtocol**). Ces deux protocoles custom se greffe au dessus de la coupe TCP/IP.

### Failures

Il ne peut y avoir que trois types d'erreurs:

- Namenode failures
- Datanode failures
- network failures

Quand un *Datanode* n'envoie pas de *heartbeat*, le *Namenode* le marque comme mort et ne lui envoie plus d'I/O. De plus le *Namenode* planifie la réplication des blocs qui se trouvaient sur ledit *Datanode*.

L'intégrité des fichiers est vérifiée grâce à un checksum de chacun des blocks que le client crée dans le même namespace que le fichier qu'il veut stocker.

Les fichiers `FsImage` et `EditLog` sont très sensibles et l'altération de l'un des deux peut causer le mauvais fonctionnement de tout le système HDFS. Pour cette raison, ils sont souvent duppliqué en mirroir.

### Data Organization

Quand un fichier est envoyé depuis un client pour être stocké sur un serveur HDFS, une copie locale du fichier est créée et à partir de cette copie, le client va initialiser la connexion dès qu'il est possible d'envoyer un block.

Le serveur va répondre au client où envoyer le block.

Une liste de *DataNode* est envoyée et les données du block sont envoyé par paquet de 4MB. Chaque paquet est reçu par le premier *DataNode* qui l'envoie au suivant après avoir copié l'information.

Une fois que le dernier paquet est envoyé depuis le client, celui-ci signale que l'entièreté du fichier a été envoyée et cloture la connexion.

Si une erreur surivent avant la fin de la communication, le fichier est perdu.

Il y a une gestion de corbeille lorsqu'un fichier est supprimé. Il peut être restauré tant qu'il se trouve dans le dossier `/trash`.

### Commands

Il y a un shell `DFSShell` qui permet les opérations POSIX.

Il y a un utilitaire `DFSAdmin` qui permet d'administrer le serveur.

