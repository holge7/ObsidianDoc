
### ¿Qué es Mongo DB?

#MongoDB es un sistema gestor de bd NO SQL (base de datos orientadas a documento, es decir, que cada registro se va a guardar en un documento en formato #JSON), usa JS como lenguaje de consulta por defecto.

La estructura que sigue Mongo DB es: Database > Collection > Document > Fields
Aquí una representación visual:

![[mongodb.svg]]

Los documentos van a estar compuesto por campos, y clasificados en colecciones que se almacenan en una base de datos.

Como mencione antes, los documentos se guardan en formato JSON,  aquí un ejemplo de un documento de una persona:
```json
{
  "_id": {
    "$oid": "58c59c6b99d4ee0af9e0f7a8"
  },
  "name": "Jorge",
  "age": 21,
  "city": "Málaga",
  "stack":["Spring", "MongoDB", "MySQL"]
}
```

La nomenclatura que se sigue para llamar a los diferentes campos de un document es la siguiente:
![[Sin título 1.png]]


Para las pruebas de esta documentación, estoy usando un cluster de bd que otorga mongoDB para hacer pruebas, esta es la uri:
`mongodb://m001-student:m001-mongodb-basics@cluster0-shard-00-00-jxeqq.mongodb.net/?tls=true&authMechanism=DEFAULT&readPreference=primary&replicaSet=Cluster0-shard-0`

### Mongo SHELL

Bien, para comenzar a trabajar con mongoDB vamos a usar la CLI, para ello deberemos haber añadido previamente mongo db como una variable de entorno y haber instalado el mongo shell añadiendolo también como variable de entorno, podemos comprobar con el comando (mongod --version), ahora, como hacemos para conectarnos a un cluster desde la mongo CLI:

Este cluster lo he creado desde: https://cloud.mongodb.com/v2/63f327340cab99585cb06da8#/clusters

````
mongosh "mongodb+srv://test.wsfvfwn.mongodb.net/myFirstDatabase" --apiVersion 1 --username holge --password Password123
````

Usaremos también la aplicación de Studio 3T para lanzar queries.



### Tipos de datos

MongoDB trabaja con estos tipos de datos:
- String: Permite almacenar cadenas (UTF-8).
- Integer32: Valores entero numéricos.
- Integer64: Valores entero numéricos.
- Double: Almacena valores flotante.
- Object: Almacena un objeto (otro documento).
- Array: Permite almacenar un array con elementos de distintos tipos.
- Boolean: true o false.
- Date: No es más que un Integer64 que representa el número de milisegundos desde la época Unix (1 de enero de 1970)

### Gestión de conexiones y usuarios

- `db.createUser()` : Crea un nuevo usuario y asigna roles para controlar el acceso a la bd.
- `db.updateUser()` : Actualiza un usuario existente con nuevos roles.
- `db.dropUser()` : Elimina un usuario existente

### Gestión de BD

En MongoDB, algunos comandos para gestionar las BD son:

- `show dbs` : Muestra una lista de todas las bd disponibles.
- `use <db>` : Selecciona una bd existente para usar (Si la base de datos no existe previamente, se creará cuando insertes el primer documento).
- `db.dropDatabase()` : Elimina la bd seleccionada actualmente.
- `db.getCollectionNames()` : Devuelve una lista de todas las colecciones en una bd seleccionada.
- `db.runCommand()` : Ejecuta un comando en la bd seleccionada.
- `db.stats()` : Devuelve información estadística sobre la bd seleccionada.

Estos comandos permiten a los administradores de bd realizar tareas importantes, como seleccionar  bd, eliminar db, obtener información de la bd, etc.

### Operaciones DDL

MongoDB es una base de datos NoSQL, lo que significa que su modelo de datos es diferente al de las bbdd relacionales. En MongoDB, no existe un lenguaje de definición de datos (DDL) formal como en SQL, ya que las colecciones y documentos se crean y modifican dinámicamente mediante operaciones CRUD.

Aún así, MongoDB proporciona algunas operaciones que se asemejan a las operaciones DDL de SQL y permiten adminsitrar la estructura de la  base de datos:

- `db.createCollection()`: Crea una nueva conlección en una bd existente.
- `db.dropCollection()`: Elimina una colección existente de la bd.
- `db.createIndex()`: Crea un índice en una o varias colecciones.
- `db.dropIndex()`: Elimina un índice existente de una colección.
- `db.renameCollection()`: Cambia el  nombre de una colección existente.
- `db.createView()`: Crea una vista que devuelve resultados basados en una consulta.

Estas operaciones permiten a los desarrolladores y administradores de bbdd realizar tareas importantes de gestión de bbdd, como la creación de nuevas colecciones y la eliminación de colecciones existentes, así como la creación y eliminación de índices y vistas.

### Operaciones CRUD

##### C
- `insertOne()` : Inserta un documento.
- `insertMany()` : Inserta varios documentos.

```mongoDB
db.usuarios.insertOne({
    nombre: "Juan",
    apellidos: "Pérez",
    edad: 30,
    email: "juan.perez@gmail.com"
});

db.usuarios.insertMany(
	[
		{
		  "nombre": "Juan",
		  "edad": 25,
		  "email": "juan@gmail.com",
		  "ciudad": "Madrid",
		  "hobbies": ["fútbol", "leer"],
		  "trabajo": {
		    "empresa": "XYZ S.L.",
		    "puesto": "desarrollador"
		  }
		},
		{
		  "nombre": "Ana",
		  "edad": 32,
		  "email": "ana@hotmail.com",
		  "ciudad": "Barcelona",
		  "hobbies": ["pintar", "viajar"],
		  "trabajo": {}
		},
		{
		  "nombre": "Jorge",
		  "edad": 21,
		  "email": "jorge@gmail.com",
		  "ciudad": "Madrid",
		  "hobbies": ["montar en bici", "cocinar"],
		  "trabajo": {
		    "empresa": "XYZ S.L.",
		    "puesto": "gerente"
		  }
		},
		{
		  "nombre": "María",
		  "edad": 28,
		  "email": "maria@yahoo.com",
		  "ciudad": "Valencia",
		  "hobbies": ["nadar", "bailar"],
		  "trabajo": {
		    "empresa": "ABC S.A.",
		    "puesto": "diseñadora"
		  }
		}
	]
);
```

Por  defecto las insercciones estan de forma "ordenada", estoquiere decir que si vamos a insertar más de un documento, y uno de ellos da fallo, mongo dejara de insertar el resto de documentos que no haya insertado a partir del que de el error, para cambiar esto, basta con especificar que la forma de insercción sea: "desordenada":

```
db.usuarios.insertMany(
	[
		{....},
		{....}
	], 
	{ 
		"ordered": false 
	}
);
```

##### R
- `find()` : Operación de lectura para recuperar documentos de una colección.

```mongoDB
db.usuarios.find();
```

##### U
- `updateOne(filter, update, options)` : Actualiza un documento.
- `updateMany()` : Actualiza varios documentos.
- `replaceOne()` : Reemplazar un solo documento.

```mongoDB
db.usuarios.updateOne(
    { _id: ObjectId("63f379d21ed204d7258df979") },
    { $set: { email: "juan.nuevoemail@gmail.com" } }
);
```

##### D
- `deleteOne()` : Elimina un documento de uan colección.
- `deleteMany()` : Elimina varios documentos de una colección.

```mongoDB
db.users.deleteOne({ "_id": ObjectId("61fcfa1a6d266a7e913e310c") })
db.users.deleteMany({ "city": "Madrid" })
```



### Cursores

El  cursor de mongoDB no es más que un index encargado de recordar cual fue el ultimo elemento que apareción en nuestra busqueda, cada vez que llamamos al método find(), además de los elementos devultos, también se retorna un objeto de clase Cursor.

Si no especificamos en ningún momento el valor de este cursor, mongo nos retornara los elementos de X en X

usando el comando: `it` podremos obtener los X siguientes.


### Operadores

#### Dato especifico

Para realizar una consulta de un dato en especifico sobre los documentos, usaremos expresiones con una clave-valor:

```mongoDB
db.usuarios.find({"_id":ObjectId("63f379d21ed204d7258df979")});
```

#### Operadores de selección
MongoDB proporciona una amplia variedad de operadores de consulta que permiten buscar y filtrar documentos en una colección de MongoDB.

Un operador consta de una clave precedida por un signo de dólar "$" y el valor que utilizara el operador de la consulta para filtrar documentos.

Algunos de los operadores de consulta más comunes incluyen:

##### Operadores de comparación
Permiten comparar valores en documentos de una colección.\
- `$eq` : igual a.
- `$ne` : no igual a.
- `$gt` : mayor que.
- `$lt` : menor que.
- `$gte` : mayor o igual que.
- `$lte` : menor o igual que.

```mongoDB

db.usuarios.find({edad: {$eq: 28}})
> María

db.usuarios.find({edad: {$ne: 32}})
>Juan, Jorge, María

db.usuarios.find({edad: {$gt: 25}})
>Ana, María

db.usuarios.find({edad: {$lt: 25}})
>Jorge

db.usuarios.find({edad: {$gte: 28}})
>Ana, María

db.usuarios.find({edad: {$lte: 25}})
>Juan, Jorge
```

##### Operadores lógicos
Permiten combinar múltiples expresiones de consulta en una sola expresión:
- `$and` : y lógico.
- `$or` : o lógico.
- `$not` : negación.

```mongoDB
db.usuarios.find({
  $and: [
    { nombre: "Juan" },
    { edad: { $gt: 20 } }
  ]
})
> Juan

db.usuarios.find({
  $or: [
    { edad: { $lt: 25 } },
    { ciudad: "Valencia" }
  ]
})
> Jorge, María

db.usuarios.find({
  nombre: { $not: { $eq: "Ana" } }
})
> Juan, Jorge, María


```

##### Operadores de elemento:
Permiten buscar documentos que contengan campos que cumplan ciertas condiciones.
- `$exists` : para buscar documentos que tengan un campo específico.
- `$type` : para buscar documentos que tengan un tipo específico.
- `$in` : para buscar documentos cuyos valores de campo estén dentro de un conjunto específico.
- `$nin` : para buscar documentos cuyo valor para un campo dado no está en una lista de valores.

```mongoDB
db.usuarios.find({"trabajo.puesto": {$exists: true}})
> Juan, Jorge, María

db.usuarios.find({"edad": {$type: "number" }})
> Juan, Ana, Jorge, María

db.usuarios.find({"edad": {"$in": [25, 28]}})
> Juan, María

db.usuarios.find({"edad": {"$nin": [25, 28]}})
> Jorge, Ana
```


##### Operadores de expresión regular
Permiten buscar documentos que contengan cadenas de texto que coincidan con patrones específicos.
- `$regex`

```mongoDB
db.usuarios.find({ email: { $regex: /@gmail.com$/ }})
> Juan, Jorge
```

##### Operadores de arrego
Permiten buscar documentos que contengan arreglos que cumplan ciertas condiciones.
- `$all` : para buscar documentos que contengan todos los elementos de un conjunto específico.
- `$elemMatch`: para buscar documentos que contengan un arreglo con elementos que coincidan con ciertas condiciones
- `$size` : para buscar documentos que contengan arreglos de un tamaño específico.

```mongoDB
db.usuarios.find({hobbies: {$all: ["leer", "fútbol"]}})
> Juan

db.usuarios.find({hobbies: {$elemMatch: {$regex: ".*viajar.*"}}})
> Ana

db.usuarios.find({hobbies: {$size: 2}})
> Juan, Ana, Jorge, María
```

##### Operadores de proyección
Permite seleccionar qué campos incluir o excluir de los resultados de una consulta.
- `$project` : para incluir o excluir campos específicos.
- `$slice` : para incluir solo una parte del arreglo.
- `$sort` : Indicar campo por el que ordenar los documentos (1 ascendiente, -1 descendiente).

```mongoDB
db.usuarios.aggregate([
  {
    $project: {
      nombre: 1,
      email: 1
    }
  }
]);

db.usuarios.aggregate([
  {
    $project: {
      hobbies: { $slice: ["$hobbies", 1] }
    }
  }

]);

db.usuarios.aggregate(
	{ $sort : { edad : 1 } }
);

```

Hay muchos otros operadores de consulta disponibles en MongoDB, y cada uno proporciona una forma poderosa de buscar y filtrar documentos en una colección.







#### Operadores de Actualización

Estos operadores se deben usar con los metodos 
- db.collection.updateOne()
- db.collection.updateMany()


##### Operadores de campos

- `$set` : establece el valor de un campo para un documento.
- `$unset` : elimina los campos que especefiquemos.
- `$rename` : renombra un campo del documento.
- `$mul` : multiplica un campo por la cantidad que especifiquemos.
- `$min` : solo actualiza un campo si el valor que ya existe es menor que le especificado.
- `$max` : solo actualiza un campo si el valor que ya existe es mayor que le especificado.
- `$currentDate` : establece el valor de la fecha actual.
- `$inc` : incrementa el valor de un campo con la cantidad que especifiquemos.

##### Operadores de vectores

- `$push` : añade elementos a un vector (al final).
- `$addToSet` : añade un elemento al array solo si no existe dicho elemento ya en el array.
- `$pop` : elimina el primero o ultimo elemento de un arrayl.
- `$pull` : elimina todos los documentos que coincidan con la consulta especificada.
- `$pullAll` : elimina todos los valores que coinciden con un array.

###### Modificadores
Los  modificadores son operadores que se pueden usar con los operadores de vectores:

- `$each` : Modifica los operadores `$push` y `$addToSet` para agregar varios elementos para las actualizaciones de un vector (insertaria 3 elementos, sin el $each insertaria un vector con 3 elementos).

```mongoDB
db.movieDetails.updateOne(
	{nombre: "The Martian"},
	{$push: {
		reviews: {
			$each: [
				{
					rating:2.5,
					reviwer: "Heidi Onda",
					text: "Justa another movie"
				},{
					rating:0.5,
					reviwer: "Esther",
					text: "The worst movie of the year"
				},{
					rating:3.5,
					reviwer: "Alex",
					text: "Nice movie"
				},
			]
		}
	}}
)
```

- `$position` : Permite cambiar la ubicación de la insercción de un elemento que va a ser insertado con `$push`, cada vez que se quiera usar `$position` tendremos que usar obligados también `$each`

```mongoDB
db.movieDetails.updateOne(
	{title; "The Martian"},
	{$push: {
		contador: {
		$each: [7],
		$position: 2
		}
	}}
)
```

Este ejemplo insertaria el numero 7 en la posición 2.