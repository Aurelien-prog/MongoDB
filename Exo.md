# Création de la table

# 1.
```javascript
db.salles.find({"smac": true}, {"nom": 1})

// On définit smac sur true pour trouver seulement ceux à qui smac est sur "true".
// On définit sur 1 le "nom" pour pouvoir l'afficher.
```

```javascript
{  _id: 1,  nom: 'AJMI Jazz Club'}
{  _id: 2,  nom: 'Paloma'}
```

# 2.
```javascript
db.salles.find( {"capacite" : {$gt : 1000}}, {"nom" : 1, "_id": 0})

// On utilise l'opérateur "gt" pour spécifier que l'on veut seulement les documents qui ont une capacité supérieur à 1000.
```

```javascript
{ nom: 'Paloma'}
```

# 3.
```javascript
db.salles.find( {"adresse.numero": { $exists: false }}, {"_id": 1})

// On vérifie avec "exists" si il existe un numero pour chaque adresse de salle.
// On affiche "id" de ceux qui n'en possèdent pas.
```

```javascript
{ _id: 3}
```

# 4.
```javascript
db.salles.find( {"avis": { $size : 1}}, {"_id": 1, "nom": 1})
// On précise la taille avec "size" pour dire que l'on veut seulement les salles avec un seul avis.
```

```javascript
{ _id: 2, nom: 'Paloma' }
```

# 5.
```javascript
db.salles.find({ "styles": "blues" }, { "styles": 1, "_id": 0 })
```

```javascript
{ styles: [    'jazz',    'soul',    'funk',    'blues'  ]}
{ styles: [    'blues',    'rock'  ]}
```

# 6.
```javascript
db.salles.find({ "styles.0": "blues" }, { "styles": 1, "_id": 0 })
// On précise avec "style.0" que l'on veut uniquement les salles avec comme premier "styles" blues.
// Dans un index, le 0 est compter comme le 1.
```

```javascript
styles: [
	'blues',
	'rock'
]
```

# 7.
```javascript
db.salles.find({ "adresse.codePostal": { $regex: /^84/ }, "capacite": { $lt: 500 } }, { "adresse.ville": 1, "_id": 0 })
// On regarde le code postal situé dans les adresses des salles.
// On utilise "regex" pour spécifier que le code postal doit commencer par '84'.
// On spécifie que la capacité doit être inférieur à '500' places avec "$lt"
```

```javascript
{  adresse: {    ville: 'Avignon'  }}
{  adresse: {    ville: 'Le Thor'  }}
```

# 8.
```javascript
db.salles.find({ $or: [ { "_id": { $mod: [ 2, 0 ] } }, { "avis": { $exists: false } } ] }, { "_id": 1 })
// "$or" est un opérateur logique pour opérer sur le tableaux qui contient les expressions "$mod" et "$exists".
// On utilise "$mod" pour effectuer une opération modulo.
```

```javascript
{  _id: 2}
{  _id: 3}
```

# 9.
```javascript
db.salles.find({ "avis.note": { $gte: 8, $lte: 10 } }, { "nom": 1, "_id": 0 })
// On utilise "$gte" ainsi que "$lte" pour pour préciser que l'on veut les notes comprise entre ces deux numéros.
```

```javascript
{  nom: 'AJMI Jazz Club'}
{  nom: 'Paloma'}
```

# 10.
```javascript
db.salles.find({ "avis.date": { $lt: new Date("2019-11-15") } }, { "nom": 1, "_id": 0 })
// On utilise "$lt" pour préciser que l'on veut spécifiquement les dates inférieur au '2019-11-15'.
// On utilise du Javascript "new Date" pour initialiser la date.
```

```javascript
{  nom: 'AJMI Jazz Club'}
{  nom: 'Paloma'}
```

# 11.
```javascript
db.salles.find({ $expr: { $gt: [ { $multiply: [ "$_id", 100 ] }, "$capacite" ] } }, { "nom": 1, "capacite": 1, "_id": 0 })
// "$expr" permet l'utilisation d'expression d'agrégation.
// On utilise "$multiply" pour multiplier l'Id par 100.
```

```javascript
{  nom: 'Sonograf',  capacite: 200}
```

# 12. A faire
```javascript
db.salles.find({ $where: function() { return obj.smac }}, { "nom": 1, "_id": 0 })
// L'expression "$where" est utilisé pour pouvoir se servir du javascript dans cette requête.
```

```javascript

```

# 13.
```javascript
db.salles.distinct("adresse.codePostal")
```

```javascript
[ '30000', '84000', '84250' ]
```

# 14.
```javascript
db.salles.updateMany({}, {$inc: {capacite: 100}})
// "$inc" pour ajouter des données.
```

```javascript
{
	acknowledged: true,
	insertedId: null,
	matchedCount: 3,
	modifiedCount: 3,
	upsertedCount: 0
}
```

# 15.
```javascript
db.salles.updateMany({styles: {$ne: "jazz"}}, {$push: {styles: "jazz"}})
// L'opérateur "$push" va ajouté la valeur "jazz" partout où elle n'est pas trouvé avec l'opérateur "$ne".
```

```javascript
{
	acknowledged: true,
	insertedId: null,
	matchedCount: 2,
	modifiedCount: 2,
	upsertedCount: 0
}
```

# 16.
```javascript
db.salles.updateMany({_id: {$ne: 2}, _id: {$ne: 3}}, {$pull: {styles: "funk"}})
```

```javascript
{  
	acknowledged: true,  
	insertedId: null,  
	matchedCount: 2,  
	modifiedCount: 1,  
	upsertedCount: 0
}
```

# 17.
```javascript
db.salles.updateOne({_id: 3}, {$addToSet: {styles: {$each: ["techno", "reggae"]}}})
// "$each" va ajouter les valeurs "techno" et "reggae" si elle n'existe pas pour la salle avec l'identifiant '3'.
```

```javascript
{  
	acknowledged: true,  
	insertedId: null,  
	matchedCount: 1,  
	modifiedCount: 1,  
	upsertedCount: 0
}
```

# 18.
```javascript
db.salles.updateMany({nom: {$regex: /^P/i}}, {$inc: {capacite: 150}, $push: {contact: {telephone: "04 11 94 00 10"}}})
// On rajoute 150 de 'capacite' et un champs telephone pour les salles dont le 'nom' commence par un 'P/p'.
```

```javascript
{  
	acknowledged: true,  
	insertedId: null,  
	matchedCount: 1,  
	modifiedCount: 1,  
	upsertedCount: 0
}
```

# 19.
```javascript
db.salles.updateMany({ nom: { $regex: /^[aeiou]+/i } }, { $push: { avis: { date: new Date(), note: 10 } } })
// On importe la "Date" avec 'new Date'.
```

```javascript
{
	acknowledged: true,
	insertedId: null,
	matchedCount: 1,
	modifiedCount: 1,
	upsertedCount: 0
}
```

# 20.
```javascript
db.salles.upsert({ nom: { $regex: /^Z/i } }, { $push: { "nom": "Pub Z" } } )
```

```javascript

```

# 21.
```javascript
db.salles.find({_id: {$type: "objectId"}}).toArray().length;
```

```javascript
{ 1 }
```

# 22.
```javascript
db.salles.find({"_id": {$not: {$type: "objectId"}}}).sort({capacite: -1}).limit(1).forEach(function(doc) { print(doc.nom); });
```

```javascript
{ 'Paloma' }
```

# 23.

```javascript

```

```javascript

```

# 24.
```javascript
db.salles.deleteOne({ _id: { $type: "objectId" }, capacite: { $lte: 60 } })
```

```javascript
{
	acknowledged: true,
	deletedCount: 1
}
```

# 25.
```javascript
db.salles.findOneAndUpdate( { ville: "Nîmes", nom: "Salle située à Nîmes" }, { $inc: { capacite: -15 } } );
```

```javascript
{ null }
```