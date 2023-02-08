
# 1.
```javascript
db.createCollection("salles", { 
	validator: { 
		$and: 
		[ 
			{nom: { $type: "string", $exists: true } }, 
			{capacite: { $type: "int", $exists: true } }, 
			{"adresse.codePostal":{ $type:"string", $exists:true } }, 
			{ "adresse.ville": { $type: "string", $exists: true } } 
		] 
		} 
	}
)
```

```javascript
{ ok: 1 }
```

```javascript
db.salles.insertOne( 
{"nom": "Super salle", "capacite": 1500, "adresse": {"ville": "Musiqueville"}} 
) 
```

```javascript
MongoServerError: Document failed validation
```

Ajouter le champ manquant "adresse.codePostal" à l'objet à insérer, de manière à ce qu'il réponde aux exigences de validation définies pour la collection "salles". La requête d'insertion pourrait alors ressembler à ceci :

```javascript 
db.salles.insertOne( {"nom": "Super salle", "capacite": 1500, "adresse": {"ville": "Musiqueville", "codePostal": "00000"}} )
```

```javascript
{
	acknowledged: true,
	insertedId: ObjectId("63e35931f48ed97edfe93408")
}
```

# 2.
```javascript
db.runCommand({ 
	collMod: "salles", 
	validationAction: "error", 
	validator: { 
		$and: [ 
			{nom: { $type: "string", $exists: true } }, 
			{capacite: { $type: "int", $exists: true } }, 
			{"adresse.codePostal":{$type:"string", $exists: true } }, 
			{ "adresse.ville": { $type: "string", $exists: true } }, 
			{ _id: { $type: ["int", "objectId"], $exists: true } } 
		] 
	} 
})
```

La méthode `db.runCommand()` avec un document de commande contenant les options pour mettre à jour la validation de la collection.

```javascript
{
	ok: 1,
	'$clusterTime': {
		clusterTime: Timestamp({ t: 1675844248, i: 1 }),
		signature: {
			hash: Binary(Buffer.from("f559228ae979b8f92064a7dda0b9cb34ae56f0e4", "hex"), 0),
			keyId: Long("7171806562036482052")
		}
	},
	operationTime: Timestamp({ t: 1675844248, i: 1 })
}
```

```
db.salles.updateMany({}, {$set: {"verifie": true}}) 
```

```javascript
{
	acknowledged: true,
	insertedId: null,
	matchedCount: 1,
	modifiedCount: 1,
	upsertedCount: 0
}
// Si on exécute cette requête, tous les documents existants dans la collection "salles" seront mis à jour avec un nouveau champ "verifie" défini à "true".
```

```javascript
db.runCommand({ collMod: "salles", validationAction: "error", validator: {} })
```

```javascript
{
	ok: 1,
	'$clusterTime': {
		clusterTime: Timestamp({ t: 1675844597, i: 1 }),
		signature: {
			hash: Binary(Buffer.from("d3a9541ab3b54f731e712b7286d4b5c626a56fdd", "hex"), 0),
			keyId: Long("7171806562036482052")
		}
	},
	operationTime: Timestamp({ t: 1675844597, i: 1 })
}
```

# 3.
```javascript
db.runCommand({
   collMod: "salles",
   validationAction: "strict",
   validator: {
      $and: [
         { nom: { $type: "string", $exists: true } },
         { capacite: { $type: "int", $exists: true } },
         { "adresse.codePostal": { $type: "string", $exists: true } },
         { "adresse.ville": { $type: "string", $exists: true } },
         {
            $or: [
               { smac: { $exists: true } },
               { stylesMusicaux: { $in: ["jazz", "soul", "funk", "blues"] } }
            ]
         }
      ]
   }
})

```

```javascript
//Si aucun document n'a un champ `_id` avec la valeur 3, alors cette requête de mise à jour ne fera aucun effet et ne modifie aucun document. Si un document existe avec un champ `_id` avec la valeur 3, il sera mis à jour pour inclure le champ `verifie` avec la valeur `false`.
```