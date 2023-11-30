# 6-Update-Operations

1. [Intro](#schema1)
2. [Updating Fields with "updateOne()", "updateMany()" and "$set"](#schema2)
3. [Incrementing & Decrementing Values](#schema3)
4. [Using $min, $max and $mul](#schema4)
5. [Getting Rid of Fields and Renaming Fields](#schema5)
6. [Undestanding "upsert()"](#schema6)
7. [Updating Matched Array Elements](#schema7)
8. [Updating All Array Elements](#schema8)
9. [Finding & Updating Specific Fields](#schema9)
10. [Adding and Removing Elements to Arrays](#schema10)


<hr>

<a name="schema1"></a>

## 1. Intro

![update](./img/1update.png)

<hr>

<a name="schema2"></a>

## 2. Updating Fields with "updateOne()", "updateMany()" and "$set"

- updateOne()

La sintaxis básica de updateOne() es la siguiente:

``` 
db.collection.updateOne(
  { /* Filtro */ },
  { $set: { /* Nuevos valores */ } }
);
```

Donde:

- collection: Es el nombre de la colección en la que deseas realizar la actualización.
- Filtro: Es la condición que determina qué documento(s) se actualizarán.
- $set: Es un operador de actualización que especifica los nuevos valores que se deben asignar al documento.

```
users> db.persons.updateOne({_id:ObjectId('6567259ef5d66fd59e935429')},{$set:{hobbies:[{title:'Sports',frequency:5},{title:'Cooking',frequency:3},{title:'Hiking',frequency:1}]}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
users> db.persons.find()
[
  
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ]
  }
]

```

- updateMany()
```
users> db.persons.updateMany({"hobbies.title":"Sports"},{$set:{isSport:true}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 3,
  modifiedCount: 3,
  upsertedCount: 0
}
users> db.persons.find({"hobbies.title":"Sports"})
[
  {
    _id: ObjectId('6567259ef5d66fd59e935426'),
    name: 'Max',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    phone: 131782734,
    isSport: true
  },
  {
    _id: ObjectId('6567259ef5d66fd59e935428'),
    name: 'Anna',
    hobbies: [
      { title: 'Sports', frequency: 2 },
      { title: 'Yoga', frequency: 3 }
    ],
    phone: '80811987291',
    age: null,
    isSport: true
  },
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSport: true
  }
]

```
```
users> db.persons.updateOne({_id:ObjectId('6567259ef5d66fd59e935429')},{$set:{age:40,phone:1233456 }})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
{
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    age: 40,
    phone: 1233456
  }
]

```

<hr>

<a name="schema3"></a>

## 3. Incrementing & Decrementing Values

- Incrementing
Incrementa la edad en 1
```
db.persons.updateOne({name:'Manuel'},{$inc:{age:1}})
```
- Decrementig

Poniendo el `-2` decrementa el valor de age.


```
db.persons.updateOne({name:'Manuel'},{$inc:{age:-2},$set:{isSporty:false}})

```

<hr>

<a name="schema4"></a>

## 4. Using $min, $max and $mul

### Min
Tenemos:

```
{
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    age: 40,
    phone: 1233456

```
Ejecutamos la sentencia:

```
db.persons.updateOne({name:'Chris'},{$min:{age:35}})
 {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    age: 35,
    phone: 1233456
  }
```
Como podemos ver el valor de age ha cambiado a 35.
Pero si ejecutamos:
```
users> db.persons.updateOne({name:'Chris'},{$min:{age:38}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 0,
  upsertedCount: 0
}
```
Vemos que no se ha modificadon nada, esto pasa porque la acción `$min` solo modifica el dato cuando el dato que se va 
a agragar es menor que el que esta.


### Max
Lo mismo pasa con `$max` que solo modifica el dato cuando el valor que se va añadir es mayor que el que está.
```
users> db.persons.updateOne({name:'Chris'},{$max:{age:38}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
users> db.persons.find()
 
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    age: 38,
    phone: 1233456
  }
]
users> 

```
### Multi

```
users> db.persons.updateOne({name:'Chris'},{$mul:{age:1.1}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
users> db.persons.find()
  
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    age: 41.800000000000004,
    phone: 1233456
  }
]

```
<hr>

<a name="schema5"></a>

## 5. Getting Rid of Fields and Renaming Fields

Cuando queremos eliminar un valor de la colección.

Primera opción usando `$set`

```
db.persons.updateMany({isSporty: true},{$set:{phone: null}})
{
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    age: 41.800000000000004,
    phone: null
  }
```
Esta opción es la correcta porque sigue aparenciendo el valor aunque sea nulo.

Mejor así:
```
users> db.persons.updateMany({isSporty: true},{$unset:{phone: ""}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 3,
  modifiedCount: 3,
  upsertedCount: 0
}
users> db.persons.find()
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    age: 41.800000000000004
}
```
### Renaming

```
users> db.persons.updateMany({},{$rename:{age:'TotalAge'}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 4,
  modifiedCount: 3,
  upsertedCount: 0
}
users> db.persons.find()
  {
    _id: ObjectId('6567259ef5d66fd59e935427'),
    name: 'Manuel',
    hobbies: [
      { title: 'Cooking', frequency: 5 },
      { title: 'Cars', frequency: 2 }
    ],
    phone: '012177972',
    isSporty: false,
    TotalAge: 33
  },


```
<hr>

<a name="schema6"></a>

## 6. Undestanding "upsert()"

```
users> db.persons.updateOne({name:'Maria'},{$set:{age:29, hobbies:[{title:'Good food', frequency: 3}],isSporty:true}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 0,
  modifiedCount: 0,
  upsertedCount: 0
}
```
No añade ni modifica nada porque le estamos pasando un nombre que no existe en la colección e intentando 
hacer un updateOne().
Pero usando `upsert:true` que por defecto siempre está a false. Crea e inserta los datos.
upsert es una combinación entre update e insertar, es decir, si está se modifica y si no está se crea.
```
db.persons.updateOne({name:'Maria'},{$set:{age:29, hobbies:[{title:'Good food', frequency: 3}],isSporty:true}},{upsert:true})
{
  acknowledged: true,
  insertedId: ObjectId('656755d39afc3679dfa0514b'),
  matchedCount: 0,
  modifiedCount: 0,
  upsertedCount: 1
}
 {
    _id: ObjectId('656755d39afc3679dfa0514b'),
    name: 'Maria',
    age: 29,
    hobbies: [ { title: 'Good food', frequency: 3 } ],
    isSporty: true
  }

```

<hr>
<a name="schema7"></a>

## 7. Updating Matched Array Elements

Vamos a analizar que pasa con esta sentencia `$and:[{"hobbies.title":"Sports"},{"hobbies.frequency":{$gte:3}}`

```
users> db.persons.find({$and:[{"hobbies.title":"Sports"},{"hobbies.frequency":{$gte:3}}]})
[
  {
    _id: ObjectId('6567259ef5d66fd59e935426'),
    name: 'Max',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true
  },
  {
    _id: ObjectId('6567259ef5d66fd59e935428'),
    name: 'Anna',
    hobbies: [
      { title: 'Sports', frequency: 2 },
      { title: 'Yoga', frequency: 3 }
    ],
    isSporty: true,
    TotalAge: null
  },
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    TotalAge: 41.800000000000004
  }
]

```
Como vemos el caso de Maria, tiene un hobbie que es Sports y tambien una frequency de valor 3.  Lo que esta pasando es 
no busca un hobbie = Sports y una frequency = 3. Lo valores no estan en el mismo diccionario pero están dentro 
de hobbies. 
```
 hobbies: [
      { title: 'Sports', frequency: 2 },
      { title: 'Yoga', frequency: 3 }
```
Para solucionar esto vamos a utilizar `$elemMatch`

```
users> db.persons.find({hobbies:{$elemMatch:{title:'Sports',frequency:{$gt:3}}}})
[
  {
    _id: ObjectId('6567259ef5d66fd59e935426'),
    name: 'Max',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true
  },
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5 },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    TotalAge: 41.800000000000004
  }
]
```

Ahora vamos a modificar los elementos anteriores, por lo tanto usamos el mismo filtro en `updateMany` y usamos
`{$set:{'hobbies.$.highFrequency'}` para hacer referencia solo a los elementos que cumplen el filtro.
```
db.persons.updateMany({hobbies:{$elemMatch:{title:'Sports',frequency:{$gt:3}}}},{$set:{'hobbies.$.highFrequency':true}})

[
  {
    _id: ObjectId('6567259ef5d66fd59e935426'),
    name: 'Max',
    hobbies: [
      { title: 'Sports', frequency: 5, highFrequency: true },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true
  },
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      { title: 'Sports', frequency: 5, highFrequency: true },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    TotalAge: 41.800000000000004
  }
]

```


<hr>
<a name="schema8"></a>

## 8. Updating All Array Elements

```
db.persons.updateMany({'hobbies.frequency':{$gt:2}},{$set:{'hobbies.$.goodFrequency':true}})
{
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      {
        title: 'Sports',
        frequency: 5,
        highFrequency: true,
        goodFrequency: true
      },
      { title: 'Cooking', frequency: 3 },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true,
    TotalAge: 41.800000000000004
  },

```

La razón por la cual solo se actualiza el primer valor que cumple el filtro es porque estás utilizando el 
operador de posición `$` en la proyección, lo cual hace que se aplique solo al primer elemento que cumple la condición.

Cuando usas `'hobbies.$.goodFrequency'` en la proyección, MongoDB interpreta eso para modificar el primer elemento 
que coincide con la condición `'hobbies.frequency': {$gt: 2}`

En este sentencia nos de un error.
```
users> db.persons.updateMany({TotalAge:{$gt:30}},{$inc:{'hobbies.frequency':-1}})
MongoServerError: Cannot create field 'frequency' in element {hobbies: [ { title: "Cooking", frequency: 5, goodFrequency: true }, { title: "Cars", frequency: 2 } ]}

```
Añadiendo `$[]` le decimos que cualquier elemento que cumpla la condición.
```
users> db.persons.updateMany({TotalAge:{$gt:30}},{$inc:{'hobbies.`$[].frequency`':-1}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 2,
  modifiedCount: 2,
  upsertedCount: 0
}
```

<hr>
<a name="schema9"></a>

## 9. Finding & Updating Specific Fields


Si deseas actualizar todos los elementos que cumplen la condición, puedes utilizar el método de filtrado 
posicional `$[<identifier>]` junto con la opción arrayFilters. Aquí hay un ejemplo de cómo podrías hacerlo:


En este ejemplo, $[el] es un identificador que se asocia con el campo que deseas actualizar. 
La opción `arrayFilters` permite filtrar los elementos del array que cumplen con la condición 
especificada ('el.frequency': { $gt: 2 }).

Esto debería actualizar todos los elementos del array que cumplen con la condición de 'hobbies.frequency': { $gt: 2 }. 

```
users> db.persons.updateMany({'hobbies.frequency':{$gt:2}},
{$set:{"hobbies.$[el].goodFrequency":true}},
{arrayFilters:[{"el.frequency":{$gt:2}}]})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 5,
  modifiedCount: 1,
  upsertedCount: 0
}
users> db.persons.find()
[
  {
    _id: ObjectId('6567259ef5d66fd59e935426'),
    name: 'Max',
    hobbies: [
      {
        title: 'Sports',
        frequency: 5,
        highFrequency: true,
        goodFrequency: true
      },
      { title: 'Cooking', frequency: 3, goodFrequency: true },
      { title: 'Hiking', frequency: 1 }
    ],
    isSporty: true
  },
  {
    _id: ObjectId('6567259ef5d66fd59e935427'),
    name: 'Manuel',
    hobbies: [
      { title: 'Cooking', frequency: 4, goodFrequency: true },
      { title: 'Cars', frequency: 1 }
    ],
    phone: '012177972',
    isSporty: false,
    TotalAge: 33
  },
  {
    _id: ObjectId('6567259ef5d66fd59e935428'),
    name: 'Anna',
    hobbies: [
      { title: 'Sports', frequency: 2 },
      { title: 'Yoga', frequency: 3, goodFrequency: true }
    ],
    isSporty: true,
    TotalAge: null
  },
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      {
        title: 'Sports',
        frequency: 4,
        highFrequency: true,
        goodFrequency: true
      },
      { title: 'Cooking', frequency: 2 },
      { title: 'Hiking', frequency: 0 }
    ],
    isSporty: true,
    TotalAge: 41.800000000000004
  },
  {
    _id: ObjectId('656755d39afc3679dfa0514b'),
    name: 'Maria',
    age: 29,
    hobbies: [ { title: 'Good food', frequency: 3, goodFrequency: true } ],
    isSporty: true
  }
]

```


<hr>
<a name="schema10"></a>

## 10. Adding and Removing Elemnts to Arrays

### Adding

Añadiendo 1 solo valor con `$push`:

```
db.persons.updateOne({name:'Maria'},{$push:{hobbies:{title:'Sports',frequency:2}}})

  {
    _id: ObjectId('656755d39afc3679dfa0514b'),
    name: 'Maria',
    age: 29,
    hobbies: [
      { title: 'Good food', frequency: 3, goodFrequency: true },
      { title: 'Sports', frequency: 2 }
    ],
    isSporty: true
  }
]

```
Añadiendo varios valores a la vez.

```
users> db.persons.updateOne({name:'Maria'},
{$push:{hobbies:{$each:[{title:'Good Wine', frequency:1},{title:'Hiking',frequency:2}],$sort:{frequency:-1}}}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
users> db.persons.find({name:'Maria'})
[
  {
    _id: ObjectId('656755d39afc3679dfa0514b'),
    name: 'Maria',
    age: 29,
    hobbies: [
      { title: 'Good food', frequency: 3, goodFrequency: true },
      { title: 'Sports', frequency: 2 },
      { title: 'Hiking', frequency: 2 },
      { title: 'Good Wine', frequency: 1 }
    ],
    isSporty: true
  }
]

```
- updateOne({ name: 'Maria' }, ...) establece la condición de filtrado para encontrar el documento con el 
campo name igual a 'Maria'.

- $push se utiliza para agregar elementos a un array. En este caso, se están agregando elementos al array
hobbies dentro del documento que cumple con la condición.

- $each se utiliza para especificar una lista de elementos que se agregarán al array. 
En este caso, se están agregando dos elementos: { title: 'Good Wine', frequency: 1 } y { title: 'Hiking', frequency: 2 }.

- $sort: { frequency: -1 } se utiliza para ordenar el array hobbies después de agregar los nuevos elementos. 
El -1 indica orden descendente basado en el campo frequency. Esto significa que los elementos con frecuencia 
más alta estarán al principio del array.

#### "$addToSeT"

```
users> db.persons.updateOne({name:'Maria'},{$addToSet:{hobbies:{title:'Hiking',frequency:2}}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 0,
  upsertedCount: 0
}
users> db.persons.find({name:'Maria'})
[
  {
    _id: ObjectId('656755d39afc3679dfa0514b'),
    name: 'Maria',
    age: 29,
    hobbies: [
      { title: 'Good food', frequency: 3, goodFrequency: true },
      { title: 'Sports', frequency: 2 },
      { title: 'Hiking', frequency: 2 },
      { title: 'Good Wine', frequency: 1 }
    ],
    isSporty: true
  }
]
```

Diferencias entre `$push` y `$addToSet`
$push: Este operador se utiliza para agregar uno o varios elementos a un array existente en un documento. 
Si el array ya contiene el valor que estás intentando agregar, $push simplemente lo añadirá nuevamente. 
No realiza ninguna verificación para evitar duplicados.

$addToSet: Este operador se utiliza para agregar elementos a un array, 
pero con la particularidad de que verifica si el valor ya existe en el array antes de agregarlo. 
Si el valor ya está presente, no se añadirá de nuevo, evitando duplicados en el array.


### Removing

- Eliminando un valor
```
users> db.persons.updateOne({name:'Chris'},{$pop:{hobbies:1}})
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
users> db.persons.find({name:'Chris'})
[
  {
    _id: ObjectId('6567259ef5d66fd59e935429'),
    name: 'Chris',
    hobbies: [
      {
        title: 'Sports',
        frequency: 4,
        highFrequency: true,
        goodFrequency: true
      },
      { title: 'Cooking', frequency: 2 }
    ],
    isSporty: true,
    TotalAge: 41.800000000000004
  }
]

```