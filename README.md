#EN 
# initiation-graphe-thingin

TP 2 : langages de requêtes NOSQL, BD graphe, requêtes sémantiques

Dans le dernier TP, nous avons mis en place injecté des données sur la plateforme thingin et brièvement aperçu son langage de requête pour observer ces données. 
Dans ce TP, nous allons utiliser la base de données multi-model arangodb (qui est utilisée pour la plateforme thingin), et introduire son langage de requête (AQL).



Part 1 : multi-model database, smart city use case
- 

Install :
- install docker on your machine 
- install arangodb docker image : ``` docker pull arangodb ``` 
- follow arangodb launch instructions ("Start an ArangoDB instance") at :  ``` https://hub.docker.com/_/arangodb ``` ; to simplify the authentication process, define a password with the option ``` ARANGO_ROOT_PASSWORD=somepassword ```
- once your image is up and running, the port 8529 is used to access your arangodb instance.
- access your instance through the browser interface : 127.0.0.1:8529
- you will be asked to enter the login/password defined earlier
- once connected, we are redirected to the "system" database which is not meant to hold data. We will create or own database and data collections. Each data collection segments the database content, similarly to SQL columns. The database can be created simply from the web interface (left panel -> databases -> +). Once it is created, you can access the new db by cliking on the db button on the top right corner of the screen. 
- we then need to create data collections. We will create a node collection for our graph, called "nodes", with type "Document" (since arangodb is a multi-model database, nodes are simply documents), and a collection for our graph's edges, called "edges" and of type "Edge" (following instructions assume you kept these names). 

- Data import : 
  - download the files nodes.json and edges.json in this repository
  - Import the nodes.json file into the "nodes" collection. To do this click on "Upload documents from JSON file" in the top right corner
  - Repeat this operation with the edges from edges.json in the edges collection
  - You should have around 6k nodes and 76 edges in your database after the import
  
  
Now we will execute some requests on this dataset to show the advantages of the multi-model database. You will translate some of the natural langage requests in arangodb's query langage (AQL) using examples
Simple Document and Graph requests : 
- Get all nodes : ```for n in nodes return n```
- Get all semantic classes of nodes in the dataset (virtual_class field): ```for n in nodes return n.virtual_class```
- Adapt the previous request using the ```distinct``` keyword to remove duplicate classes
- Construct a return payload instead containing only the domain and iri fields of the nodes : ```for n in nodes return {domain:n.domain, iri:n.iri}```
- Get all edges : ```for e in edges return e```
- Use graph traversal to fetch nodes, their inwards edges and the attached nodes: for n in nodes for v,e in 1..1 inbound n edges return {n,e,v}. What are the returned nodes ? Construct a payload to return only their semantic class and their position (the position field is named "cG9z_geom_point")

Geographic queries : for these request, a geographic indexed is required. Start by creating it on your ```nodes ``` collection. The name of the field that we will index is the position field ("cG9z_geom_point"). You can follow the following documentation to create the index : https://www.arangodb.com/docs/stable/aql/tutorial-geospatial.html. 
- Get all the nodes nearby the point of latitude 5.772045969859884, longitude 45.206042961330446, in a 100m range. Use the following documentation to create this query: https://www.arangodb.com/docs/stable/aql/functions-geo.html  (fonction geo_distance). To define the point from coordinates, you can either create a json payload, or use arangodb's "geo_point" function
- Construct a return payload containing the nodes, their outbound edges, and their respective distance to the city hall parking lot, without using its coordinates. To find the parking lot node, you can insert the following request filter : ```FILTER node.aHR0cDovL3B1cmwub3JnL2RjL3Rlcm1zL2Rlc2NyaXB0aW9u == "Parking Mairie"```

Partie 2 : building use case. Graph and semantic queries
- 
- In this part we will explore some of ThingIn's data
We will use the domain "http://thingin.orange.com/ifc/Meylan_v3/" which contains building data, but has too many nodes to be visualized directly. 



Use the following query template to filter avatars based on their domain and their class (keep the same domain). 
To choose an appropriate class, we can use ThingIn's ontology lookup service (examples : http://elite.polito.it/ontologies/dogont.owl#Lamp, http://elite.polito.it/ontologies/dogont.owl#Wall)

``` 
{
  "query": {
    "$domain": "http://thingin.orange.com/ifc/Meylan_v3/",
    "$class": ""
  },
  "view": {}
}
```

We will finally use semantic classes along with the graph to construct a query and filter the results.
Execute the following request, where the arrows indicate relations, and the "$alias" field is used for variables references: 
``` 
  {
    "query": [

    {
      "$alias": "room",
      "$iri": "http://thingin.orange.com/ifc/Meylan_v3/ifc-0m3cTj7$v4OQQ1cxNy_G8l",
      "<-http://elite.polito.it/ontologies/dogont.owl#isIn": ["objects", "cupboard"]
    },
    {
      "$alias": "cupboard",
        "$class": {
          "$in": [
            "http://elite.polito.it/ontologies/dogont.owl#Cupboard"
          ]
        },
        "<-http://elite.polito.it/ontologies/dogont.owl#isIn": "objects2"
      }
    ],
    "view": {}
  }   
``` 

What does this request do ? What avatars are expected to be returned by ? 



#FR  

# initiation-graphe-thingin

TP 2 : langages de requêtes NOSQL, BD graphe, requêtes sémantiques

Dans le dernier TP, nous avons mis en place injecté des données sur la plateforme thingin et brièvement aperçu son langage de requête pour observer ces données. 
Dans ce TP, nous allons utiliser la base de données multi-model arangodb (qui est utilisée pour la plateforme thingin), et introduire son langage de requête (AQL).



Partie 1 : requêtes BDD multi-modèle, cas d'usage smart city
- 

Installation :
- installer docker sur sa machine 
- installer l'image docker arangodb : ``` docker pull arangodb ``` 
- suivre les instructions de lancement de l'instance arangodb ("Start an ArangoDB instance") :  ``` https://hub.docker.com/_/arangodb ``` ; attention pour simplifier l'authentification utilisez un mot de passe que vous définissez avec l'option ``` ARANGO_ROOT_PASSWORD=somepassword ```
- une fois votre image arangodb lancée, le port 8529 est utilisé par défaut pour accéder à votre BDD.
- accédez à votre BDD via l'interface web exposée sur le port que vous avez spécifié (8529 par défaut) : 127.0.0.1:8529
- votre login/mot de passe vous sera demandé (en fonction de ce que vous avez indiqué dans l'étape de démarrage de l'image docker)
- une fois connecté, nous sommes redirigié vers la base de données "système". Nous allons créer une base de données de test et des collections qui vont contenir nos données. Chaque collection est un découpage de l'ensemble des données (de façon similaire à une table SQL). La base de données peuit être créée par l'interface web (panneau de gauche -> databases -> +). Une fois la base de données créée, on s'y connecte en cliquant sur l'onglet DB en haut à droite de l'écran. 
- il nous reste à créer des collections, une pour nos futurs noeuds du graphe, que l'on nommera nodes, de type "Document", et une pour les arcs du graphe que l'on nommera edges, de type "Edge" (respecter ce nommage). 

- Importation des données : 
  - télécharger les fichiers nodes.json et edges.json présents sur ce dépôt
  - Cliquer sur la collection nodes, nous allons importer le fichier nodes.json. Cliquer en haut à droite de l'écran, sur "Upload documents from JSON file"
  - Répéter l'opération depuis la collection edges, avec le fichier edges.json
  - Vous devriez avoir environ 6k documents chargés dans nodes, et 76 documents dans edges
  
  
Nous allons effectuer quelques requêtes sur le jeu de données ainsi importé, et vous trouverez comment exprimer certaines requêtes en langage naturel par vous même. Arangodb étant une base multi-modèle, nous allons rapidement illustrer ces différentes composantes  

Requêtes Document et Graphe simples : 
- Récupérer tous les noeuds : ```for n in nodes return n.```
- Récupérer les différentes classes du jeu de données (champ virtual_class) : ```for n in nodes return n.virtual_class```
- Adaptez la requête des classes virtuelles en utilisant le mot clé ```distinct``` pour supprimer les duplicats
- Construire un document contenant uniquement les champs iri et domain : ```for n in nodes return {domain:n.domain, iri:n.iri}```
- Récupérer tous les edges : ```for e in edges return e```
- Parcourir le graphe dans le sens entrant et retourner un set composé des noeuds et des edges parcourus : for n in nodes for v,e in 1..1 inbound n edges return {n,e,v}. Quels sont les noeuds retournés ? Construire un payload de retour comprenant uniquement leur classe et leur position (champ "cG9z_geom_point")

Requêtes géographiques : Pour ces requêtes, un index géographique sera nécessaire. Commencez par créer un index géographique sur votre collection ```nodes ```. Le champ géographique qui sera indexé est le suivant : La documentation suivante vous sera utile : https://www.arangodb.com/docs/stable/aql/tutorial-geospatial.html. 
- Récupérer tous les noeuds proches du point de coordonnées latitude 5.772045969859884, longitude 45.206042961330446, dans un rayon de 100m. Utilisez la documentation suivante comme exemple https://www.arangodb.com/docs/stable/aql/functions-geo.html  (fonction geo_distance). Pour définir le point avec ces coordonnées, vous pouvez au choix créer un payload json, ou utiliser la fonction geo_point
- Construisez un résultat comprenant les noeuds, leurs un arc sortants vers d'autres noeuds, et leur distance au parking de la mairie sans spécifier les coordonnées du parking. Le filtre suivant vous permet de retrouver ce noeud dans votre requête : ```FILTER node.aHR0cDovL3B1cmwub3JnL2RjL3Rlcm1zL2Rlc2NyaXB0aW9u == "Parking Mairie"```

Partie 2 : cas d'usage batiment connecté. Requêtes graphes et sémantiques
- 
- Nous allons explorer la base thingin pour cette partie 

Nous allons explorer le domaine "http://thingin.orange.com/ifc/Meylan_v3/" qui contient des données de batiment, mais contient trop ne noeuds pour être visualisé directement. 




Utiliser le template de requête suivant pour filtrer les objets par leur classe dans le domaine. 
Pour retrouver ces classes nous pouvons les rechercher sur le web, ou via le service dédié de thingin (exemples : http://elite.polito.it/ontologies/dogont.owl#Lamp, http://elite.polito.it/ontologies/dogont.owl#Wall).

``` 
{
  "query": {
    "$domain": "http://thingin.orange.com/ifc/Meylan_v3/",
    "$class": ""
  },
  "view": {}
}
```


Nous allons nous maintenant nous intéresser à un sous-ensemble des données et utiliser les classes sémantiques et le graphe pour filtrer les objets retournés. 
Exécuter la requête suivante, où les flèches indiquent des relations et le mot clé "$alias" est utilisé en tant que nom de variable : 
``` 
  {
    "query": [

    {
      "$alias": "room",
      "$iri": "http://thingin.orange.com/ifc/Meylan_v3/ifc-0m3cTj7$v4OQQ1cxNy_G8l",
      "<-http://elite.polito.it/ontologies/dogont.owl#isIn": ["objects", "cupboard"]
    },
    {
      "$alias": "cupboard",
        "$class": {
          "$in": [
            "http://elite.polito.it/ontologies/dogont.owl#Cupboard"
          ]
        },
        "<-http://elite.polito.it/ontologies/dogont.owl#isIn": "objects2"
      }
    ],
    "view": {}
  }   
``` 

D'après vous, que fait cette requête ? Quels sont les objets renvoyés ?
