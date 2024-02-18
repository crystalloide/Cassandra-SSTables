# Cluster cassandra "formation" dans un environnement virtualisé "Gitpod"

Notre cluster de démonstration contient 6 noeuds en tout : 
- 3 noeuds sur le Datacenter 1
- 3 noeuds sur le Datacenter 2


## Rappel pour retrouver les environnements éventuellement précédemment instanciés dans Gitpod : [ https://gitpod.io/workspaces ](https://gitpod.io/workspaces)

Nous allons installer un cluster Cassandra à 6 nœuds et 2 Datacenter sur un environnement virtuel accessible à partir d'un simple navigateur web, à des fins de développement et de formation. 

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/crystalloide/Cassandra-SSTables
)

### 1a°) Lancement du cluster (option 1) :

    docker compose up -d

### 1b°) Lancement du cluster (option 2) :

    docker compose -f docker-compose-01-06.yml up -d    

###	2°) Pour lister l'image récupérée  :

    docker images

### 3°) Attendre quelques minutes que les conteneurs démarrent

### 4°) Affichage des conteneurs et vérification qu'ils sont bien en cours d'exécution : 

    docker ps -a 

### 5°) Affichage des logs d'un des conteneurs en cours d'exécution : 

    docker logs cassandra11

### 6°) Surveillance d'un cluster Cassandra dans Docker :

nodetool est un programme en ligne de commande qui offre un large choix sur la façon d'examiner le cluster, de comprendre son activité et de le modifier.

nodetool permet d'obtenir des statistiques sur le cluster, de voir les plages de token maintenus par chaque nœud et les diverses tâches de gestion

(ex. : déplacement de données d'un nœud à un autre, la mise hors service de nœuds, la réparation de nœuds, etc.)

### 7°) Utilisation de nodetool : On peut utiliser la commande "nodetool status" sur n'importe lequel des nœuds pour voir l'état des nœuds du cluster :
    docker exec -it cassandra11 nodetool status

### Affichage :

    Datacenter: NORD
    ================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address     Load        Tokens  Owns (effective)  Host ID                               Rack       
    UN  172.18.0.4  75.2 KiB    16      31.9%             6ee78c05-ae68-4d93-9d35-f1ff01d9d33a  WINTERFELL2
    UN  172.18.0.6  70.22 KiB   16      36.2%             280a0c64-2395-47cc-a7ef-6cca5671e19b  WINTERFELL3
    UN  172.18.0.2  109.4 KiB   16      34.3%             aa63cc52-bb33-4d9d-a6b5-0661b7555424  WINTERFELL1
    
    Datacenter: TERRES-DE-LA-COURONNE
    =================================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address     Load        Tokens  Owns (effective)  Host ID                               Rack       
    UN  172.18.0.7  104.27 KiB  16      35.5%             ec134c10-62fe-4282-b05c-e0a0bffff722  PORT-REAL3 
    UN  172.18.0.3  75.27 KiB   16      32.0%             cfe4af2b-6886-470b-be26-afb1e5a564d8  PORT-REAL1 
    UN  172.18.0.5  70.25 KiB   16      30.2%             3e710059-131e-43d6-a0f2-919d423ae1cc  PORT-REAL2 

### 8°) Pour vérifier le bon démarrage du service cassandra sur un noeud : 

    docker logs cassandra22 | grep 'jump'

### 9°) Affichage des services en cours d'écoute sur les ports 904* :	

   On install netstat : 

    sudo apt-get install net-tools
    
   Et ensuite : 

    netstat -l | grep 904


### 10°) La commandes "nodetool info" apporte des informations complémentaires :
    docker exec -it cassandra11 nodetool info


### Affichage :

    ID                     : f4539d5f-6e29-417a-b2c2-f59d72101120
    Gossip active          : true
    Native Transport active: true
    Load                   : 75.19 KiB
    Generation No          : 1692971004
    Uptime (seconds)       : 702
    Heap Memory (MB)       : 87.96 / 251.00
    Off Heap Memory (MB)   : 0.00
    Data Center            : dc1
    Rack                   : rack2
    Exceptions             : 0
    Key Cache              : entries 11, size 984 bytes, capacity 12 MiB, 104 hits, 122 requests, 0.852 recent hit rate, 14400 save period in seconds
    Row Cache              : entries 0, size 0 bytes, capacity 0 bytes, 0 hits, 0 requests, NaN recent hit rate, 0 save period in seconds
    Counter Cache          : entries 0, size 0 bytes, capacity 6 MiB, 0 hits, 0 requests, NaN recent hit rate, 7200 save period in seconds
    Network Cache          : size 8 MiB, overflow size: 0 bytes, capacity 15 MiB
    Percent Repaired       : 100.0%
    Token                  : (invoke with -T/--tokens to see all 16 tokens)


### 11°) Connection sur un conteneur en bash : 

    docker exec -it cassandra11 bash

### 12°) Et on navigue : 

    cd /opt/cassandra/conf

   On affiche le contenu du répertoire de configuration du conteneur :  
   
    ls

   On affiche le contenu du fichier cassandra-env.sh :     
   
    cat cassandra-env.sh

   On affiche le contenu du fichier cassandra.yaml :   
   
    cat cassandra.yaml

  Et plus particulièrement, on regarde la valeur du paramètre endpoint_snitch :
    
    cat cassandra.yaml | grep endpoint_snitch
    
- Noter :

     endpoint_snitch -- Set this to a class that implements

     endpoint_snitch: GossipingPropertyFileSnitch

      cat cassandra-rackdc.properties
  
- Noter : GossipingPropertyFileSnitch

On pense à sortir du conteneur : 

    exit

### 13°) Pour afficher les détails sur la réportition des ranges de tokens dans le ring , on utilise "nodetool ring" : 

    docker exec -it cassandra11 nodetool ring

### Affichage : 
    Datacenter: dc1
    ==========
    Address          Rack        Status State   Load            Owns                Token                                       
                                                                                    8972475839137452131                         
    172.18.0.2       rack0       Up     Normal  109.38 KiB      64.66%              -9055590551661409925                        
    172.18.0.7       rack2       Up     Normal  75.19 KiB       75.99%              -8608019211763529537                        
    ....                      
    172.18.0.7       rack2       Up     Normal  75.19 KiB       75.99%              8007190946676663489                         
    172.18.0.2       rack0       Up     Normal  109.38 KiB      64.66%              8311780714600540032                         
    172.18.0.7       rack2       Up     Normal  75.19 KiB       75.99%              8721486411074324482                         
    172.18.0.6       rack1       Up     Normal  75.19 KiB       59.34%              8972475839137452131                         
    
      Warning: "nodetool ring" is used to output all the tokens of a node.
      To view status related info of a node use "nodetool status" instead.
  

## 14°) Utilisation du client CQL en ligne de commande : cqlsh  

    docker exec -it cassandra11 cqlsh

### Affichage : 

    Connected to formation at 127.0.0.1:9042
    [cqlsh 6.1.0 | Cassandra 4.1.3 | CQL spec 3.4.6 | Native protocol v5]
    Use HELP for help.

## 15°) Affichage des keyspaces présents : 

    DESCRIBE KEYSPACES;

### Affichage : 
    cqlsh> DESCRIBE KEYSPACES;
    
    system       system_distributed  system_traces  system_virtual_schema
    system_auth  system_schema       system_views 
    
### 16°) On sort de l'interpréteur CQL en ligne de commande :  
    EXIT;

### Affichage : 

    cqlsh> EXIT


## 17°) Pour arrêter le cluster :

    docker compose down


# Have fun !
