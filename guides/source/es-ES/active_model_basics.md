**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Conceptos Básicos del Active Model
==================================

Esta guía debe proporcionarle todo lo que necesita para comenzar a usar el modelo
clases Active Model permite que los ayudantes de Action Pack interactúen con
Objetos simples de Ruby. Active Model también ayuda a construir ORM personalizados para su uso
fuera del marco de Rails.

Después de leer esta guía, sabrá:

* Cómo se comporta un modelo de registro activo.
* Cómo funcionan las devoluciones de llamada y las validaciones.
* Cómo funcionan los serializadores.
* Cómo se integra Active Model con el marco de internacionalización de Rails (i18n).

--------------------------------------------------------------------------------

What is Active Model?
---------------------

Active Model es una biblioteca que contiene varios módulos utilizados en el desarrollo
clases que necesitan algunas características presentes en Active Record.
Algunos de estos módulos se explican a continuación.

### Attribute Methods

El módulo `ActiveModel::AttributeMethods` puede agregar prefijos y sufijos personalizados
sobre los métodos de una clase. Se utiliza definiendo los prefijos y sufijos y
qué métodos en el objeto los usarán.

```ruby
class Person
  include ActiveModel::AttributeMethods

  attribute_method_prefix 'reset_'
  attribute_method_suffix '_highest?'
  define_attribute_methods 'age'

  attr_accessor :age

  private
    def reset_attribute(attribute)
      send("#{attribute}=", 0)
    end

    def attribute_highest?(attribute)
      send(attribute) > 100
    end
end

person = Person.new
person.age = 110
person.age_highest?  # => true
person.reset_age     # => 0
person.age_highest?  # => false
```

### Callbacks

`ActiveModel::Callbacks` ofrece devoluciones de llamada de estilo Active Record. Esto proporciona una
capacidad de definir devoluciones de llamada que se ejecutan en los momentos apropiados.
Después de definir las devoluciones de llamada, puede envolverlas con antes, después y alrededor
métodos personalizados

```ruby
class Person
  extend ActiveModel::Callbacks

  define_model_callbacks :update

  before_update :reset_me

  def update
    run_callbacks(:update) do
      # This method is called when update is called on an object.
    end
  end

  def reset_me
    # This method is called when update is called on an object as a before_update callback is defined.
  end
end
```

### Conversion

Si una clase define los métodos `persisted?` e `id`, entonces puede incluir el
Módulo `ActiveModel::Conversion` en esa clase, y llame a la conversión Rails
métodos en objetos de esa clase.

```ruby
class Person
  include ActiveModel::Conversion

  def persisted?
    false
  end

  def id
    nil
  end
end

person = Person.new
person.to_model == person  # => true
person.to_key              # => nil
person.to_param            # => nil
```

### Dirty

Un objeto se ensucia cuando ha pasado por uno o más cambios en su
atributos y no se ha guardado. `ActiveModel::Dirty` da la capacidad de
compruebe si un objeto ha sido cambiado o no. También tiene atributos basados
métodos de acceso. Consideremos una clase de persona con atributos `first_name`
y `last_name`:

```ruby
class Person
  include ActiveModel::Dirty
  define_attribute_methods :first_name, :last_name

  def first_name
    @first_name
  end

  def first_name=(value)
    first_name_will_change!
    @first_name = value
  end

  def last_name
    @last_name
  end

  def last_name=(value)
    last_name_will_change!
    @last_name = value
  end

  def save
    # do save work...
    changes_applied
  end
end
```

#### Querying object directly for its list of all changed attributes.

```ruby
person = Person.new
person.changed? # => false

person.first_name = "First Name"
person.first_name # => "First Name"

# returns true if any of the attributes have unsaved changes.
person.changed? # => true

# returns a list of attributes that have changed before saving.
person.changed # => ["first_name"]

# returns a Hash of the attributes that have changed with their original values.
person.changed_attributes # => {"first_name"=>nil}

# returns a Hash of changes, with the attribute names as the keys, and the
# values as an array of the old and new values for that field.
person.changes # => {"first_name"=>[nil, "First Name"]}
```

#### Attribute based accessor methods

Rastree si el atributo particular ha sido cambiado o no.

```ruby
# attr_name_changed?
person.first_name # => "First Name"
person.first_name_changed? # => true
```

Rastrea el valor anterior del atributo.

```ruby
# attr_name_was accessor
person.first_name_was # => nil
```

Realice un seguimiento del valor anterior y actual del atributo modificado. Devuelve una matriz
si se cambia, de lo contrario devuelve nil.

```ruby
# attr_name_change
person.first_name_change # => [nil, "First Name"]
person.last_name_change # => nil
```

### Validations

El módulo `ActiveModel::Validations` agrega la capacidad de validar objetos
como en Active Record.

```ruby
class Person
  include ActiveModel::Validations

  attr_accessor :name, :email, :token

  validates :name, presence: true
  validates_format_of :email, with: /\A([^\s]+)((?:[-a-z0-9]\.)[a-z]{2,})\z/i
  validates! :token, presence: true
end

person = Person.new
person.token = "2b1f325"
person.valid?                        # => false
person.name = 'vishnu'
person.email = 'me'
person.valid?                        # => false
person.email = 'me@vishnuatrai.com'
person.valid?                        # => true
person.token = nil
person.valid?                        # => raises ActiveModel::StrictValidationFailed
```

### Naming

`ActiveModel::Naming` agrega una serie de métodos de clase que hacen que la denominación y el enrutamiento
Más fácil de administrar. El módulo define el método de clase `model_name` que
definirá un número de accesos usando algunos métodos `ActiveSupport::Inflector`.

```ruby
class Person
  extend ActiveModel::Naming
end

Person.model_name.name                # => "Person"
Person.model_name.singular            # => "person"
Person.model_name.plural              # => "people"
Person.model_name.element             # => "person"
Person.model_name.human               # => "Person"
Person.model_name.collection          # => "people"
Person.model_name.param_key           # => "person"
Person.model_name.i18n_key            # => :person
Person.model_name.route_key           # => "people"
Person.model_name.singular_route_key  # => "person"
```

### Model

`ActiveModel::Model` agrega la capacidad para que una clase funcione con Action Pack y
Vista de acción desde el primer momento.

```ruby
class EmailContact
  include ActiveModel::Model

  attr_accessor :name, :email, :message
  validates :name, :email, :message, presence: true

  def deliver
    if valid?
      # deliver email
    end
  end
end
```

Al incluir `ActiveModel::Model` obtienes algunas características como:

- nombre del modelo introspección
- conversiones
- traducciones
- validaciones

También le brinda la capacidad de inicializar un objeto con un hash de atributos,
como cualquier objeto de Active Record.

```ruby
email_contact = EmailContact.new(name: 'David',
                                 email: 'david@example.com',
                                 message: 'Hello World')
email_contact.name       # => 'David'
email_contact.email      # => 'david@example.com'
email_contact.valid?     # => true
email_contact.persisted? # => false
```

Cualquier clase que incluya `ActiveModel::Model` puede usarse con` form_with`,
`render` y cualquier otro método auxiliar de Action View, como Active Record
objetos.

### Serialization

`ActiveModel::Serialization` proporciona serialización básica para su objeto.
Debe declarar un atributo Hash que contiene los atributos que desea
publicar por fascículos. Los atributos deben ser cadenas, no símbolos.

```ruby
class Person
  include ActiveModel::Serialization

  attr_accessor :name

  def attributes
    {'name' => nil}
  end
end
```

Ahora puede acceder a un Hash serializado de su objeto utilizando el método `serializable_hash`.


```ruby
person = Person.new
person.serializable_hash   # => {"name"=>nil}
person.name = "Bob"
person.serializable_hash   # => {"name"=>"Bob"}
```

#### ActiveModel::Serializers

Active Model también proporciona el módulo `ActiveModel::Serializers::JSON`
para serialización / deserialización JSON. Este módulo incluye automáticamente el
discutido anteriormente módulo `ActiveModel::Serialization`.

##### ActiveModel::Serializers::

Para usar `ActiveModel::Serializers::JSON` solo necesita cambiar el
módulo que está incluyendo desde `ActiveModel::Serialization` a `ActiveModel::Serializers::JSON`.

```ruby
class Person
  include ActiveModel::Serializers::JSON

  attr_accessor :name

  def attributes
    {'name' => nil}
  end
end
```

El método `as_json`, similar a `serializable_hash`, proporciona un Hash que representa
el modelo.

```ruby
person = Person.new
person.as_json # => {"name"=>nil}
person.name = "Bob"
person.as_json # => {"name"=>"Bob"}
```

También puede definir los atributos para un modelo a partir de una cadena JSON.
Sin embargo, debe definir el método `atributos =` en su clase:

```ruby
class Person
  include ActiveModel::Serializers::JSON

  attr_accessor :name

  def attributes=(hash)
    hash.each do |key, value|
      send("#{key}=", value)
    end
  end

  def attributes
    {'name' => nil}
  end
end
```

Ahora es posible crear una instancia de `Person` y establecer atributos usando `from_json`.

```ruby
json = { name: 'Bob' }.to_json
person = Person.new
person.from_json(json) # => #<Person:0x00000100c773f0 @name="Bob">
person.name            # => "Bob"
```

### Translation

`ActiveModel::Translation` proporciona integración entre su objeto y los rieles
marco de internacionalización (i18n).

```ruby
class Person
  extend ActiveModel::Translation
end
```

Con el método `human_attribute_name`, puede transformar los nombres de atributos en un
formato más legible para los humanos. El formato legible por humanos se define en sus archivos locales.

* config/locales/app.pt-BR.yml

  ```yml
  pt-BR:
    activemodel:
      attributes:
        person:
          name: 'Nome'
  ```

```ruby
Person.human_attribute_name('name') # => "Nome"
```

### Lint Tests

`ActiveModel::Lint::Tests` le permite probar si un objeto cumple con
el API del modelo activo.

* `app/models/person.rb`

    ```ruby
    class Person
      include ActiveModel::Model
    end
    ```

* `test/models/person_test.rb`

    ```ruby
    require "test_helper"

    class PersonTest < ActiveSupport::TestCase
      include ActiveModel::Lint::Tests

      setup do
        @model = Person.new
      end
    end
    ```

```bash
$ bin/rails test

Run options: --seed 14596

# Running:

......

Finished in 0.024899s, 240.9735 runs/s, 1204.8677 assertions/s.

6 runs, 30 assertions, 0 failures, 0 errors, 0 skips
```

No se requiere un objeto para implementar todas las API para trabajar con
Action Pack. Este módulo solo tiene la intención de proporcionar orientación en caso de que desee todo
características fuera de la caja.

### SecurePassword

`ActiveModel::SecurePassword` proporciona una forma de almacenar de forma segura cualquier
contraseña en forma cifrada. Cuando incluye este módulo, se proporciona 
el método de clase `has_secure_password` que define
el acceso de `password` con ciertas validaciones por defecto.

#### Requirements

`ActiveModel::SecurePassword` depende de [`bcrypt`](https://github.com/codahale/bcrypt-ruby 'BCrypt'),
así que incluye esta gema en tu `Gemfile` para usar` ActiveModel::SecurePassword` correctamente.
Para que esto funcione, el modelo debe tener un descriptor de acceso denominado `XXX_digest`.
Donde `XXX` es el nombre del atributo de su contraseña deseada.
Las siguientes validaciones se agregan automáticamente:

1. La contraseña debe estar presente.
2. La contraseña debe ser igual a su confirmación (siempre que se pase `XXX_confirmation`).
3. La longitud máxima de una contraseña es 72 (requerida por `bcrypt` de la que depende ActiveModel::SecurePassword)

#### Examples

```ruby
class Person
  include ActiveModel::SecurePassword
  has_secure_password
  has_secure_password :recovery_password, validations: false

  attr_accessor :password_digest, :recovery_password_digest
end

person = Person.new

# When password is blank.
person.valid? # => false

# When the confirmation doesn't match the password.
person.password = 'aditya'
person.password_confirmation = 'nomatch'
person.valid? # => false

# When the length of password exceeds 72.
person.password = person.password_confirmation = 'a' * 100
person.valid? # => false

# When only password is supplied with no password_confirmation.
person.password = 'aditya'
person.valid? # => true

# When all validations are passed.
person.password = person.password_confirmation = 'aditya'
person.valid? # => true

person.recovery_password = "42password"

person.authenticate('aditya') # => person
person.authenticate('notright') # => false
person.authenticate_password('aditya') # => person
person.authenticate_password('notright') # => false

person.authenticate_recovery_password('42password') # => person
person.authenticate_recovery_password('notright') # => false

person.password_digest # => "$2a$04$gF8RfZdoXHvyTjHhiU4ZsO.kQqV9oonYZu31PRE4hLQn3xM2qkpIy"
person.recovery_password_digest # => "$2a$04$iOfhwahFymCs5weB3BNH/uXkTG65HR.qpW.bNhEjFP3ftli3o5DQC"
```
