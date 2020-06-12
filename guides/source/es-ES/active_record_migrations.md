*NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https: //guides.rubyonrails.org.**

Migraciones de Active Record
============================

Las migraciones es el modo de Active Record que le permite cambiar su
esquema de base de datos a través del tiempo. En lugar de escribir modificaciones
 el esquema en SQL puro, las migraciones utilizan un lenguaje 
de Definición de Esquemas (DSL) en Ruby para describir los cambios en sus tablas
en la base de datos.

Después de leer esta guía, sabrá:

* Los generadores que puedes usar para crearlos.
* Los métodos que proporciona Active Record para manipular su base de datos.
* Los comandos de rieles que manipulan las migraciones y su esquema.
* Cómo se relacionan las migraciones con `schema.rb`.

--------------------------------------------------------------------------------

Resumen de las Migraciones
--------------------------

Las migraciones son el modo conveniente de [cambiar el esquema de tu base de datos
 a través del tiempo](https://en.wikipedia.org/wiki/Schema_migration)
de una manera consistente.

Puedes pensar cada migración como una nueva 'versión' de la base de datos. 
Un esquema comienza sin nada, y cada migración lo modifica para agregar o
remover tablas, columnas o registros. Active Record conoce cómo actualizar su
esquema a lo largo del tiempo, llevándolo desde cualquier punto en 
el que se encuentre historia a la última versión. Active Record también actualizará su archivo 
`db / schema.rb`  con la estructura actualizada de su base de datos.

Aquí hay un ejemplo de una migración

```ruby
class CreateProducts < ActiveRecord::Migration[6.0]
  def change
    create_table :products do |t|
      t.string :name
      t.text :description

      t.timestamps
    end
  end
end
```

Esta migración añade una tabla llamada products con una columna de cadena de caractéres llamada
`name` y una columna de texto llamada `description`. Una columna de clave primaria llamada `id`
será también añadida implícitamente, como la clave primaria por defecto para todos los modelos
Active Record. Los macro timestamps añaden dos columnas, `created_at` y `updated_at`. Si existen,
estas columnas especiales son automaticamemente administradas por Active Record.

Nota que nosotros definimos el cambio que queremos que ocurra a través del tiempo.
Antes de ejecutar esta migración, no habrá una tabla. Después de la migración la tabla existirá.
Active Record, tambien sabe como retroceder esta migración: si ejecutamos la migración hacia atrás,
borrará la tabla.

En las bases de datos que admiten transacciones con declaraciones que cambian el esquema,
las migraciones se envuelven en una transacción. Si la base de datos no es compatible con esto,
cuando una migración falla, las partes de ella que tuvieron éxito no se no se revertiran.
Tendrá que revertir los cambios que se hicieron a mano.

Hay ciertas consultas que no pueden ejecutarse en una transacción. Si tu adaptador soporta
transacciones DDL puedes utilizar `disable_ddl_transaction!` para deshabilitarlas una sola migración.

Si tu deseas en una migración hacer algo que Active Record no sabe como revertir, 
puedes utilizar `reversible`:

```ruby
class ChangeProductsPrice < ActiveRecord::Migration[6.0]
  def change
    reversible do |dir|
      change_table :products do |t|
        dir.up   { t.change :price, :string }
        dir.down { t.change :price, :integer }
      end
    end
  end
end
```

Alternativamente, puedes utilizar `up` y `down` en lugar de `change`:

```ruby
class ChangeProductsPrice < ActiveRecord::Migration[6.0]
  def up
    change_table :products do |t|
      t.change :price, :string
    end
  end

  def down
    change_table :products do |t|
      t.change :price, :integer
    end
  end
end
```

Creando una Migración
---------------------

### Creating a Standalone Migration 
Creando una Migración Independiente

Las migraciones son grabadas como ficheros en el directorio `db/migrate`, una por una para 
cada clase migration. El nombre del fichero es de la forma `YYYYMMDDHHMMSS_create_products.rb`,
que es como decir un instante del tiempo UTC para identificar la migración, seguida por un 
guión bajo y el nombre de migración. El nombre de la clase migración (versión CamelCased) 
debería emparejar con la parte posterior del nombre del fichero. Por ejemplo en el fichero 
`20080906120000_create_products.rb` se deberia definir la clase `CreateProducts` y en el fichero
`20080906120001_add_details_to_products.rb` se debería definir `AddDetailsToProducts`.
Rails utiliza esta marca de tiempo para determinar cual migración debería ejecutarse en su 
orden correspondiente, entonces si estás copiando desde otra aplicación o generas el fichero por 
ti mismo, se consciente de su posicione en el orden.

Por supuesto, calcular los instantes no es divertido, entonces Active Record provee 
un generador que hace esto para usted:

```bash
$ bin/rails generate migration AddPartNumberToProducts
```
Esto creará una migración vacía apropiadamente nombrada:

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration[6.0]
  def change
  end
end
```
Este generador puede hacer mucho más que agregar una marca de tiempo al nombre del archivo.
Basado en convenciones de nomenclatura y argumentos adicionales (opcionales) puede también comienca
a desarrollar la migración.

Si el nombre de la migración es de la forma "AddXXXToYYY" o "RemoveXXXFromYYY" 
y es seguido por una lista de nombres de columnas y tipos, será creada una migración conteniendo
las declaraciones apropiadas `add_column` y `remove_column`.

```bash
$ bin/rails generate migration AddPartNumberToProducts part_number:string
```

generará

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration[6.0]
  def change
    add_column :products, :part_number, :string
  end
end
```
Si desea agregar un índice en la nueva columna, puedes hacer eso también:

```bash
$ bin/rails generate migration AddPartNumberToProducts part_number:string:index
```

genera

```ruby
class AddPartNumberToProducts < ActiveRecord::Migration[6.0]
  def change
    add_column :products, :part_number, :string
    add_index :products, :part_number
  end
end
```

Similarmente, puedes generar una migración para eliminar una columna desde la línea de comando:

```bash
$ bin/rails generate migration RemovePartNumberFromProducts part_number:string
```

genera

```ruby
class RemovePartNumberFromProducts < ActiveRecord::Migration[6.0]
  def change
    remove_column :products, :part_number, :string
  end
end
```

No está limitado a una columna generada mágicamente. Por ejemplo:

```bash
$ bin/rails generate migration AddDetailsToProducts part_number:string price:decimal
```

genera

```ruby
class AddDetailsToProducts < ActiveRecord::Migration[6.0]
  def change
    add_column :products, :part_number, :string
    add_column :products, :price, :decimal
  end
end
```

Si el nombre de la migración es de la forma "CreateXXX" y es seguida por una lista 
de nombres de columnas y tipos entonces será generada una migración para crear la 
tabla XXX con las columnas listadas. Por ejemplo:

```bash
$ bin/rails generate migration CreateProducts name:string part_number:string
```

genera

```ruby
class CreateProducts < ActiveRecord::Migration[6.0]
  def change
    create_table :products do |t|
      t.string :name
      t.string :part_number

      t.timestamps
    end
  end
end
```

Como siempre, lo que ha sido recientemente generado es un punto de partida. Puedes añadir
o remover lo que sea adecuado editando el fichero `db/migrate/YYYYMMDDHHMMSS_add_details_to_products.rb`.

También, el generador acepta tipos de columna como `references` (también disponible como
 `belongs_to`). Como ejemplo:

```bash
$ bin/rails generate migration AddUserRefToProducts user:references
```

genera

```ruby
class AddUserRefToProducts < ActiveRecord::Migration[6.0]
  def change
    add_reference :products, :user, foreign_key: true
  end
end
```

Esta migración creará una columna `user_id` y un índice apropiado.
para más opciones `add_reference`, visita a [API documentation](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_reference).

Hay también un generador que produce tablas de intersección so `JoinTable` es parte del nombre: 

```bash
$ bin/rails generate migration CreateJoinTableCustomerProduct customer product
```

producirá la siguiente migración:

```ruby
class CreateJoinTableCustomerProduct < ActiveRecord::Migration[6.0]
  def change
    create_join_table :customers, :products do |t|
      # t.index [:customer_id, :product_id]
      # t.index [:product_id, :customer_id]
    end
  end
end
```
### Model Generators
Generadores del Modelo

Los generadores de modelos y de andamiaje (Scaffold) crearán las migraciones apropiadas para añadir
un nuevo modelo.Estas migraciones ya contendrán las instrucciones para la creación de la correspondiente tablas.
Si le dices a Rails que columnas quieres, entonces las declaraciones para añadirlas también serán creadas.
Por ejemplo, ejecutando:

```bash
$ bin/rails generate model Product name:string description:text
```

creará una migración que se ve de la siguiente manera:

```ruby
class CreateProducts < ActiveRecord::Migration[6.0]
  def change
    create_table :products do |t|
      t.string :name
      t.text :description

      t.timestamps
    end
  end
end
```

Puedes añadir la cantidad de columnas nombre/tipo que quieras.

### Passing Modifiers
Modificadores de Paso

Algunos de uso común [type modifiers](#column-modifiers) puede ser pasados directamente en la 
línea de comandos.Están encerrados por llaves y después por el tipo de campo:

Por ejemplo, ejecutando:

```bash
$ bin/rails generate migration AddDetailsToProducts 'price:decimal{5,2}' supplier:references{polymorphic}
```

producirá una migración que se ve así

```ruby
class AddDetailsToProducts < ActiveRecord::Migration[6.0]
  def change
    add_column :products, :price, :decimal, precision: 5, scale: 2
    add_reference :products, :supplier, polymorphic: true
  end
end
```


CONSEJO: Eche un vistazo a la salida de ayuda de los generadores para obtener más detalles.

Escribiendo una Migración
-------------------------
¡Una vez que haya creado su migración utilizando uno de los generadores, es hora de comenzar a trabajar!

### Creating a Table
Creando una Tabla

El método `create_table` es uno de los fundamentales, pero la mayoría del tiempo,
sera genererado usando el generador del modelo o scaffold. Un típico uso podría ser

```ruby
create_table :products do |t|
  t.string :name
end
```

el cual crea una tabla `products` con una columna llamada `name` (y como se discutirá a continuación,
una columna implícita `id`).

Por defecto, `create_table` creará una clave primaria llamada `id`. Puedes cambiar
el nombre de la clave primaria con la opción `:primary_key` (no te olvides de actualizar
el modelo correspondiente) o, si no quieres ninguna clave primaria, puedes escribir la opción 
`id: false`. Si necesitas pasarle a la base de datos opciones específicas puedes escribir un 
fragmento de SQĹ en la opción `:options`. Por ejemplo:

```ruby
create_table :products, options: "ENGINE=BLACKHOLE" do |t|
  t.string :name, null: false
end
```

Esto añadirá `ENGINE=BLACKHOLE` con la declaración SQL utilizada para crear una tabla.

También se puede pasar la opción `:comment` con cualquier descripción para la tabla
que se almacenará en la base de datos y se puede ver con las herramientas de administración
de la base de datos, como MySQL Workbench o PgAdmin III. Es muy recomendable especificar comentarios 
en migraciones para aplicaciones con grandes bases de datos, ya que ayuda a las personas
para comprender el modelo de datos y generar la documentación.
Actualmente, solo los adaptadores MySQL y PostgreSQL admiten comentarios.

### Creating a Join Table
Crear Una Tabla De Intersección

El método de migración `create_join_table` crea una tabla de intersección HABTM (has and belongs to many).
Un uso común sera:

```ruby
create_join_table :products, :categories
```

la cual creará una tabla `categories_products` con dos columnas llamadas `category_id` y `product_id`.
Esas columnas tienen la opción `:null` configurada a `false` por defecto. Esto será sobrescrito especificando 
la opción `:column_options`.

```ruby
create_join_table :products, :categories, column_options: { null: true }
```

Por defecto, el nombre de la tabla de intersección proviene de la unión de los dos primeros
argumentos proporcionados a `create_join_table`, en orden alfabético.
Para personalizar el nombre de la tabla, proporcione a `:table_name`:

```ruby
create_join_table :products, :categories, table_name: :categorization
```
crea la tabla `categorization`

`create_join_table` también acepta un bloque, el cual puedes utilizar para añadir índices
(que no son creados por defecto) o columnas adicionales:

```ruby
create_join_table :products, :categories do |t|
  t.index :product_id
  t.index :category_id
end
```

### Changing Tables
Cambiando Las Tablas

Un primo cercano de `create_table` es` change_table`, utilizada para modificar tablas existentes.
Se usa de manera similar a `create_table` pero el objeto cedido sabe más trucos. Por ejemplo:

```ruby
change_table :products do |t|
  t.remove :description, :name
  t.string :part_number
  t.index :part_number
  t.rename :upccode, :upc_code
end
```


elimina las columnas `description` y` name`, crea una columna de cuerda `part_number`y 
agrega un índice. Finalmente renombra la columna `upcode`.

### Changing Columns
Modificadores De Columnas

Como el `remove_column` y `add_column` Rails provee el método de migración `change_column`.

```ruby
change_column :products, :part_number, :text
```

Esto cambia la columna `part_number` en la tabla de productos para que sea un campo`:text`.
Nota que el comando `change_column` es irreversible.

Además de `change_column`, los métodos `change_column_null` y `change_column_default`
se utilizan específicamente para cambiar una restricción no nula y predeterminada
valores de una columna.

```ruby
change_column_null :products, :name, false
change_column_default :products, :approved, from: true, to: false
```

Esto establece el campo `:name` en products en una columna` NOT NULL` y el valor predeterminado
 del campo `: approved` de verdadero a falso.

NOTA: También puede escribir la migración anterior `change_column_default` como
`change_column_default: products,: approved, false`, pero a diferencia del anterior
ejemplo, esto haría que tu migración sea irreversible.

### Column Modifiers
Modificadores De Columna

Los modificadores de columna se pueden aplicar al crear o cambiar una columna:

* `limit`        Establece el tamaño máximo del campo `string/text/binary/integer`.
* `precision`    Define la precisión para el campo`decimal`, representando el 
número total de los dígitos.
* `scale`        Define la escala para el campo `decimal`, representando el 
número total de los dígitos después del punto decimal.
* `polymorphic`  Agrega una columna `type`para asociaciones `belongs_to`.
* `null`         Permite o deshabilita los valores `NULL` en la columnalumn.
* `default`      Permite establecer un valor por defecto de la columna. Nota que si
 utilizas un valor dinámico (como una fecha), el valor por defecto solo será
 calculado la primera vez (ej: en la fecha y hora que la migración es aplicada).
* `comment`      Agrega un comentario para la columna.

Algunos adaptadores pueden admitir opciones adicionales; ver los documentos de API específicos del adaptador
para más información.

NOTA: `null` y` default` no se pueden especificar a través de la línea de comando.

### Foreign Keys
Llaves Foráneas

Mientras no es obligatorio, es posible que desee agregar restricciones de clave externa a
[guarantee referential integrity](#active-record-and-referential-integrity).

```ruby
add_foreign_key :articles, :authors
```

Esto añade una nueva clave foránea a la columna `author_id` en la tabla `articles`. 
La clave hace referencia a la columna `id` de la tabla `authors`. Si los nombres 
de las columnas no pueden ser derivados de los nombres de las tablas, puedes utilizar
las opciones `:column` y `:primary_key`.
 
 Rails generará un nombre para cada clave foránea empezando por `fk_rails_ `seguido por 10 caracteres 
 que son generados determinísticamente de la tabla `from_table` y `column`. Hay una opción `:name` para 
 especificar si un nombre diferente es necesario.

Active Record solo soporta una columna simple como clave foránea. `execute` and
`structure.sql` son requeridas para utilizar claves foráneas compuestas. Ver
[Schema Dumping and You](#schema-dumping-and-you).

Las claves foráneas también se pueden eliminar:

```ruby
# deje que Active Record descubra el nombre de la columna
remove_foreign_key :accounts, :branches

# eliminar clave foránea para una columna específica
remove_foreign_key :accounts, column: :owner_id

# eliminar clave externa por nombre
remove_foreign_key :accounts, name: :special_fk_name
```

### When Helpers aren't Enough
Cuando Los Ayudantes (Helpers) No Son Suficientes

Si los helpers provistos por Active Record no son suficientes puedes utilizar el método `execute`
para ejecutar cualquier SQL:

```ruby
Product.connection.execute("UPDATE products SET price = 'free' WHERE 1=1")
```

Para más detalles y ejemplos de métodos individuales, leer la documentación API.
En particular la documentación para
[`ActiveRecord::ConnectionAdapters::SchemaStatements`](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html)
(cual provee los métodos disponibles `change`, `up` y `down`),
[`ActiveRecord::ConnectionAdapters::TableDefinition`](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/TableDefinition.html)
(cual provee los métodos disponibles en el objecto cedido por `create_table`)
y
[`ActiveRecord::ConnectionAdapters::Table`](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/Table.html)
(cual provee los métodos disponibles en el objecto cedido por `change_table`).

### Using the `change` Method
Utilizando el Método `change`

El método `change` es la principal manera de escribir migraciones. Este funciona para 
la mayoría de los casos, donde Active Record conoce como revertir la migración automáticamente.
Actualmente, el método `change` solamente soporta estas definiciones de migración:

* add_column
* add_foreign_key
* add_index
* add_reference
* add_timestamps
* change_column_default (must supply a :from and :to option)
* change_column_null
* create_join_table
* create_table
* disable_extension
* drop_join_table
* drop_table (must supply a block)
* enable_extension
* remove_column (must supply a type)
* remove_foreign_key (must supply a second table)
* remove_index
* remove_reference
* remove_timestamps
* rename_column
* rename_index
* rename_table

`change_table` es también reversible, siempre y cuando el bloque no llame a `change`,
`change_default` o `remove`.


`remove_column` es reversible si proporciona el tipo de la columna como el tercer
argumento. También proporcione  las opciones de columna originales, de lo contrario Rails no puede
recrear la columna exactamente al retroceder (rolling back):

```ruby
remove_column :posts, :slug, :string, null: false, default: ''
```

Si va a necesitar usar cualquier otro método, debe usar `reversible`
o escriba los métodos `up` y `down` en lugar de usar el método `change`.

### Using `reversible`
Usando `reversible`

Las migraciones complejas pueden requerir un procesamiento que Active Record no sabe cómo
retroceder. Puede usar `reversible` para especificar qué hacer cuando se ejecuta una
migración y qué más hacer al revertirla. Por ejemplo:

```ruby
class ExampleMigration < ActiveRecord::Migration[6.0]
  def change
    create_table :distributors do |t|
      t.string :zipcode
    end

    reversible do |dir|
      dir.up do
        # add a CHECK constraint
        execute <<-SQL
          ALTER TABLE distributors
            ADD CONSTRAINT zipchk
              CHECK (char_length(zipcode) = 5) NO INHERIT;
        SQL
      end
      dir.down do
        execute <<-SQL
          ALTER TABLE distributors
            DROP CONSTRAINT zipchk
        SQL
      end
    end

    add_column :users, :home_page_url, :string
    rename_column :users, :email, :email_address
  end
end
```


El uso de `reversible` asegurará que las instrucciones seran ejecutadas en el
orden correcto también. Si se revierte la migración de ejemplo anterior,
el bloque `down` se ejecutará después de que se elimine la columna` home_page_url` y
justo antes de que se borre la tabla `distributors`.

A veces su migración hará algo que es simplemente irreversible; por
ejemplo, podría destruir algunos datos. En tales casos, puedes lanzar una excepción 
`ActiveRecord::IrreversibleMigration` en el bloque `down`. Si alguien intenta revertir tu migración, 
se mostrará un mensaje de error que dice que no se puede hacer.

### Using the `up`/`down` Methods
Usando Los Métodos`up`/`down`

También puedes utilizar el estilo viejo de migraciones usando `up` y `down` en
lugar del método `change`.
El método `u`p debería describir la transformación que deseas hacer a tu esquema, 
y el método down de tu migración debería revertir las transformaciones hechas por
el método `up`. En otras palabras, el esquema de la base de datos permanecerá sin cambios
 si ejecutas un `up` seguido por un `down`. 

 Por ejemplo, si creas una tabla en el método `up`, deberías borrarla en el método `down`.
 Es prudente que las transformaciones precisamente esten en el orden reverso en el 
 que fueron hechas en el método `up`. El ejemplo en la sección `reversible` es equivalente a:

```ruby
class ExampleMigration < ActiveRecord::Migration[6.0]
  def up
    create_table :distributors do |t|
      t.string :zipcode
    end

    # add a CHECK constraint
    execute <<-SQL
      ALTER TABLE distributors
        ADD CONSTRAINT zipchk
        CHECK (char_length(zipcode) = 5);
    SQL

    add_column :users, :home_page_url, :string
    rename_column :users, :email, :email_address
  end

  def down
    rename_column :users, :email_address, :email
    remove_column :users, :home_page_url

    execute <<-SQL
      ALTER TABLE distributors
        DROP CONSTRAINT zipchk
    SQL

    drop_table :distributors
  end
end
```
Si tu migración es irreversible, deberías lanzar una `ActiveRecord::IrreversibleMigration` 
desde tu método `down`. Si alguién trata de revertir la migración, un mensaje de error 
será mostrado diciendo que esto no se puede hacer.

### Reverting Previous Migrations
Revertir Migraciones Anteriores

Puede utilizar la capacidad de Active Record para revertir las migraciones utilizando el método `revert`:

```ruby
require_relative "20121212123456_example_migration"

class FixupExampleMigration < ActiveRecord::Migration[6.0]
  def change
    revert ExampleMigration

    create_table(:apples) do |t|
      t.string :variety
    end
  end
end
```

El método `revert` también acepta un bloque de instrucciones para revertir.
Esto podría ser útil para revertir partes seleccionadas de migraciones anteriores.
Por ejemplo, imaginemos que `ExampleMigration` está comprometido y más tarde se decide
que sería mejor usar validaciones de Active Record,
en lugar de la restricción `CHECK`, para verificar el código postal.

```ruby
class DontUseConstraintForZipcodeValidationMigration < ActiveRecord::Migration[6.0]
  def change
    revert do
      # copy-pasted code from ExampleMigration
      reversible do |dir|
        dir.up do
          # add a CHECK constraint
          execute <<-SQL
            ALTER TABLE distributors
              ADD CONSTRAINT zipchk
                CHECK (char_length(zipcode) = 5);
          SQL
        end
        dir.down do
          execute <<-SQL
            ALTER TABLE distributors
              DROP CONSTRAINT zipchk
          SQL
        end
      end

      # El resto de la migración estuvo bien
    end
  end
end
```

La misma migración también podría haberse escrito sin usar `revert`
pero esto habría involucrado más pasos: invertir el orden
de `create_table` y` reversible`, reemplazando `create_table`
por `drop_table`, y finalmente reemplazando` up` por `down` y viceversa.
Todo esto se soluciona mediante `revert`.


NOTA: Si desea agregar restricciones de verificación como en los ejemplos anteriores,
Tendrá que usar `structure.sql` como método de volcado. Ver 
[Schema Dumping and You](#schema-dumping-and-you).


Ejecutando migraciones
--------------------

Rails proporciona un conjunto de comandos de rails para ejecutar ciertos conjuntos de migraciones.

La primera tarea de migración que utilizarás será probablemente `rake db:migrate`. En su forma 
más básica ejecuta un método `change` o `up` para todas las migraciones que aún no se han ejecutado.
Si no hay tales migraciones, finaliza. Ejecutará esas migraciones en orden basado 
en la fecha de creación de cada migración.

Nota que ejecutar la tarea `db:migrate` también se invoca la tarea `db:schema:dump`, el cual 
actualizará tu fichero `db/schema.rb` para que coincida a la estructura de tu base de datos.

Si especificas una versión de destino, Active Record ejecturará las migraciones requeridas 
(change, up, down) hasta alcanzar la versión específica. La versión es el prefijo numérico 
en el nombre de fichero de la migración. Por ejemplo, para migrar a la versión 20080906120000 ejectuta:

```bash
$ bin/rails db:migrate VERSION=20080906120000
```

Si la versión 20080906120000 es mayor que la versión actual (un ejemplo, se está migrando hacia arriba)
, se ejectutará el método `change` o `up`) en todas las migraciones hacia arriba e incluirá la 20080906120000,
y no ejecutará ninguna migración posterior. Si se está migrando hacia abajo, esto ejecutará los metodos `down` 
ejectutará todas las migraciones hacia abajo, paro no incluirá la 20080906120000.

### Rolling Back
Deshaciendo Migraciones

Una tarea común es revertir la última migración. Por ejemplo, si hiciste un
comete un error y desea corregirlo. En lugar de rastrear la versión
número asociado con la migración anterior que puede ejecutar:

```bash
$ bin/rails db:rollback
```
Esto revertirá la última migración, ya sea revocando el método  `change`
o ejecutando el método 'down'. Si necesitas deshacer
varias migraciones puede proporcionar un parámetro `STEP`:

```bash
$ bin/rails db:rollback STEP=3
```

revertirá las últimas 3 migraciones.

El comando `db:migrate: redo` es un atajo para revertir luna migración y luego migrar
de nuevo. Al igual que con el comando `db: rollback`, puede usar el parámetro` STEP`
si necesita regresar más de una versión, por ejemplo:


```bash
$ bin/rails db:migrate:redo STEP=3
```

Ninguno de estos comandos de rails hace nada que no pueda hacer con `db: migrate`. Ellos
están ahí para su conveniencia, ya que no necesita especificar explícitamente la
versión para migrar a.

### Setup the Database
Configurar la Base de Datos

El comando `bin/rails db: setup` creará la base de datos, cargará el esquema e inicializará
con los datos iniciales.

### Resetting the Database
Recomponer la Base de Datos

El comando `bin/rails db:reset` borrará la base de datos y la configurará denuevo. Esto es
funcionalmente equivalente a `bin/rails db:drop db:setup`.

NOTA: Esto no es lo mismo que ejecutar todas las migraciones.  Esto utilizará únicamente el fichero 
`db/schema.rb` o `db/structure.sql`actual. Si una migración no puede ser revertida,
`bin/rails db:reset` no te podrá ayudar. Para descubrir más acerca del volcado del esquema vee 
[Schema Dumping and You](#schema-dumping-and-you)

### Running Specific Migrations
Ejecutando Migraciones en Diferentes Entornos

Si necesitas ejecutar una migración específica hacia arriba o abajo, las tareas `db:migrate:up` y
`db:migrate:down` harán esto. Sólo especifica la versión adecuada y la migración correspondiente 
tendrá su método `change`, `up` o `down` invocado, por ejemplo:

```bash
$ bin/rails db:migrate:up VERSION=20080906120000
```

ejecutara la migración 20080906120000 por ejecución del método `change` (o del método `up`).
Esta tarea primero comprobará si la migración está ya realizada y no hará nada si Active Record 
cree que esta ya se ha ejecutado antes.

### Running Migrations in Different Environments
 Ejecutando Migraciones en Diferentes Entornos

Por defecto cuando ejecutamos `bin/rails db:migrate` se ejecutará en el
entorno de `development`. Si quieres ejecutar las migraciones otra vez en otro entorno, 
puedes especificarlo utilizando la variable de entorno `RAILS_ENV` cuando ejecutas el comando. 
Por ejemplo para ejecutar las migraciones nuevamente en el entorno de pruebas, `test`, podrías ejecutar:

```bash
$ bin/rails db:migrate RAILS_ENV=test
```

### Changing the Output of Running Migrations
Cambiando la Salida de la Ejecución de Migraciones

Por defecto, las migraciones le dicen exactamente lo que están haciendo y cuánto tiempo les llevó.
Una migración que crea una tabla y agrega un índice puede producir resultados como este

```bash
==  CreateProducts: migrating =================================================
-- create_table(:products)
   -> 0.0028s
==  CreateProducts: migrated (0.0028s) ========================================
```

Varios métodos son provistos en las migraciones que te permiten controlar todo esto:

| Metodo             | Propósito 
| ------------------ | --------- 
| suppress_messages  | Toma un bloque como argumento y suprime cualquier salida generada por el bloque. 
| say                | Toma como argumento un mensaje y emite tal cual es. Se le puede pasar un segundo argumento booleano para especificar si se va a tabular o no.
| say_with_time      | Salida de texto junto con el tiempo que tomó para ejecutar el bloque correspondiente. Si el bloque devuelve un entero que asume que es el número de filas afectadas.

Por ejemplo, esta migración:

```ruby
class CreateProducts < ActiveRecord::Migration[6.0]
  def change
    suppress_messages do
      create_table :products do |t|
        t.string :name
        t.text :description
        t.timestamps
      end
    end

    say "Created a table"

    suppress_messages {add_index :products, :name}
    say "and an index!", true

    say_with_time 'Waiting for a while' do
      sleep 10
      250
    end
  end
end
```

genera la siguiente salida

```bash
==  CreateProducts: migrating =================================================
-- Created a table
   -> and an index!
-- Waiting for a while
   -> 10.0013s
   -> 250 rows
==  CreateProducts: migrated (10.0054s) =======================================
```

Si quieres que Active Record no muestre ninguna salida, ejecutando `rake db:migrate VERBOSE=false` 
se suprimirán todas las salidas.

Cambiando las Migraciones Existentes
------------------------------------

Ocasionalmente podemos cometer un error cuando escribimos una migración. Si has ejecutado ya
 la migración entonces no puedes editarla y ejecutarla otra vez: Rails pensará que ya se ha 
 ejecutado la migración y no hará nada cuando ejecutes `bin/rails db:migrate`. Tendras que
 revertir la migración  (por ejemplo con `bin/rails db:rollback`) edita la migración y luego escribe
 `bin/rails db:migrate` para ejecutar la versión corregida.

En general, editar una migración existente no es una buena idea. Crearás trabajo extra para ti mismo
 y tus colaboradores y causará mayores dolores de cabeza si la versión existente de una migración
 ha sido ya ejecutada en los servidores de producción. En su lugar, debe escribir una nueva migración
 que realice los cambios. Necesitas editaruna migración recién que aún no ha sido
comprometido con el control de la fuente (o, más generalmente, que no se ha propagado
ás allá de su máquina de desarrollo) es relativamente inofensivo.

El método `revert` puede ser útil al escribir una nueva migración para deshacer
migraciones anteriores en su totalidad o en parte
(ver [Reverting Previous Migrations](#reverting-previous-migrations) encima).

Volcando el Esquema y Tú
------------------------

### What are Schema Files for?
¿Para qué son los Ficheros de Esquema?

Las migraciones, con lo poderosas que pueden ser, no son la fuente fiel de tu esquema 
de base de datos. Su base de datos sigue siendo la fuente de autorización. Por defecto,
Rails genera `db / schema.rb` que intenta capturar el estado actual de base de datos.

Tiende a ser más rápido y menos propenso a errores para crear una nueva instancia de su
base de datos de la aplicación cargando el archivo de esquema a través de `bin/rails db:schema:load`
De lo que es reproducir todo el historial de migración.
[Old migrations](#old-migrations) podrian no aplicarse correctamente si las las migraciones
usan dependencias externas cambiantes o confiar en el código de la aplicación que
evoluciona por separado de sus migraciones.

Los archivos de esquema también son útiles si desea ver rápidamente qué atributos
Active Record tiene. Esta información no está en el código del modelo, eso
con frecuencia se extiende a través de varias migraciones, pero la información es 
resumido en el archivo de esquema.

### Types of Schema Dumps
Tipos de Volcado del Esquema

El formato del volcado de esquema generado por Rails está controlado por
`config.active_record.schema_format` en `config/application.rb`. Por
predeterminado, el formato es `:ruby`, pero también podria ser `:sql`.

```ruby
ActiveRecord::Schema.define(version: 2008_09_06_171750) do
  create_table "authors", force: true do |t|
    t.string   "name"
    t.datetime "created_at"
    t.datetime "updated_at"
  end

  create_table "products", force: true do |t|
    t.string   "name"
    t.text     "description"
    t.datetime "created_at"
    t.datetime "updated_at"
    t.string   "part_number"
  end
end
```

En muchos sentidos, esto es exactamente lo que es. Este archivo se crea inspeccionando la
base de datos y expresando su estructura usando `create_table`, `add_index`, etc. 

`db/ chema.rb` no puede expresar todo lo que su base de datos puede soportar, como
disparadores, secuencias, procedimientos almacenados, restricciones de verificación 
(triggers, sequences, stored procedures, check constraints), etc. Mientras 
migraciones pueden usar `execute` para crear construcciones de bases de datos que no son compatibles
 con DSL de migración de Ruby, estas construcciones pueden no ser reconstituidas por el
 esquema dumper. Si está utilizando características como estas, debe establecer el esquema
 de formatee `:sql` para obtener un archivo de esquema preciso que sea útil para
crear nuevas instancias de bases de datos.

Cuando el formato del esquema se establece en `:sql`, la estructura de la base de datos será volcada
usando una herramienta específica de la base de datos en `db/structure.sql`. Por ejemplo, para
PostgreSQL, se utiliza la utilidad `pg_dump`. Para MySQL y MariaDB, este archivo
contiene el resultado de `SHOW CREATE TABLE` para las varias tablas.

Para cargar el esquema desde `db/structure.sql`, ejecute` bin/rails db:structure:load`.
La carga de este archivo se realiza ejecutando las declaraciones SQL que contiene. Por
definición, esto creará una copia perfecta de la estructura de la base de datos.

### Schema Dumps and Source Control
Volcados del Esquema y Control de la Fuente

Debido a que los archivos de esquema se usan comúnmente para crear nuevas bases de datos,
es muy recomendable que verifique su archivo de esquema con la fuente de control.

Los conflictos de fusión pueden ocurrir en su archivo de esquema cuando dos ramas modifican el esquema.
Para resolver estos conflictos, ejecute `bin/rails db: migrate` para regenerar el archivo de esquema.

Active Record and Referential Integrity
---------------------------------------
Active Record y la Integridad Referencial

La metodología de Active Record dice que la inteligencia pertenece al modelo, no a la base de datos. 
Así bien, características tales como disparadores o restricciones, las cuales otorgan algo de 
inteligencia a la base de datos, no son fuertemente utilizadas.

Las validaciones tal como `validates :foreign_key, uniqueness: true` son un modo por el cual 
los modelos pueden forzar la integridad de los datos. La opción `:dependent` en las asociaciones 
permite a los modelos destruir los objetos que son hijos cuando el padre es destruido. 
Como cualquier cosa que opere a nivel de aplicación, eso no garantiza integridad referencial, 
entonces algunas personas se aseguran esta con restricciones sobre las claves foráneas 
[foreign key constraints](#foreign-keys)  en 
la base de datos.

Sin embargo Active Record no provee todas las herramientas para trabajar directamente con algunas características,
 el método `execute` puede ser utilizado para ejecutar un SQL arbitrario.

Migrations and Seed Data
------------------------
Migraciones y Semillas de Datos 

El objetivo principal de la función de migración de Rails es emitir comandos que modifiquen
esquema utilizando un proceso consistente. Las migraciones también se pueden usar
para agregar o modificar datos. Esto es útil en una base de datos existente que no se puede destruir.
y recreado, como una base de datos de producción.

```ruby
class AddInitialProducts < ActiveRecord::Migration[6.0]
  def up
    5.times do |i|
      Product.create(name: "Product ##{i}", description: "A product.")
    end
  end

  def down
    Product.delete_all
  end
end
```

Para agregar datos iniciales después de crear una base de datos, Rails tiene incorporado
característica `seeds` que acelera el proceso. Esto es especialmente
útil cuando se recarga la base de datos con frecuencia en entornos de desarrollo y prueba.
Para comenzar con esta función, llene `db/seeds.rb` con algunos
código en Ruby, y ejecute `bin/rails db: seed`:

```ruby
5.times do |i|
  Product.create(name: "Product ##{i}", description: "A product.")
end
```
Esta es generalmente una forma mucho más limpia de configurar la base de datos de un espacio en blanco
solicitud.

Old Migrations
--------------
Viejas migraciones

El `db/schema.rb` o` db/structure.sql` es una instantánea del estado actual de su
base de datos y es la fuente autorizada para reconstruir esa base de datos. Esta
hace posible eliminar viejos archivos de migración.

Cuando elimina archivos de migración en el directorio `db/migrate/`, cualquier entorno
endonde se ejecutaron esos archivos `bin/rails db:migrate`  esos archivos contendrán una referencia
en la marca de tiempo de migración específicamente para ellos dentro de una base de datos interna de Rails
en la tabla llamada `schema_migrations`. Esta tabla se utiliza para realizar un seguimiento si
las migraciones se han ejecutado en un entorno específico.

Si ejecuta el comando `bin/rails db: migrate: status`, que muestra el estado
(up o down) de cada migración, debería ver el texto `********** NO FILE **********`
junto de cualquier archivo de migración eliminado que alguna vez se ejecutó en un
entorno específico pero ya no se puede encontrar en el directorio `db/migrate/`.






