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

## Les opérateurs logiques

- $and -> et
- 


# /////////////////////////////////////////////////////////////////////////


# Travail à faire :

### Effectuer des recherches sur la méthode `findAndModify`.
#### Essayer de lever une erreur qui permet de vérifier que les schémas de validation appliqués fonctionnent.
