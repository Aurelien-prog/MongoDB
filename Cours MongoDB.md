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







# /////////////////////////////////////////////////////////////////////////


# Travail à faire :

### Effectuer des recherches sur la méthode `findAndModify`.
#### Essayer de lever une erreur qui permet de vérifier que les schémas de validation appliqués fonctionnent.
