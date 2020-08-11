**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Active Record Basics
============================
Lo esencial de Active Record

Esta guía es una introducción a Active Record.

Después de leer esta guía, sabrá:

* Qué son el mapeo relacional de objetos y el registro activo y cómo se usan en
  Rails.
* Cómo Active Record encaja en el paradigma Modelo-Vista-Controlador.
* Cómo usar los modelos de Active Record para manipular los datos almacenados en una relación
  base de datos.
* Convenciones de nomenclatura del esquema de registro activo.
* Los conceptos de migraciones de bases de datos, validaciones y devoluciones de llamadas.

------------------------------------------------------------------------------------------

What is Active Record?
----------------------
¿Qué es Active Record?


Active Record es la M en [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) - el
modelo - que es la capa del sistema responsable de representar a las empresas
datos y lógica. Active Record facilita la creación y el uso de objetos de negocios
cuyos datos requieren almacenamiento persistente en una base de datos. Es un
implementación del patrón Active Record, que en sí mismo es una descripción de un
Sistema de Mapeo Relacional de Objetos.

### The Active Record Pattern
El patrón de Active Record

[Active Record fue descrito por Martin Fowler](https://www.martinfowler.com/eaaCatalog/activeRecord.html)
en su libro _Patterns of Enterprise Application Architecture_. En
Active Record, los objetos llevan datos persistentes y comportamiento que
opera con esos datos. Active Record considera que garantizar
la lógica de acceso a datos como parte del objeto educará a los usuarios de ese
objeto sobre cómo escribir y leer de la base de datos.

### Object Relational Mapping
Mapeo Relacional de Objetos

[Mapeo relacional de objetos](https://en.wikipedia.org/wiki/Object-relational_mapping), comúnmente conocido como su abreviatura ORM, es
Una técnica que conecta los objetos ricos de una aplicación a tablas en
un sistema de gestión de bases de datos relacionales. Usando ORM, las propiedades y
Las relaciones de los objetos en una aplicación pueden almacenarse fácilmente y
recuperado de una base de datos sin escribir sentencias SQL directamente y con menos
código general de acceso a la base de datos.

NOTA: El conocimiento básico de los sistemas de gestión de bases de datos relacionales (RDBMS) y el lenguaje de consulta estructurado (SQL) es útil para comprender completamente Active Record. Consulte [este tutorial](https://www.w3schools.com/sql/default.asp) (o [este](http://www.sqlcourse.com/)) o estudíelos por otros medios si te gustaría aprender más.

### Active Record as an ORM Framework
Active Record como marco de referencia ORM

Active Record nos brinda varios mecanismos, el más importante es la habilidad
a:

* Representar modelos y sus datos.
* Representar asociaciones entre estos modelos.
* Representar jerarquías de herencia a través de modelos relacionados.
* Valide los modelos antes de que persistan en la base de datos.
* Realizar operaciones de base de datos de manera orientada a objetos.

Convention over Configuration in Active Record
----------------------------------------------
Convención sobre configuración en Active Record

Al escribir aplicaciones utilizando otros lenguajes de programación o marcos,
puede ser necesario escribir mucho código de configuración. Esto es particularmente cierto
para marcos ORM en general. Sin embargo, si sigue las convenciones adoptadas por
Rails, necesitará escribir muy poca configuración (en algunos casos no
configuración en absoluto) al crear modelos Active Record. La idea es que si
configura sus aplicaciones de la misma manera la mayor parte del tiempo, entonces esto
debería ser la forma predeterminada. Por lo tanto, se necesitaría una configuración explícita
solo en aquellos casos en los que no puede seguir la convención estándar.

### Naming Conventions
Convenciones de nombres

De forma predeterminada, Active Record utiliza algunas convenciones de nomenclatura para averiguar cómo
Se debe crear un mapeo entre modelos y tablas de bases de datos. Rails
pluraliza los nombres de sus clases para encontrar la tabla de base de datos respectiva. Entonces, para
una clase `Book`, debe tener una tabla de base de datos llamada **books**. Los mecanismos 
de pluralización de Rails son muy potentes, capaces de pluralizar (y
singularizar) palabras regulares e irregulares. Cuando se usan nombres de clase compuestos
de dos o más palabras, el nombre de la clase modelo debe seguir las convenciones de Ruby,
usando el formulario CajaCamello, mientras que el nombre de la tabla debe contener las palabras separadas
por guiones bajos. Ejemplos:

* Clase de modelo - Singular con la primera letra de cada palabra en mayúscula (p.ej.,
`BookClub`).
* Tabla de base de datos - Plural con guiones bajos que separan las palabras (p.ej., `book_clubs`).

| Modelo / Clase | Tabla / Esquema |
| ---------------- | -------------- |
| `Article`        | `articles`     |
| `LineItem`       | `line_items`   |
| `Deer`           | `deers`        |
| `Mouse`          | `mice`         |
| `Person`         | `people`       |

### Schema Conventions
Convenciones de Esquema

Active Record utiliza convenciones de nomenclatura para las columnas en las tablas de la base de datos,
dependiendo del propósito de estas columnas.

* **Claves foráneas**: Estos campos deben nombrarse siguiendo el patrón
  `nombre_de_tabla_singularizada_id` (p.ej.,`item_id`, `order_id`). Estos son los
  campos que Active Record buscará cuando cree asociaciones entre
  sus modelos
* **Claves primarias** - Por defecto, Active Record usará una columna entera llamada
  `id` como clave principal de la tabla (`bigint` para PostgreSQL y MySQL, `integer`
  para SQLite). Cuando se utiliza [Migraciones de registros activos](active_record_migrations.html)
  Para crear sus tablas, esta columna se creará automáticamente.

También hay algunos nombres de columna opcionales que agregarán características adicionales
a instancias de Active Record:

* `created_at` - Se establece automáticamente en la fecha y hora actuales cuando
  el registro se crea por primera vez.
* `updated_at` - Se establece automáticamente en la fecha y hora actuales siempre que
  El registro se crea o actualiza.
* `lock_version` - Agrega [optimistic
   locking](https://api.rubyonrails.org/classes/ActiveRecord/Locking.html) para
  un modelo.
* `type` - Especifica que el modelo usa [Single Table
  Inheritance](https://api.rubyonrails.org/classes/ActiveRecord/Base.html#class-ActiveRecord::Base-label-Single+table+inheritance).
* `(nombre_ asociación) _type` - Almacena el tipo para
  [asociaciones polimórficas](association_basics.html # polymorphic-asociaciones).
* `(table_name)_count` - Se usa para almacenar en caché el número de objetos pertenecientes en
  asociaciones. Por ejemplo, una columna `comments_count` en una clase` Article` que
  tiene muchas instancias de `Comentario` almacenará en caché el número de comentarios existentes
  para cada artículo.

NOTA: Si bien estos nombres de columna son opcionales, de hecho están reservados por Active Record. Manténgase alejado de las palabras clave reservadas a menos que desee la funcionalidad adicional. Por ejemplo, `type` es una palabra clave reservada que se usa para designar una tabla usando la herencia de tabla única (Singel Table Inheritance). Si no está utilizando STI, intente con una palabra clave análoga como "contexto", que aún puede describir con precisión los datos que está modelando.

Creating Active Record Models
-----------------------------
Crear modelos de Active Record


Para crear modelos de Active Record, subclasifique la clase `ApplicationRecord` y estará listo:

```ruby
class Product < ApplicationRecord
end
```

Esto creará un modelo de `Producto`, mapeado a una tabla de` productos` en
base de datos. Al hacer esto, también tendrá la capacidad de mapear las columnas de cada
fila en esa tabla con los atributos de las instancias de su modelo. Suponer
que la tabla `productos` fue creada usando una declaración SQL (o una de sus extensiones) como:

```sql
CREATE TABLE products (
   id int(11) NOT NULL auto_increment,
   name varchar(255),
   PRIMARY KEY  (id)
);
```

El esquema anterior declara una tabla con dos columnas: `id` y` name`. Cada fila de
Esta tabla representa un determinado producto con estos dos parámetros. Por lo tanto, usted
sería capaz de escribir código como el siguiente:

```ruby
p = Product.new
p.name = "Some Book"
puts p.name # "Some Book"
```
Overriding the Naming Conventions
---------------------------------
Anulación de las convenciones de nomenclatura


¿Qué sucede si necesita seguir una convención de nomenclatura diferente o si necesita usar su
Aplicación Rails con una base de datos heredada? No hay problema, puedes anular fácilmente
Las convenciones predeterminadas.

`ApplicationRecord` hereda de` ActiveRecord :: Base`, que define un
Número de métodos útiles. Puede usar el `ActiveRecord :: Base.table_name =`
Método para especificar el nombre de la tabla que se debe utilizar:

```ruby
class Product < ApplicationRecord
  self.table_name = "my_products"
end
```

Si lo hace, tendrá que definir manualmente el nombre de la clase que aloja
los accesorios (my_products.yml) usando el método `set_fixture_class` en su prueba
definición:

```ruby
class ProductTest < ActiveSupport::TestCase
  set_fixture_class my_products: Product
  fixtures :my_products
  ...
end
```

También es posible anular la columna que se debe usar como tabla
clave primaria usando el método `ActiveRecord :: Base.primary_key =`: método

```ruby
class Product < ApplicationRecord
  self.primary_key = "product_id"
end
```

NOTA: Active Record no admite el uso de columnas de clave no primaria llamadas `id`.

CRUD: Reading and Writing Data
------------------------------
CRUD: lectura y escritura de datos


CRUD es un acrónimo de los cuatro verbos que usamos para operar con datos: **C**reate,
**R**ead, **U**pdate and **D**elete. Active Record crea automáticamente métodos
para permitir que una aplicación lea y manipule datos almacenados dentro de sus tablas.

### Create

Los objetos Active Record se pueden crear a partir de un hash, un bloque o tener su
atributos establecidos manualmente después de la creación. El método `new` devolverá un nuevo
object mientras que `create` devolverá el objeto y lo guardará en la base de datos.

Por ejemplo, dado un modelo `Usuario` con atributos de` nombre` y `ocupación`,
la llamada al método `create` creará y guardará un nuevo registro en la base de datos:

```ruby
user = User.create(name: "David", occupation: "Code Artist")
```

Usando el método `new`, un objeto puede ser instanciado sin ser guardado:

```ruby
user = User.new
user.name = "David"
user.occupation = "Code Artist"
```

Una llamada a `user.save` confirmará el registro en la base de datos.

Finalmente, si un bloque es proveído, tanto `create` como` new` producirá el nuevo
inicializado dentro de un bloque:

```ruby
user = User.new do |u|
  u.name = "David"
  u.occupation = "Code Artist"
end
```

### Read

Active Record provee una rica API para acceder a datos dentro de una base de datos. De abajo
son algunos ejemplos de diferentes métodos de acceso a datos por Active Record.

```ruby
#  devuelve una colección de usuarios
users = User.all
```

```ruby
# devuelve el primer usuario
user = User.first
```

```ruby
# devuelve el primer usuario llamado David
david = User.find_by(name: 'David')
```

```ruby
# encontrar todos los usuarios llamados David que tienen de ocupación Code Artists y ordenado por created_at en sentido cronológicamente inverso
users = User.where(name: 'David', occupation: 'Code Artist').order(created_at: :desc)
```

Puedes aprender más acerca de consultar un modelo Active Record en la guía [Active Record
Query Interface](active_record_querying.html).

### Update

Una vez que un objeto Active Record ha sido recuperado, sus atributos pueden ser modificados 
y volver a ser guardados en la base de datos.

```ruby
user = User.find_by(name: 'David')
user.name = 'Dave'
user.save
```

Una forma abreviada de esto es usar un nombre de atributo de mapeo hash para el deseado
valor, así:

```ruby
user = User.find_by(name: 'David')
user.update(name: 'Dave')
```

Esta es la manera más útil al actualizar varios atributos a la vez. Si, por otro lado, 
quieres actualizar varios a la vez, encontrarás muy útil el método `update_all`:

```ruby
User.update_all "max_login_attempts = 3, must_change_password = 'true'"
```

### Delete

Asimismo, una vez que se recupera el objeto Active Record también puede ser destruído,
lo cual lo borrará de la base de datos.

```ruby
user = User.find_by(name: 'David')
user.destroy
```

Si desea eliminar varios registros de forma masiva, puede usar el método `destroy_by` 
o `destroy_all`.

```ruby
# encontrar y destruir todos los usuarios llamados David
User.destroy_by(name: 'David')

# destruir todos los usuarios
User.destroy_all
```

Validations
-----------
Validaciones

Active Record le permite validar el estado de un modelo antes de que se escriba
en la base de datos. Existen varios métodos que puede utilizar para verificar su
modelos y validar que un valor de atributo no está vacío, es único y no
ya en la base de datos, sigue un formato específico y muchos más.

La validación es un tema muy importante a considerar cuando se persiste en la base de datos, por lo que
los métodos `save` y` update` lo toman en cuenta cuando
se ejecutan: return `false` cuando una validación falla y no mantuvieron ninguna
operación en la base de datos. Todos estos tienen una contraparte explosiva, bang (que
es, `save!` y `update!`), que son más estrictos en ese sentido 
y arrojan una excepción `ActiveRecord::RecordInvalid` si la validación falla.
Un rápido ejemplo para ilustrar:

```ruby
class User < ApplicationRecord
  validates :name, presence: true
end

user = User.new
user.save  # => false
user.save! # => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```

Puedes aprender más acerca de validaciones en la guía [Active Record Validations
guide](active_record_validations.html).


Callbacks
---------

Las retrollamadas (callbacks) de Active Record le permiten adjuntar código a ciertos eventos en el
ciclo de vida de sus modelos. Esto le permite añadir comportamiento a sus modelos 
de forma transparente en la ejecución cuando estos eventos ocurren, como cuando se crea un nuevo
registro, actualizarlo, destruírlo, etc. Puedes obtener más información sobre las retrollamadas en
la [Active Record Callbacks guide](active_record_callbacks.html).

Migrations
----------

Rails provee un lenguaje de dominio específico para manejar un esquema de base de datos llamado
migraciones (migrations). Las migraciones son ficheros guardados que se ejecutan contra cualquier 
base de datos que Active Record soporte utilizando `rake`. Aquí hay una migración que 
crea una tabla:

```ruby
class CreatePublications < ActiveRecord::Migration[6.0]
  def change
    create_table :publications do |t|
      t.string :title
      t.text :description
      t.references :publication_type
      t.integer :publisher_id
      t.string :publisher_type
      t.boolean :single_issue

      t.timestamps
    end
    add_index :publications, :publication_type_id
  end
end
```

Rails mantiene el historial sobre que fichero fue actualizado en la base de datos y 
provee características para deshacer los cambios. Para realmente crear la tabla, deberías ejecutar 
`bin/rails db:migrate` y para deshacerlo `bin/rails db:rollback`.

Nota que el código de arriba es database-agnostic: se puede ejectuar en MySQL, 
PostgreSQL, Oracle y others. Puedes aprender más acerca de las migraciones en
[Active Record Migrations guide](active_record_migrations.html).
