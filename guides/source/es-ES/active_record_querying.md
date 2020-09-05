**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Interfaz de Consulta de Active Record
=====================================

Este guía cubre diferentes formas de recuperar datos de la base de datos utilizando Active Record.

Después de leer esta guía, sabrás:

* Cómo encontrar registros utilizando una variedad de métodos y condiciones.
* Cómo especificar el orden, los atributos recuperados, la agrupación y otras propiedades de los registros encontrados.
* Cómo utilizar la carga ansiosa (eager loading) para reducir la cantidad de consultas a la base de datos necesarias para la recuperación de datos.
* Cómo usar los métodos de buscador dinámico.
* Cómo usar el método de encadenamiento para usar múltiples métodos de Active Record juntos.
* Cómo verificar la existencia de registros particulares.
* Cómo realizar varios cálculos en modelos de Active Record.
* Cómo ejecutar EXPLAIN en relaciones.

--------------------------------------------------------------------------------

What is the Active Record Query Interface?
------------------------------------------
¿Qué es la interfaz de consulta de Active Record?

Si está acostumbrado a usar SQL sin procesar para buscar registros de bases de datos, generalmente encontrarás que hay mejores formas de llevar a cabo las mismas operaciones en Rails. Active Record lo aísla de la necesidad de usar SQL en la mayoría de los casos.

Active Record realizará consultas en la base de datos por usted y es compatible con la mayoría de los sistemas de bases de datos, incluidos MySQL, MariaDB, PostgreSQL y SQLite. Independientemente del sistema de base de datos que esté utilizando, el formato del método Active Record siempre será el mismo.

Los ejemplos de código a lo largo de esta guía se referirán a uno o más de los siguientes modelos:

SUGERENCIA: Todos los siguientes modelos usan `id` como su clave principal, a menos que se especifique lo contrario.

```ruby
class Client < ApplicationRecord
  has_one :address
  has_many :orders
  has_and_belongs_to_many :roles
end
```

```ruby
class Address < ApplicationRecord
  belongs_to :client
end
```

```ruby
class Order < ApplicationRecord
  belongs_to :client, counter_cache: true
end
```

```ruby
class Role < ApplicationRecord
  has_and_belongs_to_many :clients
end
```

Retrieving Objects from the Database
------------------------------------
Recuperando objetos de la base de datos

Para recuperar objetos de la base de datos, Active Record proporciona varios métodos de búsqueda. Cada método de búsqueda le permite pasar argumentos para realizar ciertas consultas en su base de datos sin escribir SQL sin formato.

Los métodos son:

* `annotate`
* `find`
* `create_with`
* `distinct`
* `eager_load`
* `extending`
* `extract_associated`
* `from`
* `group`
* `having`
* `includes`
* `joins`
* `left_outer_joins`
* `limit`
* `lock`
* `none`
* `offset`
* `optimizer_hints`
* `order`
* `preload`
* `readonly`
* `references`
* `reorder`
* `reselect`
* `reverse_order`
* `select`
* `where`

Los métodos de búsqueda que devuelven una colección, como `where` y `group`, devuelven una instancia de `ActiveRecord::Relation`. Los métodos que encuentran una sola entidad, como `find` y `first`, devuelven una sola instancia del modelo.

La operación principal de `Model.find(options)` se puede resumir como:

* Convierta las opciones proporcionadas a una consulta equivalente en SQL.
* Active la consulta SQL y recupere los resultados correspondientes de la base de datos.
* Instanciar el objeto Ruby equivalente del modelo apropiado para cada fila en los resultados.
* Ejecute `after_find` y luego `after_initialize` callbacks, si corresponde.

### Retrieving a Single Object
Recuperando un Solo Objeto

Active Record proporciona varias formas diferentes de recuperar un solo objeto.

#### `find`

Usando el método `find`, puede recuperar el objeto correspondiente a la _primary key_ especificada que coincide con las opciones proporcionadas. Por ejemplo:

```ruby
# Find the client with primary key (id) 10.
client = Client.find(10)
# => #<Client id: 10, first_name: "Ryan">
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients WHERE (clients.id = 10) LIMIT 1
```

El método `find` generará una excepción `ActiveRecord::RecordNotFound` si no se encuentra un registro coincidente.

```ruby
# Find the clients with primary keys 1 and 10.
clients = Client.find([1, 10]) # Or even Client.find(1, 10)
# => [#<Client id: 1, first_name: "Lifo">, #<Client id: 10, first_name: "Ryan">]
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients WHERE (clients.id IN (1,10))
```

ADVERTENCIA: El método `find` generará una excepción `ActiveRecord::RecordNotFound` a menos que se encuentre un registro coincidente para *all** (todas) de las claves principales proporcionadas.

#### `take`

El método `take` recupera un registro sin ningún orden implícito. Por ejemplo:

```ruby
client = Client.take
# => #<Client id: 1, first_name: "Lifo">
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients LIMIT 1
```

El método `take` devuelve `nil` si no se encuentra ningún registro y no se generará ninguna excepción.

Puede pasar un argumento numérico al método `take` para obtener ese número de resultados. Por ejemplo

```ruby
clients = Client.take(2)
# => [
#   #<Client id: 1, first_name: "Lifo">,
#   #<Client id: 220, first_name: "Sara">
# ]
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients LIMIT 2
```

El método `take!` se comporta exactamente como `take`, excepto que generará `ActiveRecord::RecordNotFound` si no se encuentra un registro coincidente.

SUGERENCIA: El registro recuperado puede variar según el motor de la base de datos.

#### `first`

El método `first` encuentra el primer registro ordenado por clave primaria (predeterminado). Por ejemplo:

```ruby
client = Client.first
# => #<Client id: 1, first_name: "Lifo">
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients ORDER BY clients.id ASC LIMIT 1
```

El método `first` devuelve `nil` si no se encuentra un registro coincidente y no se generará ninguna excepción.

Si su [alcance predeterminado](active_record_querying.html#applying-a-default-scope) contiene un método de pedido, `first` devolverá el primer registro de acuerdo con este pedido.

Puede pasar un argumento numérico al método `first` para obtener ese número de resultados. Por ejemplo

```ruby
clients = Client.first(3)
# => [
#   #<Client id: 1, first_name: "Lifo">,
#   #<Client id: 2, first_name: "Fifo">,
#   #<Client id: 3, first_name: "Filo">
# ]
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients ORDER BY clients.id ASC LIMIT 3
```

En una colección que se ordena usando `order`, `first` devolverá el primer registro ordenado por el atributo especificado en `order`.

```ruby
client = Client.order(:first_name).first
# => #<Client id: 2, first_name: "Fifo">
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients ORDER BY clients.first_name ASC LIMIT 1
```

El método `first!` se comporta exactamente como `first`, excepto que generará `ActiveRecord::RecordNotFound` si no se encuentra un registro coincidente.

#### `last`

El método `last` encuentra el último registro ordenado por clave primaria (predeterminado). Por ejemplo:

```ruby
client = Client.last
# => #<Client id: 221, first_name: "Russel">
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients ORDER BY clients.id DESC LIMIT 1
```

El método `last` devuelve `nil` si no se encuentra un registro coincidente y no se generará ninguna excepción.

Si su [default scope](active_record_querying.html#applying-a-default-scope) contiene un método de pedido, `last` devolverá el último registro de acuerdo con este pedido.

You can pass in a numerical argument to the `last` method to return up to that number of results. For example

```ruby
clients = Client.last(3)
# => [
#   #<Client id: 219, first_name: "James">,
#   #<Client id: 220, first_name: "Sara">,
#   #<Client id: 221, first_name: "Russel">
# ]
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients ORDER BY clients.id DESC LIMIT 3
```

En una colección que se ordena usando `order`, `last` devolverá el último registro ordenado por el atributo especificado para `order`.

```ruby
client = Client.order(:first_name).last
# => #<Client id: 220, first_name: "Sara">
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients ORDER BY clients.first_name DESC LIMIT 1
```

El método `last!` Se comporta exactamente como `last`, excepto que generará `ActiveRecord::RecordNotFound` si no se encuentra un registro coincidente.

#### `find_by`

El método `find_by` encuentra el primer registro que coincide con algunas condiciones. Por ejemplo:

```ruby
Client.find_by first_name: 'Lifo'
# => #<Client id: 1, first_name: "Lifo">

Client.find_by first_name: 'Jon'
# => nil
```

Es equivalente a escribir:

```ruby
Client.where(first_name: 'Lifo').take
```

El SQL equivalente de lo anterior es:

```sql
SELECT * FROM clients WHERE (clients.first_name = 'Lifo') LIMIT 1
```

El método `find_by!` Se comporta exactamente igual que `find_by`, excepto que generará `ActiveRecord::RecordNotFound` si no se encuentra un registro coincidente. Por ejemplo:

```ruby
Client.find_by! first_name: 'does not exist'
# => ActiveRecord::RecordNotFound
```

Es equivalente a escribir:

```ruby
Client.where(first_name: 'does not exist').take!
```

### Retrieving Multiple Objects in Batches
Recuperando múltiples objetos en lotes

A menudo necesitamos iterar sobre un gran conjunto de registros, como cuando enviamos un boletín a un gran conjunto de usuarios, o cuando exportamos datos.

Esto puede parecer sencillo:

```ruby
# This may consume too much memory if the table is big.
User.all.each do |user|
  NewsMailer.weekly(user).deliver_now
end
```

Pero este enfoque se vuelve cada vez menos práctico a medida que aumenta el tamaño de la tabla, ya que `User.all.each` instruye a Active Record a buscar _the entire table_  (la tabla completa) en una sola pasada, construir un objeto modelo por fila y luego mantener toda la matriz de objetos modelo en memoria. De hecho, si tenemos una gran cantidad de registros, la colección completa puede exceder la cantidad de memoria disponible.

Rails proporciona dos métodos que abordan este problema dividiendo los registros en lotes amigables para el procesamiento. El primer método, `find_each`, recupera un lote de registros y luego entrega _each_ cada uno de los registros al bloque individualmente como modelo. El segundo método, `find_in_batches`, recupera un lote de registros y luego entrega _the entire batch_ al bloque como una matriz de modelos.

SUGERENCIA: Los métodos `find_each` y `find_in_batches` están diseñados para usarse en el procesamiento por lotes de una gran cantidad de registros que no caben en la memoria de una vez. Si solo necesita recorrer más de mil registros, los métodos de búsqueda regulares son la opción preferida.

#### `find_each`

El método `find_each` recupera registros en lotes y luego arroja _each_ (cada uno) al bloque. En el siguiente ejemplo, `find_each` recupera usuarios en lotes de 1000 y los entrega al bloque uno por uno:


```ruby
User.find_each do |user|
  NewsMailer.weekly(user).deliver_now
end
```

Este proceso se repite, obteniendo más lotes según sea necesario, hasta que se hayan procesado todos los registros.

`find_each` funciona en clases de modelos, como se ve arriba, y también en relaciones:

```ruby
User.where(weekly_subscriber: true).find_each do |user|
  NewsMailer.weekly(user).deliver_now
end
```

siempre que no tengan orden, ya que el método necesita forzar una orden
internamente para iterar.

Si hay una orden presente en el receptor, el comportamiento depende de la bandera
`config.active_record.error_on_ignored_order`. Si es true (verdadero), el `ArgumentError` es
elevado, de lo contrario se ignora la orden y se emite una advertencia, que es el
defecto. Esto se puede anular con la opción `:error_on_ignore`, explicada
abajo.

##### Options for `find_each`

**`:batch_size`**

La opción `:batch_size` le permite especificar el número de registros que se recuperarán en cada lote, antes de pasarlos individualmente al bloque. Por ejemplo, para recuperar registros en lotes de 5000:

```ruby
User.find_each(batch_size: 5000) do |user|
  NewsMailer.weekly(user).deliver_now
end
```

**`:start`**

De forma predeterminada, los registros se obtienen en orden ascendente de la clave primaria. La opción `:start` le permite configurar la primera ID de la secuencia siempre que la ID más baja no sea la que necesita. Esto sería útil, por ejemplo, si desea reanudar un proceso por lotes interrumpido, siempre que haya guardado la última ID procesada como un punto de control.

Por ejemplo, para enviar boletines solo a usuarios con la clave principal a partir de 2000:

```ruby
User.find_each(start: 2000) do |user|
  NewsMailer.weekly(user).deliver_now
end
```

**`:finish`**

Similar a la opción `:start`,`:finish` le permite configurar la última ID de la secuencia siempre que la ID más alta no sea la que necesita.

Esto sería útil, por ejemplo, si desea ejecutar un proceso por lotes utilizando un subconjunto de registros basados ​​en `:inicio` y `:finalización`.

Por ejemplo, para enviar boletines solo a usuarios con la clave principal a partir de 2000 hasta 10000:

```ruby
User.find_each(start: 2000, finish: 10000) do |user|
  NewsMailer.weekly(user).deliver_now
end
```

Otro ejemplo sería si quisieras que varios trabajadores manejan la misma
cola de procesamiento. Puede hacer que cada trabajador maneje 10000 registros configurando
opciones apropiadas de `:start` y`:finish` para cada trabajador.

**`:error_on_ignore`**

Invalida la configuración de la aplicación para especificar si se debe generar un error cuando un
orden este presente en la relación.

#### `find_in_batches`

El método `find_in_batches` es similar a `find_each`, ya que ambos recuperan lotes de registros. La diferencia es que `find_in_batches` produce _batches_ para el bloque como una matriz de modelos, en lugar de individualmente. El siguiente ejemplo le dará al bloque suministrado una matriz de hasta 1000 facturas a la vez, con el bloque final que contiene las facturas restantes:

```ruby
# Give add_invoices an array of 1000 invoices at a time.
Invoice.find_in_batches do |invoices|
  export.add_invoices(invoices)
end
```

`find_in_batches` funciona en clases de modelos, como se ve arriba, y también en relaciones:

```ruby
Invoice.pending.find_in_batches do |invoices|
  pending_invoices_export.add_invoices(invoices)
end
```

siempre que no tengan un ordern, ya que el método necesita forzar un orden
internamente para iterar.

##### Opciones para `find_in_batches`

El método `find_in_batches` acepta las mismas opciones que `find_each`:

**`:batch_size`**

Al igual que para `find_each`, `batch_size` establece cuántos registros se recuperarán en cada grupo. Por ejemplo, la recuperación de lotes de 2500 registros se puede especificar como:

```ruby
Invoice.find_in_batches(batch_size: 2500) do |invoices|
  export.add_invoices(invoices)
end
```

**`:start`**

La opción `start` permite especificar la ID inicial desde donde se seleccionarán los registros. Como se mencionó anteriormente, por defecto los registros se obtienen en orden ascendente de la clave primaria. Por ejemplo, para recuperar facturas que comienzan en ID: 5000 en lotes de 2500 registros, se puede usar el siguiente código:

```ruby
Invoice.find_in_batches(finish: 7000) do |invoices|
  export.add_invoices(invoices)
end
```

**`:error_on_ignore`**

La opción `error_on_ignore` anula la configuración de la aplicación para especificar si se debe generar un error cuando hay un orden específico en la relación.

Conditions
----------

Condiciones

El método `where` le permite especificar condiciones para limitar los registros devueltos, representando la parte` WHERE` de la instrucción SQL. Las condiciones se pueden especificar como una cadena, matriz o hash.

### Pure String Conditions
Condiciones de cadena pura

Si desea agregar condiciones a su búsqueda, puede especificarlas allí, como `Client.where ("orders_count = '2'")`. Esto encontrará a todos los clientes donde el valor del campo `orders_count` es 2.

ADVERTENCIA: Construir sus propias condiciones como cadenas puras puede dejarlo vulnerable a los ataques de inyección SQL. Por ejemplo, `Client.where("first_name LIKE '%#{params[:first_name]}%'")` no es seguro. Consulte la siguiente sección para conocer la forma preferida de manejar las condiciones utilizando una matriz.

### Array Conditions
Condiciones de la matriz

Ahora, ¿qué pasa si ese número puede variar, digamos como un argumento desde algún lugar? El hallazgo entonces tomaría la forma:

```ruby
Client.where("orders_count = ?", params[:orders])
```

Active Record tomará el primer argumento como la cadena de condiciones y cualquier argumento adicional reemplazará los signos de interrogación `(?)` en él.

Si desea especificar condiciones múltiples:

```ruby
Client.where("orders_count = ? AND locked = ?", params[:orders], false)
```

En este ejemplo, el primer signo de interrogación se reemplazará con el valor en `params[:orders]` y el segundo se reemplazará con la representación SQL de `false`, que depende del adaptador.

Este código es altamente preferible:

```ruby
Client.where("orders_count = ?", params[:orders])
```

a este código:

```ruby
Client.where("orders_count = #{params[:orders]}")
```

por razones de seguridad. Poner la variable directamente en la cadena de condiciones pasará la variable a la base de datos **as-is** (tal cual). Esto significa que será una variable sin escape directamente de un usuario que pueda tener intenciones maliciosas. Si hace esto, pone en riesgo toda su base de datos porque una vez que un usuario se entera de que puede explotar su base de datos, puede hacer casi cualquier cosa. Nunca ponga sus argumentos directamente dentro de la cadena de condiciones.

SUGERENCIA: Para obtener más información sobre los peligros de la inyección SQL, consulte la publicación [Ruby on Rails Security Guide](security.html#sql-injection).

#### Placeholder Conditions

Similar al estilo de reemplazo de  como (`?)`, también se puede especificar claves en la cadena de condiciones junto con un hash de claves/valores correspondiente:

```ruby
Client.where("created_at >= :start_date AND created_at <= :end_date",
  {start_date: params[:start_date], end_date: params[:end_date]})
```

Esto permite una legibilidad más clara si tiene una gran cantidad de condiciones variables.

### Hash Conditions
Condiciones de hash

Active Record también le permite pasar en condiciones hash que pueden aumentar la legibilidad de la sintaxis de sus condiciones. Con condiciones hash, pasa un hash con las claves de los campos que desea calificar y los valores de cómo desea calificarlos:

NOTA: Solo es posible la verificación de igualdad, rango y subconjunto con condiciones Hash.

#### Equality Conditions
Condiciones de igualdad

```ruby
Client.where(locked: true)
```

Esto generará SQL como este:

```sql
SELECT * FROM clients WHERE (clients.locked = 1)
```

El nombre del campo también puede ser una cadena:

```ruby
Client.where('locked' => true)
```

En el caso de una relación belong_to, se puede usar una asociación de clave para especificar el modelo si se usa un objeto Active Record como valor. Este método también funciona con relaciones polimórficas.

```ruby
Article.where(author: author)
Author.joins(:articles).where(articles: { author: author })
```

#### Range Conditions
Condiciones de rango

```ruby
Client.where(created_at: (Time.now.midnight - 1.day)..Time.now.midnight)
```

Esto encontrará todos los clientes creados ayer mediante el uso de una instrucción SQL `BETWEEN`

```sql
SELECT * FROM clients WHERE (clients.created_at BETWEEN '2008-12-21 00:00:00' AND '2008-12-22 00:00:00')
```

Esto demuestra una sintaxis más corta para los ejemplos en [Array Conditions](#array-conditions)

#### Subset Conditions
Condiciones de subconjunto

Si desea buscar registros utilizando la expresión `IN`, puede pasar una matriz a las condiciones hash:

```ruby
Client.where(orders_count: [1,3,5])
```

Este código generará SQL así:

```sql
SELECT * FROM clients WHERE (clients.orders_count IN (1,3,5))
```

### NOT Conditions
Condiciones NOT

`NOT` SQL queries can be built by `where.not`:

```ruby
Client.where.not(locked: true)
```

En otras palabras, esta consulta se puede generar llamando a `where` sin argumento, e inmediatamente encadenando con` not` pasando `where` condiciones. Esto generará SQL como este:

```sql
SELECT * FROM clients WHERE (clients.locked != 1)
```

### OR Conditions
Condiciones OR

Las condiciones `OR` entre dos relaciones se pueden construir llamando a `o` en la primera
relación, y pasando el segundo como argumento.

```ruby
Client.where(locked: true).or(Client.where(orders_count: [1,3,5]))
```

```sql
SELECT * FROM clients WHERE (clients.locked = 1 OR clients.orders_count IN (1,3,5))
```

Ordering
--------
Ordenar

Para recuperar registros de la base de datos en un orden específico, puede usar el método `order`.

Por ejemplo, si obtiene un conjunto de registros y desea ordenarlos en orden ascendente por el campo `created_at` de su tabla:

```ruby
Client.order(:created_at)
# OR
Client.order("created_at")
```

También se puede especificar `ASC` o` DESC`:

```ruby
Client.order(created_at: :desc)
# OR
Client.order(created_at: :asc)
# OR
Client.order("created_at DESC")
# OR
Client.order("created_at ASC")
```

O ordenando por múltiples campos:


```ruby
Client.order(orders_count: :asc, created_at: :desc)
# OR
Client.order(:orders_count, created_at: :desc)
# OR
Client.order("orders_count ASC, created_at DESC")
# OR
Client.order("orders_count ASC", "created_at DESC")
```

Si desea llamar a `order` varias veces, los pedidos posteriores se agregarán al primero:

```ruby
Client.order("orders_count ASC").order("created_at DESC")
# SELECT * FROM clients ORDER BY orders_count ASC, created_at DESC
```

ADVERTENCIA: En la mayoría de los sistemas de bases de datos, al seleccionar campos con `distinct` de un conjunto de resultados utilizando métodos como `select`, `pluck` e `ids`; el método `order` generará una excepción `ActiveRecord::StatementInvalid` a menos que los campos utilizados en la cláusula `order` se incluyan en la lista de selección. Consulte la siguiente sección para seleccionar campos del conjunto de resultados.

Selecting Specific Fields
-------------------------
Seleccionar campos específicos

Por defecto, `Model.find` selecciona todos los campos del conjunto de resultados usando `select *`.

Para seleccionar solo un subconjunto de campos del conjunto de resultados, puede especificar el subconjunto mediante el método `select`.

Por ejemplo, para seleccionar solo columnas `viewable_by` y` bloqueadas`:

```ruby
Client.select(:viewable_by, :locked)
# OR
Client.select("viewable_by, locked")
```

La consulta SQL utilizada por esta llamada de búsqueda será algo así como:

```sql
SELECT viewable_by, locked FROM clients
```

Tenga cuidado porque esto también significa que está inicializando un objeto modelo con solo los campos que haz seleccionado. Si intenta acceder a un campo que no está en el registro inicializado, recibirá:

```bash
ActiveModel::MissingAttributeError: missing attribute: <attribute>
```

Donde `<attribute>` es el atributo que solicitó. El método `id` no generará el `ActiveRecord::MissingAttributeError`, así que tenga cuidado al trabajar con asociaciones porque necesitan el método `id` para funcionar correctamente.

Si desea obtener solo un registro por valor único en un campo determinado, puede usar `distinct`:

```ruby
Client.select(:name).distinct
```

Esto generaría SQL como:

```sql
SELECT DISTINCT name FROM clients
```

También puede eliminar la restricción de unicidad:

```ruby
query = Client.select(:name).distinct
# => Returns unique names

query.distinct(false)
# => Returns all names, even if there are duplicates
```

Limit and Offset
----------------
Límite y compensación

Para aplicar `LIMIT` al SQL disparado por el` Model.find`, puede especificar el `LIMIT` usando los métodos `limit` y `offset` en la relación.

Puede usar `limit` para especificar el número de registros que se recuperarán, y usar `offset` para especificar el número de registros que se omitirán antes de comenzar a devolver los registros. Por ejemplo

```ruby
Client.limit(5)
```

devolverá un máximo de 5 clientes y dado que no especifica ningún desplazamiento, devolverá los primeros 5 de la tabla. El SQL que ejecuta se ve así:

```sql
SELECT * FROM clients LIMIT 5
```

Añadiendo `offset` a eso

```ruby
Client.limit(5).offset(30)
```

devolverá en su lugar un máximo de 5 clientes a partir del 31. El SQL se ve así:

```sql
SELECT * FROM clients LIMIT 5 OFFSET 30
```

Group
-----

Grupo

Para aplicar una cláusula `GROUP BY` al SQL disparado por el buscador, puede usar el método `group`.

Por ejemplo, si desea encontrar una colección de las fechas en que se crearon los pedidos:

```ruby
Order.select("date(created_at) as ordered_date, sum(price) as total_price").group("date(created_at)")
```

Y esto le dará un único objeto `Order` para cada fecha en la que haya pedidos en la base de datos.

El SQL que se ejecutaría sería algo como esto:

```sql
SELECT date(created_at) as ordered_date, sum(price) as total_price
FROM orders
GROUP BY date(created_at)
```

### Total of grouped items
Total de artículos agrupados

Para obtener el total de elementos agrupados en una sola consulta, llame a `count` después del `group`.

```ruby
Order.group(:status).count
# => { 'awaiting_approval' => 7, 'paid' => 12 }
```

El SQL que se ejecutaría sería algo como esto:

```sql
SELECT COUNT (*) AS count_all, status AS status
FROM "orders"
GROUP BY status
```

Having
------

SQL usa la cláusula `HAVING` para especificar condiciones en los campos `GROUP BY`. Puede agregar la cláusula `HAVING` al SQL disparado por el `Model.find` agregando el método `having` al hallazgo.

Por ejemplo:

```ruby
Order.select("date(created_at) as ordered_date, sum(price) as total_price").
  group("date(created_at)").having("sum(price) > ?", 100)
```

El SQL que se ejecutaría sería algo como esto:

```sql
SELECT date(created_at) as ordered_date, sum(price) as total_price
FROM orders
GROUP BY date(created_at)
HAVING sum(price) > 100
```

Esto devuelve la fecha y el precio total de cada objeto de pedido, agrupados por el día en que se ordenaron y donde el precio es superior a $ 100.


Overriding Conditions
---------------------
Condiciones primordiales

### `unscope`

Puede especificar ciertas condiciones que se eliminarán utilizando el método `unscope`. Por ejemplo:

```ruby
Article.where('id > 10').limit(20).order('id asc').unscope(:order)
```

El SQL que se ejecutaría:

```sql
SELECT * FROM articles WHERE id > 10 LIMIT 20

# Original query without `unscope`
SELECT * FROM articles WHERE id > 10 ORDER BY id asc LIMIT 20

```

También puede desmarcar cláusulas específicas 'where'. Por ejemplo:


```ruby
Article.where(id: 10, trashed: false).unscope(where: :id)
# SELECT "articles".* FROM "articles" WHERE trashed = 0
```

Una relación que ha usado `unscope` afectará cualquier relación en la que se fusione:

```ruby
Article.order('id asc').merge(Article.unscope(:order))
# SELECT "articles".* FROM "articles"
```

### `only`

También puede anular condiciones utilizando el método `only`. Por ejemplo:

```ruby
Article.where('id > 10').limit(20).order('id desc').only(:order, :where)
```

El SQL que se ejecutaría:

```sql
SELECT * FROM articles WHERE id > 10 ORDER BY id DESC

# Original query without `only`
SELECT * FROM articles WHERE id > 10 ORDER BY id DESC LIMIT 20

```

### `reselect`

El método `reselect` anula una instrucción select existente. Por ejemplo:

```ruby
Post.select(:title, :body).reselect(:created_at)
```

El SQL que se ejecutaría:

```sql
SELECT `posts`.`created_at` FROM `posts`
```

En caso de que no se use la cláusula `reselect`,

```ruby
Post.select(:title, :body).select(:created_at)
```

el SQL ejecutado sería:

```sql
SELECT `posts`.`title`, `posts`.`body`, `posts`.`created_at` FROM `posts`
```

### `reorder`

El método `reorder` anula el orden de alcance predeterminado. Por ejemplo:


```ruby
class Article < ApplicationRecord
  has_many :comments, -> { order('posted_at DESC') }
end

Article.find(10).comments.reorder('name')
```

El SQL que se ejecutaría:

```sql
SELECT * FROM articles WHERE id = 10 LIMIT 1
SELECT * FROM comments WHERE article_id = 10 ORDER BY name
```

En el caso donde no se usa la cláusula `reorder`, el SQL ejecutado sería:

```sql
SELECT * FROM articles WHERE id = 10 LIMIT 1
SELECT * FROM comments WHERE article_id = 10 ORDER BY posted_at DESC
```

### `reverse_order`

El método `reverse_order` revierte la cláusula de ordenación si se especifica.

```ruby
Client.where("orders_count > 10").order(:name).reverse_order
```

El SQL que se ejecutaría:

```sql
SELECT * FROM clients WHERE orders_count > 10 ORDER BY name DESC
```

Si no se especifica una cláusula de pedido en la consulta, el `reverse_order` ordena por la clave primaria en orden inverso.

```ruby
Client.where("orders_count > 10").reverse_order
```
El SQL que se ejecutaría:

```sql
SELECT * FROM clients WHERE orders_count > 10 ORDER BY clients.id DESC
```

Este método ** no ** acepta argumentos. 

### `rewhere`

El método `rewhere` anula una condición existente, llamada where. Por ejemplo:

```ruby
Article.where(trashed: true).rewhere(trashed: false)
```

El SQL que se ejecutar

```sql
SELECT * FROM articles WHERE `trashed` = 1 AND `trashed` = 0
```

Null Relation
-------------

El método `none` devuelve una relación encadenable sin registros. Cualquier condición posterior encadenada a la relación devuelta continuará generando relaciones vacías. Esto es útil en escenarios donde se necesita una respuesta encadenable a un método o un alcance que podría devolver ningun resultados.

```ruby
Article.none # returns an empty Relation and fires no queries.
```

```ruby
# The visible_articles method below is expected to return a Relation.
@articles = current_user.visible_articles.where(name: params[:name])

def visible_articles
  case role
  when 'Country Manager'
    Article.where(country: country)
  when 'Reviewer'
    Article.published
  when 'Bad User'
    Article.none # => returning [] or nil breaks the caller code in this case
  end
end
```

Readonly Objects
----------------

Active Record proporciona el método `readonly` en una relación para rechazar explícitamente la modificación de cualquiera de los objetos devueltos. Cualquier intento de alterar un registro de solo lectura no tendrá éxito, generando una excepción `ActiveRecord::ReadOnlyRecord`.

```ruby
client = Client.readonly.first
client.visits += 1
client.save
```

Como `client` está configurado explícitamente para ser un objeto de solo lectura, el código anterior generará una excepción `ActiveRecord::ReadOnlyRecord` cuando llame a `client.save` con un valor actualizado de _visits_.

Locking Records for Update
--------------------------

El bloqueo es útil para prevenir las condiciones de carrera al actualizar registros en la base de datos y garantizar actualizaciones atómicas.

Active Record proporciona dos mecanismos de bloqueo:

* Bloqueo Optimistic (optimista)
* Bloqueo Pessimistic (pesimista)

### Optimistic Locking

El bloqueo optimista permite que varios usuarios accedan al mismo registro para las ediciones, y supone un mínimo de conflictos con los datos. Lo hace comprobando si otro proceso ha realizado cambios en un registro desde que se abrió. Se produce una excepción `ActiveRecord::StaleObjectError` si eso ha ocurrido y se ignora la actualización.

**Optimistic locking column**

Para utilizar el bloqueo optimista, la tabla debe tener una columna llamada `lock_version` de tipo entero. Cada vez que se actualiza el registro, Active Record incrementa la columna `lock_version`. Si se realiza una solicitud de actualización con un valor inferior en el campo `lock_version` que el que está actualmente en la columna` lock_version` en la base de datos, la solicitud de actualización fallará con un `ActiveRecord::StaleObjectError`. Ejemplo:

```ruby
c1 = Client.find(1)
c2 = Client.find(1)

c1.first_name = "Michael"
c1.save

c2.name = "should fail"
c2.save # Raises an ActiveRecord::StaleObjectError
```

Entonces es su responsabilidad de lidiar con el conflicto rescatando la excepción y revertiendo, fusionando o aplicando la lógica comercial necesaria para resolver el conflicto.

Este comportamiento se puede desactivar estableciendo `ActiveRecord::Base.lock_optimistically = false`.

Para anular el nombre de la columna `lock_version`,` ActiveRecord::Base` proporciona un atributo de clase llamado `Lock_column`:

```ruby
class Client < ApplicationRecord
  self.locking_column = :lock_client_column
end
```

### Pessimistic Lockin

El bloqueo pesimista utiliza un mecanismo de bloqueo proporcionado por la base de datos subyacente. El uso de `lock` cuando se construye una relación obtiene un bloqueo exclusivo en las filas seleccionadas. Las relaciones que usan `lock` generalmente se envuelven dentro de una transacción para evitar condiciones de punto muerto.

Por ejemplo:

```ruby
Item.transaction do
  i = Item.lock.first
  i.name = 'Jones'
  i.save!
end
```

La sesión anterior produce el siguiente SQL para un servidor MySQL:

```sql
SQL (0.2ms)   BEGIN
Item Load (0.3ms)   SELECT * FROM `items` LIMIT 1 FOR UPDATE
Item Update (0.4ms)   UPDATE `items` SET `updated_at` = '2009-02-07 18:05:56', `name` = 'Jones' WHERE `id` = 1
SQL (0.8ms)   COMMIT
```

También puede pasar SQL sin formato al método `lock` para permitir diferentes tipos de bloqueos. Por ejemplo, MySQL tiene una expresión llamada `LOCK IN SHARE MODE` donde puede bloquear un registro pero aún permitir que otras consultas lo lean. Para especificar esta expresión, simplemente pásala como la opción de bloqueo:

```ruby
Item.transaction do
  i = Item.lock("LOCK IN SHARE MODE").find(1)
  i.increment!(:views)
end
```

Si ya tiene una instancia de su modelo, puede iniciar una transacción y adquirir el bloqueo de una vez usando el siguiente código:

```ruby
item = Item.first
item.with_lock do
  # This block is called within a transaction,
  # item is already locked.
  item.increment!(:views)
end
```

Joining Tables
--------------

Active Record proporciona dos métodos de búsqueda para especificar cláusulas `JOIN` en
SQL resultante: `joins` y` left_outer_joins`.
En donde `joins` se debe usarse para `INNER JOIN` o consultas personalizadas,
`left_outer_joins` se usa para consultas que usan` LEFT OUTER JOIN`.

### `une`

Hay varias formas de usar el método `join`.

#### Using a String SQL Fragment

Simplemente puede proporcionar el SQL sin formato que especifica la cláusula `JOIN` a` join`:

```ruby
Author.joins("INNER JOIN posts ON posts.author_id = authors.id AND posts.published = 't'")
```

Esto dará como resultado el siguiente SQL:

```sql
SELECT authors.* FROM authors INNER JOIN posts ON posts.author_id = authors.id AND posts.published = 't'
```

#### Using Array/Hash of Named Associations

Active Record le permite usar los nombres de las [associations](association_basics.html) definidas en el modelo como un acceso directo para especificar cláusulas `JOIN` para esas asociaciones cuando se utiliza el método `join`.

Por ejemplo, considere los siguientes modelos de `Category`,` Article`, `Comment`,` Guest` y `Tag`:

```ruby
class Category < ApplicationRecord
  has_many :articles
end

class Article < ApplicationRecord
  belongs_to :category
  has_many :comments
  has_many :tags
end

class Comment < ApplicationRecord
  belongs_to :article
  has_one :guest
end

class Guest < ApplicationRecord
  belongs_to :comment
end

class Tag < ApplicationRecord
  belongs_to :article
end
```

Ahora, todo lo siguiente producirá las consultas de unión esperadas usando `INNER JOIN`:

##### Joining a Single Association

```ruby
Category.joins(:articles)
```

Esto produce:

```sql
SELECT categories.* FROM categories
  INNER JOIN articles ON articles.category_id = categories.id
```

O, en español: "devolver un objeto ategoría para todas las categorías con artículos". Tenga en cuenta que verá categorías duplicadas si más de un artículo tiene la misma categoría. Si desea categorías únicas, puede usar `Category.joins(:articles).distinct`.

#### Joining Multiple Associations

```ruby
Article.joins(:category, :comments)
```

Esto produce:

```sql
SELECT articles.* FROM articles
  INNER JOIN categories ON categories.id = articles.category_id
  INNER JOIN comments ON comments.article_id = articles.id
```

O, en español: "devolver todos los artículos que tengan una categoría y al menos un comentario". Tenga en cuenta nuevamente que los artículos con múltiples comentarios aparecerán varias veces.

##### Joining Nested Associations (Single Level)

```ruby
Article.joins(comments: :guest)
```

Esto produce:

```sql
SELECT articles.* FROM articles
  INNER JOIN comments ON comments.article_id = articles.id
  INNER JOIN guests ON guests.comment_id = comments.id
```

O, en español: "devolver todos los artículos que tengan un comentario hecho por un invitado".

##### Joining Nested Associations (Multiple Level)

```ruby
Category.joins(articles: [{ comments: :guest }, :tags])
```

Esto produce:

```sql
SELECT categories.* FROM categories
  INNER JOIN articles ON articles.category_id = categories.id
  INNER JOIN comments ON comments.article_id = articles.id
  INNER JOIN guests ON guests.comment_id = comments.id
  INNER JOIN tags ON tags.article_id = articles.id
```

O, en español: "devuelve todas las categorías que tienen artículos, donde esos artículos tienen un comentario hecho por un invitado y donde esos artículos también tienen una etiqueta".

#### Specifying Conditions on the Joined Tables

Puede especificar condiciones en las tablas unidas utilizando las condiciones normales [Array](#array-condition) y [String](#pure-string-condition). [Hash conditions](#hash-conditions) proporcionan una sintaxis especial para especificar condiciones para las tablas unidas:

```ruby
time_range = (Time.now.midnight - 1.day)..Time.now.midnight
Client.joins(:orders).where('orders.created_at' => time_range)
```

Una sintaxis alternativa y más limpia es anidar las condiciones hash:

```ruby
time_range = (Time.now.midnight - 1.day)..Time.now.midnight
Client.joins(:orders).where(orders: { created_at: time_range })
```

Esto encontrará a todos los clientes que tienen pedidos que se crearon ayer, nuevamente utilizando una expresión SQL `BETWEEN`.

### `left_outer_joins`

Si desea seleccionar un conjunto de registros, estén o no asociados
registros que puede usar el método `left_outer_joins`.

```ruby
Author.left_outer_joins(:posts).distinct.select('authors.*, COUNT(posts.*) AS posts_count').group('authors.id')
```

Que produce:

```sql
SELECT DISTINCT authors.*, COUNT(posts.*) AS posts_count FROM "authors"
LEFT OUTER JOIN posts ON posts.author_id = authors.id GROUP BY authors.id
```

Lo que significa: "devolver a todos los autores con su recuento de publicaciones, ya sea que
tiene alguna publicación"

Eager Loading Associations
--------------------------

La carga ansiosa es el mecanismo para cargar los registros asociados de los objetos devueltos por `Model.find` utilizando la menor cantidad de consultas posible.

**Problema de consultas N + 1**

Considere el siguiente código, que encuentra 10 clientes e imprime sus códigos postales:

```ruby
clients = Client.limit(10)

clients.each do |client|
  puts client.address.postcode
end
```

Este código se ve bien a primera vista. Pero el problema radica en el número total de consultas ejecutadas. El código anterior ejecuta 1 (para encontrar 10 clientes) + 10 (uno por cada cliente para cargar la dirección) = **11** consultas en total.

**Solución al problema de consultas N + 1**

Active Record le permite especificar de antemano todas las asociaciones que se van a cargar. Esto es posible especificando el método `includes` de la llamada `Model.find`. Con `includes`, Active Record garantiza que todas las asociaciones especificadas se carguen utilizando el mínimo número posible de consultas.

```sql
SELECT * FROM clients LIMIT 10
SELECT addresses.* FROM addresses
  WHERE (addresses.client_id IN (1,2,3,4,5,6,7,8,9,10))
```

### Eager Loading Multiple Associations

Active Record le permite cargar cualquier número de asociaciones con una sola llamada `Model.find` mediante el uso de una matriz, hash o un hash anidado de matriz / hash con el método `include`.

#### Array of Multiple Associations

```ruby
Article.includes(:category, :comments)
```

Esto carga todos los artículos y la categoría asociada y comentarios para cada artículo.

#### Nested Associations Hash

```ruby
Category.includes(articles: [{ comments: :guest }, :tags]).find(1)
```

Esto encontrará la categoría con ID 1 y cargará ansiosamente todos los artículos asociados, las etiquetas y comentarios de los artículos asociados y la asociación de invitados de cada comentario.

### Specifying Conditions on Eager Loaded Associations

A pesar de que Active Record le permite especificar condiciones en las asociaciones cargadas ansiosas al igual que `joins`, la forma recomendada es utilizar [joins](#joining-tables) en su lugar.

Sin embargo, si debe hacer esto, puede usar `where` como lo haría normalmente.

```ruby
Article.includes(:comments).where(comments: { visible: true })
```

Esto generaría una consulta que contiene un `LEFT OUTER JOIN` mientras que el
El método `joins` generaría uno usando la función `INNER JOIN` en su lugar.

```ruby
  SELECT "articles"."id" AS t0_r0, ... "comments"."updated_at" AS t1_r5 FROM "articles" LEFT OUTER JOIN "comments" ON "comments"."article_id" = "articles"."id" WHERE (comments.visible = 1)
```

Si no existiera la condición `where`, esto generaría el conjunto normal de dos consultas.

NOTA: Usar `where` de esta manera solo funcionará cuando le pase un Hash. por
Fragmentos de SQL end donde se necesita usar `referencias` para forzar tablas unidas:

```ruby
Article.includes(:comments).where("comments.visible = true").references(:comments)
```

Si, en el caso de esta consulta `includes`, no hubo comentarios para
artículos, todos los artículos aún se cargarían. Mediante el uso de `joins` (un INNER
JOIN), las condiciones de unión **deben** coincidir, de lo contrario no se registrarán registros
devuelto.

NOTA: Si una asociación está ansiosamente cargada como parte de una unión, los campos de una cláusula de selección personalizada no estarán presentes en los modelos cargados.
Esto se debe a que es ambiguo si deberían aparecer en el registro primario o en el secundario.

Scopes
------

El alcance le permite especificar consultas de uso común a las que se puede hacer referencia como llamadas a métodos en los objetos o modelos de asociación. Con estos ámbitos, puede utilizar todos los métodos cubiertos anteriormente, como `where`,` join` e `include`. Todos los cuerpos de ámbito deben devolver un `ActiveRecord::Relation` o `nil` para permitir que se invoquen otros métodos (como otros ámbitos).

Para definir un alcance simple, usamos el método `scope` dentro de la clase, pasando la consulta que nos gustaría ejecutar cuando se llama a este alcance:

```ruby
class Article < ApplicationRecord
  scope :published, -> { where(published: true) }
end
```

Los ámbitos también se pueden encadenar dentro de los ámbitos:

```ruby
class Article < ApplicationRecord
  scope :published,               -> { where(published: true) }
  scope :published_and_commented, -> { published.where("comments_count > 0") }
end
```

Para llamar a este alcance `published` podemos llamarlo en la clase:

```ruby
Article.published # => [published articles]
```

O en una asociación que consta de objetos `Article`:

```ruby
category = Category.first
category.articles.published # => [published articles belonging to this category]
```

### Passing in arguments

Su alcance puede tomar argumentos:

```ruby
class Article < ApplicationRecord
  scope :created_before, ->(time) { where("created_at < ?", time) }
end
```

Llame al ámbito como si fuera un método de clase:

```ruby
Article.created_before(Time.zone.now)
```

Sin embargo, esto es solo duplicar la funcionalidad que le proporcionaría un método de clase.

```ruby
class Article < ApplicationRecord
  def self.created_before(time)
    where("created_at < ?", time)
  end
end
```

Estos métodos seguirán siendo accesibles en los objetos de asociación:

```ruby
category.articles.created_before(time)
```

### Using conditionals

Su alcance puede utilizar condicionales:

```ruby
class Article < ApplicationRecord
  scope :created_before, ->(time) { where("created_at < ?", time) if time.present? }
end
```

Al igual que los otros ejemplos, esto se comportará de manera similar a un método de clase.

```ruby
class Article < ApplicationRecord
  def self.created_before(time)
    where("created_at < ?", time) if time.present?
  end
end
```

Sin embargo, hay una advertencia importante: un ámbito siempre devolverá un objeto `ActiveRecord::Relation`, incluso si el condicional se evalúa como `false`, mientras que un método de clase devolverá `nil`. Esto puede causar `NoMethodError` al encadenar métodos de clase con condicionales, si alguno de los condicionales devuelve `falso`.


### Applying a default scope

Si deseamos que se aplique un alcance en todas las consultas al modelo, podemos usar el
Método `default_scope` dentro del modelo mismo.

```ruby
class Client < ApplicationRecord
  default_scope { where("removed_at IS NULL") }
end
```

Cuando las consultas se ejecutan en este modelo, la consulta SQL ahora se verá similar a
esta:

```sql
SELECT * FROM clients WHERE removed_at IS NULL
```

Si necesita hacer cosas más complejas con un alcance predeterminado, también puede
definirlo como un método de clase:

```ruby
class Client < ApplicationRecord
  def self.default_scope
    # Should return an ActiveRecord::Relation.
  end
end
```

NOTA: El `default_scope` también se aplica al crear / construir un registro
cuando los argumentos de alcance se dan como un `Hash`. No se aplica mientras
actualizar un registro P.ej.:

```ruby
class Client < ApplicationRecord
  default_scope { where(active: true) }
end

Client.new          # => #<Client id: nil, active: true>
Client.
```

Tenga en cuenta que, cuando se proporciona en el formato `Array`, los argumentos de consulta` default_scope`
no se puede convertir a un `Hash` para la asignación de atributos predeterminada. P.ej.:

```ruby
class Client < ApplicationRecord
  default_scope { where("active = ?", true) }
end

Client.new # => #<Client id: nil, active: nil>
```

### Merging of scopes

Al igual que los alcances de las cláusulas `where` se fusionan utilizando condiciones `AND`.

```ruby
class User < ApplicationRecord
  scope :active, -> { where state: 'active' }
  scope :inactive, -> { where state: 'inactive' }
end

User.active.inactive
# SELECT "users".* FROM "users" WHERE "users"."state" = 'active' AND "users"."state" = 'inactive'
```

Podemos mezclar y combinar las condiciones de `scope` y `here` y el SQL final
tendrá todas las condiciones unidas con `AND`.

```ruby
User.active.where(state: 'finished')
# SELECT "users".* FROM "users" WHERE "users"."state" = 'active' AND "users"."state" = 'finished'
```

Si queremos que gane la última cláusula `where`, entonces `Relation#merge` puede
ser usado.

```ruby
User.active.merge(User.inactive)
# SELECT "users".* FROM "users" WHERE "users"."state" = 'inactive'
```

Una advertencia importante es que `default_scope` se antepondrá en
condiciones de `scope` y `where`.

```ruby
class User < ApplicationRecord
  default_scope { where state: 'pending' }
  scope :active, -> { where state: 'active' }
  scope :inactive, -> { where state: 'inactive' }
end

User.all
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending'

User.active
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending' AND "users"."state" = 'active'

User.where(state: 'inactive')
# SELECT "users".* FROM "users" WHERE "users"."state" = 'pending' AND "users"."state" = 'inactive'
```

Como puede ver arriba, el `default_scope` se está fusionando en ambos
condiciones de `scope` y `where`.

### Removing All Scoping

Si deseamos eliminar el alcance por cualquier motivo, podemos utilizar el método `unscoped`. Esto es
especialmente útil si se especifica un `default_scope` en el modelo y no debe ser
solicitó esta consulta en particular.

```ruby
Client.unscoped.load
```

Este método elimina todo el alcance y hará una consulta normal en la tabla.

```ruby
Client.unscoped.all
# SELECT "clients".* FROM "clients"

Client.where(published: false).unscoped.all
# SELECT "clients".* FROM "clients"
```

`unscoped` can also accept a block.

```ruby
Client.unscoped {
  Client.created_before(Time.zone.now)
}
```

Dynamic Finders
---------------

Para cada campo (también conocido como atributo) que defina en su tabla, Active Record proporciona un método de búsqueda. Si tiene un campo llamado `first_name` en su modelo `Client`, por ejemplo, obtiene `find_by_first_name` gratis de Active Record. Si tiene un campo `locked` en el modelo` Client`, también obtiene el método `find_by_locked`.

Puede especificar un signo de exclamación (`!`) Al final de los buscadores dinámicos para que generen un error `ActiveRecord::RecordNotFound` si no devuelven ningún registro, como `Client.find_by_name!("Ryan")`

Si desea encontrar tanto por nombre como bloqueado, puede encadenar estos buscadores simplemente escribiendo "`and`" entre los campos. Por ejemplo, `Client.find_by_first_name_and_locked("Ryan", true)`.


Enums
-----

La macro `enum` asigna una columna entera a un conjunto de valores posibles.

```ruby
class Book < ApplicationRecord
  enum availability: [:available, :unavailable]
end
```

Esto creará automáticamente los [scopes](#scopes) correspondientes para consultar el
modelo. Los métodos para la transición entre estados y consultar el estado actual también son
adicional.

```ruby
# Both examples below query just available books.
Book.available
# or
Book.where(availability: :available)

book = Book.new(availability: :available)
book.available?   # => true
book.unavailable! # => true
book.available?   # => false
```

Lea la documentación completa sobre las enumeraciones.
[in the Rails API docs](https://api.rubyonrails.org/classes/ActiveRecord/Enum.html).

Understanding The Method Chaining
---------------------------------

El patrón de registro activo implementa [Method Chaining](https://en.wikipedia.org/wiki/Method_chaining),
lo que nos permite usar múltiples métodos Active Record juntos de una manera simple y directa.

Puede encadenar métodos en una instrucción cuando el método anterior llamado devuelve un
`ActiveRecord::Relation`, como` all`, `where` y `joins`. Métodos que regresan
un solo objeto (ver [Retrieving a Single Object Section](#retrieving-a-single-object))
tiene que estar al final de la declaración.

Hay algunos ejemplos a continuación. Esta guía no cubrirá todas las posibilidades, solo algunas como ejemplos.
Cuando se llama a un método de Registro activo, la consulta no se genera de inmediato y no se envía a la base de datos,
esto solo sucede cuando los datos son realmente necesarios. Entonces, cada ejemplo a continuación genera una sola consulta.

### Retrieving filtered data from multiple tables

```ruby
Person
  .select('people.id, people.name, comments.text')
  .joins(:comments)
  .where('comments.created_at > ?', 1.week.ago)
```

El resultado debería ser algo como esto:

```sql
SELECT people.id, people.name, comments.text
FROM people
INNER JOIN comments
  ON comments.person_id = people.id
WHERE comments.created_at > '2015-01-01'
```

### Retrieving specific data from multiple tables

```ruby
Person
  .select('people.id, people.name, companies.name')
  .joins(:company)
  .find_by('people.name' => 'John') # this should be the last
```

Lo anterior debería generar:

```sql
SELECT people.id, people.name, companies.name
FROM people
INNER JOIN companies
  ON companies.person_id = people.id
WHERE people.name = 'John'
LIMIT 1
```

NOTA: Tenga en cuenta que si una consulta coincide con varios registros, `find_by`
busque solo el primero e ignore a los demás (consulte la declaración anterior 
`LIMIT 1`).

Find or Build a New Object
--------------------------

Es común que necesite encontrar un registro o crearlo si no existe. Puede hacerlo con los métodos `find_or_create_by` y `find_or_create_by!`.

### `find_or_create_by`

El método `find_or_create_by` verifica si existe un registro con los atributos especificados. Si no es así, se llama a `create`. Veamos un ejemplo.

Supongamos que desea encontrar un cliente llamado 'Andy', y si no hay ninguno, cree uno. Puede hacerlo ejecutando:

```ruby
Client.find_or_create_by(first_name: 'Andy')
# => #<Client id: 1, first_name: "Andy", orders_count: 0, locked: true, created_at: "2011-08-30 06:09:27", updated_at: "2011-08-30 06:09:27">
```

El SQL generado por este método se ve así:

```sql
SELECT * FROM clients WHERE (clients.first_name = 'Andy') LIMIT 1
BEGIN
INSERT INTO clients (created_at, first_name, locked, orders_count, updated_at) VALUES ('2011-08-30 05:22:57', 'Andy', 1, NULL, '2011-08-30 05:22:57')
COMMIT
```

`find_or_create_by` devuelve el registro que ya existe o el nuevo registro. En nuestro caso, todavía no teníamos un cliente llamado Andy, por lo que el registro se crea y se devuelve.

El nuevo registro podría no guardarse en la base de datos; eso depende de si las validaciones pasaron o no (al igual que `create`).

Supongamos que queremos establecer el atributo `locked` en `false` si estamos
creando un nuevo registro, pero no queremos incluirlo en la consulta. Entonces
queremos encontrar al cliente llamado "Andy", o si ese cliente no
existe, crea un cliente llamado "Andy" que no está bloqueado.

Podemos lograr esto de dos maneras. El primero es usar `create_with`:

```ruby
Client.create_with(locked: false).find_or_create_by(first_name: 'Andy')
```

La segunda forma es usar un bloque:

```ruby
Client.find_or_create_by(first_name: 'Andy') do |c|
  c.locked = false
end
```

El bloque solo se ejecutará si se está creando el cliente.
La segunda vez que ejecutamos este código, el bloque será ignorado.

### `find_or_create_by!`

También puede usar `find_or_create_by!` para generar una excepción si el nuevo registro no es válido. Las validaciones no están cubiertas en esta guía, pero supongamos por un momento que agregue temporalmente


```ruby
validates :orders_count, presence: true
```

a su modelo de `Client`. Si intenta crear un nuevo `Client` sin pasar un` orders_count`, el registro no será válido y se generará una excepción:

```ruby
Client.find_or_create_by!(first_name: 'Andy')
# => ActiveRecord::RecordInvalid: Validation failed: Orders count can't be blank
```

### `find_or_initialize_by`

El método `find_or_initialize_by` funcionará igual que
`find_or_create_by` pero llamará a `new` en lugar de `create`. Esta
significa que se creará una nueva instancia de modelo en la memoria pero no se
guardado en la base de datos. Continuando con el ejemplo `find_or_create_by`, nosotros
ahora quiero que el cliente se llame 'Nick':

```ruby
nick = Client.find_or_initialize_by(first_name: 'Nick')
# => #<Client id: nil, first_name: "Nick", orders_count: 0, locked: true, created_at: "2011-08-30 06:09:27", updated_at: "2011-08-30 06:09:27">

nick.persisted?
# => false

nick.new_record?
# => true
```

Debido a que el objeto aún no está almacenado en la base de datos, el SQL generado se ve así:

```sql
SELECT * FROM clients WHERE (clients.first_name = 'Nick') LIMIT 1
```

Cuando desee guardarlo en la base de datos, simplemente llame a `save`:

```ruby
nick.save
# => true
```

Finding by SQL
--------------

Si desea usar su propio SQL para buscar registros en una tabla, puede usar `find_by_sql`. El método `find_by_sql` devolverá una matriz de objetos incluso si la consulta subyacente devuelve un solo registro. Por ejemplo, podría ejecutar esta consulta:

```ruby
Client.find_by_sql("SELECT * FROM clients
  INNER JOIN orders ON clients.id = orders.client_id
  ORDER BY clients.created_at desc")
# =>  [
#   #<Client id: 1, first_name: "Lucas" >,
#   #<Client id: 2, first_name: "Jan" >,
#   ...
# ]
```

`find_by_sql` le proporciona una forma sencilla de realizar llamadas personalizadas a la base de datos y recuperar objetos instanciados.

### `select_all`

`find_by_sql` tiene un pariente cercano llamado `connection#select_all`. `select_all` recuperará
objetos de la base de datos usando SQL personalizado como `find_by_sql` pero no los instanciará.
Este método devolverá una instancia de la clase `ActiveRecord::Result` y llamará a `to_a` en este
objeto le devolvería una matriz de hashes donde cada hash indica un registro.

```ruby
Client.connection.select_all("SELECT first_name, created_at FROM clients WHERE id = '1'").to_a
# => [
#   {"first_name"=>"Rafael", "created_at"=>"2012-11-10 23:23:45.281189"},
#   {"first_name"=>"Eileen", "created_at"=>"2013-12-09 11:22:35.221282"}
# ]
```

### `pluck`

`pluck` puede usarse para consultar columnas simples o múltiples desde la tabla subyacente de un modelo. Acepta una lista de nombres de columnas como argumento y devuelve una matriz de valores de las columnas especificadas con el tipo de datos correspondiente.


```ruby
Client.where(active: true).pluck(:id)
# SELECT id FROM clients WHERE active = 1
# => [1, 2, 3]

Client.distinct.pluck(:role)
# SELECT DISTINCT role FROM clients
# => ['admin', 'member', 'guest']

Client.pluck(:id, :name)
# SELECT clients.id, clients.name FROM clients
# => [[1, 'David'], [2, 'Jeremy'], [3, 'Jose']]
```

`pluck` hace posible reemplazar código como:

```ruby
Client.select(:id).map { |c| c.id }
# or
Client.select(:id).map(&:id)
# or
Client.select(:id, :name).map { |c| [c.id, c.name] }
```

con:

```ruby
Client.pluck(:id)
# or
Client.pluck(:id, :name)
```

La diferencia de `select`,` pluck` convierte directamente el resultado de una base de datos en un Ruby `Array`,
sin construir objetos `ActiveRecord`. Esto puede significar un mejor rendimiento para
una consulta grande o de ejecución frecuente. Sin embargo, cualquier anulación de método modelo
No estar disponible. Por ejemplo:

```ruby
class Client < ApplicationRecord
  def name
    "I am #{super}"
  end
end

Client.select(:name).map &:name
# => ["I am David", "I am Jeremy", "I am Jose"]

Client.pluck(:name)
# => ["David", "Jeremy", "Jose"]
```

No está limitado a consultar campos desde una sola tabla, también puede consultar varias tablas.


```ruby
Client.joins(:comments, :categories).pluck("clients.email, comments.title, categories.name")
```

Además, a diferencia de `select` y otros ámbitos de` Relation`, `pluck` desencadena un inmediato
consulta y, por lo tanto, no se puede encadenar con otros ámbitos, aunque puede funcionar con
ámbitos ya construidos anteriormente:

```ruby
Client.pluck(:name).limit(1)
# => NoMethodError: undefined method `limit' for #<Array:0x007ff34d3ad6d8>

Client.limit(1).pluck(:name)
# => ["David"]
```

NOTA: También debe saber que el uso de `pluck` activará la carga ansiosa si el objeto de relación contiene valores de inclusión, incluso si la carga ansiosa no es necesaria para la consulta. Por ejemplo:

```ruby
# store association for reusing it
assoc = Company.includes(:account)
assoc.pluck(:id)
# SELECT "companies"."id" FROM "companies" LEFT OUTER JOIN "accounts" ON "accounts"."id" = "companies"."account_id"
```

Una forma de evitar esto es `unscope` la inclusión

```ruby
assoc.unscope(:includes).pluck(:id)
```

### `ids`

Los `ids` se pueden usar para seleccionar todas las ID de la relación utilizando la clave primaria de la tabla.

```ruby
Person.ids
# SELECT id FROM people
```

```ruby
class Person < ApplicationRecord
  self.primary_key = "person_id"
end

Person.ids
# SELECT person_id FROM people
```

Existence of Objects
--------------------

Si simplemente desea verificar la existencia del objeto, hay un método llamado `exists?`.
Este método consultará la base de datos utilizando la misma consulta que `find`, pero en lugar de devolver un
objeto o colección de objetos devolverá `true` o `false`.


```ruby
Client.exists?(1)
```

El método `exists?`también toma múltiples valores, pero el problema es que devolverá `true` si alguno
uno de esos registros existe.

```ruby
Client.exists?(id: [1,2,3])
# or
Client.exists?(name: ['John', 'Sergei'])
```

Incluso es posible usar `exists?` sin ningún argumento sobre un modelo o una relación.

```ruby
Client.where(first_name: 'Ryan').exists?
```

The above returns `true` if there is at least one client with the `first_name` 'Ryan' and `false`
otherwise.

```ruby
Client.exists?
```

Lo anterior devuelve `false` si la tabla `clients` está vacía y `true` de lo contrario.

También puede usar `any?` Y `many?` Para verificar la existencia en un modelo o relación.

```ruby
# via a model
Article.any?
Article.many?

# via a named scope
Article.recent.any?
Article.recent.many?

# via a relation
Article.where(published: true).any?
Article.where(published: true).many?

# via an association
Article.first.categories.any?
Article.first.categories.many?
```

Calculations
------------

Esta sección utiliza el conteo como método de ejemplo en este preámbulo, pero las opciones descritas se aplican a todas las subsecciones.

Todos los métodos de cálculo funcionan directamente en un modelo:

```ruby
Client.count
# SELECT COUNT(*) FROM clients
```

O en una relación:

```ruby
Client.where(first_name: 'Ryan').count
# SELECT COUNT(*) FROM clients WHERE (first_name = 'Ryan')
```

También puede usar varios métodos de búsqueda en una relación para realizar cálculos complejos:

```ruby
Client.includes("orders").where(first_name: 'Ryan', orders: { status: 'received' }).count
```

Que ejecutará:

```sql
SELECT COUNT(DISTINCT clients.id) FROM clients
  LEFT OUTER JOIN orders ON orders.client_id = clients.id
  WHERE (clients.first_name = 'Ryan' AND orders.status = 'received')
```

### Count

Si desea ver cuántos registros hay en la tabla de su modelo, puede llamar a `Client.count` y eso devolverá el número. Si desea ser más específico y encontrar todos los clientes con su edad presente en la base de datos, puede usar `Client.count(:age)`.

Para ver las opciones, consulte la sección principal, [Calculations](#calculations).

### Average

Si desea ver el promedio de un cierto número en una de sus tablas, puede llamar al método `average` en la clase que se relaciona con la tabla. Esta llamada al método se verá así:

```ruby
Client.average("orders_count")
```

Esto devolverá un número (posiblemente un número de coma flotante como 3.14159265) que representa el valor promedio en el campo.

Para ver las opciones, consulte la sección principal, [Calculations](#calculations).

### Minimum

Si desea encontrar el valor mínimo de un campo en su tabla, puede llamar al método `minimum` en la clase que se relaciona con la tabla. Esta llamada al método se verá así:

```ruby
Client.minimum("age")
```

Para ver las opciones, consulte la sección principal, [Calculations](#calculations).

### Maximum

Si desea encontrar el valor máximo de un campo en su tabla, puede llamar al método `maximum` en la clase que se relaciona con la tabla. Esta llamada al método se verá así:

```ruby
Client.maximum("age")
```

Para ver las opciones, consulte la sección principal, [Calculations](#calculations).

### Sum

Si desea encontrar la suma de un campo para todos los registros en su tabla, puede llamar al método `sum` en la clase que se relaciona con la tabla. Esta llamada al método se verá así:

```ruby
Client.sum("orders_count")
```

Para ver las opciones, consulte la sección principal, [Calculations](#calculations).

Running EXPLAIN
---------------

Puede ejecutar EXPLAIN en las consultas desencadenadas por las relaciones. Por ejemplo,

```ruby
User.where(id: 1).joins(:articles).explain
```

puede rendir

```sql
EXPLAIN for: SELECT `users`.* FROM `users` INNER JOIN `articles` ON `articles`.`user_id` = `users`.`id` WHERE `users`.`id` = 1
+----+-------------+----------+-------+---------------+
| id | select_type | table    | type  | possible_keys |
+----+-------------+----------+-------+---------------+
|  1 | SIMPLE      | users    | const | PRIMARY       |
|  1 | SIMPLE      | articles | ALL   | NULL          |
+----+-------------+----------+-------+---------------+
+---------+---------+-------+------+-------------+
| key     | key_len | ref   | rows | Extra       |
+---------+---------+-------+------+-------------+
| PRIMARY | 4       | const |    1 |             |
| NULL    | NULL    | NULL  |    1 | Using where |
+---------+---------+-------+------+-------------+

2 rows in set (0.00 sec)
```

bajo MySQL y MariaDB.

Active Record realiza una bonita impresión que emula la del
base de datos correspondiente. Entonces, la misma consulta que se ejecuta con el
el adaptador PostgreSQL produciría en su lugar

```sql
EXPLAIN for: SELECT "users".* FROM "users" INNER JOIN "articles" ON "articles"."user_id" = "users"."id" WHERE "users"."id" = 1
                                  QUERY PLAN
------------------------------------------------------------------------------
 Nested Loop Left Join  (cost=0.00..37.24 rows=8 width=0)
   Join Filter: (articles.user_id = users.id)
   ->  Index Scan using users_pkey on users  (cost=0.00..8.27 rows=1 width=4)
         Index Cond: (id = 1)
   ->  Seq Scan on articles  (cost=0.00..28.88 rows=8 width=4)
         Filter: (articles.user_id = 1)
(6 rows)
```

La carga ansiosa puede desencadenar más de una consulta debajo del capó, y algunas consultas
puede necesitar los resultados de los anteriores. Por eso, `explain` en realidad
ejecuta la consulta y luego solicita los planes de consulta. Por ejemplo,

```ruby
User.where(id: 1).includes(:articles).explain
```

rendimientos

```sql
EXPLAIN for: SELECT `users`.* FROM `users`  WHERE `users`.`id` = 1
+----+-------------+-------+-------+---------------+
| id | select_type | table | type  | possible_keys |
+----+-------------+-------+-------+---------------+
|  1 | SIMPLE      | users | const | PRIMARY       |
+----+-------------+-------+-------+---------------+
+---------+---------+-------+------+-------+
| key     | key_len | ref   | rows | Extra |
+---------+---------+-------+------+-------+
| PRIMARY | 4       | const |    1 |       |
+---------+---------+-------+------+-------+

1 row in set (0.00 sec)

EXPLAIN for: SELECT `articles`.* FROM `articles`  WHERE `articles`.`user_id` IN (1)
+----+-------------+----------+------+---------------+
| id | select_type | table    | type | possible_keys |
+----+-------------+----------+------+---------------+
|  1 | SIMPLE      | articles | ALL  | NULL          |
+----+-------------+----------+------+---------------+
+------+---------+------+------+-------------+
| key  | key_len | ref  | rows | Extra       |
+------+---------+------+------+-------------+
| NULL | NULL    | NULL |    1 | Using where |
+------+---------+------+------+-------------+


1 row in set (0.00 sec)
```

bajo MySQL y MariaDB.



### Interpreting EXPLAIN

La interpretación del resultado de EXPLAIN está más allá del alcance de esta guía. los
siguientes consejos pueden ser útiles:

* SQLite3: [EXPLAIN QUERY PLAN](https://www.sqlite.org/eqp.html)

* MySQL: [EXPLAIN Output Format](https://dev.mysql.com/doc/refman/en/explain-output.html)

* MariaDB: [EXPLAIN](https://mariadb.com/kb/en/mariadb/explain/)

* PostgreSQL: [Using EXPLAIN](https://www.postgresql.org/docs/current/static/using-explain.html)
