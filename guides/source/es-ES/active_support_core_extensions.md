**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Extensiones de Núcleo de Active Support
=======================================

Active Support es el componente Ruby on Rails responsable de proporcionar extensiones de lenguaje Ruby, utilidades y otras cosas transversales.

Ofrece un resultado final más rico a nivel de lenguaje, dirigido tanto al desarrollo de aplicaciones Rails como al desarrollo de Ruby on Rails.

Después de leer esta guía, sabrá:

* Qué son las extensiones principales.
* Cómo cargar todas las extensiones.
* Cómo seleccionar las extensiones que desee.
* Qué extensiones proporciona Active Support.

--------------------------------------------------------------------------------

How to Load Core Extensions
---------------------------

### Stand-Alone Active Support

Para tener una huella predeterminada cercana a cero, Active Support no carga nada de forma predeterminada. Está dividido en pedazos pequeños para que pueda cargar justo lo que necesita, y también tiene algunos puntos de entrada convenientes para cargar extensiones relacionadas de una sola vez, incluso todo.

Por lo tanto, después de un simple requerimiento como:

```ruby
require "active_support"
```

objects do not even respond to `blank?`. Let's see how to load its definition.

#### Cherry-picking a Definition

La forma más ligera de dejar `blank?` es seleccionar el archivo que lo define.

Para cada método definido como una extensión principal, esta guía tiene una nota que dice dónde se define dicho método. En el caso de `blank?`, La nota dice:

NOTE: Definido en `active_support/core_ext/object/blank.rb`.

Eso significa que puede solicitarlo así:

```ruby
require "active_support"
require "active_support/core_ext/object/blank"
```

Active Support se ha revisado cuidadosamente para que la selección de un archivo solo cargue las dependencias estrictamente necesarias, si las hubiera.

#### Loading Grouped Core Extensions

El siguiente nivel es simplemente cargar todas las extensiones en `Object`. Como regla general, las extensiones de `SomeClass` están disponibles de una sola vez cargando `active_support/core_ext/some_class`.

Por lo tanto, para cargar todas las extensiones a `Object` (incluyendo `blank?`):

```ruby
require "active_support"
require "active_support/core_ext/object"
```

#### Loading All Core Extensions

Es posible que prefiera cargar todas las extensiones principales, hay un archivo para eso:

```ruby
require "active_support"
require "active_support/core_ext"
```

#### Loading All Active Support

Y finalmente, si desea tener disponible todo el Soporte activo, simplemente emita:

```ruby
require "active_support/all"
```

Eso ni siquiera pone todo el Active Support en la memoria por adelantado, de hecho, algunas cosas se configuran mediante `autoload`, por lo que solo se carga si se usa.

### Active Support Within a Ruby on Rails Application

Una aplicación Ruby on Rails carga todo el soporte activo a menos que `config.active_support.bare` sea verdadero. En ese caso, la aplicación solo cargará lo que el propio marco elija para sus propias necesidades, y aún puede hacerlo en cualquier nivel de granularidad, como se explicó en la sección anterior.

Extensions to All Objects
-------------------------

### `blank?` and `present?`

The following values are considered to be blank in a Rails application:

* `nil` y` false`,

* cadenas compuestas solo de espacios en blanco (ver nota a continuación),

* matrices y hashes vacíos, y

* cualquier otro objeto que responda a "¿vacío?" y esté vacío.

INFO: El predicado para cadenas utiliza la clase de caracteres compatible con Unicode `[:space:]`, por lo que, por ejemplo, U + 2029 (separador de párrafo) se considera un espacio en blanco.

WARNING: Tenga en cuenta que no se mencionan los números. En particular, 0 y 0.0 son ** no ** en blanco.

Por ejemplo, este método de `ActionController::HttpAuthentication::Token::ControllerMethods` usa `blank?` Para verificar si un token está presente:

```ruby
def authenticate(controller, &login_procedure)
  token, options = token_and_options(controller.request)
  unless token.blank?
    login_procedure.call(token, options)
  end
end
```

El método `present?` Es equivalente a `!blank?`. Este ejemplo está tomado de `ActionDispatch::Http::Cache::Response`:

```ruby
def set_conditional_cache_control!
  return if self["Cache-Control"].present?
  ...
end
```

NOTE: Definido en `active_support/core_ext/object/blank.rb`.
                 
### `presence`

El método `presence` devuelve su receptor si `present?` y `nil` en caso contrario. Es útil para modismos como este:

```ruby
host = config[:host].presence || 'localhost'
```

NOTE: Definido en `active_support/core_ext/object/blank.rb`.
                  
### `duplicable?`

A partir de Ruby 2.5, la mayoría de los objetos se pueden duplicar mediante `dup` o `clone`:


```ruby
"foo".dup           # => "foo"
"".dup              # => ""
Rational(1).dup     # => (1/1)
Complex(0).dup      # => (0+0i)
1.method(:+).dup    # => TypeError (allocator undefined for Method)
```

Active Support proporciona `duplicable?` Para consultar un objeto sobre esto:

```ruby
"foo".duplicable?           # => true
"".duplicable?              # => true
Rational(1).duplicable?     # => true
Complex(1).duplicable?      # => true
1.method(:+).duplicable?    # => false
```
WARNING: Cualquier clase puede prohibir la duplicación eliminando `dup` y `clone` o generando excepciones. Por lo tanto, solo "rescue" puede decir si un objeto arbitrario dado es duplicable. `duplicable?` depende de la lista codificada anterior, pero es mucho más rápido que `rescue`. Úselo solo si sabe que la lista codificada es suficiente en su caso de uso.

NOTE: Definido en `active_support/core_ext/object/duplicable.rb`.

## `deep_dup`

The `deep_dup` El método devuelve una copia profunda de un objeto dado. Normalmente, cuando `dup` un objeto que contiene otros objetos, Ruby no los "duplica", por lo que crea una copia superficial del objeto. Si tiene una matriz con una cadena, por ejemplo, se verá así:

```ruby
array     = ['string']
duplicate = array.dup

duplicate.push 'another-string'

# the object was duplicated, so the element was added only to the duplicate
array     # => ['string']
duplicate # => ['string', 'another-string']

duplicate.first.gsub!('string', 'foo')

# first element was not duplicated, it will be changed in both arrays
array     # => ['foo']
duplicate # => ['foo', 'another-string']
```

Como puede ver, después de duplicar la instancia de `Array`, obtuvimos otro objeto, por lo tanto, podemos modificarlo y el objeto original permanecerá sin cambios. Sin embargo, esto no es cierto para los elementos de la matriz. Dado que `dup` no hace una copia profunda, la cadena dentro de la matriz sigue siendo el mismo objeto.

Si necesita una copia profunda de un objeto, debe usar `deep_dup`. Aquí hay un ejemplo:

```ruby
array     = ['string']
duplicate = array.deep_dup

duplicate.first.gsub!('string', 'foo')

array     # => ['string']
duplicate # => ['foo']
```

Si el objeto no es duplicable, `deep_dup` simplemente lo devolverá:

```ruby
number = 1
duplicate = number.deep_dup
number.object_id == duplicate.object_id   # => true
```

NOTE: Definido en `active_support/core_ext/object/deep_dup.rb`.
                  
### `try`

Cuando desea llamar a un método en un objeto solo si no es `nil`, la forma más sencilla de lograrlo es con declaraciones condicionales, agregando desorden innecesario. La alternativa es usar `try`. `try` es como` Object # send` excepto que devuelve `nil` si se envía a` nil`.

Aquí hay un ejemplo:

```ruby
# without try
unless @number.nil?
  @number.next
end

# with try
@number.try(:next)
```

Otro ejemplo es este código de `ActiveRecord::ConnectionAdapters::AbstractAdapter` donde `@logger` podría ser` nil`. Puede ver que el código usa `try` y evita una verificación innecesaria.

```ruby
def log_info(sql, name, ms)
  if @logger.try(:debug?)
    name = '%s (%.1fms)' % [name || 'SQL', ms]
    @logger.debug(format_log_entry(name, sql.squeeze(' ')))
  end
end
```

`try` también se puede llamar sin argumentos pero un bloque, que solo se ejecutará si el objeto no es nil:

```ruby
@person.try { |p| "#{p.first_name} #{p.last_name}" }
```

Tenga en cuenta que `try` se tragará los errores sin método, devolviendo nil en su lugar. Si desea protegerse contra errores tipográficos, utilice `try!` En su lugar:

```ruby
@number.try(:nest)  # => nil
@number.try!(:nest) # NoMethodError: undefined method `nest' for 1:Integer
```

NOTE: Definido en `active_support/core_ext/object/try.rb`.
                  
### `class_eval(*args, &block)`

Puede evaluar código en el contexto de la clase singleton de cualquier objeto usando `class_eval`:

```ruby
class Proc
  def bind(object)
    block, time = self, Time.current
    object.class_eval do
      method_name = "__bind_#{time.to_i}_#{time.usec}"
      define_method(method_name, &block)
      method = instance_method(method_name)
      remove_method(method_name)
      method
    end.bind(object)
  end
end
```

NOTE: Definido en `active_support/core_ext/kernel/singleton_class.rb`.
                  
### `acts_like?(duck)`

El método `acts_libe?` Proporciona una manera de comprobar si alguna clase actúa como otra clase basándose en una convención simple: una clase que proporciona la misma interfaz que define `String`

```ruby
def acts_like_string?
end
```

que es solo un marcador, su cuerpo o valor de retorno son irrelevantes. Luego, el código del cliente puede consultar la seguridad del tipo pato de esta manera:

```ruby
some_klass.acts_like?(:string)
```

Rails tiene clases que actúan como `Date` o `Time` y siguen este contrato.

NOTE: Definido en `active_support/core_ext/object/acts_like.rb`.
                  
### `to_param`

Todos los objetos en Rails responden al método `to_param`, que está destinado a devolver algo que los represente como valores en una cadena de consulta o como fragmentos de URL.

Por defecto, `to_param` solo llama a `to_s`:

```ruby
7.to_param # => "7"
```

El valor de retorno de `to_param` **no** debe escaparse:

```ruby
"Tom & Jerry".to_param # => "Tom & Jerry"
```

Varias clases de Rails sobrescriben este método.

Por ejemplo, `nil`,` true` y `false` se devuelven ellos mismos. `Array#to_param` llama a `to_param` en los elementos y une el resultado con "/":

```ruby
[0, true, String].to_param # => "0/true/String"
```

En particular, el sistema de enrutamiento Rails llama a `to_param` en los modelos para obtener un valor para el marcador de posición`: id`. `ActiveRecord :: Base # to_param` devuelve el `id` de un modelo, pero puedes redefinir ese método en tus modelos. Por ejemplo, dado

```ruby
class User
  def to_param
    "#{id}-#{name.parameterize}"
  end
end
```

Nosotras obtenemos:

```ruby
user_path(@user) # => "/users/357-john-smith"
```

ADVERTENCIA. Los controladores deben estar al tanto de cualquier redefinición de `to_param` porque cuando una solicitud como esa viene en "357-john-smith" es el valor de `params[:id] `.

NOTA: Definido en `active_support/core_ext/object/to_param.rb`.
                  
### `to_query`

A excepción de los hash, dada una `key` sin escape, este método construye la parte de una cadena de consulta que mapeará dicha clave a lo que devuelve `to_param`. Por ejemplo, dado

```ruby
class User
  def to_param
    "#{id}-#{name.parameterize}"
  end
end
```

Nosotras obtenemos:

```ruby
current_user.to_query('user') # => "user=357-john-smith"
```

Este método escapa a lo que sea necesario, tanto para la clave como para el valor:

```ruby
account.to_query('company[name]')
# => "company%5Bname%5D=Johnson+%26+Johnson"
```

por lo que su salida está lista para usarse en una cadena de consulta.

Las matrices devuelven el resultado de aplicar `to_query` a cada elemento con `key[]` como clave, y unen el resultado con "&":

```ruby
[3.4, -45.6].to_query('sample')
# => "sample%5B%5D=3.4&sample%5B%5D=-45.6"
```

Los hash también responden a `to_query` pero con una firma diferente. Si no se pasa ningún argumento, una llamada genera una serie ordenada de asignaciones de clave / valor que llaman a `to_query(key)` en sus valores. Luego une el resultado con "&":

```ruby
{c: 3, b: 2, a: 1}.to_query # => "a=1&b=2&c=3"
```

El método `Hash#to_query` acepta un espacio de nombre opcional para las claves:

```ruby
{id: 89, name: "John Smith"}.to_query('user')
# => "user%5Bid%5D=89&user%5Bname%5D=John+Smith"
```
NOTA: Definido en `active_support/core_ext/object/to_query.rb`.

### `with_options`

El método `with_options` proporciona una manera de descartar opciones comunes en una serie de llamadas a métodos.

Dado un hash de opciones predeterminado, `with_options` cede un objeto proxy a un bloque. Dentro del bloque, los métodos llamados en el proxy se envían al receptor con sus opciones fusionadas. Por ejemplo, se deshace de la duplicación en:

```ruby
class Account < ApplicationRecord
  has_many :customers, dependent: :destroy
  has_many :products,  dependent: :destroy
  has_many :invoices,  dependent: :destroy
  has_many :expenses,  dependent: :destroy
end
```

de esta manera:

```ruby
class Account < ApplicationRecord
  with_options dependent: :destroy do |assoc|
    assoc.has_many :customers
    assoc.has_many :products
    assoc.has_many :invoices
    assoc.has_many :expenses
  end
end
```

Ese idioma también puede transmitir _grouping_ al lector. Por ejemplo, digamos que desea enviar un boletín cuyo idioma depende del usuario. En algún lugar del correo, podría agrupar bits dependientes de la configuración regional como este:

```ruby
I18n.with_options locale: user.locale, scope: "newsletter" do |i18n|
  subject i18n.t :subject
  body    i18n.t :body, user_name: user.name
end
```

TIP: Since `with_options` forwards calls to its receiver they can be nested. Each nesting level will merge inherited defaults in addition to their own.

NOTE: Definido en `active_support/core_ext/object/with_options.rb`.

### JSON support

Active Support proporciona una mejor implementación de `to_json` que la gem `json` que normalmente proporciona para los objetos Ruby. Esto se debe a que algunas clases, como `Hash`, `OrderedHash` y `Process::Status` necesitan un manejo especial para proporcionar una representación JSON adecuada.

NOTA: Definido en `active_support/core_ext/object/json.rb`.

### Instance Variables

Active Support proporciona varios métodos para facilitar el acceso a las variables de instancia.

#### `instance_values`

El método `instance_values` devuelve un hash que asigna los nombres de las variables de instancia sin" @ "a su
valores correspondientes. Las claves son cadenas:

```ruby
class C
  def initialize(x, y)
    @x, @y = x, y
  end
end

C.new(0, 1).instance_values # => {"x" => 0, "y" => 1}
```

NOTE: Definido en `active_support/core_ext/object/instance_variables.rb`.
                  
#### `instance_variable_names`

El método `instance_variable_names` devuelve una matriz. Cada nombre incluye el signo "@".

```ruby
class C
  def initialize(x, y)
    @x, @y = x, y
  end
end

C.new(0, 1).instance_variable_names # => ["@x", "@y"]
```

NOTE: Definido en `active_support/core_ext/object/instance_variables.rb`.

### Silencing Warnings and Exceptions

Los métodos `silent_warnings` y `enable_warnings` cambian el valor de `$VERBOSE` en consecuencia durante la duración de su bloqueo, y lo restablecen después:

```ruby
silence_warnings { Object.const_set "RAILS_DEFAULT_LOGGER", logger }
```

También es posible silenciar las excepciones con `suppress`. Este método recibe un número arbitrario de clases de excepción. Si se genera una excepción durante la ejecución del bloque y es `kind_of?` Cualquiera de los argumentos, `suppress` la captura y regresa silenciosamente. De lo contrario, la excepción no se captura:

```ruby
# If the user is locked, the increment is lost, no big deal.
suppress(ActiveRecord::StaleObjectError) do
  current_user.increment! :visits
end
```

NOTE: Definido en `active_support/core_ext/kernel/reporting.rb`.

### `in?`

El predicado `in?` Prueba si un objeto está incluido en otro objeto. Se generará una excepción `ArgumentError` si el argumento pasado no responde a `include?`.

Ejemplos de `in?`:

```ruby
1.in?([1,2])        # => true
"lo".in?("hello")   # => true
25.in?(30..50)      # => false
1.in?(1)            # => ArgumentError
```

NOTE: Definido en `active_support/core_ext/object/inclusion.rb`.
                  
Extensions to `Module`
----------------------

### Attributes

#### `alias_attribute`

```ruby
class User < ApplicationRecord
  # You can refer to the email column as "login".
  # This can be meaningful for authentication code.
  alias_attribute :login, :email
end
```

NOTE: Definido en `active_support/core_ext/module/aliasing.rb`.


#### Internal Attributes

Cuando está definiendo un atributo en una clase que está destinado a ser subclasificado, las colisiones de nombres son un riesgo. Eso es muy importante para las bibliotecas.

Active Support define las macros `attr_internal_reader`, `attr_internal_writer` y `attr_internal_accessor`. Se comportan como sus contrapartes `attr_ *` incorporadas en Ruby, excepto que nombran la variable de instancia subyacente de una manera que hace que las colisiones sean menos probables.

La macro `attr_internal` es sinónimo de `attr_internal_accessor`:

```ruby
# library
class ThirdPartyLibrary::Crawler
  attr_internal :log_level
end

# client code
class MyCrawler < ThirdPartyLibrary::Crawler
  attr_accessor :log_level
end
```

En el ejemplo anterior podría darse el caso de que `:log_level` no pertenezca a la interfaz pública de la biblioteca y solo se utilice para desarrollo. El código del cliente, inconsciente del posible conflicto, subclasifica y define su propio `: log_level`. Gracias a `attr_internal` no hay colisión.

De forma predeterminada, la variable de instancia interna se nombra con un guión bajo inicial, `@ _log_level` en el ejemplo anterior. Eso se puede configurar a través de `Module.attr_internal_naming_format`, sin embargo, puede pasar cualquier cadena de formato similar a` sprintf` con una `@` inicial y un `% s` en algún lugar, que es donde se colocará el nombre. El valor predeterminado es `" @ _% s "`.

Rails usa atributos internos en algunos puntos, como ejemplos de vistas:

```ruby
module ActionView
  class Base
    attr_internal :captures
    attr_internal :request, :layout
    attr_internal :controller, :template
  end
end
```

NOTE: Definido en `active_support/core_ext/module/attr_internal.rb`.
                  
#### Module Attributes

Las macros `mattr_reader`, `mattr_writer` y `mattr_accessor` son las mismas que las macros `cattr_ * `definidas para la clase. De hecho, las macros `cattr_ *` son solo alias para las macros `mattr_ *`. Marque [Class Attributes](#class-attributes).

Por ejemplo, el mecanismo de dependencias las usa:

```ruby
module ActiveSupport
  module Dependencies
    mattr_accessor :warnings_on_first_load
    mattr_accessor :history
    mattr_accessor :loaded
    mattr_accessor :mechanism
    mattr_accessor :load_paths
    mattr_accessor :load_once_paths
    mattr_accessor :autoloaded_constants
    mattr_accessor :explicitly_unloadable_constants
    mattr_accessor :constant_watch_stack
    mattr_accessor :constant_watch_stack_mutex
  end
end
```

`active_support/core_ext/module/attribute_accessors.rb`.

### Parents

#### `module_parent`

El método `module_parent` en un módulo con nombre anidado devuelve el módulo que contiene su constante correspondiente:

```ruby
module X
  module Y
    module Z
    end
  end
end
M = X::Y::Z

X::Y::Z.module_parent # => X::Y
M.module_parent       # => X::Y
```

Si el módulo es anónimo o pertenece al nivel superior, `module_parent` devuelve `Object`.

WARNING: Tenga en cuenta que en ese caso `module_parent_name` devuelve `nil`.

NOTE: Definido en  `active_support/core_ext/module/introspection.rb`.


#### `module_parent_name`

El método `module_parent_name` en un módulo con nombre anidado devuelve el nombre completo del módulo que contiene su constante correspondiente:

```ruby
module X
  module Y
    module Z
    end
  end
end
M = X::Y::Z

X::Y::Z.module_parent_name # => "X::Y"
M.module_parent_name       # => "X::Y"
```

Para módulos anónimos o de nivel superior, `module_parent_name` devuelve `nil`.

WARNING: Tenga en cuenta que en ese caso `module_parent` devuelve `Object`.

NOTE: Definido en `active_support/core_ext/module/introspection.rb`.

#### `module_parents`

El método `module_parents` llama a `module_parent` en el receptor y hacia arriba hasta que se alcanza el `Object`. La cadena se devuelve en una matriz, de abajo hacia arriba:

```ruby
module X
  module Y
    module Z
    end
  end
end
M = X::Y::Z

X::Y::Z.module_parents # => [X::Y, X, Object]
M.module_parents       # => [X::Y, X, Object]
```

NOTE: Definido en `active_support/core_ext/module/introspection.rb`.

### Anonymous

Un módulo puede tener o no un nombre:

```ruby
module M
end
M.name # => "M"

N = Module.new
N.name # => "N"

Module.new.name # => nil
```

Puede comprobar si un módulo tiene un nombre con el predicado `anonymous?`:

```ruby
module M
end
M.anonymous? # => false

Module.new.anonymous? # => true
```

Tenga en cuenta que ser inalcanzable no implica ser anónimo:

```ruby
module M
end

m = Object.send(:remove_const, :M)

m.anonymous? # => false
```

aunque un módulo anónimo es inalcanzable por definición.

NOTE: Definido en `active_support/core_ext/module/anonymous.rb`.

### Method Delegation

#### `delegate`

La macro `delegate` ofrece una manera fácil de reenviar métodos.

Imaginemos que los usuarios de alguna aplicación tienen información de inicio de sesión en el modelo `User`, pero el nombre y otros datos en un modelo `Profile` separado:

```ruby
class User < ApplicationRecord
  has_one :profile
end
```

Con esa configuración, obtienes el nombre de un usuario a través de su perfil, `user.profile.name`, pero podría ser útil poder acceder a dicho atributo directamente:

```ruby
class User < ApplicationRecord
  has_one :profile

  def name
    profile.name
  end
end
```

Eso es lo que `delegate` hace por ti:

```ruby
class User < ApplicationRecord
  has_one :profile

  delegate :name, to: :profile
end
```

Es más corto y la intención más obvia.

El método debe ser público en el destino.

La macro `delegate` acepta varios métodos:

```ruby
delegate :name, :age, :address, :twitter, to: :profile
```

Cuando se interpola en una cadena, la opción `:to` debería convertirse en una expresión que evalúe el objeto al que se delega el método. Normalmente una cadena o un símbolo. Tal expresión se evalúa en el contexto del receptor:

```ruby
# delegates to the Rails constant
delegate :logger, to: :Rails

# delegates to the receiver's class
delegate :table_name, to: :class
```

WARNING: Si la opción `:prefix` es `true`, esto es menos genérico, ver más abajo.

De forma predeterminada, si la delegación genera "NoMethodError" y el objetivo es "nil", la excepción se propaga. Puede pedir que se devuelva `nil` en su lugar con la opción `:allow_nil`:

```ruby
delegate :name, to: :profile, allow_nil: true
```

Con `:allow_nil` la llamada `user.name` devuelve `nil` si el usuario no tiene perfil.

La opción `:prefix` agrega un prefijo al nombre del método generado. Esto puede ser útil, por ejemplo, para obtener un mejor nombre:

```ruby
delegate :street, to: :address, prefix: true
```

El ejemplo anterior genera `address_street` en lugar de `street`.

WARNING: Dado que en este caso el nombre del método generado está compuesto por el objeto de destino y los nombres del método de destino, la opción `: to` debe ser un nombre de método.

También se puede configurar un prefijo personalizado:

```ruby
delegate :size, to: :attachment, prefix: :avatar
```

En el ejemplo anterior, la macro genera `avatar_size` en lugar de `size`.

La opción `:private` cambia el alcance de los métodos:

```ruby
delegate :date_of_birth, to: :profile, private: true
```

Los métodos delegados son públicos de forma predeterminada. Pase `private: true` para cambiar eso.

NOTE: Definido en `active_support/core_ext/module/delegation.rb`

#### `delegate_missing_to`

Imagina que te gustaría delegar todo lo que falta en el objeto `User`,
al de `Profile`. La macro `delegate_missing_to` le permite implementar esto
en una brisa:

```ruby
class User < ApplicationRecord
  has_one :profile

  delegate_missing_to :profile
end
```

El objetivo puede ser cualquier cosa invocable dentro del objeto, p. Ej. variables de instancia,
métodos, constantes, etc. Sólo se delegan los métodos públicos del objetivo.

NOTE: Definido en `active_support/core_ext/module/delegation.rb`.
                  
### Redefining Methods

Hay casos en los que necesitas definir un método con `define_method`, pero no sabes si ya existe un método con ese nombre. Si es así, se emite una advertencia si están habilitados. No es gran cosa, pero tampoco limpia.

El método `redefine_method` previene tal advertencia potencial, eliminando el método existente antes si es necesario.

También puede usar `silent_redefinition_of_method` si necesita definir
el método de reemplazo usted mismo (porque está usando `delegate`, para
ejemplo).


NOTE: Definido en `active_support/core_ext/module/redefine_method.rb`.
                  
Extensions to `Class`
---------------------

### Class Attributes

#### `class_attribute`

El método `class_attribute` declara uno o más atributos de clase heredables que pueden anularse en cualquier nivel de la jerarquía.

```ruby
class A
  class_attribute :x
end

class B < A; end

class C < B; end

A.x = :a
B.x # => :a
C.x # => :a

B.x = :b
A.x # => :a
C.x # => :b

C.x = :c
A.x # => :a
B.x # => :b
```

Por ejemplo, `ActionMailer::Base` define:

```ruby
class_attribute :default_params
self.default_params = {
  mime_version: "1.0",
  charset: "UTF-8",
  content_type: "text/plain",
  parts_order: [ "text/plain", "text/enriched", "text/html" ]
}.freeze
```

También se puede acceder a ellos y anularlos a nivel de instancia.

```ruby
A.x = 1

a1 = A.new
a2 = A.new
a2.x = 2

a1.x # => 1, comes from A
a2.x # => 2, overridden in a2
```

La generación del método de instancia de escritor se puede evitar configurando la opción `:instance_writer` en `false`.

```ruby
module ActiveRecord
  class Base
    class_attribute :table_name_prefix, instance_writer: false, default: "my"
  end
end
```

Un modelo puede encontrar esa opción útil como una forma de evitar que la asignación masiva establezca el atributo.

La generación del método de la instancia del lector puede evitarse configurando la opción `:instance_reader` en `false`.

```ruby
class A
  class_attribute :x, instance_reader: false
end

A.new.x = 1
A.new.x # NoMethodError
```

Por conveniencia, `class_attribute` también define un predicado de instancia que es la doble negación de lo que devuelve el lector de instancias. En los ejemplos anteriores se llamaría `x?`.

Cuando `:instance_reader` es `false`, el predicado de instancia devuelve un `NoMethodError` al igual que el método reader.

Si no desea el predicado de instancia, pase `instance_predicate: false` y no se definirá.

NOTE: Definido en  `active_support/core_ext/class/attribute.rb`.

#### `cattr_reader`, `cattr_writer`, and `cattr_accessor`

Las macros `cattr_reader`,` cattr_writer` y `cattr_accessor` son análogas a sus contrapartes` attr_ * `pero para clases. Inicializan una variable de clase en `nil` a menos que ya exista, y generan los métodos de clase correspondientes para acceder a ella:

```ruby
class MysqlAdapter < AbstractAdapter
  # Generates class methods to access @@emulate_booleans.
  cattr_accessor :emulate_booleans
end
```

Además, puede pasar un bloque a `cattr_ *` para configurar el atributo con un valor predeterminado:

```ruby
class MysqlAdapter < AbstractAdapter
  # Generates class methods to access @@emulate_booleans with default value of true.
  cattr_accessor :emulate_booleans, default: true
end
```

Los métodos de instancia también se crean por conveniencia, son solo sustitutos del atributo de clase. Por lo tanto, las instancias pueden cambiar el atributo de clase, pero no pueden anularlo como sucede con `class_attribute` (ver más arriba). Por ejemplo dado

```ruby
module ActionView
  class Base
    cattr_accessor :field_error_proc, default: Proc.new { ... }
  end
end
```

podemos acceder a `field_error_proc` en las vistas.

La generación del método de instancia del lector se puede evitar configurando `:instance_reader` en `false` y la generación del método de instancia del escritor se puede evitar configurando `:instance_writer` en `false`. La generación de ambos métodos se puede evitar configurando `:instance_accessor` en `false`. En todos los casos, el valor debe ser exactamente `false` y no ningún valor falso.

```ruby
module A
  class B
    # No first_name instance reader is generated.
    cattr_accessor :first_name, instance_reader: false
    # No last_name= instance writer is generated.
    cattr_accessor :last_name, instance_writer: false
    # No surname instance reader or surname= writer is generated.
    cattr_accessor :surname, instance_accessor: false
  end
end
```

Un modelo puede encontrar útil establecer `:instance_accessor` en `false` como una forma de evitar que la asignación masiva establezca el atributo.

NOTE: Definido en `active_support/core_ext/module/attribute_accessors.rb`.

### Subclasses & Descendants

#### `subclasses`

El método `subclasses` devuelve las subclases del receptor:

```ruby
class C; end
C.subclasses # => []

class B < C; end
C.subclasses # => [B]

class A < B; end
C.subclasses # => [B]

class D < C; end
C.subclasses # => [B, D]
```

El orden en el que se devuelven estas clases no se especifica.

NOTE: Definido en `active_support/core_ext/class/subclasses.rb`.
                  
#### `descendants`

El método `descendants` devuelve todas las clases que son` <`que su receptor:

```ruby
class C; end
C.descendants # => []

class B < C; end
C.descendants # => [B]

class A < B; end
C.descendants # => [B, A]

class D < C; end
C.descendants # => [B, A, D]
```

El orden en el que se devuelven estas clases no se especifica.

NOTE: Definido en `active_support/core_ext/class/subclasses.rb`.

Extensions to `String`
----------------------

### Output Safety

#### Motivation

Insertar datos en plantillas HTML requiere un cuidado especial. Por ejemplo, no puede simplemente interpolar `@review.title` literalmente en una página HTML. Por un lado, si el título de la reseña es "Flanagan & Matz rules!" la salida no estará bien formada porque un ampersand debe escaparse como "&amp;amp;". Además, dependiendo de la aplicación, eso puede ser un gran agujero de seguridad porque los usuarios pueden inyectar HTML malicioso configurando un título de revisión hecho a mano. Consulte la sección sobre secuencias de comandos entre sitios en la [Security guide](security.html#cross-site-scripting-xss) para obtener más información sobre los riesgos.

#### Safe Strings

Active Support tiene el concepto de _(html) safe_ strings. Una cadena segura es aquella que está marcada como insertable en HTML tal cual. Es de confianza, sin importar si se ha escapado o no.

Las cadenas se consideran _inseguras_ por defecto:

```ruby
"".html_safe? # => false
```

Puede obtener una cadena segura de una determinada con el método `html_safe`:

```ruby
s = "".html_safe
s.html_safe? # => true
```

Es importante entender que `html_safe` no realiza ningún tipo de escape, es solo una afirmación:

```ruby
s = "<script>...</script>".html_safe
s.html_safe? # => true
s            # => "<script>...</script>"
```

Es su responsabilidad asegurarse de que llamar a `html_safe` en una cadena en particular esté bien.

Si agrega una cadena segura, ya sea en el lugar con `concat`/`<< `, o con `+`, el resultado es una cadena segura. Se escapan los argumentos inseguros:

```ruby
"".html_safe + "<" # => "&lt;"
```

Los argumentos seguros se añaden directamente:

```ruby
"".html_safe + "<".html_safe # => "<"
```

Estos métodos no deben utilizarse en vistas normales. Los valores no seguros se escapan automáticamente:

```erb
<%= @review.title %> <%# fine, escaped if needed %>
```

Para insertar algo textualmente use el ayudante `raw` en lugar de llamar a `html_safe`:

```erb
<%= raw @cms.current_template %> <%# inserts @cms.current_template as is %>
```

o, de manera equivalente, use `<%==`:

```erb
<%== @cms.current_template %> <%# inserts @cms.current_template as is %>
```

El ayudante `raw` llama a `html_safe` para usted:

```ruby
def raw(stringish)
  stringish.to_s.html_safe
end
```

NOTE: Definido en `active_support/core_ext/string/output_safety.rb`.
                  
### Transformation

Como regla general, excepto quizás para la concatenación como se explicó anteriormente, cualquier método que pueda cambiar una cadena le da una cadena insegura. Estos son `downcase`,` gsub`, `strip`,` chomp`, `underscore`, etc.

En el caso de transformaciones in situ como `gsub!`, El receptor en sí se vuelve inseguro.

INFO: El bit de seguridad se pierde siempre, sin importar si la transformación realmente cambió algo.

#### Conversion and Coercion

Llamar a `to_s` en una cadena segura devuelve una cadena segura, pero la coerción con `to_str` devuelve una cadena insegura.

#### Copying

Llamar a `dup` o `clone` en cadenas seguras produce cadenas seguras.

### `remove`

El método `remove` eliminará todas las apariciones del patrón:

```ruby
"Hello World".remove(/Hello /) # => "World"
```

También existe la versión destructiva `String#remove!`.

NOTE: Definido en `active_support/core_ext/string/filters.rb`.

### `squish`

El método `squish` elimina los espacios en blanco iniciales y finales, y sustituye los espacios en blanco con un solo espacio cada uno:

```ruby
" \n  foo\n\r \t bar \n".squish # => "foo bar"
```

También existe la versión destructiva `String#squish!`.

Tenga en cuenta que maneja espacios en blanco ASCII y Unicode.

NOTE: Definido en `active_support/core_ext/string/filters.rb`.
                  
### `truncate`

El método `truncate` devuelve una copia de su receptor truncado después de una `length` dada:

```ruby
"Oh dear! Oh dear! I shall be late!".truncate(20)
# => "Oh dear! Oh dear!..."
```

La elipsis se puede personalizar con la opción `:omission`:

```ruby
"Oh dear! Oh dear! I shall be late!".truncate(20, omission: '&hellip;')
# => "Oh dear! Oh &hellip;"
```

Tenga en cuenta en particular que el truncamiento tiene en cuenta la longitud de la cadena de omisión.

Pase un `:separator` para truncar la cadena en un salto natural:

```ruby
"Oh dear! Oh dear! I shall be late!".truncate(18)
# => "Oh dear! Oh dea..."
"Oh dear! Oh dear! I shall be late!".truncate(18, separator: ' ')
# => "Oh dear! Oh..."
```

La opción `:separator` puede ser una expresión regular:

```ruby
"Oh dear! Oh dear! I shall be late!".truncate(18, separator: /\s/)
# => "Oh dear! Oh..."
```

En los ejemplos anteriores, "dear" se corta primero, pero luego `:separator` lo impide.

### `truncate_bytes`

El método `truncate_bytes` devuelve una copia de su receptor truncado como máximo en bytes de `bytesize`:

```ruby
"👍👍👍👍".truncate_bytes(15)
# => "👍👍👍…"
```

La elipsis se puede personalizar con la opción `: omission`:

```ruby
"👍👍👍👍".truncate_bytes(15, omission: "🖖")
# => "👍👍🖖"
```

NOTE: Definido en `active_support/core_ext/string/filters.rb`.
                  
### `truncate_words`

El método `truncate_words` devuelve una copia de su receptor truncado después de un número determinado de palabras:

```ruby
"Oh dear! Oh dear! I shall be late!".truncate_words(4)
# => "Oh dear! Oh dear!..."
```

La elipsis se puede personalizar con la opción `:omission`:

```ruby
"Oh dear! Oh dear! I shall be late!".truncate_words(4, omission: '&hellip;')
# => "Oh dear! Oh dear!&hellip;"
```

Pase un `:separator` para truncar la cadena en un salto natural:

```ruby
"Oh dear! Oh dear! I shall be late!".truncate_words(3, separator: '!')
# => "Oh dear! Oh dear! I shall be late..."
```

La opción `:separator` puede ser una expresión regular:

```ruby
"Oh dear! Oh dear! I shall be late!".truncate_words(4, separator: /\s/)
# => "Oh dear! Oh dear!..."
```

NOTE: Definido en `active_support/core_ext/string/filters.rb`.
                  
### `inquiry`

El método `query` convierte una cadena en un objeto `StringInquirer` haciendo más bonitas las verificaciones de igualdad.

```ruby
"production".inquiry.production? # => true
"active".inquiry.inactive?       # => false
```

NOTE: Definido en `active_support/core_ext/string/query.rb`.

### `starts_with?` and `ends_with?`

Active Support define los alias en tercera persona de `String#start_with?` y `String#end_with?`:

```ruby
"foo".starts_with?("f") # => true
"foo".ends_with?("o")   # => true
```

NOTE: Definido en `active_support/core_ext/string/starts_ends_with.rb`.
                  
### `strip_heredoc`

El método `strip_heredoc` elimina la sangría en heredocs.

Por ejemplo en

```ruby
if options[:usage]
  puts <<-USAGE.strip_heredoc
    This command does such and such.

    Supported options are:
      -h         This message
      ...
  USAGE
end
```

el usuario vería el mensaje de uso alineado con el margen izquierdo.

Técnicamente, busca la línea con menos sangría en toda la cadena y elimina
esa cantidad de espacios en blanco iniciales.

NOTE: Definido en `active_support/core_ext/string/strip.rb`.
                  
 ### `indent`

Sangra las líneas en el receptor:

```ruby
<<EOS.indent(2)
def some_method
  some_code
end
EOS
# =>
  def some_method
    some_code
  end
```

El segundo argumento, `indent_string`, especifica qué cadena de sangría usar. El valor predeterminado es `nil`, que le dice al método que haga una conjetura educada mirando la primera línea con sangría, y retroceda a un espacio si no hay ninguno.

```ruby
"  foo".indent(2)        # => "    foo"
"foo\n\t\tbar".indent(2) # => "\t\tfoo\n\t\t\t\tbar"
"foo".indent(2, "\t")    # => "\t\tfoo"
```

Si bien `indent_string` suele ser un espacio o tabulación, puede ser cualquier cadena.

El tercer argumento, `indent_empty_lines`, es una bandera que indica si las líneas vacías deben tener sangría. El valor predeterminado es falso.

```ruby
"foo\n\nbar".indent(2)            # => "  foo\n\n  bar"
"foo\n\nbar".indent(2, nil, true) # => "  foo\n  \n  bar"
```

El método `indent!` Realiza la sangría en el lugar.

NOTE: Definido en `active_support/core_ext/string/indent.rb`.
                  
### Access

#### `at(position)`

Devuelve el carácter de la cadena en la posición `position`:

```ruby
"hello".at(0)  # => "h"
"hello".at(4)  # => "o"
"hello".at(-1) # => "o"
"hello".at(10) # => nil
```

NOTE: Definido en `active_support/core_ext/string/access.rb`.
                  
#### `from(position)`

Devuelve la subcadena de la cadena que comienza en la posición `position`:

```ruby
"hello".from(0)  # => "hello"
"hello".from(2)  # => "llo"
"hello".from(-2) # => "lo"
"hello".from(10) # => nil
```

NOTE: Definido en `active_support/core_ext/string/access.rb`.
                  
 #### `to(position)`

Devuelve la subcadena de la cadena hasta la posición `position`:

```ruby
"hello".to(0)  # => "h"
"hello".to(2)  # => "hel"
"hello".to(-2) # => "hell"
"hello".to(10) # => "hello"
```

NOTE: Definido en `active_support/core_ext/string/access.rb`.
                  
#### `first(limit = 1)`

La llamada `str.first(n)` es equivalente a `str.to(n-1)` si `n` > 0, y devuelve una cadena vacía para` n` == 0.

NOTA: Definido en `active_support/core_ext/string/access.rb`.


#### `last(limit = 1)`

La llamada `str.last (n)` es equivalente a `str.from (-n)` si `n`> 0, y devuelve una cadena vacía para` n` == 0.

NOTE: Definido en `active_support/core_ext/string/access.rb`.

### Inflections

#### `pluralize`

El método `pluralize` devuelve el plural de su receptor:

```ruby
"table".pluralize     # => "tables"
"ruby".pluralize      # => "rubies"
"equipment".pluralize # => "equipment"
```

Como muestra el ejemplo anterior, Active Support conoce algunos plurales irregulares y sustantivos incontables. Las reglas integradas se pueden ampliar en `config/initializers/inflections.rb`. Este archivo es generado por defecto por el comando `rails new` y tiene instrucciones en los comentarios.

`pluralize` también puede tomar un parámetro opcional `count`. Si `count == 1` se devolverá la forma singular. Para cualquier otro valor de `count` se devolverá la forma plural:

```ruby
"dude".pluralize(0) # => "dudes"
"dude".pluralize(1) # => "dude"
"dude".pluralize(2) # => "dudes"
```

Active Record utiliza este método para calcular el nombre de tabla predeterminado que corresponde a un modelo:


```ruby
# active_record/model_schema.rb
def undecorated_table_name(class_name = base_class.name)
  table_name = class_name.to_s.demodulize.underscore
  pluralize_table_names ? table_name.pluralize : table_name
end
```

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.

#### `singularize`

El inverso de `pluralize`:

```ruby
"tables".singularize    # => "table"
"rubies".singularize    # => "ruby"
"equipment".singularize # => "equipment"
```

Las asociaciones calculan el nombre de la clase asociada predeterminada correspondiente utilizando este método:

```ruby
# active_record/reflection.rb
def derive_class_name
  class_name = name.to_s.camelize
  class_name = class_name.singularize if collection?
  class_name
end
```

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.

#### `camelize`

El método `camelize` devuelve su receptor en caso de camello:

```ruby
"product".camelize    # => "Product"
"admin_user".camelize # => "AdminUser"
```

Como regla general, puede pensar en este método como el que transforma las rutas en nombres de módulos o clases Ruby, donde se cortan espacios de nombres separados:

```ruby
"backoffice/session".camelize # => "Backoffice::Session"
```

Por ejemplo, Action Pack usa este método para cargar la clase que proporciona una determinada tienda de sesión:

```ruby
# action_controller/metal/session_management.rb
def session_store=(store)
  @@session_store = store.is_a?(Symbol) ?
    ActionDispatch::Session.const_get(store.to_s.camelize) :
    store
end
```

`camelize` acepta un argumento opcional, que puede ser`:upper` (predeterminado) o `:lower`. Con este último la primera letra se vuelve minúscula:

```ruby
"visual_effect".camelize(:lower) # => "visualEffect"
```

Eso puede ser útil para calcular nombres de métodos en un lenguaje que siga esa convención, por ejemplo JavaScript.

INFO: Como regla general, puede pensar en `camelize` como el inverso de `undercore`, aunque hay casos en los que eso no se cumple: `" SSLError ".underscore.camelize` devuelve` "SslError" `. Para apoyar casos como este, Active Support le permite especificar acrónimos en `config/initializers/inflections.rb`:

```ruby
ActiveSupport::Inflector.inflections do |inflect|
  inflect.acronym 'SSL'
end

"SSLError".underscore.camelize # => "SSLError"
```

`camelize` tiene el alias de `camelcase`.

NOTE: Definido en  `active_support/core_ext/string/inflections.rb`.

#### `underscore`

El método `underscore` va al revés, desde el caso del camello hasta las rutas:

```ruby
"Product".underscore   # => "product"
"AdminUser".underscore # => "admin_user"
```

También convierte "::" a "/":

```ruby
"Backoffice::Session".underscore # => "backoffice/session"
```

y entiende cadenas que comienzan con minúsculas:

```ruby
"visualEffect".underscore # => "visual_effect"
```

Sin embargo, el `underscore` no acepta ningún argumento.

La carga automática de módulos y clases de Rails utiliza un guión bajo para inferir la ruta relativa sin extensión de un archivo que definiría una constante faltante dada:

```ruby
# active_support/dependencies.rb
def load_missing_constant(from_mod, const_name)
  ...
  qualified_name = qualified_name_for from_mod, const_name
  path_suffix = qualified_name.underscore
  ...
end
```

INFO: Como regla general, puede pensar en `underscore` como el inverso de `camelize`, aunque hay casos en los que eso no se cumple. Por ejemplo, `" SSLError ".underscore.camelize` devuelve` "SslError" `.

NOTA: Definido en`active_support/core_ext/string/inflections.rb`.

#### `titleize`

El método `titleize` capitaliza las palabras en el receptor:

```ruby
"alice in wonderland".titleize # => "Alice In Wonderland"
"fermat's enigma".titleize     # => "Fermat's Enigma"
```

`titleize` tiene el alias de `titlecase`.

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.
                  
#### `dasherize`

El método `dasherize` reemplaza los guiones bajos en el receptor con guiones:

```ruby
"name".dasherize         # => "name"
"contact_data".dasherize # => "contact-data"
```

El serializador XML de modelos utiliza este método para dividir los nombres de los nodos:

```ruby
# active_model/serializers/xml.rb
def reformat_name(name)
  name = name.camelize if camelize?
  dasherize? ? name.dasherize : name
end
```

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.
                  
#### `demodulize`

Given a string with a qualified constant name, `demodulize` returns the very constant name, that is, the rightmost part of it:

```ruby
"Product".demodulize                        # => "Product"
"Backoffice::UsersController".demodulize    # => "UsersController"
"Admin::Hotel::ReservationUtils".demodulize # => "ReservationUtils"
"::Inflections".demodulize                  # => "Inflections"
"".demodulize                               # => ""

```

Active Record, por ejemplo, utiliza este método para calcular el nombre de una columna de caché de contador:

```ruby
# active_record/reflection.rb
def counter_cache_column
  if options[:counter_cache] == true
    "#{active_record.name.demodulize.underscore.pluralize}_count"
  elsif options[:counter_cache]
    options[:counter_cache]
  end
end
```

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.
                  
#### `deconstantize`

Dada una cadena con una expresión de referencia constante calificada, `deconstantize` elimina el segmento más a la derecha, generalmente dejando el nombre del contenedor de la constante:

```ruby
"Product".deconstantize                        # => ""
"Backoffice::UsersController".deconstantize    # => "Backoffice"
"Admin::Hotel::ReservationUtils".deconstantize # => "Admin::Hotel"
```

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.
                   
#### `parameterize`

El método `parameterize` normaliza su receptor de una forma que se puede utilizar en URL bonitas.

```ruby
"John Smith".parameterize # => "john-smith"
"Kurt Gödel".parameterize # => "kurt-godel"
```

Para preservar el caso de la cadena, establezca el argumento `preserve_case` en verdadero. De forma predeterminada, `preserve_case` se establece en falso.

```ruby
"John Smith".parameterize(preserve_case: true) # => "John-Smith"
"Kurt Gödel".parameterize(preserve_case: true) # => "Kurt-Godel"
```

Para usar un separador personalizado, anule el argumento `separator`.

```ruby
"John Smith".parameterize(separator: "_") # => "john\_smith"
"Kurt Gödel".parameterize(separator: "_") # => "kurt\_godel"
```

NOTE: Definido en  `active_support/core_ext/string/inflections.rb`.

#### `tableize`

El método `tableize` es`underscore` seguido de `pluralize`.

```ruby
"Person".tableize      # => "people"
"Invoice".tableize     # => "invoices"
"InvoiceLine".tableize # => "invoice_lines"
```

Como regla general, `tableize` devuelve el nombre de la tabla que corresponde a un modelo dado para casos simples. De hecho, la implementación real en Active Record no es un "tableize" directo, porque también demoduliza el nombre de la clase y verifica algunas opciones que pueden afectar la cadena devuelta.

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.

#### `classify`

El método `classify` es el inverso de `tableize`. Te da el nombre de clase correspondiente al nombre de una tabla:

```ruby
"people".classify        # => "Person"
"invoices".classify      # => "Invoice"
"invoice_lines".classify # => "InvoiceLine"
```

El método comprende tabla calificada

```ruby
"highrise_production.companies".classify # => "Company"
```

Tenga en cuenta que `classify` devuelve un nombre de clase como una cadena. Puede obtener el objeto de clase real invocando `constantize`, explicado a continuación.

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.
                  
#### `constantize`~

El método `constantize` resuelve la expresión de referencia constante en su receptor:

```ruby
"Integer".constantize # => Integer

module M
  X = 1
end
"M::X".constantize # => 1
```

Si la cadena no se evalúa como una constante conocida, o su contenido ni siquiera es un nombre de constante válido, `constantize` genera `NameError`.

La resolución constante de nombres por `constantize` comienza siempre en el `Object` de nivel superior, incluso si no hay un "::" inicial.

```ruby
X = :in_Object
module M
  X = :in_M

  X                 # => :in_M
  "::X".constantize # => :in_Object
  "X".constantize   # => :in_Object (!)
end
```

Entonces, en general, no es equivalente a lo que haría Ruby en el mismo lugar, si se evaluara una constante real.

Los casos de prueba de mailer obtienen el mailer que se está probando a partir del nombre de la clase de prueba usando `constantize`:

```ruby
# action_mailer/test_case.rb
def determine_default_mailer(name)
  name.sub(/Test$/, '').constantize
rescue NameError => e
  raise NonInferrableMailerError.new(name)
end
```

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.

#### `humanize`

El método `humanize` modifica un nombre de atributo para mostrarlo a los usuarios finales.

Específicamente realiza estas transformaciones:

  * Aplica reglas de inflexión humana al argumento.
  * Elimina los guiones bajos iniciales, si los hay.
  * Elimina un sufijo "_id" si está presente.
  * Reemplaza los guiones bajos con espacios, si los hubiera.
  * Reduce todas las palabras excepto los acrónimos.
  * Capitaliza la primera palabra.

La mayúscula de la primera palabra se puede desactivar configurando la
opción `:capitalize` en falso (el valor predeterminado es verdadero).

```ruby
"name".humanize                         # => "Name"
"author_id".humanize                    # => "Author"
"author_id".humanize(capitalize: false) # => "author"
"comments_count".humanize               # => "Comments count"
"_id".humanize                          # => "Id"
```

Si "SSL" se definió como un acrónimo:

```ruby
'ssl_error'.humanize # => "SSL error"
```

El método auxiliar `full_messages` usa `humanize` como alternativa para incluir
nombres de atributos:

```ruby
def full_messages
  map { |attribute, message| full_message(attribute, message) }
end

def full_message
  ...
  attr_name = attribute.to_s.tr('.', '_').humanize
  attr_name = @base.class.human_attribute_name(attribute, default: attr_name)
  ...
end
```

NOTE: Definido en `active_support/core_ext/string/inflections.rb`.

#### `foreign_key`

El método `foreign_key` da un nombre de columna de clave externa de un nombre de clase. Para hacerlo, demoduliza, subraya y agrega "_id":

```ruby
"User".foreign_key           # => "user_id"
"InvoiceLine".foreign_key    # => "invoice_line_id"
"Admin::Session".foreign_key # => "session_id"
```

Pase un argumento falso si no desea el guión bajo en "_id":

```ruby
"User".foreign_key(false) # => "userid"
```

Las asociaciones usan este método para inferir claves externas, por ejemplo, `has_one` y `has_many` hacen esto:

```ruby
# active_record/associations.rb
foreign_key = options[:foreign_key] || reflection.active_record.name.foreign_key
```

NOTE: Definido en  `active_support/core_ext/string/inflections.rb`.

### Conversions

#### `to_date`, `to_time`, `to_datetime`

Los métodos `to_date`, `to_time` y `to_datetime` son básicamente envoltorios de conveniencia alrededor de `Date._parse`:

```ruby
"2010-07-27".to_date              # => Tue, 27 Jul 2010
"2010-07-27 23:37:00".to_time     # => 2010-07-27 23:37:00 +0200
"2010-07-27 23:37:00".to_datetime # => Tue, 27 Jul 2010 23:37:00 +0000
```

`to_time` recibe un argumento opcional`:utc` o `:local`, para indicar en qué zona horaria quieres la hora:

```ruby
"2010-07-27 23:42:00".to_time(:utc)   # => 2010-07-27 23:42:00 UTC
"2010-07-27 23:42:00".to_time(:local) # => 2010-07-27 23:42:00 +0200
```

El valor predeterminado es `:local`.

Consulte la documentación de `Date._parse` para obtener más detalles.

INFO: Los tres devuelven `nil` para los receptores en blanco.

NOTE: Definido en `active_support/core_ext/string/conversions.rb`.
                  
Extensions to `Symbol`
----------------------

### `starts_with?` and `ends_with?`

Active Support define los alias en tercera persona de `Symbol#start_with?` y `Symbol#end_with?`:

```ruby
:foo.starts_with?("f") # => true
:foo.ends_with?("o")   # => true
```

NOTE: Definido en `active_support/core_ext/symbol/starts_ends_with.rb`.
                  
Extensions to `Numeric`
-----------------------

### Bytes

Todos los números responden a estos métodos:

```ruby
bytes
kilobytes
megabytes
gigabytes
terabytes
petabytes
exabytes
```

Devuelven la cantidad correspondiente de bytes, usando un factor de conversión de 1024:

```ruby
2.kilobytes   # => 2048
3.megabytes   # => 3145728
3.5.gigabytes # => 3758096384
-4.exabytes   # => -4611686018427387904
```

Las formas singulares tienen un alias para que pueda decir:

```ruby
1.megabyte # => 1048576
```

NOTE: Definido en `active_support/core_ext/numeric/bytes.rb`.
                  
### Time

Permite el uso de cálculos y declaraciones de tiempo, como `45.minutes + 2.hours + 4.weeks`.

Estos métodos usan Time # advance para cálculos de fecha precisos cuando se usa from_now, ago, etc.
así como sumar o restar sus resultados de un objeto Time. Por ejemplo:

```ruby
# equivalent to Time.current.advance(months: 1)
1.month.from_now

# equivalent to Time.current.advance(weeks: 2)
2.weeks.from_now

# equivalent to Time.current.advance(months: 4, weeks: 5)
(4.months + 5.weeks).from_now
```

WARNING. Para otras duraciones, consulte las extensiones de tiempo para `Integer`.

NOTE: Definido en `active_support/core_ext/numeric/time.rb`.
                  
### Formatting

Habilita el formateo de números de varias formas.

Producir una representación de cadena de un número como un número de teléfono:

```ruby
5551234.to_s(:phone)
# => 555-1234
1235551234.to_s(:phone)
# => 123-555-1234
1235551234.to_s(:phone, area_code: true)
# => (123) 555-1234
1235551234.to_s(:phone, delimiter: " ")
# => 123 555 1234
1235551234.to_s(:phone, area_code: true, extension: 555)
# => (123) 555-1234 x 555
1235551234.to_s(:phone, country_code: 1)
# => +1-123-555-1234
```

Produzca una representación de cadena de un número como moneda:

```ruby
1234567890.50.to_s(:currency)                 # => $1,234,567,890.50
1234567890.506.to_s(:currency)                # => $1,234,567,890.51
1234567890.506.to_s(:currency, precision: 3)  # => $1,234,567,890.506
```

Produzca una representación de cadena de un número como porcentaje:

```ruby
100.to_s(:percentage)
# => 100.000%
100.to_s(:percentage, precision: 0)
# => 100%
1000.to_s(:percentage, delimiter: '.', separator: ',')
# => 1.000,000%
302.24398923423.to_s(:percentage, precision: 5)
# => 302.24399%
```

Produce una representación de cadena de un número en forma delimitada:

```ruby
12345678.to_s(:delimited)                     # => 12,345,678
12345678.05.to_s(:delimited)                  # => 12,345,678.05
12345678.to_s(:delimited, delimiter: ".")     # => 12.345.678
12345678.to_s(:delimited, delimiter: ",")     # => 12,345,678
12345678.05.to_s(:delimited, separator: " ")  # => 12,345,678 05
```

Produce una representación de cadena de un número redondeado con una precisión:

```ruby
123.to_s(:human_size)                  # => 123 Bytes
1234.to_s(:human_size)                 # => 1.21 KB
12345.to_s(:human_size)                # => 12.1 KB
1234567.to_s(:human_size)              # => 1.18 MB
1234567890.to_s(:human_size)           # => 1.15 GB
1234567890123.to_s(:human_size)        # => 1.12 TB
1234567890123456.to_s(:human_size)     # => 1.1 PB
1234567890123456789.to_s(:human_size)  # => 1.07 EB
```

Produzca una representación de cadena de un número en palabras legibles por humanos:

```ruby
123.to_s(:human)               # => "123"
1234.to_s(:human)              # => "1.23 Thousand"
12345.to_s(:human)             # => "12.3 Thousand"
1234567.to_s(:human)           # => "1.23 Million"
1234567890.to_s(:human)        # => "1.23 Billion"
1234567890123.to_s(:human)     # => "1.23 Trillion"
1234567890123456.to_s(:human)  # => "1.23 Quadrillion"
```

NOTE: Definido en `active_support/core_ext/numeric/conversions.rb`.
                  
Extensions to `Integer`
-----------------------

### `multiple_of?`

El método `multiple_of?` Prueba si un número entero es múltiplo del argumento:

```ruby
2.multiple_of?(1) # => true
1.multiple_of?(2) # => false
```

NOTE: Definido en `active_support/core_ext/integer/multiple.rb`.
                  
### `ordinal`

El método `ordinal` devuelve la cadena de sufijo ordinal correspondiente al entero receptor:

```ruby
1.ordinal    # => "st"
2.ordinal    # => "nd"
53.ordinal   # => "rd"
2009.ordinal # => "th"
-21.ordinal  # => "st"
-134.ordinal # => "th"
```

NOTE: Definido en `active_support/core_ext/integer/inflections.rb`.
                  
### `ordinalize`

El método `ordinalize` devuelve la cadena ordinal correspondiente al entero receptor. En comparación, tenga en cuenta que el método `ordinal` devuelve ** solo ** la cadena de sufijo.

```ruby
1.ordinalize    # => "1st"
2.ordinalize    # => "2nd"
53.ordinalize   # => "53rd"
2009.ordinalize # => "2009th"
-21.ordinalize  # => "-21st"
-134.ordinalize # => "-134th"
```

NOTE: Definido en `active_support/core_ext/integer/inflections.rb`.
                  
### Time

Habilita el uso de declaraciones y cálculos de tiempo, como `4.months + 5.years`.

Estos métodos usan Time # advance para cálculos de fecha precisos cuando se usa from_now, ago, etc.
así como sumar o restar sus resultados de un objeto Time. Por ejemplo:

```ruby
# equivalent to Time.current.advance(months: 1)
1.month.from_now

# equivalent to Time.current.advance(years: 2)
2.years.from_now

# equivalent to Time.current.advance(months: 4, years: 5)
(4.months + 5.years).from_now
```

WARNING. Para otras duraciones, consulte las extensiones de tiempo para `Numeric`.

NOTE: Definido en `active_support/core_ext/integer/time.rb`.
                  
Extensions to `BigDecimal`
--------------------------
### `to_s`

El método `to_s` proporciona un especificador predeterminado de" F ". Esto significa que una simple llamada a `to_s` dará como resultado una representación de punto flotante en lugar de una notación de ingeniería:

```ruby
BigDecimal(5.00, 6).to_s       # => "5.0"
```

y que también se admiten especificadores de símbolos:

```ruby
BigDecimal(5.00, 6).to_s(:db)  # => "5.0"
```

Aún se admite la notación de ingeniería:

```ruby
BigDecimal(5.00, 6).to_s("e")  # => "0.5E1"
```

Extensions to `Enumerable`
--------------------------

### `sum`

El método `sum` agrega los elementos de un enumerable:

```ruby
[1, 2, 3].sum # => 6
(1..100).sum  # => 5050
```

La adición solo asume que los elementos responden a '+':

```ruby
[[1, 2], [2, 3], [3, 4]].sum    # => [1, 2, 2, 3, 3, 4]
%w(foo bar baz).sum             # => "foobarbaz"
{a: 1, b: 2, c: 3}.sum          # => [:a, 1, :b, 2, :c, 3]
```

La suma de una colección vacía es cero de forma predeterminada, pero es personalizable:

```ruby
[].sum    # => 0
[].sum(1) # => 1
```

Si se proporciona un bloque, `sum` se convierte en un iterador que produce los elementos de la colección y suma los valores devueltos:

```ruby
(1..5).sum {|n| n * 2 } # => 30
[2, 4, 6, 8, 10].sum    # => 30
```

La suma de un receptor vacío también se puede personalizar en este formulario:

```ruby
[].sum(1) {|n| n**3} # => 1
```

NOTE: Definido en `active_support/core_ext/enumerable.rb`.
                  
### `index_by`

El método `index_by` genera un hash con los elementos de un enumerable indexado por alguna clave.

Repite la colección y pasa cada elemento a un bloque. El elemento estará codificado por el valor devuelto por el bloque:

```ruby
invoices.index_by(&:number)
# => {'2009-032' => <Invoice ...>, '2009-008' => <Invoice ...>, ...}
```

WARNING. Las claves normalmente deberían ser únicas. Si el bloque devuelve el mismo valor para diferentes elementos, no se crea una colección para esa clave. El último artículo ganará.

NOTE: Definido en `active_support/core_ext/enumerable.rb`.
                  
### `index_with`

El método `index_with` genera un hash con los elementos de un enumerable como claves. El valor
es un valor predeterminado pasado o devuelto en un bloque.

```ruby
post = Post.new(title: "hey there", body: "what's up?")

%i( title body ).index_with { |attr_name| post.public_send(attr_name) }
# => { title: "hey there", body: "what's up?" }

WEEKDAYS.index_with(Interval.all_day)
# => { monday: [ 0, 1440 ], … }
```

NOTE: Definido en `active_support/core_ext/enumerable.rb`.
                  
### `many?`

El método `many?` Es una abreviatura de `collection.size > 1`:

```erb
<% if pages.many? %>
  <%= pagination_links %>
<% end %>
```

Si se da un bloque opcional, `many?` Solo tiene en cuenta aquellos elementos que devuelven verdadero:

```ruby
@see_more = videos.many? {|video| video.category == params[:category]}
```

NOTE: Definido en `active_support/core_ext/enumerable.rb`.
                  
### `exclude?`

El predicado `exclude?` Comprueba si un objeto dado ** no ** pertenece a la colección. Es la negación del `include?` incorporado:

```ruby
to_visit << node if visited.exclude?(node)
```

NOTE: Definido en `active_support/core_ext/enumerable.rb`.
                  
### `including`

El método `including` devuelve un nuevo enumerable que incluye los elementos pasados:

```ruby
[ 1, 2, 3 ].including(4, 5)                    # => [ 1, 2, 3, 4, 5 ]
["David", "Rafael"].including %w[ Aaron Todd ] # => ["David", "Rafael", "Aaron", "Todd"]
```
NOTE: Definido en `active_support/core_ext/enumerable.rb`.
                  
### `excluding`

El método `excluding` devuelve una copia de un enumerable con los elementos especificados
remoto:

```ruby
["David", "Rafael", "Aaron", "Todd"].excluding("Aaron", "Todd") # => ["David", "Rafael"]
```

`excluding` tiene el alias de`without`.

NOTE: Definido en `active_support/core_ext/enumerable.rb`.
                  
### `pluck`

El método `pluck` extrae la clave dada de cada elemento:

```ruby
[{ name: "David" }, { name: "Rafael" }, { name: "Aaron" }].pluck(:name) # => ["David", "Rafael", "Aaron"]
[{ id: 1, name: "David" }, { id: 2, name: "Rafael" }].pluck(:id, :name) # => [[1, "David"], [2, "Rafael"]]
```

NOTE: Definido en `active_support/core_ext/enumerable.rb`.
                  
### `pick`

El método `pick` extrae la clave dada del primer elemento:

```ruby
[{ name: "David" }, { name: "Rafael" }, { name: "Aaron" }].pick(:name) # => "David"
[{ id: 1, name: "David" }, { id: 2, name: "Rafael" }].pick(:id, :name) # => [1, "David"]
```

NOTE: Definido en `active_support/core_ext/enumerable.rb`.
                  
Extensions to `Array`
---------------------

### Accessing

Active Support aumenta la API de matrices para facilitar ciertas formas de acceder a ellas. Por ejemplo, `to` devuelve el subarreglo de elementos hasta el del índice pasado:

```ruby
%w(a b c d).to(2) # => ["a", "b", "c"]
[].to(7)          # => []
```

De manera similar, `from` devuelve la cola del elemento en el índice pasado hasta el final. Si el índice es mayor que la longitud de la matriz, devuelve una matriz vacía.

```ruby
%w(a b c d).from(2)  # => ["c", "d"]
%w(a b c d).from(10) # => []
[].from(0)           # => []
```

El método `including` devuelve una nueva matriz que incluye los elementos pasados:

```ruby
[ 1, 2, 3 ].including(4, 5)          # => [ 1, 2, 3, 4, 5 ]
[ [ 0, 1 ] ].including([ [ 1, 0 ] ]) # => [ [ 0, 1 ], [ 1, 0 ] ]
```

El método `including` devuelve una copia del Array excluyendo los elementos especificados.
Esta es una optimización de `Enumerable#excluding` que usa` Array#-`
en lugar de "Array # rechazar" por motivos de rendimiento.

```ruby
["David", "Rafael", "Aaron", "Todd"].excluding("Aaron", "Todd") # => ["David", "Rafael"]
[ [ 0, 1 ], [ 1, 0 ] ].excluding([ [ 1, 0 ] ])                  # => [ [ 0, 1 ] ]
```

Los métodos `second`, `third`, `fourth` y `fifth` devuelven el elemento correspondiente, al igual que `second_to_last` y `third_to_last` (`first` y `last` están integrados). Gracias a la sabiduría social y a la constructividad positiva en todos lados, `forty_two` también está disponible.

```ruby
%w(a b c d).third # => "c"
%w(a b c d).fifth # => nil
```

NOTE: Definido en `active_support/core_ext/array/access.rb`.
                  
### Extracting

El método `extract!` Elimina y devuelve los elementos para los que el bloque devuelve un valor verdadero.
Si no se proporciona ningún bloque, se devuelve un enumerador en su lugar.

```ruby
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
odd_numbers = numbers.extract! { |number| number.odd? } # => [1, 3, 5, 7, 9]
numbers # => [0, 2, 4, 6, 8]
```

NOTE: Definido en `active_support/core_ext/array/extract.rb`.
                 
### Options Extraction

Cuando el último argumento en una llamada a un método es un hash, excepto quizás por un argumento `&block`, Ruby le permite omitir los corchetes:


```ruby
User.exists?(email: params[:email])
```

Ese azúcar sintáctico se usa mucho en Rails para evitar argumentos posicionales donde habría demasiados, ofreciendo en su lugar interfaces que emulan parámetros con nombre. En particular, es muy idiomático usar un hash final para las opciones.

Sin embargo, si un método espera un número variable de argumentos y usa `*` en su declaración, dicho hash de opciones termina siendo un elemento de la matriz de argumentos, donde pierde su función.

En esos casos, puede darle a un hash de opciones un tratamiento distinguido con `extract_options!`. Este método verifica el tipo del último elemento de una matriz. Si es un hash, lo abre y lo devuelve; de ​​lo contrario, devuelve un hash vacío.

Veamos, por ejemplo, la definición de la macro del controlador `caches_action`:

```ruby
def caches_action(*actions)
  return unless cache_configured?
  options = actions.extract_options!
  ...
end
```

Este método recibe un número arbitrario de nombres de acción y un hash opcional de opciones como último argumento. Con la llamada a `extract_options!` Obtienes el hash de opciones y lo eliminas de `actions` de una manera simple y explícita.

NOTE: Definido en `active_support/core_ext/array/extract_options.rb`.
                  
### Conversions

#### `to_sentence`

El método `to_sentence` convierte una matriz en una cadena que contiene una oración que enumera sus elementos:

```ruby
%w().to_sentence                # => ""
%w(Earth).to_sentence           # => "Earth"
%w(Earth Wind).to_sentence      # => "Earth and Wind"
%w(Earth Wind Fire).to_sentence # => "Earth, Wind, and Fire"
```

Este método acepta tres opciones:

* `:two_words_connector`: Lo que se utiliza para matrices de longitud 2. El valor predeterminado es "y".
* `:words_connector`: Lo que se usa para unir los elementos de arreglos con 3 o más elementos, excepto los dos últimos. El valor predeterminado es ",".
* `:last_word_connector`: Qué se usa para unir los últimos elementos de una matriz con 3 o más elementos. El valor predeterminado es "y".

Los valores predeterminados para estas opciones se pueden localizar, sus claves son:

| Option                 | I18n key                            |
| ---------------------- | ----------------------------------- |
| `:two_words_connector` | `support.array.two_words_connector` |
| `:words_connector`     | `support.array.words_connector`     |
| `:last_word_connector` | `support.array.last_word_connector` |

NOTE: Definido en `active_support/core_ext/array/conversions.rb`.
                  
#### `to_formatted_s`

El método `to_formatted_s` actúa como` to_s` por defecto.

Sin embargo, si la matriz contiene elementos que responden a `id`, el símbolo
`:db` se puede pasar como argumento. Eso se usa típicamente con
colecciones de objetos Active Record. Las cadenas devueltas son:

```ruby
[].to_formatted_s(:db)            # => "null"
[user].to_formatted_s(:db)        # => "8456"
invoice.lines.to_formatted_s(:db) # => "23,567,556,12"
```

Se supone que los números enteros en el ejemplo anterior provienen de las respectivas llamadas a `id`.

NOTE: Definido en `active_support/core_ext/array/conversions.rb`.
                 
#### `to_xml`

El método `to_xml` devuelve una cadena que contiene una representación XML de su receptor:

```ruby
Contributor.limit(2).order(:rank).to_xml
# =>
# <?xml version="1.0" encoding="UTF-8"?>
# <contributors type="array">
#   <contributor>
#     <id type="integer">4356</id>
#     <name>Jeremy Kemper</name>
#     <rank type="integer">1</rank>
#     <url-id>jeremy-kemper</url-id>
#   </contributor>
#   <contributor>
#     <id type="integer">4404</id>
#     <name>David Heinemeier Hansson</name>
#     <rank type="integer">2</rank>
#     <url-id>david-heinemeier-hansson</url-id>
#   </contributor>
# </contributors>
```

Para hacerlo, envía `to_xml` a cada elemento por turno y recopila los resultados en un nodo raíz. Todos los elementos deben responder a `to_xml`; de lo contrario, se genera una excepción.

Por defecto, el nombre del elemento raíz es el plural subrayado y discontinuo del nombre de la clase del primer elemento, siempre que el resto de elementos pertenezcan a ese tipo (marcados con `is_a?`) Y no sean hashes. En el ejemplo anterior, eso es "contribuyentes".

Si hay algún elemento que no pertenece al tipo del primero, el nodo raíz se convierte en "objetos":

```ruby
[Contributor.first, Commit.first].to_xml
# =>
# <?xml version="1.0" encoding="UTF-8"?>
# <objects type="array">
#   <object>
#     <id type="integer">4583</id>
#     <name>Aaron Batalion</name>
#     <rank type="integer">53</rank>
#     <url-id>aaron-batalion</url-id>
#   </object>
#   <object>
#     <author>Joshua Peek</author>
#     <authored-timestamp type="datetime">2009-09-02T16:44:36Z</authored-timestamp>
#     <branch>origin/master</branch>
#     <committed-timestamp type="datetime">2009-09-02T16:44:36Z</committed-timestamp>
#     <committer>Joshua Peek</committer>
#     <git-show nil="true"></git-show>
#     <id type="integer">190316</id>
#     <imported-from-svn type="boolean">false</imported-from-svn>
#     <message>Kill AMo observing wrap_with_notifications since ARes was only using it</message>
#     <sha1>723a47bfb3708f968821bc969a9a3fc873a3ed58</sha1>
#   </object>
# </objects>
```

Si el receptor es una matriz de hashes, el elemento raíz por defecto también es "objetos":

```ruby
[{a: 1, b: 2}, {c: 3}].to_xml
# =>
# <?xml version="1.0" encoding="UTF-8"?>
# <objects type="array">
#   <object>
#     <b type="integer">2</b>
#     <a type="integer">1</a>
#   </object>
#   <object>
#     <c type="integer">3</c>
#   </object>
# </objects>
```

WARNING. Si la colección está vacía, el elemento raíz es por defecto "nil-classes". Eso es un problema, por ejemplo, el elemento raíz de la lista de contribuyentes anterior no sería "contribuyentes" si la colección estuviera vacía, sino "nil-classes". Puede usar la opción `:root` para asegurar un elemento raíz consistente.

El nombre de los nodos hijos es por defecto el nombre del nodo raíz singularizado. En los ejemplos anteriores, hemos visto "contribuyente" y "objeto". La opción `:children` le permite establecer estos nombres de nodo.

El constructor XML predeterminado es una instancia nueva de `Builder::XmlMarkup`. Puede configurar su propio constructor a través de la opción `:builder`. El método también acepta opciones como `:dasherize` y amigos, se reenvían al constructor:

```ruby
Contributor.limit(2).order(:rank).to_xml(skip_types: true)
# =>
# <?xml version="1.0" encoding="UTF-8"?>
# <contributors>
#   <contributor>
#     <id>4356</id>
#     <name>Jeremy Kemper</name>
#     <rank>1</rank>
#     <url-id>jeremy-kemper</url-id>
#   </contributor>
#   <contributor>
#     <id>4404</id>
#     <name>David Heinemeier Hansson</name>
#     <rank>2</rank>
#     <url-id>david-heinemeier-hansson</url-id>
#   </contributor>
# </contributors>
```

NOTE: Definido en `active_support/core_ext/array/conversions.rb`.
                  
### Wrapping

El método `Array.wrap` envuelve su argumento en una matriz a menos que ya sea una matriz (o similar a una matriz).

Específicamente:

* Si el argumento es `nil`, se devuelve una matriz vacía.
* De lo contrario, si el argumento responde a `to_ary` se invoca, y si el valor de` to_ary` no es `nil`, se devuelve.
* De lo contrario, se devuelve una matriz con el argumento como elemento único.

```ruby
Array.wrap(nil)       # => []
Array.wrap([1, 2, 3]) # => [1, 2, 3]
Array.wrap(0)         # => [0]
```

Este método tiene un propósito similar al de `Kernel#Array`, pero hay algunas diferencias:

* Si el argumento responde a `to_ary`, se invoca el método. `Kernel#Array` pasa a intentar `to_a` si el valor devuelto es `nil`, pero `Array.wrap` devuelve una matriz con el argumento como su único elemento de inmediato.
* Si el valor devuelto de `to_ary` no es ni` nil` ni un objeto `Array`, `Kernel#Array` genera una excepción, mientras que `Array.wrap` no lo hace, simplemente devuelve el valor.
* No llama a `to_a` en el argumento, si el argumento no responde a` to_ary`, devuelve una matriz con el argumento como su único elemento.

El último punto es particularmente digno de comparar para algunos enumerables:

```ruby
Array.wrap(foo: :bar) # => [{:foo=>:bar}]
Array(foo: :bar)      # => [[:foo, :bar]]
```

También hay un modismo relacionado que usa el operador splat:

```ruby
[*object]
```

NOTE: Definido en `active_support/core_ext/array/wrap.rb`.
                  
### Duplicating

El método `Array#deep_dup` se duplica a sí mismo y a todos los objetos dentro
de forma recursiva con el método de soporte activo `Object#deep_dup`. Funciona como `Array#map` con el envío del método `deep_dup` a cada objeto dentro.

```ruby
array = [1, [2, 3]]
dup = array.deep_dup
dup[1][2] = 4
array[1][2] == nil   # => true
```

NOTE: Definido en `active_support/core_ext/object/deep_dup.rb`.
                  
### Grouping

#### `in_groups_of(number, fill_with = nil)`

El método `in_groups_of` divide una matriz en grupos consecutivos de cierto tamaño. Devuelve una matriz con los grupos:

```ruby
[1, 2, 3].in_groups_of(2) # => [[1, 2], [3, nil]]
```

O los cede a su vez si se pasa un bloque:

```html+erb
<% sample.in_groups_of(3) do |a, b, c| %>
  <tr>
    <td><%= a %></td>
    <td><%= b %></td>
    <td><%= c %></td>
  </tr>
<% end %>
```

El primer ejemplo muestra que `in_groups_of` llena el último grupo con tantos elementos` nil` como sea necesario para tener el tamaño solicitado. Puede cambiar este valor de relleno utilizando el segundo argumento opcional:

```ruby
[1, 2, 3].in_groups_of(2, 0) # => [[1, 2], [3, 0]]
```

Y puede decirle al método que no complete el último grupo pasando `false`:

```ruby
[1, 2, 3].in_groups_of(2, false) # => [[1, 2], [3]]
```

Como consecuencia, `false` no se puede utilizar como valor de relleno.

NOTE: Definido en `active_support/core_ext/array/grouping.rb`.

#### `in_groups(number, fill_with = nil)`

El método `in_groups` divide una matriz en un cierto número de grupos. El método devuelve una matriz con los grupos:

```ruby
%w(1 2 3 4 5 6 7).in_groups(3)
# => [["1", "2", "3"], ["4", "5", nil], ["6", "7", nil]]
```

o los cede a su vez si se pasa un bloque:

```ruby
%w(1 2 3 4 5 6 7).in_groups(3) {|group| p group}
["1", "2", "3"]
["4", "5", nil]
["6", "7", nil]
```

Los ejemplos anteriores muestran que `in_groups` llena algunos grupos con un elemento `nil` final según sea necesario. Un grupo puede obtener como máximo uno de estos elementos adicionales, el más a la derecha si lo hay. Y los grupos que los tienen son siempre los últimos.

Puede cambiar este valor de relleno utilizando el segundo argumento opcional:

```ruby
%w(1 2 3 4 5 6 7).in_groups(3, "0")
# => [["1", "2", "3"], ["4", "5", "0"], ["6", "7", "0"]]
```

Y puede decirle al método que no complete los grupos más pequeños pasando `false`:

```ruby
%w(1 2 3 4 5 6 7).in_groups(3, false)
# => [["1", "2", "3"], ["4", "5"], ["6", "7"]]
```

Como consecuencia, "false" no se puede utilizar como valor de relleno.

NOTE: Definido en `active_support/core_ext/array/grouping.rb`.
                  
#### `split(value = nil)`

El método `split` divide una matriz por un separador y devuelve los fragmentos resultantes.

Si se pasa un bloque, los separadores son aquellos elementos de la matriz para los que el bloque devuelve verdadero:

```ruby
(-5..5).to_a.split { |i| i.multiple_of?(4) }
# => [[-5], [-3, -2, -1], [1, 2, 3], [5]]
```

De lo contrario, el valor recibido como argumento, cuyo valor predeterminado es `nil`, es el separador:

```ruby
[0, 1, -5, 1, 1, "foo", "bar"].split(1)
# => [[0], [-5], [], ["foo", "bar"]]
```

TIP: Observe en el ejemplo anterior que los separadores consecutivos dan como resultado matrices vacías.

NOTA: Definido en `active_support/core_ext/array/grouping.rb`.
                  
Extensions to `Hash`
--------------------

### Conversions

#### `to_xml`

El método `to_xml` devuelve una cadena que contiene una representación XML de su receptor:

```ruby
{"foo" => 1, "bar" => 2}.to_xml
# =>
# <?xml version="1.0" encoding="UTF-8"?>
# <hash>
#   <foo type="integer">1</foo>
#   <bar type="integer">2</bar>
# </hash>
```

Para hacerlo, el método recorre los pares y construye nodos que dependen de los _valores_. Dado un par de `clave`,` valor`:

* Si `value` es un hash, hay una llamada recursiva con `key` como `:root`.

* Si `value` es una matriz, hay una llamada recursiva con `key` como `:root`, y` key` singularizada como `: children`.

* Si `valor` es un objeto invocable, debe esperar uno o dos argumentos. Dependiendo de la aridad, el invocable se invoca con el hash `options` como primer argumento con `key` como `:root`, y` key` singularizado como segundo argumento. Su valor de retorno se convierte en un nuevo nodo.

* Si `value` responde a` to_xml`, el método se invoca con `key` como `:root`.

* De lo contrario, se crea un nodo con `key` como etiqueta con una representación de cadena de `value` como nodo de texto. Si `value` es` nil`, se agrega un atributo "nil" establecido en "true". A menos que la opción `:skip_types` exista y sea verdadera, también se agrega un atributo "tipo" de acuerdo con la siguiente asignación:

```ruby
XML_TYPE_NAMES = {
  "Symbol"     => "symbol",
  "Integer"    => "integer",
  "BigDecimal" => "decimal",
  "Float"      => "float",
  "TrueClass"  => "boolean",
  "FalseClass" => "boolean",
  "Date"       => "date",
  "DateTime"   => "datetime",
  "Time"       => "datetime"
}
```

Por defecto, el nodo raíz es "hash", pero eso se puede configurar a través de la opción `:root`.

El constructor XML predeterminado es una instancia nueva de `Builder::XmlMarkup`. Puede configurar su propio constructor con la opción `:constructor`. El método también acepta opciones como `:dasherize` y amigos, se reenvían al constructor.

NOTA: Definido en `active_support/core_ext/hash/conversions.rb`.
                  
### Merging

Ruby tiene un método incorporado `Hash#merge` que fusiona dos hashes:

```ruby
{a: 1, b: 1}.merge(a: 0, c: 2)
# => {:a=>0, :b=>1, :c=>2}
```

Active Support define algunas formas más de fusionar hashes que pueden ser convenientes.

#### `reverse_merge` and `reverse_merge!`

En caso de colisión, la clave en el hash del argumento gana en `merge`. Puede admitir hash de opciones con valores predeterminados de una manera compacta con este modismo:

```ruby
options = {length: 30, omission: "..."}.merge(options)
```

Active Support define `reverse_merge` en caso de que prefiera esta notación alternativa:

```ruby
options = options.reverse_merge(length: 30, omission: "...")
```

Y una versión explosiva `reverse_merge!` Que realiza la fusión en su lugar:

```ruby
options.reverse_merge!(length: 30, omission: "...")
```

WARNING. Tenga en cuenta que `reverse_merge!` Puede cambiar el hash en la persona que llama, lo que puede ser una buena idea o no.

NOTE: Definido en `active_support/core_ext/hash/reverse_merge.rb`.
                  
#### `reverse_update`

El método `reverse_update` es un alias de` reverse_merge!`, Explicado anteriormente.

WARNING. Tenga en cuenta que "reverse_update" no tiene explosión.

NOTE: Definido en `active_support/core_ext/hash/reverse_merge.rb`.
                  
#### `deep_merge` and `deep_merge!`

Como puede ver en el ejemplo anterior, si se encuentra una clave en ambos hashes, gana el valor del que está en el argumento.

Active Support define `Hash#deep_merge`. En una fusión profunda, si se encuentra una clave en ambos hashes y sus valores son hashes a su vez, su _merge_ se convierte en el valor del hash resultante:

```ruby
{a: {b: 1}}.deep_merge(a: {c: 2})
# => {:a=>{:b=>1, :c=>2}}
```

El método `deep_merge!` Realiza una fusión profunda en su lugar.

NOTE: Definido en `active_support/core_ext/hash/deep_merge.rb`.
                  
### Deep duplicating

El método `Hash # deep_dup` se duplica a sí mismo y a todas las claves y valores
dentro de forma recursiva con el método de soporte activo `Object#deep_dup`. Funciona como `Enumerator#each_with_object` con el envío del método `deep_dup` a cada par dentro.

```ruby
hash = { a: 1, b: { c: 2, d: [3, 4] } }

dup = hash.deep_dup
dup[:b][:e] = 5
dup[:b][:d] << 5

hash[:b][:e] == nil      # => true
hash[:b][:d] == [3, 4]   # => true
```
NOTE: Definido en `active_support/core_ext/object/deep_dup.rb`.
                  
### Working with Keys

#### `except` and `except!`

El método `except` devuelve un hash con las claves de la lista de argumentos eliminadas, si están presentes:

```ruby
{a: 1, b: 2}.except(:a) # => {:b=>2}
```

Si el receptor responde a `convert_key`, se llama al método en cada uno de los argumentos. Esto permite que `except` juegue bien con hashes con acceso indiferente, por ejemplo:

```ruby
{a: 1}.with_indifferent_access.except(:a)  # => {}
{a: 1}.with_indifferent_access.except("a") # => {}
```

There's also the bang variant `except!` that removes keys in the very receiver.

NOTE: Definido en `active_support/core_ext/hash/except.rb`.
                  
#### `stringify_keys` and `stringify_keys!`

El método `stringify_keys` devuelve un hash que tiene una versión en cadena de las claves en el receptor. Lo hace enviándoles `to_s`:


```ruby
{nil => nil, 1 => 1, a: :a}.stringify_keys
# => {"" => nil, "1" => 1, "a" => :a}
```

En caso de colisión de claves, el valor será el que se haya insertado más recientemente en el hash:

```ruby
{"a" => 1, a: 2}.stringify_keys
# The result will be
# => {"a"=>2}
```

This method may be useful for example to easily accept both symbols and strings as options. For instance `ActionView::Helpers::FormHelper` defines:

```ruby
def to_check_box_tag(options = {}, checked_value = "1", unchecked_value = "0")
  options = options.stringify_keys
  options["type"] = "checkbox"
  ...
end
```

La segunda línea puede acceder de forma segura a la tecla "tipo" y permitir que el usuario pase ": tipo" o "tipo".

También existe la variante bang `stringify_keys!` Que secuencia las claves en el mismo receptor.

Además de eso, uno puede usar `deep_stringify_keys` y `deep_stringify_keys!` para secuenciar todas las claves en el hash dado y todos los hash anidados en él. Un ejemplo del resultado es:

```ruby
{nil => nil, 1 => 1, nested: {a: 3, 5 => 5}}.deep_stringify_keys
# => {""=>nil, "1"=>1, "nested"=>{"a"=>3, "5"=>5}}
```

NOTE: Definido en `active_support/core_ext/hash/keys.rb`.
                  
#### `symbolize_keys` and `symbolize_keys!`

El método `symbolize_keys` devuelve un hash que tiene una versión simbolizada de las claves en el receptor, cuando es posible. Lo hace enviándoles `to_sym`:

```ruby
{nil => nil, 1 => 1, "a" => "a"}.symbolize_keys
# => {nil=>nil, 1=>1, :a=>"a"}
```

WARNING. Tenga en cuenta que en el ejemplo anterior solo se simbolizó una tecla.

En caso de colisión de claves, el valor será el que se haya insertado más recientemente en el hash:

```ruby
{"a" => 1, a: 2}.symbolize_keys
# The result will be
# => {:a=>2}
```

Este método puede ser útil, por ejemplo, para aceptar fácilmente símbolos y cadenas como opciones. Por ejemplo, `ActionText::TagHelper` define

```ruby
def rich_text_area_tag(name, value = nil, options = {})
  options = options.symbolize_keys

  options[:input] ||= "trix_input_#{ActionText::TagHelper.id += 1}
  ...
end
```

La tercera línea puede acceder de forma segura a la tecla `:input`, y permitir que el usuario pase`: input` o "input".

También existe la variante bang `symbolize_keys!` Que simboliza las teclas en el mismo receptor.

Además de eso, uno puede usar `deep_symbolize_keys` y `deep_symbolize_keys!` para simbolizar todas las claves en el hash dado y todos los hashes anidados en él. Un ejemplo del resultado es:

```ruby
{nil => nil, 1 => 1, "nested" => {"a" => 3, 5 => 5}}.deep_symbolize_keys
# => {nil=>nil, 1=>1, nested:{a:3, 5=>5}}
```

NOTE: Definido en `active_support/core_ext/hash/keys.rb`.
                  
#### `to_options` and `to_options!`

Los métodos `to_options` y` to_options! `Son, respectivamente, alias de `symbolize_keys` y `symbolize_keys!`.

NOTE: Definido en `active_support/core_ext/hash/keys.rb`.
                  
#### `assert_valid_keys`

El método `assert_valid_keys` recibe un número arbitrario de argumentos y comprueba si el receptor tiene alguna clave fuera de esa lista blanca. Si lo hace, se genera ArgumentError.

```ruby
{a: 1}.assert_valid_keys(:a)  # passes
{a: 1}.assert_valid_keys("a") # ArgumentError
```

Active Record no acepta opciones desconocidas al crear asociaciones, por ejemplo. Implementa ese control a través de `assert_valid_keys`.

NOTE: Definido en `active_support/core_ext/hash/keys.rb`.
                  
### Working with Values

#### `deep_transform_values` and `deep_transform_values!`

El método `deep_transform_values` devuelve un nuevo hash con todos los valores convertidos por la operación de bloque. Esto incluye los valores del hash raíz y de todos los hash y matrices anidados.

```ruby
hash = { person: { name: 'Rob', age: '28' } }

hash.deep_transform_values{ |value| value.to_s.upcase }
# => {person: {name: "ROB", age: "28"}}
```
También existe la variante bang `deep_transform_values!` Que convierte destructivamente todos los valores mediante la operación de bloque.

NOTE: Definido en `active_support/core_ext/hash/deep_transform_values.rb`.
                  
### Slicing

El método `slice!` Reemplaza el hash solo con las claves dadas y devuelve un hash que contiene los pares clave/valor eliminados.

```ruby
hash = {a: 1, b: 2}
rest = hash.slice!(:a) # => {:b=>2}
hash                   # => {:a=>1}
```

NOTE: Definido en `active_support/core_ext/hash/slice.rb`.
                  
### Extracting

El método `extract!` Elimina y devuelve los pares clave/valor que coinciden con las claves dadas.

```ruby
hash = {a: 1, b: 2}
rest = hash.extract!(:a) # => {:a=>1}
hash                     # => {:b=>2}
```
El método `extract!` Devuelve la misma subclase de Hash que el receptor.

```ruby
hash = {a: 1, b: 2}.with_indifferent_access
rest = hash.extract!(:a).class
# => ActiveSupport::HashWithIndifferentAccess
```

NOTE: Definido en `active_support/core_ext/hash/slice.rb`.
                   
### Indifferent Access`

El método `with_indifferent_access` devuelve un` ActiveSupport::HashWithIndifferentAccess` de su receptor:

```ruby
{a: 1}.with_indifferent_access["a"] # => 1
```

NOTE: Definido en `active_support/core_ext/hash/indifferent_access.rb`.
                  
Extensions to `Regexp`
----------------------

### `multiline?`

El método `multiline?` Dice si una expresión regular tiene el indicador `/ m` establecido, es decir, si el punto coincide con las nuevas líneas.

```ruby
%r{.}.multiline?  # => false
%r{.}m.multiline? # => true

Regexp.new('.').multiline?                    # => false
Regexp.new('.', Regexp::MULTILINE).multiline? # => true
```

Rails utiliza este método en un solo lugar, también en el código de enrutamiento. Las expresiones regulares multilínea no están permitidas para los requisitos de ruta y este indicador facilita la aplicación de esa restricción.

```ruby
def verify_regexp_requirements(requirements)
  ...
  if requirement.multiline?
    raise ArgumentError, "Regexp multiline option is not allowed in routing requirements: #{requirement.inspect}"
  end
  ...
end
```

NOTE: Definido en `active_support/core_ext/regexp.rb`.

Extensions to `Range`
---------------------

### `to_s`

Active Support extiende el método `Range#to_s` para que comprenda un argumento de formato opcional. En el momento de escribir estas líneas, el único formato no predeterminado admitido es `:db`:
```ruby
(Date.today..Date.tomorrow).to_s
# => "2009-10-25..2009-10-26"

(Date.today..Date.tomorrow).to_s(:db)
# => "BETWEEN '2009-10-25' AND '2009-10-26'"
```

Como muestra el ejemplo, el formato `:db` genera una cláusula SQL` BETWEEN`. Eso lo utiliza Active Record en su apoyo a valores de rango en condiciones.

NOTE: Definido en `active_support/core_ext/range/conversions.rb`.
                  
### `===`, `include?`, and `cover?`

Los métodos `Range#===`, `Range#include?` y `Range#cover?` Dicen si algún valor se encuentra entre los extremos de una instancia determinada:

```ruby
(2..3).include?(Math::E) # => true
```

Active Support amplía estos métodos para que el argumento pueda ser otro rango a su vez. En ese caso, probamos si los extremos del rango del argumento pertenecen al receptor mismo:

```ruby
(1..10) === (3..7)  # => true
(1..10) === (0..7)  # => false
(1..10) === (3..11) # => false
(1...9) === (3..9)  # => false

(1..10).include?(3..7)  # => true
(1..10).include?(0..7)  # => false
(1..10).include?(3..11) # => false
(1...9).include?(3..9)  # => false

(1..10).cover?(3..7)  # => true
(1..10).cover?(0..7)  # => false
(1..10).cover?(3..11) # => false
(1...9).cover?(3..9)  # => false
```

NOTE: Definido en  `active_support/core_ext/range/compare_range.rb`.
                  
### `overlaps?`

El método `Range#overlaps?` dice si dos rangos dados tienen una intersección no nula:        

```ruby
(1..10).overlaps?(7..11)  # => true
(1..10).overlaps?(0..7)   # => true
(1..10).overlaps?(11..27) # => false
```

NOTE: Definido en `active_support/core_ext/range/overlaps.rb`.
                  
Extensions to `Date`
--------------------

### Calculations

INFO: Los siguientes métodos de cálculo tienen casos extremos en octubre de 1582, ya que los días 5..14 simplemente no existen. Esta guía no documenta su comportamiento en esos días por brevedad, pero basta con decir que hacen lo que cabría esperar. Es decir, `Date.new(1582, 10, 4).Tomorrow` devuelve` Date.new (1582, 10, 15) `y así sucesivamente. Compruebe `test/core_ext/date_ext_test.rb` en el conjunto de pruebas de Active Support para conocer el comportamiento esperado.

#### `Date.current`

Active Support define "Date.current" como hoy en la zona horaria actual. Eso es como `Date.today`, excepto que respeta la zona horaria del usuario, si está definida. También define `Date.yesterday` y `Date.tomorrow`, y la instancia predica `past?`, `today?`, `tomorrow?`, `next_day?`, `yesterday?`, `prev_day?`, `future?`, `on_weekday?` y `on_weekend?`, todos ellos relativos a `Date.current`.

Cuando realice comparaciones de fechas utilizando métodos que respeten la zona horaria del usuario, asegúrese de utilizar `Date.current` y no `Date.today`. Hay casos en los que la zona horaria del usuario podría estar en el futuro en comparación con la zona horaria del sistema, que `Date.today` usa de forma predeterminada. Esto significa que `Date.today` puede ser igual a `Date.yesterday`.

NOTE: Definido en `active_support/core_ext/date/calculations.rb`.
                  
#### Named dates

##### `beginning_of_week`, `end_of_week`

Los métodos `begin_of_week` y `end_of_week` devuelven las fechas para
principio y fin de semana, respectivamente. Se supone que las semanas comienzan en
Lunes, pero eso se puede cambiar pasando un argumento, configurando el hilo local
`Date.beginning_of_week` o `config.beginning_of_week`.

```ruby
d = Date.new(2010, 5, 8)     # => Sat, 08 May 2010
d.beginning_of_week          # => Mon, 03 May 2010
d.beginning_of_week(:sunday) # => Sun, 02 May 2010
d.end_of_week                # => Sun, 09 May 2010
d.end_of_week(:sunday)       # => Sat, 08 May 2010
```

`beginning_of_week` tiene el alias de `at_beginning_of_week` y `end_of_week` tiene el alias de`at_end_of_week`.

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.

##### `monday`, `sunday`

Los métodos `monday` y `sunday` devuelven las fechas del lunes anterior y
el próximo domingo, respectivamente.

```ruby
d = Date.new(2010, 5, 8)     # => Sat, 08 May 2010
d.monday                     # => Mon, 03 May 2010
d.sunday                     # => Sun, 09 May 2010

d = Date.new(2012, 9, 10)    # => Mon, 10 Sep 2012
d.monday                     # => Mon, 10 Sep 2012

d = Date.new(2012, 9, 16)    # => Sun, 16 Sep 2012
d.sunday                     # => Sun, 16 Sep 2012
```

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.
                  
##### `prev_week`, `next_week`

El método `next_week` recibe un símbolo con un nombre de día en inglés (por defecto es el hilo local `Date.beginning_of_week`, o `config.beginning_of_week`, o `:monday`) y devuelve la fecha correspondiente a ese día.

```ruby
d = Date.new(2010, 5, 9) # => Sun, 09 May 2010
d.next_week              # => Mon, 10 May 2010
d.next_week(:saturday)   # => Sat, 15 May 2010
```

El método `prev_week` es análogo:

```ruby
d.prev_week              # => Mon, 26 Apr 2010
d.prev_week(:saturday)   # => Sat, 01 May 2010
d.prev_week(:friday)     # => Fri, 30 Apr 2010
```

`prev_week` tiene el alias de `last_week`.

Tanto `next_week` como `prev_week` funcionan como se esperaba cuando se configuran `Date.beginning_of_week` o `config.beginning_of_week`.

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.
                  
##### `beginning_of_month`, `end_of_month`

Los métodos `begin_of_month` y `end_of_month` devuelven las fechas para el comienzo y el final del mes:

```ruby
d = Date.new(2010, 5, 9) # => Sun, 09 May 2010
d.beginning_of_month     # => Sat, 01 May 2010
d.end_of_month           # => Mon, 31 May 2010
```

`begin_of_month` tiene un alias de `at_beginning_of_month`, y `end_of_month` tiene un alias de `at_end_of_month`.

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.
                  
##### `beginning_of_quarter`, `end_of_quarter`

Los métodos `begin_of_quarter` y `end_of_quarter` devuelven las fechas para el inicio y el final del trimestre del año calendario del receptor:

```ruby
d = Date.new(2010, 5, 9) # => Sun, 09 May 2010
d.beginning_of_quarter   # => Thu, 01 Apr 2010
d.end_of_quarter         # => Wed, 30 Jun 2010
```

`begin_of_quarter` tiene el alias de `at_beginning_of_quarter`, y `end_of_quarter` tiene el alias de `at_end_of_quarter`.

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.
                  
##### `beginning_of_year`, `end_of_year`

Los métodos `begin_of_year` y `end_of_year` devuelven las fechas para el comienzo y el final del año:


```ruby
d = Date.new(2010, 5, 9) # => Sun, 09 May 2010
d.beginning_of_year      # => Fri, 01 Jan 2010
d.end_of_year            # => Fri, 31 Dec 2010
```

`beginning_of_year` is aliased to `at_beginning_of_year`, and `end_of_year` is aliased to `at_end_of_year`.

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.
                  
#### Other Date Computations

##### `years_ago`, `years_since`

El método `years_ago` recibe un número de años y devuelve la misma fecha que hace muchos años:

```ruby
date = Date.new(2010, 6, 7)
date.years_ago(10) # => Wed, 07 Jun 2000
```

`years_since` moves forward in time:

```ruby
date = Date.new(2010, 6, 7)
date.years_since(10) # => Sun, 07 Jun 2020
```

If such a day does not exist, the last day of the corresponding month is returned:

```ruby
Date.new(2012, 2, 29).years_ago(3)     # => Sat, 28 Feb 2009
Date.new(2012, 2, 29).years_since(3)   # => Sat, 28 Feb 2015
```

`last_year` is short-hand for `#years_ago(1)`.

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.
                  
##### `months_ago`, `months_since`

Los métodos `months_ago` y `months_since` funcionan de manera análoga durante meses:


```ruby
Date.new(2010, 4, 30).months_ago(2)   # => Sun, 28 Feb 2010
Date.new(2010, 4, 30).months_since(2) # => Wed, 30 Jun 2010
```

Si ese día no existe, se devuelve el último día del mes correspondiente:

```ruby
Date.new(2010, 4, 30).months_ago(2)    # => Sun, 28 Feb 2010
Date.new(2009, 12, 31).months_since(2) # => Sun, 28 Feb 2010
```

`last_month` is short-hand for `#months_ago(1)`.

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.
                  
##### `weeks_ago`

The method `weeks_ago` works analogously for weeks:

```ruby
Date.new(2010, 5, 24).weeks_ago(1)    # => Mon, 17 May 2010
Date.new(2010, 5, 24).weeks_ago(2)    # => Mon, 10 May 2010
```

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.
                  
##### `advance`

La forma más genérica de saltar a otros días es "avance". Este método recibe un hash con las claves `: años`,`: meses`, `: semanas`,`: días`, y devuelve una fecha avanzada tanto como las claves presentes indican:

```ruby
date = Date.new(2010, 6, 6)
date.advance(years: 1, weeks: 2)  # => Mon, 20 Jun 2011
date.advance(months: 2, days: -2) # => Wed, 04 Aug 2010
```

Tenga en cuenta en el ejemplo anterior que los incrementos pueden ser negativos.

Para realizar el cálculo, el método primero incrementa años, luego meses, luego semanas y finalmente días. Este orden es importante hacia finales de meses. Digamos, por ejemplo, que estamos a finales de febrero de 2010 y queremos avanzar un mes y un día.

El método `advance` avanza primero un mes, y luego un día, el resultado es:

```ruby
Date.new(2010, 2, 28).advance(months: 1, days: 1)
# => Sun, 29 Mar 2010
```

Mientras que si lo hiciera al revés el resultado sería diferente:

```ruby
Date.new(2010, 2, 28).advance(days: 1).advance(months: 1)
# => Thu, 01 Apr 2010
```


NOTE: Definido en `active_support/core_ext/date/calculations.rb`.
                  
#### Changing Components

El método `change` le permite obtener una nueva fecha que es la misma que la del receptor, excepto para el año, mes o día dado:


```ruby
Date.new(2010, 12, 23).change(year: 2011, month: 11)
# => Wed, 23 Nov 2011
```

Este método no es tolerante con fechas inexistentes, si el cambio no es válido, se genera un error de argumento:

```ruby
Date.new(2010, 1, 31).change(month: 2)
# => ArgumentError: invalid date
```

NOTE: Definido en `active_support/core_ext/date/calculations.rb`.
                 
#### Durations

Las duraciones se pueden sumar y restar de las fechas:

```ruby
d = Date.current
# => Mon, 09 Aug 2010
d + 1.year
# => Tue, 09 Aug 2011
d - 3.hours
# => Sun, 08 Aug 2010 21:00:00 UTC +00:00
```

Se traducen en llamadas a `since` o `advance`. Por ejemplo aquí obtenemos el salto correcto en la reforma del calendario:

```ruby
Date.new(1582, 10, 4) + 1.day
# => Fri, 15 Oct 1582
```

#### Timestamps

INFO: Los siguientes métodos devuelven un objeto `Time` si es posible, de lo contrario un` DateTime`. Si se configura, respetan la zona horaria del usuario.

##### `beginning_of_day`, `end_of_day`

El método `begin_of_day` devuelve una marca de tiempo al comienzo del día (00:00:00):


```ruby
date = Date.new(2010, 6, 7)
date.beginning_of_day # => Mon Jun 07 00:00:00 +0200 2010
```

The method `end_of_day` returns a timestamp at the end of the day (23:59:59):

```ruby
date = Date.new(2010, 6, 7)
date.end_of_day # => Mon Jun 07 23:59:59 +0200 2010
```

`begin_of_day` tiene el alias de `at_beginning_of_day`, `midnight`, `at_midnight`.

NOTE: Definido en `active_support/core_ext/date/calculations.rb`.
                  
##### `beginning_of_hour`, `end_of_hour`

El método `begin_of_hour` devuelve una marca de tiempo al comienzo de la hora (hh: 00: 00):

```ruby
date = DateTime.new(2010, 6, 7, 19, 55, 25)
date.beginning_of_hour # => Mon Jun 07 19:00:00 +0200 2010
```

El método `end_of_hour` devuelve una marca de tiempo al final de la hora (hh: 59: 59):

```ruby
date = DateTime.new(2010, 6, 7, 19, 55, 25)
date.end_of_hour # => Mon Jun 07 19:59:59 +0200 2010
```

`begin_of_hour` tiene el alias de `at_beginning_of_hour`.

NOTE: Definido en `active_support/core_ext/date_time/calculations.rb`.
                  
##### `beginning_of_minute`, `end_of_minute`

The method `beginning_of_minute` returns a timestamp at the beginning of the minute (hh:mm:00):

```ruby
date = DateTime.new(2010, 6, 7, 19, 55, 25)
date.beginning_of_minute # => Mon Jun 07 19:55:00 +0200 2010
```

El método `end_of_minute` devuelve una marca de tiempo al final del minuto (hh: mm: 59):

```ruby
date = DateTime.new(2010, 6, 7, 19, 55, 25)
date.end_of_minute # => Mon Jun 07 19:55:59 +0200 2010
```

`begin_of_minute` tiene el alias de `at_beginning_of_minute`.

INFO: `begin_of_hour`, `end_of_hour`, `begin_of_minute` y` end_of_minute` se implementan para `Time` y `DateTime` pero **no** `Date` ya que no tiene sentido solicitar el comienzo o el final de un hora o minuto en una instancia de `Date`.

##### `ago`, `since`

El método `ago` recibe una cantidad de segundos como argumento y devuelve una marca de tiempo de hace muchos segundos desde la medianoche:

```ruby
date = Date.current # => Fri, 11 Jun 2010
date.ago(1)         # => Thu, 10 Jun 2010 23:59:59 EDT -04:00
```

Del mismo modo, `since` avanza:

```ruby
date = Date.current # => Fri, 11 Jun 2010
date.since(1)       # => Fri, 11 Jun 2010 00:00:01 EDT -04:00
```

NOTE: Definido en `active_support/core_ext/date/calculations.rb`.
                  
#### Other Time Computations

### Conversions

Extensions to `DateTime`
------------------------

WARNING: `DateTime` no conoce las reglas de DST, por lo que algunos de estos métodos tienen casos extremos cuando se produce un cambio de DST. Por ejemplo, `seconds_since_midnight` podría no devolver la cantidad real en ese día.

### Calculations

La clase `DateTime` es una subclase de ` Date` por lo que al cargar `active_support/core_ext/date/calculations.rb` hereda estos métodos y sus alias, excepto que siempre devolverán fechas y horas.

Los siguientes métodos se han vuelto a implementar para que ** no ** necesite cargar `active_support/core_ext/date/calculations.rb` para estos:

```ruby
beginning_of_day (midnight, at_midnight, at_beginning_of_day)
end_of_day
ago
since (in)
```

Por otro lado, `advance` y `change` también se definen y admiten más opciones, que se documentan a continuación.

Los siguientes métodos solo se implementan en `active_support / core_ext / date_time / calculations.rb` ya que solo tienen sentido cuando se usan con una instancia de `DateTime`:

```ruby
beginning_of_hour (at_beginning_of_hour)
end_of_hour
```

#### Named Datetimes

##### `DateTime.current`

Active Support define `DateTime.current` como `Time.now.to_datetime`, excepto que respeta la zona horaria del usuario, si está definida. También define `DateTime.yesterday` y` DateTime.tomorrow`, y la instancia predica `past?` y `future?` relativo a `DateTime.current`.

NOTE: Definido en `active_support/core_ext/date_time/calculations.rb`.
                  
#### Other Extensions

##### `seconds_since_midnight`

El método `seconds_since_midnight` devuelve el número de segundos desde la medianoche:

```ruby
now = DateTime.current     # => Mon, 07 Jun 2010 20:26:36 +0000
now.seconds_since_midnight # => 73596
```

NOTE: Definido en `active_support/core_ext/date_time/calculations.rb`.
                  
##### `utc`

El método `utc` le da la misma fecha y hora en el receptor expresada en UTC.

```ruby
now = DateTime.current # => Mon, 07 Jun 2010 19:27:52 -0400
now.utc                # => Mon, 07 Jun 2010 23:27:52 +0000
```

Este método también tiene un alias como "getutc".

NOTE: Definido `active_support/core_ext/date_time/calculations.rb`.
               
##### `utc?`

El predicado `utc?` dice si el receptor tiene UTC como zona horaria:

```ruby
now = DateTime.now # => Mon, 07 Jun 2010 19:30:47 -0400
now.utc?           # => false
now.utc.utc?       # => true
```

NOTE: Definido en `active_support/core_ext/date_time/calculations.rb`.
                  
##### `advance`

La forma más genérica de saltar a otros días es `advance`. Este método recibe un hash con las claves `:years`, `:months`, `:weeks`, `:days`, y devuelve una fecha avanzada tanto como las claves presentes indican:

```ruby
d = DateTime.current
# => Thu, 05 Aug 2010 11:33:31 +0000
d.advance(years: 1, months: 1, days: 1, hours: 1, minutes: 1, seconds: 1)
# => Tue, 06 Sep 2011 12:34:32 +0000
```

Este método primero calcula la fecha de destino pasando `:years`, `:months`, `:weeks`, y `:days` hasta la `Date#advance` documentada anteriormente. Después de eso, ajusta el tiempo llamando "desde" con el número de segundos para avanzar. Este orden es relevante, un orden diferente daría diferentes fechas y horas en algunos casos extremos. Se aplica el ejemplo de `Date # advance`, y podemos ampliarlo para mostrar la relevancia del orden relacionada con los bits de tiempo.

Si primero movemos los bits de fecha (que también tienen un orden relativo de procesamiento, como se documentó anteriormente), y luego los bits de tiempo obtenemos, por ejemplo, el siguiente cálculo:

```ruby
d = DateTime.new(2010, 2, 28, 23, 59, 59)
# => Sun, 28 Feb 2010 23:59:59 +0000
d.advance(months: 1, seconds: 1)
# => Mon, 29 Mar 2010 00:00:00 +0000
```

pero si los calculamos al revés, el resultado sería diferente:

```ruby
d.advance(seconds: 1).advance(months: 1)
# => Thu, 01 Apr 2010 00:00:00 +0000
```

WARNING: Dado que "DateTime" no tiene en cuenta el horario de verano, puede terminar en un punto no existente en el tiempo sin ninguna advertencia o error que se lo indique.

NOTE: Definido en `active_support/core_ext/date_time/calculations.rb`.

#### Changing Components

El método `change` le permite obtener una nueva fecha y hora que es la misma que la del receptor excepto por las opciones dadas, que pueden incluir `:year`, `:month`, `:day`, `:hour`, `:min`, `:sec`, `:offset`, `:start`:

```ruby
now = DateTime.current
# => Tue, 08 Jun 2010 01:56:22 +0000
now.change(year: 2011, offset: Rational(-6, 24))
# => Wed, 08 Jun 2011 01:56:22 -0600
```

Si las horas se ponen a cero, los minutos y los segundos también (a menos que hayan dado valores):

```ruby
now.change(hour: 0)
# => Tue, 08 Jun 2010 00:00:00 +0000
```

Del mismo modo, si los minutos se ponen a cero, los segundos también (a menos que se haya dado un valor):

```ruby
now.change(min: 0)
# => Tue, 08 Jun 2010 01:00:00 +0000
```

Este método no es tolerante con fechas inexistentes, si el cambio no es válido, se genera un error de argumento:

```ruby
DateTime.current.change(month: 2, day: 30)
# => ArgumentError: invalid date
```

NOTE: Definido en active_support/core_ext/date_time/calculations.rb`.
                  
#### Durations

Las duraciones se pueden sumar y restar de las fechas y horas:

```ruby
now = DateTime.current
# => Mon, 09 Aug 2010 23:15:17 +0000
now + 1.year
# => Tue, 09 Aug 2011 23:15:17 +0000
now - 1.week
# => Mon, 02 Aug 2010 23:15:17 +0000
```

Se traducen en llamadas a `since` o `advance`. Por ejemplo aquí obtenemos el salto correcto en la reforma del calendario:

```ruby
DateTime.new(1582, 10, 4, 23) + 1.hour
# => Fri, 15 Oct 1582 00:00:00 +0000
```

Extensions to `Time`
--------------------

### Calculations

Son análogos. Consulte la documentación anterior y tenga en cuenta las siguientes diferencias:

* `change` acepta una opción adicional `:usec`.
* `Time` comprende el horario de verano, por lo que obtiene los cálculos correctos de DST como en

```ruby
Time.zone_default
# => #<ActiveSupport::TimeZone:0x7f73654d4f38 @utc_offset=nil, @name="Madrid", ...>

# In Barcelona, 2010/03/28 02:00 +0100 becomes 2010/03/28 03:00 +0200 due to DST.
t = Time.local(2010, 3, 28, 1, 59, 59)
# => Sun Mar 28 01:59:59 +0100 2010
t.advance(seconds: 1)
# => Sun Mar 28 03:00:00 +0200 2010
```

* Si `since` o` ago` salta a una hora que no se puede expresar con `Time`, se devuelve un objeto` DateTime` en su lugar.

#### `Time.current`

Active Support define "Time.current" como hoy en la zona horaria actual. Eso es como "Time.now", excepto que respeta la zona horaria del usuario, si está definida. También define los predicados de instancia `past?`, `today?`, `tomorrow?`, `next_day?`, `yesterday?`, `prev_day?` y `future?`, Todos ellos relativos a `Time.current`.

Al hacer comparaciones de tiempo utilizando métodos que respeten la zona horaria del usuario, asegúrese de utilizar `Time.current` en lugar de` Time.now`. Hay casos en los que la zona horaria del usuario podría estar en el futuro en comparación con la zona horaria del sistema, que `Time.now` utiliza de forma predeterminada. Esto significa que "Time.now.to_date" puede ser igual a "Date.yesterday".

NOTE: Definido en `active_support/core_ext/time/calculations.rb`.

#### `all_day`, `all_week`, `all_month`, `all_quarter` and `all_year`

El método `all_day` devuelve un rango que representa el día completo de la hora actual.

```ruby
now = Time.current
# => Mon, 09 Aug 2010 23:20:05 UTC +00:00
now.all_day
# => Mon, 09 Aug 2010 00:00:00 UTC +00:00..Mon, 09 Aug 2010 23:59:59 UTC +00:00
```

De manera análoga, `all_week`,` all_month`, `all_quarter` y `all_year` sirven para generar rangos de tiempo.

```ruby
now = Time.current
# => Mon, 09 Aug 2010 23:20:05 UTC +00:00
now.all_week
# => Mon, 09 Aug 2010 00:00:00 UTC +00:00..Sun, 15 Aug 2010 23:59:59 UTC +00:00
now.all_week(:sunday)
# => Sun, 16 Sep 2012 00:00:00 UTC +00:00..Sat, 22 Sep 2012 23:59:59 UTC +00:00
now.all_month
# => Sat, 01 Aug 2010 00:00:00 UTC +00:00..Tue, 31 Aug 2010 23:59:59 UTC +00:00
now.all_quarter
# => Thu, 01 Jul 2010 00:00:00 UTC +00:00..Thu, 30 Sep 2010 23:59:59 UTC +00:00
now.all_year
# => Fri, 01 Jan 2010 00:00:00 UTC +00:00..Fri, 31 Dec 2010 23:59:59 UTC +00:00
```
NOTE: Defined in `active_support/core_ext/date_and_time/calculations.rb`.

#### `prev_day`, `next_day`

`prev_day` y `next_day` devuelven la hora del último día o del día siguiente:


```ruby
t = Time.new(2010, 5, 8) # => 2010-05-08 00:00:00 +0900
t.prev_day               # => 2010-05-07 00:00:00 +0900
t.next_day               # => 2010-05-09 00:00:00 +0900
```

NOTE: Definido en  `active_support/core_ext/time/calculations.rb`.

#### `prev_month`, `next_month`

`prev_month` y `next_month` devuelven la hora con el mismo día en el último o el próximo mes:

```ruby
t = Time.new(2010, 5, 8) # => 2010-05-08 00:00:00 +0900
t.prev_month             # => 2010-04-08 00:00:00 +0900
t.next_month             # => 2010-06-08 00:00:00 +0900
```

Si ese día no existe, se devuelve el último día del mes correspondiente:

```ruby
Time.new(2000, 5, 31).prev_month # => 2000-04-30 00:00:00 +0900
Time.new(2000, 3, 31).prev_month # => 2000-02-29 00:00:00 +0900
Time.new(2000, 5, 31).next_month # => 2000-06-30 00:00:00 +0900
Time.new(2000, 1, 31).next_month # => 2000-02-29 00:00:00 +0900
```

NOTE: Definido en  `active_support/core_ext/time/calculations.rb`.

#### `prev_year`, `next_year`

`prev_year` y `next_year` devuelven una hora con el mismo día / mes en el último o el próximo año:

```ruby
t = Time.new(2010, 5, 8) # => 2010-05-08 00:00:00 +0900
t.prev_year              # => 2009-05-08 00:00:00 +0900
t.next_year              # => 2011-05-08 00:00:00 +0900
```

Si la fecha es el 29 de febrero de un año bisiesto, obtiene el 28:

```ruby
t = Time.new(2000, 2, 29) # => 2000-02-29 00:00:00 +0900
t.prev_year               # => 1999-02-28 00:00:00 +0900
t.next_year               # => 2001-02-28 00:00:00 +0900
```

NOTE: Definido en `active_support/core_ext/time/calculations.rb`.
                  
#### `prev_quarter`, `next_quarter`

`prev_quarter` and `next_quarter` return the date with the same day in the previous or next quarter:

```ruby
t = Time.local(2010, 5, 8) # => 2010-05-08 00:00:00 +0300
t.prev_quarter             # => 2010-02-08 00:00:00 +0200
t.next_quarter             # => 2010-08-08 00:00:00 +0300
```

Si ese día no existe, se devuelve el último día del mes correspondiente:

```ruby
Time.local(2000, 7, 31).prev_quarter  # => 2000-04-30 00:00:00 +0300
Time.local(2000, 5, 31).prev_quarter  # => 2000-02-29 00:00:00 +0200
Time.local(2000, 10, 31).prev_quarter # => 2000-07-31 00:00:00 +0300
Time.local(2000, 11, 31).next_quarter # => 2001-03-01 00:00:00 +0200
```

`prev_quarter` is aliased to `last_quarter`.

NOTE: Definido en `active_support/core_ext/date_and_time/calculations.rb`.
                  
### Time Constructors

Active Support define `Time.current` como `Time.zone.now` si hay una zona horaria definida por el usuario, con un retorno a `Time.now`:

```ruby
Time.zone_default
# => #<ActiveSupport::TimeZone:0x7f73654d4f38 @utc_offset=nil, @name="Madrid", ...>
Time.current
# => Fri, 06 Aug 2010 17:11:58 CEST +02:00
```

De manera análoga a `DateTime`, los predicados `past?` y `future?` Son relativos a `Time.current`.

#### Durations

Las duraciones se pueden sumar y restar de los objetos de tiempo:

```ruby
now = Time.current
# => Mon, 09 Aug 2010 23:20:05 UTC +00:00
now + 1.year
# => Tue, 09 Aug 2011 23:21:11 UTC +00:00
now - 1.week
# => Mon, 02 Aug 2010 23:21:11 UTC +00:00
```

Se traducen en llamadas a `since` o `advance`. Por ejemplo aquí obtenemos el salto correcto en la reforma del calendario:

```ruby
Time.utc(1582, 10, 3) + 5.days
# => Mon Oct 18 00:00:00 UTC 1582
```

Extensions to `File`
--------------------

### `atomic_write`

Con el método de clase `File.atomic_write` puedes escribir en un archivo de una manera que evitará que cualquier lector vea contenido a medio escribir.

El nombre del archivo se pasa como argumento y el método produce un identificador de archivo abierto para escritura. Una vez que el bloque está hecho, atomic_write cierra el identificador del archivo y completa su trabajo.

For example, Action Pack uses this method to write asset cache files like `all.css`:

```ruby
File.atomic_write(joined_asset_path) do |cache|
  cache.write(join_asset_file_contents(asset_paths))
end
```

Para lograr esto, atomic_write crea un archivo temporal. Ese es el archivo en el que realmente escribe el código del bloque. Al finalizar, se cambia el nombre del archivo temporal, que es una operación atómica en los sistemas POSIX. Si el archivo de destino existe, `atomic_write` lo sobrescribe y conserva los propietarios y los permisos. Sin embargo, hay algunos casos en los que atomic_write no puede cambiar la propiedad o los permisos del archivo, este error se detecta y se omite confiando en el usuario/sistema de archivos para garantizar que el archivo sea accesible para los procesos que lo necesitan.

NOTE. Debido a la operación chmod que realiza `atomic_write`, si el archivo de destino tiene una ACL configurada, esta ACL será recalculada / modificada.

WARNING. Tenga en cuenta que no puede agregar con `atomic_write`.

El archivo auxiliar está escrito en un directorio estándar para archivos temporales, pero puede pasar un directorio de su elección como segundo argumento.

NOTE: Definido en `active_support/core_ext/file/atomic.rb`.
            
Extensions to `Marshal`
-----------------------

### `load`

Active Support agrega soporte constante de carga automática a `load`.

Por ejemplo, el almacén de caché de archivos se deserializa de esta manera:

```ruby
File.open(file_name) { |f| Marshal.load(f) }
```

Si los datos almacenados en caché hacen referencia a una constante desconocida en ese momento, se activa el mecanismo de carga automática y, si tiene éxito, la deserialización se vuelve a intentar de forma transparente.

ADVERTENCIA. Si el argumento es un `IO`, debe responder a `rewind` para poder volver a intentarlo. Los archivos normales responden a `rewind`.

NOTE: Definido en `active_support/core_ext/marshal.rb`.

Extensions to `NameError`
-------------------------

Active Support agrega `missing_name?` a `NameError`, que prueba si la excepción se generó debido al nombre pasado como argumento.

El nombre puede darse como símbolo o cadena. Un símbolo se compara con el nombre de constante simple, una cadena se compara con el nombre de constante completo.

TIP: Un símbolo puede representar un nombre de constante totalmente calificado como en `:" ActiveRecord :: Base "`, por lo que el comportamiento de los símbolos se define por conveniencia, no porque tenga que ser así técnicamente.

Por ejemplo, cuando una acción de `ArticlesController` se llama, Rails intenta de forma optimista usar `ArticlesHelper`. Está bien que el módulo auxiliar no exista, por lo que si se genera una excepción para ese nombre constante, se debe silenciar. Pero podría darse el caso de que `articles_helper.rb` genere un` NameError` debido a una constante desconocida real. Eso debería volver a plantearse. El método `missing_name?` Proporciona una forma de distinguir ambos casos:

```ruby
def default_helper_module!
  module_name = name.sub(/Controller$/, '')
  module_path = module_name.underscore
  helper module_path
rescue LoadError => e
  raise e unless e.is_missing? "helpers/#{module_path}_helper"
rescue NameError => e
  raise e unless e.missing_name? "#{module_name}Helper"
end
```

NOTE: Definido en active_support/core_ext/name_error.rb`.
                  
Extensions to `LoadError`
-------------------------

Active Support agrega `is_missing?` a `LoadError`.

Dado un nombre de ruta, `is_missing?` Prueba si la excepción se generó debido a ese archivo en particular (excepto quizás por la extensión ".rb").

Por ejemplo, cuando una acción de `ArticlesController` se llama, Rails intenta cargar `articles_helper.rb`, pero es posible que ese archivo no exista. Está bien, el módulo de ayuda no es obligatorio, por lo que Rails silencia un error de carga. Pero podría darse el caso de que el módulo auxiliar exista y, a su vez, requiera otra biblioteca que falte. En ese caso, Rails debe volver a generar la excepción. El método `is_missing?` Proporciona una forma de distinguir ambos casos:

```ruby
def default_helper_module!
  module_name = name.sub(/Controller$/, '')
  module_path = module_name.underscore
  helper module_path
rescue LoadError => e
  raise e unless e.is_missing? "helpers/#{module_path}_helper"
rescue NameError => e
  raise e unless e.missing_name? "#{module_name}Helper"
end
```

NOTE: Definido en `active_support/core_ext/load_error.rb`.
