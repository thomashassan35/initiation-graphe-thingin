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
  
  
- Nous allons effectuer quelques requêtes sur le jeu de données ainsi importé, et vous trouverez comment exprimer certaines requêtes en langage naturel par vous même. Arangodb étant une base multi-modèle, nous allons rapidement illustrer ces différentes composantes  

- Requête document simple : 
- Récupérer les différentes classes distinctes du jeu de données (champ virtual_class) : 

Requêtes Graphe simples : 
- Récupérer tous les noeuds : for n in nodes return n
- Récupérer tous les edges : for e in edges return e
- Parcourir le graphe dans le sens entrant et retourner un set composé des noeuds et des edges parcourus : for n in nodes for v,e in 1..1 inbound n edges return {n,e,v}. Quels sont les noeuds retournés ? Construire un payload de retour comprenant uniquement leur classe et leur position (champ "cG9z_geom_point")

Requêtes géographiques : Pour ces requêtes, un index géographique sera nécessaire. Commencez par créer un index géographique sur votre collection ```nodes ```. Le champ géographique qui sera indexé est le suivant : La documentation suivante vous sera utile : https://www.arangodb.com/docs/stable/aql/tutorial-geospatial.html. 
- Récupérer tous les noeuds proches du point de coordonnées latitude 5.772045969859884, longitude 45.206042961330446, dans un rayon de 100m. Utilisez la documentation suivante comme exemple https://www.arangodb.com/docs/stable/aql/functions-geo.html  (fonction geo_distance). Pour définir le point avec ces coordonnées, vous pouvez au choix créer un payload json, ou utiliser la fonction geo_point
- Construisez un résultat comprenant les noeuds, leurs un arc sortants vers d'autres noeuds, et leur distance au parking de la mairie sans spécifier les coordonnées du parking (pour retrouver ce noeud, faites une requêtes document sur la propriété "aHR0cDovL3B1cmwub3JnL2RjL3Rlcm1zL2Rlc2NyaXB0aW9u" qui doit contenir la chaîne :"Parking Mairie"

Partie 2 : cas d'usage batiment connecté. Requêtes graphes et sémantiques
- 
- Nous allons explorer la base thingin pour cette partie 

Nous allons explorer le domaine "http://thingin.orange.com/ifc/Meylan_v3/" qui contient des données de batiment, mais contient trop ne noeuds pour être visualisé directement. 




Utiliser le template de requête suivant pour filtrer les objets par leur classe dans le domaine. Pour retrouver ces classes nous pouvons les rechercher sur le web, ou via le service dédié de thingin [indications pour le choix de classe à l'oral]. 

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
Exécuter la requête suivante : 
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
