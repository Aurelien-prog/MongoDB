### Collection : 
Groupe de documents MongoDB. C'est l'équivalent d'une table dans un système de gestion de base de données. Les collections n'imposent pas de schéma précis.

Ceci est un 'objet vide' en JSON :
``` JSON
{ }
```

Les propriétés d'un objet JSON sont matérialisé par une partie clé/valeur :
``` JSON
{ 
	"clé" : "valeur",
	"clé2" : "valeur2",
	
	"un_tableau" : 
	[
		 "clé" : "valeur" 
	];
}
```


## Création d'une base de données :

``` shell
use tmp

db.maCollection.insertOne({
	"une_cle":"une_valeur"
})
```

## Supression d'une base de données

```shell
use laDaDbASupprimer

db.dropDatabase()
```

## Les helpers

```shell
db.runCommand({"collstats" : "uneCollection"})

db.adminCommand("currentOp")
```

# Gestion des collections 

## Les collations

```JSON
{
	**locale**: < string >,
	caseLevel: < boolean >,
	caseFirst: < string >,
	strength: < entier >,
	numericOrdering: < boolean >
}
```

Au sein de ce type de document le champ "locale" est obligatoire !!

## Créer une collection

```shell
db.createCollection("maCollection", { "collation": {"locale": "fr"} })
```

## Supprimer une collection

```shell
db.maCollection.Drop()
```

# Les documents

## Insertion d'un document

```javascript
db.maCollection.insert(< document /*ou un tableau de documents*/ >)
```

Insérer un tableau de documents dans une collection "personnes" :
```javascript
db.personnes.insert(
[
	{"nom": "Deschamps", "prenom": "Didier"},
	{"nom": "Marinade", "prenom": "Jean", "age": 20}
])
```

## Update un document au sein d'une collection

```javascript
db.collection.updateOne(<filtre>, <modification>)

db.personnes.update(
	{"_id": ObjectId("Deschamps")},
	{ // "$set" est un opérateur
		$set : { "nom": "Deschamp" }
	},
	{ "multi" : true }
)
```

### Cas de l'upsert :
#### La combinaison d'update et d'insert !

```javascript
db.personnes.updateOne(
	{"prenom": "remi"},
	{
		$set : { "prenom": "Remi" }
	}
	{ "upsert": true }
)
```

## Validation des documents

```javascript
var athletes = [
	{
		"nom": "Eclair",
		"prenom": "Jean-Michel",
		"discipline": "Course"
	},
	{
		
	}
]

db.athletes.insertMany(athletes)
```


#### Maintenant que nos règles sont établies, nous allons modifier la collection à l'aide de la commande `collMod`

```javascript
db.runCommand(
	{
		"collMod": "athletes",
		"validator": 
		{
			$jsonSchema: 
			{
				"bsonType": "object",
				"required": [ "nom", "prenom", "discipline" ],
				"properties": "proprieties"	
			}	
		}
	}
)
```


## Les opérateurs 

- $ne -> différent de ...
- $gt -> suppérieur à ...
- $gte -> suppérieur ou égal à ...
- $lt -> inférieur à ...
- $lte -> inférieur ou égal à ...
- $in /  $nin -> absence ou présence de ...
- $eq -> égale à ...

## Les opérateurs logiques

- $and -> et

## L'opérateur $expr

Il permet d'utiliser des expressions dans nos requêtes. Ces expressions pourront contenir des operateurs, des objets ou encore des chemins d'objet pointant vers des champs.

```javascript
// Requête affichant le nom des personnes dont la longueur du nom multiplié par 12 est supérieur à leur âge.
db.personnes.find({
	"nom": { $exists: 1},
	"age": { $exists: 1 },
	$expr: { $gt: [ { $multiply: [ { $strLenCP: "$nom" }, 12 ] }, "$age" ] } 
},
{ // Projection
	"nom": 1,
	"_id": 0
})
```

```javascript
// Requête permettant d'afficher les comptes dont la sommes des opérations de débit est supérieur au montant du crédit.
db.banque.find({
	"debit": {$exists: 1},
	$expr: {
		$gt: [
			{ $sum: "$debit" },
			"$credit"
		]
	}
})
```

## L'opérateur $type

```javascript
{ champ: { $type: < type BSON > } }
```

```javascript
db.personnes.insertOne({
	"nom": "Zidane", "prenom": "Z", "age": numberInt(50)
})
```

## L'opérateur $mod

```javascript
{ champ: { $mod: [diviseur, reste ] } }
```

## L'opérateur $where

```javascript
db.personnes.find ({ $where: "this.nom.length > 6" })

db.personnes.find ({ $where: function() {
	return obj.nom.length > 6
} })
```

## Les opérateurs de tableau

```javascript
{ $push: { <champ>: <valeur>, ... } }
{ $pull: { <champ>: <valeur>, ... } }

db.hobbies.updateOne ({ "nom": "Yves" }, { $push: { "passions": "Line" } })
```

## L'opérateur $addToSet

```javascript
db.personnes.find({ "interets": "jardinage" })
// Requêtes permettant d'afficher les personnes ayant l'un des deux interets (peu importe l'ordre.)
db.personnes.find({ "interets": { $all : [ "bridge", "jardinage" ] } })

db.personnes.find({ "interets.1": "jardinage" })
// Récupère les personnes ayant deux interets exactement.
db.personnes.find({ "interets": { $size: 2 } })
// Récupère les personnes ayant au moins deux interets.
db.personnes.find({ "interets.1": { $exists:1 } })
```

## L'opérateur $elemMatch

```javascript
db.eleves.find({ "notes": { $elemMatch: { $gt: 0, $lt: 10 } }})
// Affiche les élèves ayant une note entre 5 et 18.50.
db.eleves.find({ "notes": { $all: [5, 7.50] } })
```

## Les tableaux de documents

```javascript
db.eleves.find({ "notes.note": 10 })
```

Renvoyer les documents dont les élèves ont au moins une note entre 10 et 15 dans une matière quelqconque :

```javascript
db.eleves.find({
	"notes" : { 
		$elemMatch: { 
			"note": { $gt: 10, $lte: 15 }
		} 
	} 
})

db.eleve.find({  })

db.eleves.find({ "notes.0.note": { $lt: 10 } })
```

## Le tri

```javascript
curseur.sort( <tri> )
```

# Les index

La nature de votre application devra impacter votre logique d'indexation : 
Est-elle orientée écriture (write-heavy) ? ou lecture (read-heavy) ?

```javascript
db.collection.createIndex(<champ + type>, <options?>)

db.collection.createIndex({ "age": -1 })
{
	"createCollectionAutomatically": false,
	"numIndexesBefore": 1,
	..
	ok:1
}

db.personnes.getIndexes()
// Retourne un tableau

db.personnes.dropIndex( "age_-1" )
db.personnes.createIndex({ "age": -1 }, { "name": "index_age" })
```

Un index peut porter sur plus d'un champ :
C'est ce qu'on appelle un index composé.

```javascript
db.personnes.createIndex({ "age": })
```

MongoDB permet l'utilisation de deux types d'Index qui permettent de gérer les requêtes géospatiales :
- Les indexs de type `2dsphere` sont utilisés par des requêtes géospatiales intervenant sur une surface sphérique.
- Les indexs `2d` concernent des requêtes intervenant sur un plan Euclidien.

Pour un champ nommé `donneesSpatiales` d'une collection `cartographie` vous pouvez par exemple créer un index de type `2d` avec la commande :

``` javascript
db.cartographie.createIndex({ "donneesSpatiales": "2d" })
```

Pour la création d'un index `2dsphere` on utilisera plutôt :

```javascript
db.cartographie.createIndex({ "donneesSpatiales": "2dsphere" })
```

Les indexs 2d font intervenir dess coordonnées de type `legacy`

```javascript
db.plan.insertOne({ "nom": "firstPoint", "geodata": [1, 1] })

db.plan.insertOne({ "nom": "firstPoint_bis", "geodata": [4.7, 44.5] })

db.plan.insertOne({ "nom": "firstPoint_bis", "geodata": [ "lon": 4.7, "lat": 44.5] })
```

## Les objets GeoJSON

```javascript
{ type: <type d_objet>, coordinates: <coordonnees> }
```

### Le type Point

```javascript
{ 
	"type": "Point", 
	"coordinates": [ 14.0, 1.0 ]
}
```

### Le type MultiPoint

```javascript
{
	"type": "MultiPoint",
	"coordinates": [
		[13.0, 1.0], [13.0, 3.0]
	]
}
```

### Le type LineString

```javascript
{
	"type": "LineString",
	"coordinates": [
		[13.0, 1.0], [13.0, 3.0]
	]
}
```

### Le type Polygon

```javascript
{
	"type": "Polygon",
	"coordinates": [
		[13.0, 1.0], [13.0, 3.0]
	]
	[
		[13.0, 1.0], [13.0, 3.0]
	]
}	
```

#### Création d'index

```javascript
db.avignon.createIndex({ "localisation": "2dsphere" })

db.avignon2d.createIndex({ "localisation": "2d" })
```

## L'opérateur $nearSphere

```javascript
{
	$nearSphere: {
		$geometry: {
			type: "Point",
			coordinates: [<longitude>, <latitude>]
		},
		$minDistance: <distance en metres>,
		$maxDistance: <distance en metres>
	}
}

{
	$nearSphere: [ <x>, <y> ],
	$minDistance: <distance en radians>,
	$maxDistance: <distance en radians>
}
```

```javascript
var opera = { type: "Point", coordinates: [43.949749, 4.805325]}

//Requête sur la collection avignon
db.avignon.find(
	{
		"localisation": {
			$nearSphere: {
				$geometry: opera
			}	
		}
	}, { "_id": 0, "nom": 1 }
)
```

## L'opérateur $geoWithin
Cet opérateur n'effectue aucun tri et ne necessite pas la création d'un index geospatiale, on l'utilise de la manière suivante:

```javascript
{
	<champ des documents contenant les coordonnées>: {
		$geoWithin: {
			<operateur de forme>: <coordonnees>
		}
	}
}
```

Création d'un polygone de notre exemple :
```javascript
var polygone = [
	[43.9548, 4.80143],
	[43.95475, 4.80779],
	[43.95045, 4.81097],
	[43.4657, 4.80449],
]
```

La requête suivante utilise ce polygone :
```javascript
db.avignon2d.find(
	{
		"localisation": {
			$geoWithin: {
				$polygon: polygone
			}
		}
	}, { "_id": 0, "nom": 1 }	
)
```

## Signature pour le cas d'utilisation d'objets GeoJSON

```javascript
{
	<champ des documents contenant les coordonnées>: {
		$geoWithin: {
			type: < "Polygon" ou bien "MultiPolygon" >,
			coordinates: [<coordonnees>]
		}
	}
}
```

```javascript
var polygone = [
	[43.9548, 4.80143],
	[43.95475, 4.80779],
	[43.95045, 4.81097],
	[43.4657, 4.80449],
]

db.avignon2d.find(
	{
		"localisation": {
			$geoWithin: {
				$geometry: {
					type: "Polygon",
					coordinates: [polygone]
				}
			}
		}
	},	{ "_id": 0, "nom": 1 }
)
```

## Le framework d'agrégation

MongoDB met à disposition un puissant outil d'analyse et de traitement de l'information: 
Le pipeline d'agreagation (ou framework)

Métaphore du tapis roulant d'usine.

Méthode utilisée :
```javascript
db.collection.aggregate(pipeline, options)
```

- pipeline : désigne un tableau d'étapes
- options : désigne un document

Parmis les options, nous retiendrons :

- Les collations, permet d'affecter une collation à l'opération d'aggrégation.
- bypassDocumentValidation : Fonctionne avec un opérateur appelé `$out` et permet de passer au travers de la validation des documents.
- allowDiskUse : Donne la possibilité de faire déborder les opérations d'écriture sur le disque.

Vous pouvez appeler aggregate sans arguments : 
```javascript
db.personnes.aggregate()
```

Au sein du `shell`, nous allons créer une variable pipeline :
```javascript
var pipeline = []

db.personnes.aggregate(pipeline)
db.personnes.aggregate(
	pipeline, {
		"collation": { "locale": "fr" }
	}
)
```

## L'opérateur $addFilds

```javascript
{ $addFields: { <nouveau champ> : <expression>, ... } }

db.personnes.aggregate([
	{
		$addFields: {
			"numero_secu_s": ""
		}
	}
])
 
 db.personnes.aggregate([
	{
		$project: {
			......
			"numero_secu_s": 1
		}
	}
])
```


## L'opérateur $group
```javascript
{
	$group: {
		"_id": <expression>,
		<champ>: { <operateur d_accumulation> }
	}
}
```

Opérateur d'accumulation: `$push`, `$sum`, `$avg`, `$min`, `$max`.

```javascript
var pipeline = [{
	$group: {
		"_id": "age",
		"nombre_personnes": { $sum: 1 }
	}
},
{
	$sort: {
		"nombre_personnes": 1
	}
}]

db.personnes.aggregate(pipeline)


db.personnes.aggregate([
	{
		$sortByCount: "$age"
	}
])


var pipeline = [{
	$group: {
		"_id": null,
		"nombre_personnes": { $sum: 1 }
	}
}]


var pipeline = [
	{
		$match: {
			"age": { $exists: true }
		}
	},
	{
		$group: {
			"_id": null,
			"$avg": {
				$avg: "$age"
			}
		}
	},
	{
		$project: {
			"_id": 0,
			"age_moyen": {
				"$ceil": "$avg"
			}
		}
	}
]
```










# ///////////////////////////////////////////


# Travail à faire :

### Effectuer des recherches sur la méthode `findAndModify`.
#### Essayer de lever une erreur qui permet de vérifier que les schémas de validation appliqués fonctionnent.
