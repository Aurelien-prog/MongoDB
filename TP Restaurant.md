# 1.
## Importation d'une dataset
## a.
```javascript
mongoimport --db Restaurant --collection Business --file yelp_academic_dataset_business.json
```

## Vérification que les coordonnées existe
## b.
```javascript
db.Business.find({ "location.latitude": 40.3381827, "location.longitude": -75.4716585 }, { "name": 1, "_id": 0 })
// Commande pour rechercher que les restaurants possèdent bien une 'latitude' et une 'longitude'.
```

```javascript
{ name: 'Perkiomen Valley Brewery' }
// Coordonnées du premier restaurant du dataset.
```

# 2.
## Les restaurants ouverts à partir de 18H
## a. A finir
```js
db.Business.find({
	"hours.Wednesday": { $regex: /^18/ }, { "name": 1, "_id": 0 }
})
```

## Trie des restaurants de la meilleur note à la pire.
## b.
```js
db.Business.find({},
	{
		"name": 1,
		"stars": 1,
		"_id": 0
	})
	.sort(
	{ 
		stars: -1
	}
)
```

```js
{  name: 'Adams Dental',  stars: 5}
{  name: 'Abby Rappoport, LAC, CMQ',  stars: 5}
{  name: 'Indian Walk Veterinary Center',  stars: 5}
{  name: 'Altitude Trampoline Park - Boise',  stars: 5}
{  name: 'Sierra Pro Events',  stars: 5}
{  name: 'Stomel Elliot Attorney-At-Law',  stars: 5}
{  name: 'All In Shipping',  stars: 5}
{  name: 'Core de Roma',  stars: 5}
{  name: 'Jennie Deckert',  stars: 5}
{  name: 'Paws The Cat Cafe',  stars: 5}
{  name: 'Mr. Margarita',  stars: 5}
{  name: "Roy's Appliance Service",  stars: 5}
{  name: 'James Dant',  stars: 5}
{  name: 'Impasto',  stars: 5}
{  name: 'Premier Mortgage Resources',  stars: 5}
{  name: 'Cornerstone Physical Therapy Associates',  stars: 5}
{  name: 'Tinkle Belle Diaper Service',  stars: 5}
{  name: "Cook's Glass & Mirror",  stars: 5}
{  name: 'David Gower, Jr.  - Coldwell Banker Preferred',  stars: 5}
{  name: "P's & Q's - Premium Quality",  stars: 5}
```

# 3.
## Indexation
## a.
```js
db.Business.createIndex({ "hours": 1 }, { "name": "IndexOuverture" })
```

## Vérifie que l'index est créer avec `listIndexes`
## b.
```js
db.runCommand ({ listIndexes: "Business" }).cursor.firstBatch
```

 ```js
[
	{ v: 2, key: { _id: 1 }, name: '_id_' },
	{ v: 2, key: { hours: 1 }, name: 'IndexOuverture' }
]
```

# 4.
## Rechercher les restaurants dans une zone de 2km
## a.
```js
db.Business.createIndex({ "location": "2dsphere" })

db.Business.find(
	{
		"location": {
			$nearSphere: {
				$geometry: {
					type: "Point",
					coordinates: [-90.335695, 38.551126]
				},
				$maxDistance: 2000
			}
		}
		
	}, { "_id": 0, "name": 1, "address": 1, "city": 1 }
)
```
