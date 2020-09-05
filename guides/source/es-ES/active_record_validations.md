**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Validaciones de Active Record
=========================
Resumen de Validaciones

Esta guía le enseña cómo validar el estado de los objetos antes de que entren
en la base de datos utilizando la característica de validación de Active Record.

Después de leer esta guía, sabrá:

* Cómo usar los ayudantes(helpers) de validación de Active Record.
* Cómo crear tus propios métodos de validación.
* Cómo trabajar con los mensajes de error generados por el proceso de validación.

--------------------------------------------------------------------------------

Validations Overview
--------------------
Resumen de Validaciones

Aquí hay un ejemplo simple de una validación:

```ruby
class Person < ApplicationRecord
  validates :name, presence: true
end

Person.create(name: "John Doe").valid? # => true
Person.create(name: nil).valid? # => false
```

Como puedes ver, nuestra validación nos permite saber que nuestra `Person` no es válida
sin un atributo `name`. La segunda `Person` no será persistente sobre la
base de datos.

Antes de entrar en más detalles, hablemos sobre cómo las validaciones quedan en
panorama general de su aplicación.

### Why Use Validations?
¿Por qué Utilizamos Validaciones?

Las validaciones se utilizan para garantizar que solo se guarden datos válidos en su
base de datos. Por ejemplo, puede ser importante para su aplicación asegurarse de que
cada usuario proporciona una dirección de correo electrónico y una dirección postal. 
Las validaciones del nivel de modelo (Model-level) son la mejor manera de garantizar
que solo se guarden datos válidos en su base de datos. Son independientes de la base de datos,
no pueden ser ignorados por los usuarios finales y son conveniente para probar y mantener.
Rails incorpora ayuda en helpers para necesidades comúnes
, y también le permite crear sus propios métodos de validación .

Hay otras formas de validar los datos antes de guardarlos en su
base de datos, incluidas restricciones nativas de bases de datos, validaciones del lado del cliente y
validaciones a nivel de controlador. Aquí hay un resumen de los pros y los contras:

* Restricciones de la base de datos, y/o procedimientos almacenados que hacen el mecanismo de validación 
  dependiente del motor de la base de datos y pueden hacer las pruebas y el mantenimiento más dificil. 
  Sin embargo, si tu base de datos es utilizada por otras aplicaciones, puede ser una buena idea utilizar 
  algunas restricciones en el nivel de base de datos. Adicionalmente, las validaciones en el nivel de base
  de datos, pueden mantener la seguridad en algunas cosas (como la unicidad
  en tablas muy utilizadas) que son difíciles de implementar de otra manera.
* Las validaciones del lado del cliente suelen ser utilizadas, pero son generalmente poco confiables si se 
  utilizan solas. Si son implementadas utilizando JavaScript, pueden ser sobrepasadas si el JavaScript 
  está quitado en el navegador del usuario. Sin embargo, si se combinan con otras técnicas, las validaciones
 del lado del cliente pueden ser una manera conveniente de proporcionar una respuesta inmediata a sus usuarios.
* Las validaciones a nivel de controlador pueden ser tentadoras de usar, pero a menudo se vuelven
  difícil de manejar y difícil de probar y mantener. Siempre que sea posible, es un buen
  idea para mantener sus controladores delgados, ya que hará que su aplicación sea
  placer de trabajar a largo plazo.

Elija estos en ciertos casos específicos. Es la opinión del equipo de Rails.
que las validaciones a nivel de modelo son las más apropiadas en la mayoría de las circunstancias.

### When Does Validation Happen?
¿Cuándo sucede la validación?

Hay dos clases de objetos Active Record: aquellos que se corresponden a un registro en la base de datos y los que no.
Cuando crea un objeto nuevo, por ejemplo usando el método `new`, ese objeto todavía no pertenece a la base de datos.
Una vez que se llame `save` sobre ese objeto, se guardará en la tabla de base de datos adecuada.
Active Record utiliza el método de instancia new_record? para determinar si un objeto está ya en la base de datos o no.
Considera la siguiente clase Active Record:

```ruby
class Person < ApplicationRecord
end
```

Podemos ver cómo funciona mirando algunos resultados de  `bin/rails console`:

```ruby
$ bin/rails console
>> p = Person.new(name: "John Doe")
=> #<Person id: nil, name: "John Doe", created_at: nil, updated_at: nil>
>> p.new_record?
=> true
>> p.save
=> true
>> p.new_record?
=> false
```

Crear y guardar un nuevo registro enviará una operación SQL `INSERT` a la
base de datos. La actualización de un registro que existente enviará una operación SQL `UPDATE`.
Las validaciones generalmente se ejecutan antes de que estos comandos se envíen a
base de datos. Si una validación falla, el objeto se marcará como inválido y
Active Record no realizará la operación `INSERT` o `UPDATE`. Esto evita
almacenar un objeto que no es válido en la base de datos.
Puedes elegir que validaciones específicas sean ejecutadas cuando se
crea, guarda o actualiza un objeto.

PRECAUCIÓN: Hay muchas formas de cambiar el estado de un objeto en la base de datos.
Algunos métodos activarán validaciones, pero otros no. Esto significa que es
es posible guardar un objeto en la base de datos en un estado no válido si uno
no tiene cuidado.

Los siguientes métodos disparan validaciones, y guardarán un objeto en la 
base de datos solo si el objeto es válido:

* `create`
* `create!`
* `save`
* `save!`
* `update`
* `update!`

Las versiones de bang (p.e. `save!`) generan una excepción si el 
registro no es válido. Las versiones no-bang `save` y `update` devuelve 
`falso`, y `create` devuelve el objeto.

### Skipping Validations
Saltando las Validaciones

Los siguientes métodos omiten las validaciones y guardarán el objeto en la
base de datos sin darle importancia a su validez. Deben usarse con precaución.

* `decrement!`
* `decrement_counter`
* `increment!`
* `increment_counter`
* `toggle!`
* `touch`
* `update_all`
* `update_attribute`
* `update_column`
* `update_columns`
* `update_counters`

 Nota que `save` también tiene la capacidad de omitir validaciones si se pasa `validate:
false` como un argumento. Esta técnica debe usarse con precaución.

* `save(validate: false)`

### `valid?` and `invalid?`

Antes de guardar un objeto Active Record, Rails ejecuta sus validaciones.
Si estas validaciones producen algún error, Rails no guarda el objeto.

También puede ejecutar estas validaciones por su propia cuenta. `valid?` activa sus validaciones
y devuelve verdadero si no se encontraron errores en el objeto, y falso de lo contrario.
Como viste arriba:

```ruby
class Person < ApplicationRecord
  validates :name, presence: true
end

Person.create(name: "John Doe").valid? # => true
Person.create(name: nil).valid? # => false
```

Después de que Active Record haya realizado las validaciones, se puede acceder cualquier error
encontrado a través del método de instancia `errors`, que devuelve una colección de errores.
Por definición, un objeto es válido si esta colección está vacía después de ejecutarse
validaciones

Nota que un objeto instanciado con `new` no informará errores
incluso si es técnicamente inválido, porque las validaciones se ejecutan automáticamente
solo cuando se guarda el objeto, tal como con los métodos `create` o` save`.

```ruby
class Person < ApplicationRecord
  validates :name, presence: true
end

>> p = Person.new
# => #<Person id: nil, name: nil>
>> p.errors.size
# => 0

>> p.valid?
# => false
>> p.errors.objects.first.full_message
# => "Name can't be blank"

>> p = Person.create
# => #<Person id: nil, name: nil>
>> p.errors.objects.first.full_message
# => "Name can't be blank"

>> p.save
# => false

>> p.save!
# => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank

>> Person.create!
# => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank
```

`inválido?` es el inverso de `válido?`. Activa sus validaciones,
devuelve verdadero si se encontraron errores en el objeto, y falso de
lo contrario.

### `errors[]`

Para verificar si un atributo particular de un objeto es válido o no, puede
use `errors[:attribute]`. Devuelve una matriz de todos los mensajes de error para
`:attribute`. Si no hay errores en el atributo especificado, se regresa una matriz vacía.

Este método solo es útil _después_ de haberse ejecutado las validaciones, porque solo
inspecciona la recopilación de errores y no activa validaciones en sí mismo. Es
diferente del método `ActiveRecord::Base#invalid?`  que fue explicado anteriormente porque
no verifica la validez del objeto. Solo comprueba para ver
si hay errores que fueron encontrados en un atributo individual del objeto.

```ruby
class Person < ApplicationRecord
  validates :name, presence: true
end

>> Person.new.errors[:name].any? # => false
>> Person.create.errors[:name].any? # => true
```

Cubriremos errores de validación en mayor profundidad en la sección [Working with Validation
Errors](#working-with-validation-errors)

Validation Helpers
------------------
Helpers de Validación

Active Record ofrece muchos ayudantes de validación predefinidos que se pueden usar
directamente dentro de las definiciones de tus clases. Estos ayudantes proveen reglas
comúnes de validación. Cada vez que una validación falla, un error se agrega a la
colección de `errors` del objeto, y esto está asociado con el atributo que se esta validado.

Cada ayudante acepta un número arbitrario de nombres de atributos, entonces 
con una línea de código puedes añadir algún tipo de validadación a varios atributos.

Todos ellos aceptan las opciones `:on` y`:message`, que definen cuándo
se debe ejecutar la validación y qué mensaje se debe agregar a la colección de `errors`
si falla, respectivamente. La opción de `: on` toma uno de los valores
`:create` o`:update`. Hay un  mensaje de error por defecto
para cada uno de los ayudantes de validación. Estos mensajes se utilizan cuando
la opción `:mensaje` no está especificada. Echemos un vistazo a cada uno de los
Ayudantes disponibles. Vamos a ver a cada uno de los helpers disponibles.

### `acceptance`

Este método valida que se haya marcado una casilla de verificación en la interfaz de 
usuario cuando se envie el formulario. Esto normalmente se usa cuando el usuario 
necesita aceptar términos de servicio de la aplicación, confirme que se lee algún texto
o un concepto similar.

```ruby
class Person < ApplicationRecord
  validates :terms_of_service, acceptance: true
end
```

Esta verificación se realiza solo si `terms_of_service` no es `nil`.
El mensaje de error predeterminado para este helper es _"must be accepted"_.
También se puede pasar un mensaje personalizado a través de la opción ``message`.

```ruby
class Person < ApplicationRecord
  validates :terms_of_service, acceptance: { message: 'must be abided' }
end
```

También se puede recibir una opción `:accept`, que determina los valores permitidos
que serán considerado como aceptado. Su valor predeterminado es `['1', true]` y puede ser
cambiado fácilmente.

```ruby
class Person < ApplicationRecord
  validates :terms_of_service, acceptance: { accept: 'yes' }
  validates :eula, acceptance: { accept: ['TRUE', 'accepted'] }
end
```

Esta validación es específicamente para aplicaciones del web y este
`acceptance` no necesita registrarse en ningún lugar de su base de datos.
Si tu no tienes un campo para el, el ayudante creará un atributo virtual.
Si el campo si existe en su base de datos, la opción `accept` debe establecerse en
o incluir `true` o de lo contrario la validación no se ejecutará.

### `validates_associated`

Debe usar este asistente cuando su modelo tenga asociaciones con otros modelos
y también se necesitan ser validados. Cuando intentas guardar tu objeto, `valid`
se llamará a cada uno de los objetos asociados.

```ruby
class Library < ApplicationRecord
  has_many :books
  validates_associated :books
end
```

Esta validación funcionará con todos los tipos de asociación.

PRECAUCIÓN: No use `validates_associated` en ambos extremos de sus asociaciones.
Se llamarían entre sí en un bucle infinito.

El mensaje de error predeterminado para `validates_associated` es _"is invalid"_. Nota
que cada objeto asociado contendrá su propia colección de "`errors`; los errores 
no burbujean hasta el modelo que lo llama.

### `confirmation`

Debería usar este asistente cuando tenga dos campos de texto que deberían recibir
exactamente el mismo contenido. Por ejemplo, es posible que desee confirmar una 
dirección de correo electrónico o una contraseña. Esta validación crea un atributo
virtual cuyo nombre es el nombre del campo que debe confirmarse adjuntando
con "_confirmación" .

```ruby
class Person < ApplicationRecord
  validates :email, confirmation: true
end
```
En su plantilla de vista, podría usar algo como

```erb
<%= text_field :person, :email %>
<%= text_field :person, :email_confirmation %>
```

Esta verificación se realiza solo si `email_confirmation` no es `nil`. Para requerir
confirmación, asegúrese de agregar una verificación de presencia para el atributo de confirmación
(veremos `presence` más adelante en esta guía):

```ruby
class Person < ApplicationRecord
  validates :email, confirmation: true
  validates :email_confirmation, presence: true
end
```

También hay una opción `:case_sensitive` que puede usar para definir si
la restricción de confirmación se distingue entre mayúsculas y minúsculas o no. Esta opción
por defecto es cierto.

```ruby
class Person < ApplicationRecord
  validates :email, confirmation: { case_sensitive: false }
end
```

El mensaje de error predeterminado para este asistente es _"doesn't match confirmation"_.

### `exclusion`

Este helper valida que los valores de los atributos no estén incluidos en un
conjunto determinado. De hecho, este conjunto puede ser cualquier objeto enumerable.

```ruby
class Account < ApplicationRecord
  validates :subdomain, exclusion: { in: %w(www us ca jp),
    message: "%{value} is reserved." }
end
```

Este helper `exclusion` tiene una opción `:in` que recibe que recibe el conjunto de valores que
no serán aceptado para los atributos validados.  La opción :`in` tiene un alias llamado `:within`
que puedes utilizar para el mismo propósito, si lo quieres. Este ejemplo utiliza la opción
`:message` para demostrar como puedes incluir los valores de los atributos. Para ver las opciones 
completas del argumento del mensaje, consulte el [message documentation](#message).

El mensaje de error predeterminado es _"is reserved"_.

### `format`

Este ayudante valida los valores de los atributos probando si coinciden con un
dada la expresión regular, que se especifica usando la opción `: with`.

```ruby
class Product < ApplicationRecord
  validates :legacy_code, format: { with: /\A[a-zA-Z]+\z/,
    message: "only allows letters" }
end
```

Alternativamente, puedes requerir que el atributo específico no responda a la expresión regular utilizando la opción `:without`.

El mensaje de error por defecto es _"is invalid"_.

### `inclusion`

Este ayudante valida que los valores de los atributos están incluidos en un conjunto dado.
De hecho, este conjunto puede ser cualquier objeto enumerable.

```ruby
class Coffee < ApplicationRecord
  validates :size, inclusion: { in: %w(small medium large),
    message: "%{value} is not a valid size" }
end
```

Este helper `inclusion` tiene una opción `:in` que recibe que recibe el conjunto de valores que
serán aceptado. La opción :`in` tiene un alias llamado `:within` que puedes utilizar para 
el mismo propósito, si lo quieres. El ejemplo anterior usa la opción `:message` para mostrar 
cómo puede incluir el valor del atributo. Para ver las opciones completas del argumento 
del mensaje, consulte el [message documentation](#message).

El mensaje de error predeterminado es _"is not included in the list"_.

### `length`

Este ayudante valida la longitud de los valores de los atributos. Proporciona una
variedad de opciones, por lo que puede especificar restricciones de longitud de diferentes maneras:

```ruby
class Person < ApplicationRecord
  validates :name, length: { minimum: 2 }
  validates :bio, length: { maximum: 500 }
  validates :password, length: { in: 6..20 }
  validates :registration_number, length: { is: 6 }
end
```

Las opciones de restricción de longitud son:

* `:minimum` - El atributo no puede tener menos caracteres que una longitud específica.
* `:maximum` - El atributo no puede tener más caracteres que una longitud específica.
* `:in` (or `:within`) - La longitud del atributo debe estar contenida en un intervalo dado. 
El valor para esta opción de ser un rango.
* `:is` -  La longitud del atributo debe ser igual a un valor dado.

Los mensajes de error por defecto dependen del tipo de validación de longitud que es utilizada. 
Puedes personalizar estos mensajes utilizando las opciones `:wrong_length`, `:too_long`, y
`:too_short` y `%{count}` como referencia para mostrar el número que corresponda para 
la restricción que se está utilizando. Todavía puedes usar la opción `:message` para especificar 
un mensaje de error.
 
```ruby
class Person < ApplicationRecord
  validates :bio, length: { maximum: 1000,
    too_long: "%{count} characters is the maximum allowed" }
end
```

Nota que los mensajes de error predeterminados son plurales. Por predeterminado, coincidirá con 
un signo opcional seguido de una integral o número de punto flotante.

 Para especificar que solo se permiten números integrales, establezca `:only_integer` 
 en verdadero. Entonces usará la     

```ruby
/\A[+-]?\d+\z/
```

expresión regular para validar el valor del atributo. De lo contrario, intentará
convertir el valor a un número usando `Float`. Los `Float` se convierten en` BigDecimal`
utilizando el valor de precisión de la columna o 15.

```ruby
class Player < ApplicationRecord
  validates :points, numericality: true
  validates :games_played, numericality: { only_integer: true }
end
```

El mensaje de error predeterminado es  _"must be an integer"_.

Además :only_integer, este helper también acepta las siguientes opciones para 
añadir restricciones a los valores acceptables:

* `:greater_than` - Especifica que el valor del atributo debe debe ser mayor que
 un valor determinado. El mensaje de error para esta opción es  _"must be greater than
  %{count}"_.
* `:greater_than_or_equal_to` - Especifica que el valor del atributo debe ser mayor o 
igual a un valor determinado. El mensaje de error por defecto para esta opción es
  _"must be greater than or equal to %{count}"_.
* `:equal_to` - Especifica que el valor del atributo debe ser igual a un valor 
determinado. El mensaje de error por defecto de esta opción es _"must be equal to %{count}"_.
* `:less_than` - Especifica que el valor del atributo debe ser menor que un determinado valor.
 El mensaje de error de esta opción ess _"must be less than %{count}"_.
* `:less_than_or_equal_to` - Especifica que el valor del atributo debe ser menor o igual 
de un valor determinado. El mensaje de error por defecto es _"must be
  less than or equal to %{count}"_.
* `:other_than` - Especifica que el valor debe ser distinto del valor proporcionado. El 
mensaje de error por defecto es _"must be other than %{count}"_.
* `:odd` -  Especifica que el valor debe ser un número impar si es configurado a true. El 
mensaje de error por defecto es _"must be odd"_.
* `:even` - especifica que el valor debe ser un número par si se establece a true. El 
 mensaje de error por defecto es _"must be even"_.

NOTE: Por defecto, `numericality` no permite valores `nil`. Puedes utilizar la opción`allow_nil: true` para permitirlo.

El mensaje de error para esta opción es _"is not a number"_.

### `presence`

Este ayudante valida que los atributos especificados no estén vacíos. Utiliza el método `blank?` para comprobar
si el valor es `nil` o una cadena esta en blancoese, es decir, una cadena que está vacía o que consta de 
espacios en blanco.

```ruby
class Person < ApplicationRecord
  validates :name, :login, :email, presence: true
end
```

Si quieres estar seguro de que hay una asociación presente, tendras que probar
si el objeto asociado en sí está presente, y no la clave foránea utilizada para asignar la asociación.
De esta manera, no solo se verifica que la clave foránea no este vacía sino que también 
el objeto referenciado exista.

```ruby
class LineItem < ApplicationRecord
  belongs_to :order
  validates :order, presence: true
end
```

Para validar qu los registros asociados cuya presencia se requiere, debe
especifique la opción `:inverse_of` para la asociación:

```ruby
class Order < ApplicationRecord
  has_many :line_items, inverse_of: :order
end
```

Para validar los registros asociados cuya presencia se requiere, debe
especifica la opción `:inverse_of` para la asociación:

```ruby
class Order < ApplicationRecord
  has_many :line_items, inverse_of: :order
end
```

Si validas la presencia de un objeto asociado a través de una relación `has_one`
o `has_many`, comprobará que el objeto no este en `blank?` ni `marked_for_destruction?`.

Como `false.blank?` es verdadero, si quieres validar la presencia de un booleano
debes usar una de las siguientes validaciones:

```ruby
validates :boolean_field_name, inclusion: { in: [true, false] }
validates :boolean_field_name, exclusion: { in: [nil] }
```

Al utilizar una de estas validaciones, se asegurará de que el valor NO sea `nil`
daría como resultado un valor `NULL` en la mayoría de los casos.

### `absence`

Este ayudante valida que los atributos especificados están ausentes. Utiliza el
método `present?` para verificar si el valor no es `nil` o una cadena en blanco, que
es decir, una cadena que está vacía o que consta de espacios en blanco.

```ruby
class Person < ApplicationRecord
  validates :name, :login, :email, absence: true
end
```

Si desea estar seguro de que una asociación está ausente, deberá probar si el objeto
asociado está ausente y no la clave foránea utilizada para mapear la asociación.

```ruby
class LineItem < ApplicationRecord
  belongs_to :order
  validates :order, absence: true
end
```

Para validar los registros asociados cuyo absence es necesario, debe especificar 
la opción `:inverse_of` para la asociación:

```ruby
class Order < ApplicationRecord
  has_many :line_items, inverse_of: :order
end
```

Si valida la ausencia de un objeto asociado a través de una relación `has_one` o 
`has_many`, comprobará que el objeto no está `present?` ni `marked_for_destruction?`.

Como `false.present?` es falso, si quieres validar la presencia de un booleano
debes usar `validates :field_name, exclusion: { in: [true, false] }`.

El mensaje de error predeterminado es  _"must be blank"_.

### `uniqueness`

Este asistente valida que el valor del atributo es único justo antes de
que el objeto se guarda. No crea una restricción de unicidad en la base de datos,
por lo tanto, puede suceder que dos conexiones de base de datos diferentes crean dos registros
con el mismo valor para una columna que pretende ser única. Para evitar eso,
debe crear un índice único en esa columna en su base de datos.

```ruby
class Account < ApplicationRecord
  validates :email, uniqueness: true
end
```

La validación se realiza realizando una consulta SQL en la tabla del modelo,
buscando un registro existente con el mismo valor en ese atributo.

Hay una opción de `:scope` que puede utilizar para especificar otros atributos que se utilizan
para limitar la comprobación de unicidad:

```ruby
class Holiday < ApplicationRecord
  validates :name, uniqueness: { scope: :year,
    message: "should happen once per year" }
end
```

Si desea crear una restricción de la base de datos para evitar posibles violaciones de una validación de unicidad utilizando la opción `:scope`, debe crear un índice único en ambas columnas de su base de datos. Consulte [the MySQL manual](https://dev.mysql.com/doc/refman/en/multiple-column-indexes.html) para obtener más detalles sobre los índices de columnas múltiples  o [the PostgreSQL manual](https://www.postgresql.org/docs/current/static/ddl-constraints.html) para ver ejemplos de restricciones únicas que se refieren a un grupo de columnas.

También hay una opción `:case_sensitive` que puede utilizar para definir si la restricción de unicidad
será sensible a mayúsculas o minúsculas. Esta opción predeterminada es true.

```ruby
class Person < ApplicationRecord
  validates :name, uniqueness: { case_sensitive: false }
end
```

ADVERTENCIA. Tenga en cuenta que algunas bases de datos están configuradas para realizar búsquedas sin distinción 
con mayúsculas y minúsculas.

El mensaje de error predeterminado es  _"has already been taken"_.

### `validates_with`

Este ayudante pasa el registro a una clase separada para validación.

```ruby
class GoodnessValidator < ActiveModel::Validator
  def validate(record)
    if record.first_name == "Evil"
      record.errors.add :base, "This person is evil"
    end
  end
end

class Person < ApplicationRecord
  validates_with GoodnessValidator
end
```

Nota: Los errores agregados a `record.errors [:base]` se relacionan con el estado del registro
en conjunto, y no a un atributo específico.

El ayudante `validates_with` toma una clase, o una lista de clases para utilizar para esa validación.
No hay un mensaje de error predeterminado para `validates_with`. Debe agregar manualmente los errores
a la colección de errores del registro en la clase validator.

Para implementar el método validate, debe tener definido un parámetro `record,` que es el 
registro a validar.

Como todas las demás validaciones, `validates_with` toma las opciones `:if` ,`:unless` y `:on`. 
Si pasa otras opciones, se enviarán las opciones a la clase validator como `options`:

```ruby
class GoodnessValidator < ActiveModel::Validator
  def validate(record)
    if options[:fields].any?{|field| record.send(field) == "Evil" }
      record.errors.add :base, "This person is evil"
    end
  end
end

class Person < ApplicationRecord
  validates_with GoodnessValidator, fields: [:first_name, :last_name]
end
```

Nota que el validador se inicializará *sólo una vez* para todo el ciclo de vida de la aplicación, 
y no en cada ejecución de validación, así que tenga cuidado al usar variables de instancia dentro de ella.

Si su validador es lo suficientemente complejo como para que desee variables de instancia, puede utilizar 
fácilmente un objeto Ruby antiguo en su lugar:

```ruby
class Person < ApplicationRecord
  validate do |person|
    GoodnessValidator.new(person).validate
  end
end

class GoodnessValidator
  def initialize(person)
    @person = person
  end

  def validate
    if some_complex_condition_involving_ivars_and_private_methods?
      @person.errors.add :base, "This person is evil"
    end
  end

  # ...
end
```

Este helper valida los atributos contra un bloque. No tiene una función de validación
predefinida. Debes crear uno usando un bloque, y cada atributo es
pasado a `validates_each` y será probado contra él. En el siguiente ejemplo,
no queremos que los nombres y apellidos comiencen con minúsculas.

```ruby
class Person < ApplicationRecord
  validates_each :name, :surname do |record, attr, value|
    record.errors.add(attr, 'must start with upper case') if value =~ /\A[[:lower:]]/
  end
end
```

El bloque recibe el registro, el nombre del atributo y el valor del atributo.
Puedes hacer lo que quieras para verificar los datos válidos dentro del bloque. Si tu
validación falla, debrias agregar un error al modelo, por lo tanto
haciéndolo inválido.

Common Validation Options
-------------------------
Opciones de validación comunes

Estas son opciones de validación comunes:

### `:allow_nil`

La `:allow_nil`  opción omite la validación cuando el valor que se valida es
`nil`.

```ruby
class Coffee < ApplicationRecord
  validates :size, inclusion: { in: %w(small medium large),
    message: "%{value} is not a valid size" }, allow_nil: true
end
```

Para ver las opciones completas del argumento del mensaje, consulte el
[message documentation](#message).

### `:allow_blank`

La `:allow_blank` opción es similar a la opción `:allow_nil`. Esta opción
dejará pasar la validación si el valor del atributo es `blank?`, como `nil` o por ejemplo
una cadena vacía. 

```ruby
class Topic < ApplicationRecord
  validates :title, length: { is: 5 }, allow_blank: true
end

Topic.create(title: "").valid?  # => true
Topic.create(title: nil).valid? # => true
```

### `:message`

Como ya has visto, la opción `:message` te permite especificar el mensaje que
se agregará a la colección de `errors` cuando falla la validación. Cuando esta
opción no se usa, Active Record usará el respectivo mensaje de error predeterminado 
para cada ayudante de validación. La opción `:message` acepta un `String` o `Proc`.

Un valor `String` `:message` opcionalmente puede contener cualquiera/todos del `%{value}`,
`%{attribute}`, y `%{model}` que serán reemplazados dinámicamente cuando
la validación falla. Este reemplazo se realiza utilizando la gem I18n, y los 
marcadores de posición deben coincidir exactamente, no se permiten espacios.

Un valor `Proc`: `mensaje` tiene dos argumentos: el objeto que se esta validando y
un hash con los pares clave-valor `:modelo`,`:atributo` y `:valor`.

```ruby
class Person < ApplicationRecord
  # Hard-coded message
  validates :name, presence: { message: "must be given please" }

  # Message with dynamic attribute value. %{value} will be replaced with
  # the actual value of the attribute. %{attribute} and %{model} also
  # available.
  validates :age, numericality: { message: "%{value} seems wrong" }

  # Proc
  validates :username,
    uniqueness: {
      # object = person object being validated
      # data = { model: "Person", attribute: "Username", value: <username> }
      message: ->(object, data) do
        "Hey #{object.name}!, #{data[:value]} is taken already! Try again #{Time.zone.tomorrow}"
      end
    }
end
```

### `:on`

La opción `:on` le permite especificar cuándo se debe realizarse la validación. El 
comportamiento predeterminado para todos los ayudantes de validación integrados se 
debe ejecutar al guardar (save) (tanto cuando está creando un nuevo registro como 
cuando lo está actualizando). Si tu deseas cambiarlo, puedes usar `on: :create` para ejecutar
la validación solo cuando un nuevo registro es creado o `on: :update` para ejecutar 
la validación solo cuando un registro se está actualizado.

```ruby
class Person < ApplicationRecord
  # it will be possible to update email with a duplicated value
  validates :email, uniqueness: true, on: :create

  # it will be possible to create the record with a non-numerical age
  validates :age, numericality: true, on: :update

  # the default (validates on both create and update)
  validates :name, presence: true
end
```

También se puede usar `on:` para definir contextos personalizados. Los contextos 
personalizados deben ser se activados explícitamente pasando el nombre del contexto 
a `valid?`, `invalid?`, or `save`.

```ruby
class Person < ApplicationRecord
  validates :email, uniqueness: true, on: :account_setup
  validates :age, numericality: true, on: :account_setup
end

person = Person.new(age: 'thirty-three')
person.valid? # => true
person.valid?(:account_setup) # => false
person.errors.messages
 # => {:email=>["has already been taken"], :age=>["is not a number"]}
```

`person.valid?(:account_setup)` ejecuta ambas validaciones sin guardar
el modelo. `person.save(context: :account_setup)` valida `person` en el contexto
`account_setup` antes de guardar.

Cuando se desencadena por un contexto explícito, las validaciones se ejecutan para ese contexto,
así como cualquier validación  _without_ contexto.

```ruby
class Person < ApplicationRecord
  validates :email, uniqueness: true, on: :account_setup
  validates :age, numericality: true, on: :account_setup
  validates :name, presence: true
end

person = Person.new
person.valid?(:account_setup) # => false
person.errors.messages
 # => {:email=>["has already been taken"], :age=>["is not a number"], :name=>["can't be blank"]}
```

Strict Validations
------------------
Validaciones estrictas

También se puede especificar validaciones para que sean estrictas y elevar
`ActiveModel::StrictValidationFailed` cuando el objeto no es válido.

```ruby
class Person < ApplicationRecord
  validates :name, presence: { strict: true }
end

Person.new.valid?  # => ActiveModel::StrictValidationFailed: Name can't be blank
```

También existe la posibilidad de pasar una excepción personalizada a la opción `:strict`.

```ruby
class Person < ApplicationRecord
  validates :token, presence: true, uniqueness: true, strict: TokenGenerationException
end

Person.new.valid?  # => TokenGenerationException: Token can't be blank
```

Conditional Validation
----------------------
Validación Condicional

A veces tendrá sentido validar un objeto solo cuando un predicado dado
es satisfecho. Puedes hacerlo utilizando las opciones `:if` y `:unless`, que
pueden tomar un símbolo, un `Proc` o un` Array`. Puede usar la opción `:if`
cuando desee especificar cuándo la validación **should** debe ocurrir. Si tu
deseas especificar cuándo la validación **should not** no deberia ocurrir, 
entoncess puede usar la opción `:unless`.

### Using a Symbol with `:if` and `:unless`

Puedes asociar las opciones `:if` y `:unless` con un símbolo correspondiente
al nombre del método que se llamará justo antes de que ocurra la validación.
Esta es la opción más utilizada.

```ruby
class Order < ApplicationRecord
  validates :card_number, presence: true, if: :paid_with_card?

  def paid_with_card?
    payment_type == "card"
  end
end
```

### Using a Proc with `:if` and `:unless`

Es posible asociar `:if` y`:unless` con un objeto `Proc` que sera llamado. 
El uso de un objeto `Proc` le da la capacidad de escribir una condición en 
línea en lugar de un método separado. Esta opción es la más adecuada para
una línea.

```ruby
class Account < ApplicationRecord
  validates :password, confirmation: true,
    unless: Proc.new { |a| a.password.blank? }
end
```

Como las `Lambdas` son un tipo de` Proc`, también se pueden usar para escribir en línea
condiciones de una manera más corta.

```ruby
validates :password, confirmation: true, unless: -> { password.blank? }
```

### Grouping Conditional validations
Agrupación de validaciones condicionales

A veces es útil que varias validaciones usen una condición. Se puede lograr 
fácilmente usando `with_options`.

```ruby
class User < ApplicationRecord
  with_options if: :is_admin? do |admin|
    admin.validates :password, length: { minimum: 10 }
    admin.validates :email, presence: true
  end
end
```

Todas las validaciones dentro del bloque `with_options` tendrán automáticamente
pasó la condición `if::is_admin?`

### Combining Validation Conditions
Combinando condiciones de validación

Por otro lado, cuando  condiciones múltiples definen si una validación debe o no
debe suceder, se puede usar un `Array`. Además, puedes aplicar los dos `:if` como
`:unless` a la misma validación

```ruby
class Computer < ApplicationRecord
  validates :mouse, presence: true,
                    if: [Proc.new { |c| c.market.retail? }, :desktop?],
                    unless: Proc.new { |c| c.trackpad.present? }
end
```

La validación solo se ejecuta cuando todas las condiciones `:if` y ninguna de las
`:unless` las condiciones se evalúan para `true`.

Performing Custom Validations
-----------------------------
Realizar validaciones personalizadas

Cuando los ayudantes de validación integrados no son suficientes para sus necesidades, puedes\
escribir sus propios validadores o métodos de validación como prefiera.

### Custom Validators
Validadores personalizados

Los validadores personalizados son clases que heredan de `ActiveModel::Validator`. Estas 
clases deben implementar el método `validate` que toma un registro como argumento
y realiza la validación en él. El validador personalizado se llama utilizando el
método `validates_with`.

```ruby
class MyValidator < ActiveModel::Validator
  def validate(record)
    unless record.name.start_with? 'X'
      record.errors.add :name, "Need a name starting with X please!"
    end
  end
end

class Person
  include ActiveModel::Validations
  validates_with MyValidator
end
````

La forma más fácil de agregar validadores personalizados para validar atributos individuales
es con el conveniente `ActiveModel::EachValidator`. En este caso, la clase del validador 
personalizado debe implementar un método `validate_each` que requiere tres
argumentos: registro, atributo y valor. Estos corresponden a la instancia, del
atributo a validar, y el valor del atributo en el pasado.

```ruby
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      record.errors.add attribute, (options[:message] || "is not an email")
    end
  end
end

class Person < ApplicationRecord
  validates :email, presence: true, email: true
end
```

Como se muestra en el ejemplo, también puede combinar validaciones estándar con su
propios validadores personalizados.

### Custom Methods
Métodos personalizados

También se puede crear métodos que verifiquen el estado de sus modelos y agreguen
errores en la colección `errors` cuando no son válidos. Debes entonces
registrar estos métodos utilizando el `validate`
([API](https://api.rubyonrails.org/classes/ActiveModel/Validations/ClassMethods.html#method-i-validate))
método de clase, pasando los símbolos para los nombres de los métodos de validación.

Se puede pasar más de un símbolo para cada método de la clase y las las validaciones
respectivas se ejecutarán en el mismo orden en que se registraron.

El método `valid?` verificará que la colección de errores esté vacía,
por lo que sus métodos de validación personalizados deberían agregarle errores cuando
desear que la validación falle:

```ruby
class Invoice < ApplicationRecord
  validate :expiration_date_cannot_be_in_the_past,
    :discount_cannot_be_greater_than_total_value

  def expiration_date_cannot_be_in_the_past
    if expiration_date.present? && expiration_date < Date.today
      errors.add(:expiration_date, "can't be in the past")
    end
  end

  def discount_cannot_be_greater_than_total_value
    if discount > total_value
      errors.add(:discount, "can't be greater than total value")
    end
  end
end
```

Por defecto, tales validaciones se ejecutarán cada vez que llame `valid?`
o se guarde el objeto. Pero también es posible controlar cuándo ejecutar estas
validaciones personalizadas dando una opción `:on` al método `validate`,
con: `:create` o `:update`.

```ruby
class Invoice < ApplicationRecord
  validate :active_customer, on: :create

  def active_customer
    errors.add(:customer_id, "is not active") unless customer.active?
  end
end
```

Working with Validation Errors
------------------------------
Trabajando con errores de validación

Los métodos `valid?` o `invalid?` Solo proporcionan un resumen del estado de validez. Sin embargo, puede profundizar en cada error individual utilizando varios métodos de la colección `errors`.

La siguiente es una lista de los métodos más utilizados. Consulte la documentación de `ActiveModel::Errors` para obtener una lista de todos los métodos disponibles.

### `errors`

La puerta de enlace a través de la cual se puede profundizar en varios detalles de cada error.

Esto devuelve una instancia de la clase `ActiveModel::Errors` que contiene todos los errores,
cada error está representado por un objeto `ActiveModel::Error`.

```ruby
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end

person = Person.new
person.valid? # => false
person.errors.full_messages
 # => ["Name can't be blank", "Name is too short (minimum is 3 characters)"]

person = Person.new(name: "John Doe")
person.valid? # => true
person.errors.full_messages # => []
```

### `errors[]`

`errors[]` se usa cuando desea verificar los mensajes de error para un atributo específico. Devuelve una matriz de cadenas con todos los mensajes de error para el atributo dado, cada cadena con un mensaje de error. Si no hay errores relacionados con el atributo, devuelve una matriz vacía.

```ruby
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end

person = Person.new(name: "John Doe")
person.valid? # => true
person.errors[:name] # => []

person = Person.new(name: "JD")
person.valid? # => false
person.errors[:name] # => ["is too short (minimum is 3 characters)"]

person = Person.new
person.valid? # => false
person.errors[:name]
 # => ["can't be blank", "is too short (minimum is 3 characters)"]
```

### `errors.where` and error object

A veces podemos necesitar más información sobre cada error junto a su mensaje. Cada error se encapsula como un objeto `ActiveModel::Error`, y el método `where` es la forma más común de acceso.

`where` devuelve una matriz de objetos de error, filtrados por varios grados de condiciones.

```ruby
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end

person = Person.new
person.valid? # => false

>> person.errors.where(:name) # errors linked to :name attribute
>> person.errors.where(:name, :too_short) # further filtered to only :too_short type error
```

Puede leer información diversa de estos objetos de error:

```ruby
>> error = person.errors.where(:name).last
>> error.attribute # => :name
>> error.type # => :too_short
>> error.options[:count] # => 3
```

También se puede generar el mensaje de error:

>> error.message # => "is too short (minimum is 3 characters)"
>> error.full_message # => "Name is too short (minimum is 3 characters)"

El método `full_message` genera un mensaje más fácil de usar, con el nombre del atributo en mayúscula antepuesto.

### `errors.add`

El método `add` crea el objeto de error tomando el `attribute`, el error `type` y opciones adicionales en un hash. Esto es útil para escribir su propio validador.

```ruby
class Person < ApplicationRecord
  validate do |person|
    errors.add :name, :too_plain, message: "is not cool enough"
  end
end

person = Person.create
person.errors.where(:name).first.type # => :too_plain
person.errors.where(:name).first.full_message # => "Name is not cool enough"
```

### `errors[:base]

Puedes agregar errores relacionados con el estado del objeto como un todo, en lugar de estar relacionados con un atributo específico. Puede agregar errores a `:base` cuando desee decir que el objeto no es válido, sin importar los valores de sus atributos.

```ruby
class Person < ApplicationRecord
  validate do |person|
    errors.add :base, :invalid, message: "This person is invalid because ..."
  end
end

person = Person.create
person.errors.where(:base).first.full_message # => "This person is invalid because ..."
```

### `errors.clear`

El método `clear` se usa cuando intencionalmente desea borrar la colección `errors`. Por supuesto, llamar a `errors.clear` sobre un objeto no válido no lo hará realmente válido: la colección `errors` ahora estará vacía, pero la próxima vez que llame a `valid?` o cualquier método que intente guardar este objeto a la base de datos, las validaciones se ejecutarán nuevamente. Si alguna de las validaciones falla, la colección de `errores` se completará nuevamente.

```ruby
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end

person = Person.new
person.valid? # => false
person.errors.empty? # => false

person.errors.clear
person.errors.empty? # => true

person.save # => false

person.errors.empty? # => false
```

### `errors.size`

El método `size` devuelve el número total de errores para el objeto.


```ruby
class Person < ApplicationRecord
  validates :name, presence: true, length: { minimum: 3 }
end

person = Person.new
person.valid? # => false
person.errors.size # => 2

person = Person.new(name: "Andrea", email: "andrea@example.com")
person.valid? # => true
person.errors.size # => 0
```

Displaying Validation Errors in Views
-------------------------------------
Mostrar errores de validación en vistas

Una vez que haya creado un modelo y agregado las validaciones, si ese modelo se crea a través de
un formulario en la web, es probable que desee mostrar un mensaje de error cuando uno de los
las validaciones fallan.

Debido a que cada aplicación maneja este tipo de cosas de manera diferente, Rails no incluye
ningún asistente de vista para ayudarlo a generar estos mensajes directamente.
Sin embargo, debido a la gran cantidad de métodos que Rails le brinda para interactuar con
validaciones en general, puede construir el suyo propio. Además, cuando
generando un dcaffold, Rails pondrá algo de ERB en el `_form.html.erb` que
genera que muestra la lista completa de errores en ese modelo.

Suponiendo que tenemos un modelo que se ha guardado en un variable de instancia llamado
`@artículo`, se ve así:

```ruby
<% if @article.errors.any? %>
  <div id="error_explanation">
    <h2><%= pluralize(@article.errors.count, "error") %> prohibited this article from being saved:</h2>

    <ul>
    <% @article.errors.each do |error| %>
      <li><%= error.full_message %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```

Además, si usa los ayudantes de formulario de Rails para generar sus formularios, cuando
se produce un error de validación en un campo, generará un `<div>` adicional alrededor de
la entrada.

```html
<div class="field_with_errors">
 <input id="article_title" name="article[title]" size="30" type="text" value="">
</div>
```

Luego puedes diseñar este div como quieras. El scaffold predeterminado que
Rails genera, por ejemplo, agrega esta regla CSS:

```css
.field_with_errors {
  padding: 2px;
  background-color: red;
  display: table;
}
```

Esto significa que cualquier campo con un error termina con un borde rojo de 2 píxeles.
