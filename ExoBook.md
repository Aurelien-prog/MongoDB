
# 1.
``` javascript
use sample_db

db.createCollection("employees")
show collections

// "show" collection sert à vérifier si la collection à bien été créer !
```
Retourne la collection "employees".

# 2.
```javascript
db.employees.insertMany ([ 
	{ name: "John Doe", age: 35, job: "Manager", salary: 80000 },
	{ name: "Jane Doe", age: 32, job: "Developer", salary: 75000 },
	{ name: "Jim Smith", age: 40, job: "Manager", salary: 85000 } 
])

// J'utilise "insertMany" pour pouvoir créer plusieurs documents. 
```

```javascript
{
	acknowledged: true,
	insertedIds: 
	{
		'0': ObjectId("63e11c0dfee85882267503e1"),
		'1': ObjectId("63e11c0dfee85882267503e2"),
		'2': ObjectId("63e11c0dfee85882267503e3")
	}
}
```


# 3.
```javascript
db.employees.find({}) 

// "find" sert pour chercher quelque chose, et les accolades dans les parenthèses indiquent que l'on veut tous ce qu'il y à dans la collection "employees".
```

```javascript
{  _id: ObjectId("63e11c0dfee85882267503e1"),  name: 'John Doe',  age: 35,  job: 'Manager',  salary: 80000}
{  _id: ObjectId("63e11c0dfee85882267503e2"),  name: 'Jane Doe',  age: 32,  job: 'Developer',  salary: 75000}
{  _id: ObjectId("63e11c0dfee85882267503e3"),  name: 'Jim Smith',  age: 40,  job: 'Manager',  salary: 85000}
```

# 4.
```javascript
db.employees.find({ age: { $gt: 33 } }) 

// On utilise toujour "find" mais avec cette fois ci des paramètres tel que l'âge, ainsi que "$gt" qui est un opérateur qui signifie (supérieur à ...).
```

```javascript
{  _id: ObjectId("63e11c0dfee85882267503e1"),  name: 'John Doe',  age: 35,  job: 'Manager',  salary: 80000}
{  _id: ObjectId("63e11c0dfee85882267503e3"),  name: 'Jim Smith',  age: 40,  job: 'Manager',  salary: 85000}
```

# 5.
```javascript
db.employees.find({}).sort({ salary: -1 }) 

// On se sert içi de "sort" qui va nous servir à trié les salaires.
// Aussi le "-1" lui va servir à faire le trie dans l'ordre décroissant. ("1" pour l'ordre croissant).
```

```javascript
{  _id: ObjectId("63e11c0dfee85882267503e3"),  name: 'Jim Smith',  age: 40,  job: 'Manager',  salary: 85000}
{  _id: ObjectId("63e11c0dfee85882267503e1"),  name: 'John Doe',  age: 35,  job: 'Manager',  salary: 80000}
{  _id: ObjectId("63e11c0dfee85882267503e2"),  name: 'Jane Doe',  age: 32,  job: 'Developer',  salary: 75000}
```

# 6.
```javascript
db.employees.find({}, { name: 1, job: 1, _id: 0 }) 

// Içi on va afficher seulement le nom et le job de tous les employées.
// Il faut assigné la valeur "0" ou "false" à "_id" car il sera affiché par défaut.
```

```javascript
{  name: 'John Doe',  job: 'Manager'}
{  name: 'Jane Doe',  job: 'Developer'}
{  name: 'Jim Smith',  job: 'Manager'}
```

# 7.
```javascript
db.employees.aggregate([ { $group: { _id: "$job", count: { $sum: 1 } } } ]) 

// L'opérateur "group" va nous servir à regrouper les documents par valeur.
// L'opérateur "sum" va nous servir à compter le nombre de documents dans chaques groupes.
```

```javascript
{  _id: 'Manager',  count: 2}
{  _id: 'Developer',  count: 1}
```

# 8.
```javascript
db.employees.updateMany( { job: "Developer" }, { $set: { salary: 80000 } } ) 

// Içi on utilise l'opérateur "set" pour définir une valeur, içi sur le salaire à '80000'.
// Les modification seront apporté a tous les documents qui ont pour job "Developer".
// "updateMany" sert a mettre à jour tous les documents concerner en une seule commande.
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
Tous les documents qui auront pour job "Developer" auront désormais comme salaire "80000".
