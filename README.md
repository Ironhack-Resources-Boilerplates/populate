# .populate()

Population es un método de Mongoose que nos permite reemplazar automáticamente una ruta en un documento con documentos reales de otras colecciones. Esto te permite tener un esquema para cada uno de ellos y mantener las cosas bien ordenadas. Veámos un ejemplo con usuarios y posts:

#### Paso 1: Crea tus Schemas

Necesitamos un Schema para cada colección. Uno para usuarios y otro para los posts. Así que vamos a hacerlo:

###### Modelo User

```js
const mongoose = require("mongoose");
const Schema   = mongoose.Schema;

const userSchema = new Schema({
  username: String,
  posts: [{
    type: Schema.Types.ObjectId,
    ref: 'Post'
  }]
})

const User = mongoose.model("User", userSchema);

module.exports = User;
```

###### Modelo Post

```js
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const postSchema = new Schema ({
  content: String,
  author {
  type: Schema.Types.ObjectId,
  ref: 'User'
}
})

const Post = mongoose.model('Post', postSchema);

module.exports = Post;
```

Las propiedades donde queremos usar .populate() son las propiedades donde hay el **type: Schema.Types.ObjectId**. Este type le está diciendo a Mongoose '¡Hey! ¡Voy a referenciar documentos de otras colecciones!'. La siguiente parte de la propiedad es **ref**. El ref le dice a Mongoose 'Estos documentos estarán en la colección ___________________________'

Entonces, en nuestro Schema de User, referenciamos la colección Post, porque queremos que el usuario esté vinculado a las cosas que publica y queremos poder acceder fácilmente a esas publicaciones sin tener que crear más consultas. 

#### Paso 2: Crear correctamente usuarios y publicaciones

Cuando vinculamos colecciones al modelo usando el type y ref apropiados, los datos reales almacenados serán el **_id** del otro documento. Se almacenará como un string. Esto también funcionaria en el caso de tener un **array de _id**.

Veamos lo que comentamos. Nuestro esquema dice esto:

```js
const userSchema = new Schema({
  username: String,
  posts: [{
    type: Schema.Types.ObjectId,
    ref: 'Post'
  }]
})
```

La propiedad **posts** se leerá como algo así:

```js
{
  _id: 59ab1c92ea84486fb4ba9f28,
  username: JD,
  posts: [
    "59ab1b43ea84486fb4ba9ef0",
    "59ab1b43ea84486fb4ba9ef1"
  ]
}
```

El primer _id que encontramos es el _id del usuario y dentro de la propiedad posts encontramos una array con los id. Estos id en forma de string son los id de cada Post. 

Hay que tener en cuenta que este es el documento almacenado. Todavía no hemos llamado a la función .populate(). Una vez la llamemos, irá a la colección apropiada, buscará esos dos _ids y devolverá un array con los posts reales. Vamos a verlo:

####  Paso 3: Implementamos .populate()

Aquí tenemos la función:

```js
function getUserWithPosts(username){
  return User.findOne({username: username})
  .populate('posts')
  .exec((err, posts) =>{
    console.log('Populated User' + posts)
  })
}
```

.populate() necesita una consulta a la que adjuntarse, por lo que usamos el método **User.findOne()** para encontrar el usuario que coincida con el nombre de usuario que le proporcionamos como argumento. Esto nos devuelve el usuario encontrado y aquí es donde .populate() toma el control. A la función .populate() le pasamos el argumento 'posts'. Al proporcionale este argumento le estamos diciendo a .populate() con qué propiedad de nuestro usuario queremos que funcione. 

Llamar a .exec() simplemente ejecuta algo una vez que .populate() ha hecho su función. 

De manera que, con el .populate() conseguimos esto:

```js
{ 
  _id: 59ab1c92ea84486fb4ba9f28,
  username: 'JD',
  posts:
    [ 
      { 
        _id: 59ab1b43ea84486fb4ba9ef0,
        content: "Is it dark out?"
      },{
        _id: 59ab1b43ea84486fb4ba9ef1,
        content: "Hey anyone got a cup of sugar?"
      }
    ]
  }
```



¡y eso es todo! ¡Hemos creado un único objeto usando 2 Schemas, 2 Modelos y 2 colecciones!

