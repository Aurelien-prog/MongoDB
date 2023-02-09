# 1.
## Importation d'une dataset
## a.
```javascript
mongoimport --db Restaurant --collection Business --file yelp_academic_dataset_business.json
```

## Vérification que les coordonnées existe
## b.
```javascript
db.Business.find({ 
	"location.latitude": 40.3381827, 
	"location.longitude": -75.4716585 }, { "name": 1, "_id": 0 })
// Commande pour rechercher que les restaurants possèdent bien une 'latitude' et une 'longitude'.
```

```javascript
{ name: 'Perkiomen Valley Brewery' }
// Coordonnées du premier restaurant du dataset.
```

# 2.
## Les restaurants ouverts à partir de 18H
## a.
```js
db.business.find({ $or: 
	[ 
		{"hours.Monday.open":{ $regex: /^18:/ }}, 
		{"hours.Tuesday.open":{ $regex: /^18:/ }}, 
		{"hours.Wednesday":{ $regex: /^18:/ }}, 
		{"hours.Thursday.open":{ $regex: /^18:/ }}, 
		{"hours.Friday.open":{ $regex: /^18:/ }}, 
		{"hours.Saturday.open":{ $regex: /^18:/ }}, 
		{"hours.Sunday.open":{ $regex: /^18:/ }} 
	] 
}, {"name": 1}) 
// Les restaurants qui ouvre a 8h un jour dans la semaine 

db.business.aggregate([ 
	{ 
		$project: { 
			name:1, 
			ouverture: { $objectToArray : "$hours" } 
		} 
	}, { $match: { "ouverture.v.open":{ $regex: /^18:/ } } }, 
	{ $project: { name:1 } } 
])
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
db.Business.createIndex({ "hours": 1 }, { "name": "IndexHoraire" })

db.Business.createIndex({ "hours.Monday.open": 1 }, { "name": "IndexOuvertureLundi" }) db.Business.createIndex({ "hours.Tuesday.open": 1 }, { "name": "IndexOuvertureMardi" }) db.Business.createIndex({ "hours.Wednesday": 1 }, { "name": "IndexOuvertureMercredi" }) db.Business.createIndex({ "hours.Thursday.open": 1 }, { "name": "IndexOuvertureJeudi" }) db.Business.createIndex({ "hours.Friday.open": 1 }, { "name": "IndexOuvertureVendredi" }) db.Business.createIndex({ "hours.Saturday.open": 1 }, { "name": "IndexOuvertureSamedi" }) db.Business.createIndex({ "hours.Sunday.open": 1 }, { "name": "IndexOuvertureDimanche" })

db.Business.createIndex({ 
	"hours.Monday.open": 1, 
	"hours.Tuesday.open":1, 
	"hours.Wednesday":1, 
	"hours.Thursday.open":1, 
	"hours.Friday.open":1, 
	"hours.Saturday.open":1, 
	"hours.Sunday.open":1 },
	{ "name": "IndexOuvertureSemaine"
})
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
db.Business.createIndex({ lieu: "2dsphere" })

db.Business.find(
	{ 
		lieu: { 
			$near: { 
				$geometry: { 
					type: "Point",
					coordinate: [ -119.7111968, 34.4266787 ]
			    }, $maxDistance: 2000
		    } 
		}
	}, {name: 1, _id: 0, address: 1, city: 1}
)
```

```js
{  name: 'Abby Rappoport, LAC, CMQ', address: '1616 Chapala St, Ste 2', city: 'Santa Barbara' }
{  name: 'Axxess Card', address: '1616 Chapala St, Ste 1', city: 'Santa Barbara' }
{  name: 'James House Santa Barbara', address: '1632 Chapala St', city: 'Santa Barbara' }
{  name: 'Cochrane Property Management, Inc.', address: '102 W Arrellaga St', city: 'Santa Barbara' }
{  name: 'Cheshire Cat Inn', address: '36 W Valerio St', city: 'Santa Barbara' }
{  name: 'El Prado Inn', address: '1601 State St', city: 'Santa Barbara' }
{  name: 'Kunz John David, MD', address: '101 W Arrellaga St, Ste B', city: 'Santa Barbara' }
{  name: 'Optometry Care Santa Barbara', address: '1629 State St, Ste 1', city: 'Santa Barbara' }
{  name: 'Heath Montgomery DMD', address: '14 W Valerio St, Ste C', city: 'SANTA BARBARA' }
{  name: 'Century 21 Butler Realty', address: '1635 State St', city: 'Santa Barbara' }
{  name: 'La Quinta Inn & Suites by Wyndham Santa Barbara Downtown', address: '1601 State Street', city: 'Santa Barbara' }
{  name: "Cantwell's Market & Deli", address: '1533 State St', city: 'Santa Barbara' }
{  name: 'IHOP', address: '1701 State St', city: 'Santa Barbara' }
{  name: 'Cami Ferris-Wong, DDS', address: '1525 State St, Ste 201', city: 'Santa Barbara' }
{  name: 'Robert Half', address: '1525 State St, Ste 101', city: 'Santa Barbara' }
{  name: 'The Presidio Hotel', address: '1620 State St', city: 'Santa Barbara' }
{  name: 'Financial Credit Network', address: '7 W Figueroa St, Ste 302', city: 'Santa Barbara' }
{  name: 'The Supply Room - The Presidio Motel', address: '1620 State St', city: 'Santa Barbara' }
{  name: 'Santa Barbara Hair and Makeup', address: '1634 De La Vina St', city: 'Santa Barbara' }
{  name: 'Lucent Skincare Santa Barbara', address: '1525 State St, Ste 206', city: 'Santa Barbara' }
```

## Rechercher les restaurants dans un rayon autour d'un point
## b.
```js
db.Business.find(
	{ 
		lieu: { 
			$near: { 
				$geometry: { 
					type: "Point",
					coordinate: [ -119.7111968, 34.4266787 ]
			    }, $maxDistance: //Définir la distance
		    } 
		}
	}, {name: 1, _id: 0, address: 1, city: 1}
)
```

# Framework d'agrégation
## Calculez la moyenne des notes des restaurants
## a.
```js
db.Business.aggregate([ { $group: { _id: null, moy: { $avg: "$stars" } } } ])
```

```js
{
	_id: null,
	moy: 3.5967235576603303
}
```

## Trouver les restaurants les plus populaires avec le nombre de commentaires
## b.
```js
db.Business.aggregate([ 
	{ $sort: 
	   { review_count: -1 } 
	},
	{ $limit: 20 },
	{ $project: 
		{ 
			_id: 0,
			name: 1,
			city: 1, 
			review_count: 1 
		}
	} 
])
```

```js
{  name: 'Acme Oyster House',  city: 'New Orleans',  review_count: 7568}
{  name: 'Oceana Grill',  city: 'New Orleans',  review_count: 7400}
{  name: 'Hattie B’s Hot Chicken - Nashville',  city: 'Nashville',  review_count: 6093}
{  name: 'Reading Terminal Market',  city: 'Philadelphia',  review_count: 5721}
{  name: 'Ruby Slipper - New Orleans',  city: 'New Orleans',  review_count: 5193}
{  name: "Mother's Restaurant",  city: 'New Orleans',  review_count: 5185}
{  name: 'Royal House',  city: 'New Orleans',  review_count: 5070}
{  name: "Commander's Palace",  city: 'New Orleans',  review_count: 4876}
{  name: 'Luke',  city: 'New Orleans',  review_count: 4554}
{  name: 'Cochon',  city: 'New Orleans',  review_count: 4421}
{  name: "Pat's King of Steaks",  city: 'Philadelphia',  review_count: 4250}
{  name: 'Biscuit Love: Gulch',  city: 'Nashville',  review_count: 4207}
{  name: "Pappy's Smokehouse",  city: 'Saint Louis',  review_count: 3999}
{  name: "Felix's Restaurant & Oyster Bar",  city: 'New Orleans',  review_count: 3966}
{  name: 'Gumbo Shop',  city: 'New Orleans',  review_count: 3902}
{  name: 'Cochon Butcher',  city: 'New Orleans',  review_count: 3837}
{  name: 'Los Agaves',  city: 'Santa Barbara',  review_count: 3834}
{  name: "Willie Mae's Scotch House",  city: 'New Orleans',  review_count: 3582}
{  name: "Geno's Steaks",  city: 'Philadelphia',  review_count: 3401}
{  name: 'Grand Sierra Resort and Casino',  city: 'Reno',  review_count: 3345}
```

# Export de la base de données
## Exporter les données en fichier `csv`
## a.
```js
mongoexport --collection=Business --db=Restaurant --out=Restaurant.csv
```

