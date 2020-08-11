**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Active Record Callbacks
=======================
Active Record Devoluciones de Llamada 

Esta guía le enseña cómo conectarse al ciclo de vida de su Registro Activo
objetos.

Después de leer esta guía, sabrá:

* El ciclo de vida de los objetos Active Record.
* Cómo crear métodos de devolución de llamada que respondan a eventos en el ciclo de vida del objeto.
* Cómo crear clases especiales que encapsulan un comportamiento común para sus devoluciones de llamada.

--------------------------------------------------------------------------------

The Object Life Cycle
---------------------
El Ciclo de Vida del Objeto

Durante el funcionamiento normal de una aplicación de Rails, se pueden crear, actualizar y destruir los objetos. Active Record proporciona ganchos en este *object life cycle* para que pueda controlar su aplicación y sus datos.

Las devoluciones de llamada le permiten activar la lógica antes o después de una alteración del estado de un objeto.

Callbacks Overview
------------------
Descripción General de Devoluciones de Llamada

Las devoluciones de llamada son métodos que se llaman en ciertos momentos del ciclo de vida de un objeto. Con las devoluciones de llamada, es posible escribir código que se ejecutará siempre que se cree, guarde, actualice, elimine, valide o cargue un objeto Active Record desde la base de datos.

```ruby
class User < ApplicationRecord
  validates :login, :email, presence: true

  before_validation :ensure_login_has_a_value

  private
    def ensure_login_has_a_value
      if login.nil?
        self.login = email unless email.blank?
      end
    end
end
```

Los métodos de clase de estilo macro también pueden recibir un bloque. Considere usar este estilo si el código dentro de su bloque es tan corto que cabe en una sola línea:

```ruby
class User < ApplicationRecord
  validates :login, :email, presence: true

  before_create do
    self.name = login.capitalize if name.blank?
  end
end
```

Las devoluciones de llamada también se pueden registrar para que solo se activen en ciertos eventos del ciclo de vida:

```ruby
class User < ApplicationRecord
  before_validation :normalize_name, on: :create

  # :on takes an array as well
  after_validation :set_location, on: [ :create, :update ]

  private
    def normalize_name
      self.name = name.downcase.titleize
    end

    def set_location
      self.location = LocationService.query(self)
    end
end
```

Se considera una buena práctica declarar los métodos de devolución de llamada como privados. Si se dejan públicos, se pueden llamar desde fuera del modelo y violan el principio de encapsulación de objetos.

Available Callbacks
-------------------
Callbacks Disponibles

Aquí hay una lista con todas las devoluciones de llamada de Active Record disponibles, enumeradas en el mismo orden en que se las llamará durante las respectivas operaciones:

### Creating an Object

* `before_validation`
* `after_validation`
* `before_save`
* `around_save`
* `before_create`
* `around_create`
* `after_create`
* `after_save`
* `after_commit/after_rollback`

### Updating an Object

* `before_validation`
* `after_validation`
* `before_save`
* `around_save`
* `before_update`
* `around_update`
* `after_update`
* `after_save`
* `after_commit/after_rollback`

### Destroying an Object

* `before_destroy`
* `around_destroy`
* `after_destroy`
* `after_commit/after_rollback`

ADVERTENCIA. `after_save` se ejecuta tanto en crear como en actualizar, pero siempre _after_ de las devoluciones de llamada más específicas `after_create` y `after_update`, sin importar el orden en que se ejecutaron las llamadas macro.

ADVERTENCIA. Se debe tener cuidado en las devoluciones de llamada para evitar actualizar los atributos. Por ejemplo, evite ejecutar `update(attribute: "value")`  y un código similar durante las devoluciones de llamada. Esto puede alterar el estado del modelo y puede provocar efectos secundarios inesperados durante la confirmación. En su lugar, debe intentar asignar valores en las devoluciones de llamada `before_create` o anteriores.

NOTA: las devoluciones de llamada `before_destroy` deben colocarse antes de las asociaciones  `dependent: :destroy`
(o use la opción `prepend: true`), para asegurarse de que se ejecuten antes que
los registros sean eliminados por `dependent: :destroy`.

### `after_initialize` and `after_find`

La devolución de llamada `after_initialize` se llamará siempre que se cree una instancia de un objeto Active Record, ya sea utilizando directamente `new` o cuando se carga un registro desde la base de datos. Puede ser útil para evitar la necesidad de anular directamente el método de `initialize` de Active Record.

La devolución de llamada `after_find` se llamará siempre que Active Record cargue un registro de la base de datos. `after_find` se llama antes de` after_initialize` si ambos están definidos.

La devolución de llamada `after_initialize` y` after_find` no tienen contrapartidas `before_ *`, pero pueden registrarse como las otras devoluciones de llamadas de Active Record.

```ruby
class User < ApplicationRecord
  after_initialize do |user|
    puts "You have initialized an object!"
  end

  after_find do |user|
    puts "You have found an object!"
  end
end

>> User.new
You have initialized an object!
=> #<User id: nil>

>> User.first
You have found an object!
You have initialized an object!
=> #<User id: 1>
```

### `after_touch`

La devolución de llamada `after_touch` se llamará cada vez que se toque un objeto Active Record.

```ruby
class User < ApplicationRecord
  after_touch do |user|
    puts "You have touched an object"
  end
end

>> u = User.create(name: 'Kuldeep')
=> #<User id: 1, name: "Kuldeep", created_at: "2013-11-25 12:17:49", updated_at: "2013-11-25 12:17:49">

>> u.touch
You have touched an object
=> true
```

It can be used along with `belongs_to`:

```ruby
class Employee < ApplicationRecord
  belongs_to :company, touch: true
  after_touch do
    puts 'An Employee was touched'
  end
end

class Company < ApplicationRecord
  has_many :employees
  after_touch :log_when_employees_or_company_touched

  private
    def log_when_employees_or_company_touched
      puts 'Employee/Company was touched'
    end
end

>> @employee = Employee.last
=> #<Employee id: 1, company_id: 1, created_at: "2013-11-25 17:04:22", updated_at: "2013-11-25 17:05:05">

# triggers @employee.company.touch
>> @employee.touch
An Employee was touched
Employee/Company was touched
=> true
```

Running Callbacks
-----------------
Ejecutar Devoluciones de Llamada

Los siguientes métodos desencadenan las devoluciones de llamada:

* `create`
* `create!`
* `destroy`
* `destroy!`
* `destroy_all`
* `save`
* `save!`
* `save(validate: false)`
* `toggle!`
* `touch`
* `update_attribute`
* `update`
* `update!`
* `valid?`

Además, la devolución de llamada `after_find` se activa mediante los siguientes métodos de búsqueda:

* `all`
* `first`
* `find`
* `find_by`
* `find_by_*`
* `find_by_*!`
* `find_by_sql`
* `last`

La devolución de llamada `after_initialize` se activa cada vez que se inicializa un nuevo objeto de la clase.

NOTA: Los métodos `find_by_*` y `find_by_*!` son buscadores dinámicos generados automáticamente para cada atributo. Obtenga más información sobre ellos en la [Dynamic finders section](active_record_querying.html#dynamic-finders)

Skipping Callbacks
------------------
Saltar Devoluciones de Llamada

Igual que con las validaciones, también es posible omitir devoluciones de llamada utilizando los siguientes métodos:

* `decrement!`
* `decrement_counter`
* `delete`
* `delete_all`
* `increment!`
* `increment_counter`
* `update_column`
* `update_columns`
* `update_all`
* `update_counters`

Sin embargo, estos métodos se deben usarse con precaución, ya que las reglas comerciales importantes y la lógica de la aplicación pueden mantenerse en devoluciones de llamada. Eludirlos sin comprender las posibles implicaciones puede conducir a datos no válidos.

Halting Execution
-----------------
Detener la Ejecución

Cuando comience a registrar nuevas devoluciones de llamada para sus modelos, se pondrán en cola para su ejecución. Esta cola incluirá todas las validaciones de su modelo, las devoluciones de llamada registradas y la operación de la base de datos que se ejecutará.

```ruby
throw :abort
```

ADVERTENCIA. Rails volverá a plantear cualquier excepción que no sea `ActiveRecord::Rollback` o` ActiveRecord::RecordInvalid` después de detener la cadena de devolución de llamada. Elevando una excepción que no sea `ActiveRecord::Rollback` o` ActiveRecord::RecordInvalid` puede romper el código que no espera métodos como `save` y `update` (que normalmente intentan devolver `true` o` false`) una excepción.

```ruby
class User < ApplicationRecord
  has_many :articles, dependent: :destroy
end

class Article < ApplicationRecord
  after_destroy :log_destroy_action

  def log_destroy_action
    puts 'Article destroyed'
  end
end

>> user = User.first
=> #<User id: 1>
>> user.articles.create!
=> #<Article id: 1, user_id: 1>
>> user.destroy
Article destroyed
=> #<User id: 1>
```

Conditional Callbacks
---------------------
Condicionales de Callbacks 

Al igual que con las validaciones, también podemos condicionar la invocación de un método de devolución de llamada a la satisfacción de un predicado dado. Podemos hacer esto usando las opciones `:if` y `:unless`, que pueden tomar un símbolo, un `Proc` o un `Array`. Puede usar la opción `:if` cuando desee especificar en qué condiciones se debe llamar a la devolución de llamada *unless*. Si desea especificar las condiciones bajo las cuales no se debe llamar a la devolución de llamada **should not**, puede usar la opción `:unless`

### Using `:if` and `:unless` with a `Symbol`

Puede asociar las opciones `:if` y `:unlesss` con un símbolo correspondiente al nombre de un método de predicado que se llamará justo antes de la devolución de llamada. Cuando se usa la opción `:if`, la devolución de llamada no se ejecutará si el método de predicado devuelve falso; cuando se usa la opción `:unless`, la devolución de llamada no se ejecutará si el método de predicado devuelve verdadero. Esta es la opción más común. Al usar esta forma de registro, también es posible registrar varios predicados diferentes que se deben llamar para verificar si se debe ejecutar la devolución de llamada.

```ruby
class Order < ApplicationRecord
  before_save :normalize_card_number, if: :paid_with_card?
end
```

### Using `:if` and `:unless` with a `Proc`

Es posible asociar `:if` y `:unless` con un objeto `Proc`. Esta opción es más adecuada cuando se escriben métodos de validación cortos, generalmente de una sola línea:

```ruby
class Order < ApplicationRecord
  before_save :normalize_card_number,
    if: Proc.new { |order| order.paid_with_card? }
end
```

Como el proceso se evalúa en el contexto del objeto, también es posible escribir esto como:

```ruby
class Order < ApplicationRecord
  before_save :normalize_card_number, if: Proc.new { paid_with_card? }
end
```

### Multiple Conditions for Callbacks

Al escribir devoluciones de llamada condicionales, es posible mezclar tanto `:if` como `:unless` en la misma declaración de devolución de llamada:

```ruby
class Comment < ApplicationRecord
  after_create :send_email_to_author, if: :author_wants_emails?,
    unless: Proc.new { |comment| comment.article.ignore_comments? }
end
```

### Combining Callback Conditions

Cuando varias condiciones definen si se debe realizar una devolución de llamada o no, se puede usar una 'Matriz'. Además, puede aplicar `: if` y `:unless` a la misma devolución de llamada.

```ruby
class Comment < ApplicationRecord
  after_create :send_email_to_author,
    if: [Proc.new { |c| c.user.allow_send_email? }, :author_wants_emails?],
    unless: Proc.new { |c| c.article.ignore_comments? }
end
```

La devolución de llamada solo se ejecuta cuando todas las condiciones `:if` y ninguna de las condiciones`:unless` se evalúan como `true`.

Callback Classes
----------------
Clases de Devolución de Llamada

A veces, los métodos de devolución de llamada que escribirá serán lo suficientemente útiles como para ser reutilizados por otros modelos. Active Record hace posible crear clases que encapsulan los métodos de devolución de llamada, para que puedan reutilizarse.

Aquí hay un ejemplo en el que creamos una clase con una devolución de llamada `after_destroy` para un modelo` PictureFile`:

```ruby
class PictureFileCallbacks
  def after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end
```

Cuando se declara dentro de una clase, como se indicó anteriormente, los métodos de devolución de llamada recibirán el objeto modelo como un parámetro. Ahora podemos usar la clase de devolución de llamada en el modelo:

```ruby
class PictureFile < ApplicationRecord
  after_destroy PictureFileCallbacks.new
end
```

Tenga en cuenta que necesitábamos crear una instancia de un nuevo objeto `PictureFileCallbacks`, ya que declaramos nuestra devolución de llamada como método de instancia. Esto es particularmente útil si las devoluciones de llamada hacen uso del estado del objeto instanciado. Sin embargo, a menudo tendrá más sentido declarar las devoluciones de llamada como métodos de clase:

```ruby
class PictureFileCallbacks
  def self.after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end
```

Si el método de devolución de llamada se declara de esta manera, no será necesario instanciar un objeto `PictureFileCallbacks`.

```ruby
class PictureFile < ApplicationRecord
  after_destroy PictureFileCallbacks
end
```

Puede declarar tantas devoluciones de llamada como desee dentro de sus clases de devolución de llamada.

Transaction Callbacks
---------------------
Callbacks de transacciones

Hay dos devoluciones de llamada adicionales que se activan al completar una transacción de base de datos: `after_commit` y `after_rollback`. Estas devoluciones de llamada son muy similares a la devolución de llamada `after_save`, excepto que no se ejecutan hasta después de que los cambios en la base de datos se hayan confirmado o revertido. Son más útiles cuando sus modelos de registros activos necesitan interactuar con sistemas externos que no forman parte de la transacción de la base de datos.

Considere, por ejemplo, el ejemplo anterior en el que el modelo `PictureFile` necesita eliminar un archivo después de que se destruya el registro correspondiente. Si algo genera una excepción después de que se llama la devolución de llamada `after_destroy` y la transacción se revierte, el archivo se habrá eliminado y el modelo quedará en un estado inconsistente. Por ejemplo, suponga que `picture_file_2` en el código a continuación no es válido y que el método `save!` genera un error.

```ruby
PictureFile.transaction do
  picture_file_1.destroy
  picture_file_2.save!
end
```

Al usar la devolución de llamada `after_commit` podemos dar cuenta de este caso.

```ruby
class PictureFile < ApplicationRecord
  after_commit :delete_picture_file_from_disk, on: :destroy

  def delete_picture_file_from_disk
    if File.exist?(filepath)
      File.delete(filepath)
    end
  end
end
```

NOTA: La opción `:on` especifica cuándo se activará una devolución de llamada. Si tu
no proporcione la opción `:on`, la devolución de llamada se activará para cada acción.

Dado que el uso de la devolución de llamada `after_commit` solo en crear, actualizar o eliminar es
común, hay alias para esas operaciones:

* `after_create_commit`
* `after_update_commit`
* `after_destroy_commit`

```ruby
class PictureFile < ApplicationRecord
  after_destroy_commit :delete_picture_file_from_disk

  def delete_picture_file_from_disk
    if File.exist?(filepath)
      File.delete(filepath)
    end
  end
end
```

ADVERTENCIA. Cuando se completa una transacción, se llaman las devoluciones de llamada `after_commit` o` after_rollback` para todos los modelos creados, actualizados o destruidos dentro de esa transacción. Sin embargo, si se genera una excepción dentro de una de estas devoluciones de llamada, la excepción se disparará y los métodos restantes `after_commit` o` after_rollback` no se ejecutarán. Como tal, si su código de devolución de llamada podría generar una excepción, deberá rescatarlo y manejarlo dentro de la devolución de llamada para permitir que se ejecuten otras devoluciones de llamada.

ADVERTENCIA. El código ejecutado dentro de las devoluciones de llamada `after_commit` o `after_rollback` no están incluido en una transacción.

ADVERTENCIA. El uso de `after_create_commit` y `after_update_commit` en el mismo modelo solo permitirá que la última devolución de la llamada definida surta efecto y anulará todas las demás.

```ruby
class User < ApplicationRecord
  after_create_commit :log_user_saved_to_db
  after_update_commit :log_user_saved_to_db

  private
  def log_user_saved_to_db
    puts 'User was saved to database'
  end
end

# prints nothing
>> @user = User.create

# updating @user
>> @user.save
=> User was saved to database
```

También hay un alias para usar la devolución de llamada `after_commit` para crear y actualizar juntos:

* `after_save_commit`

```ruby
class User < ApplicationRecord
  after_save_commit :log_user_saved_to_db

  private
  def log_user_saved_to_db
    puts 'User was saved to database'
  end
end

# creating a User
>> @user = User.create
=> User was saved to database

# updating @user
>> @user.save
=> User was saved to database
```
