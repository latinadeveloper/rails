**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Active Job Basics
=================

Este guía le proporciona todo lo que necesita para comenzar a crear,
poner en cola y ejecutar trabajos en segundo plano.

Después de leer esta guía, sabrá:

* Cómo crear empleo.
* Cómo poner trabajos en cola.
* Cómo ejecutar trabajos en segundo plano.
* Cómo enviar correos electrónicos desde su aplicación de forma asincrónica.

--------------------------------------------------------------------------------


What is Active Job?
-------------------

Active Job es un marco para declarar trabajos y hacer que se ejecuten en una variedad
de backends de cola. Estos trabajos pueden ser de cualquier cosa, desde ser programas que regularmente
dan limpiezas, gastos de facturación, envíos postales. Todo lo que pueda cortarse
en pequeñas unidades de trabajo y ejecutar en paralelo, de verdad.

The Purpose of Active Job
-----------------------------

El punto principal es garantizar que todas las aplicaciones de Rails tengan una infraestructura de trabajo
en su lugar. Luego podemos tener características de marco y otras gemas construidas sobre eso,
sin tener que preocuparse por las diferencias de API entre varios corredores de trabajo, como
trabajo retrasado y rescates. Elegir su backend de cola se vuelve más como una preocupación operacional.
Y podrás cambiar entre ellos sin tener que volver a escribir
sus trabajos.

NOTE: Rails viene de forma predeterminada con una implementación de cola asincrónica que
ejecuta trabajos con un grupo de subprocesos en proceso. Los trabajos se ejecutarán de forma asincrónica, pero cualquiera
de los trabajos de la cola se eliminarán al reiniciar.

Creating a Job
--------------

Esta sección proporcionará una guía paso a paso para crear un trabajo y ponerlo en cola.

### Create the Job

Active Job proporciona un generador de Rails para crear trabajos. Lo siguiente creará un
trabajo en `app/jobs` (con un caso de prueba adjunto en `test/jobs`):

```bash
$ bin/rails generate job guests_cleanup
invoke  test_unit
create    test/jobs/guests_cleanup_job_test.rb
create  app/jobs/guests_cleanup_job.rb
```

También puede crear un trabajo que se ejecutará en una cola específica:

```bash
$ bin/rails generate job guests_cleanup --queue urgent
```

Si no desea utilizar un generador, puede crear su propio archivo dentro de
`app/jobs`, solo asegúrate de que herede de `ApplicationJob`.

Así es como se ve un trabajo:

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :default

  def perform(*guests)
    # Do something later
  end
end
```

Tenga en cuenta que puede definir `perform` con tantos argumentos como desee.

### Enqueue the Job

Ponga en cola un trabajo como este:


```ruby
# Enqueue a job to be performed as soon as the queuing system is
# free.
GuestsCleanupJob.perform_later guest
```

```ruby
# Enqueue a job to be performed tomorrow at noon.
GuestsCleanupJob.set(wait_until: Date.tomorrow.noon).perform_later(guest)
```

```ruby
# Enqueue a job to be performed 1 week from now.
GuestsCleanupJob.set(wait: 1.week).perform_later(guest)
```

```ruby
# `perform_now` and `perform_later` will call `perform` under the hood so
# you can pass as many arguments as defined in the latter.
GuestsCleanupJob.perform_later(guest1, guest2, filter: 'some_filter')
```

¡Eso es todo!

Job Execution
-------------

Para poner en cola y ejecutar trabajos en producción, debe configurar un backend de cola,
es decir, debe decidir qué biblioteca de colas de terceros Rails debe de usar.
Rails solo proporciona un sistema de cola en proceso, que solo mantiene los trabajos en RAM.
Si el proceso falla o la máquina se reinicia, todos los trabajos pendientes se pierden con el
backend asincrónico predeterminado. Esto puede estar bien para aplicaciones más pequeñas o trabajos que no son críticos,
pero la mayoría de las aplicaciones de producción deberán elegir un backend persistente.

### Backends

Active Job tiene adaptadores integrados para múltiples backends de cola (Sidekiq,
Resque, Delayed Job y otros). Para obtener una lista actualizada de los adaptadores
consulte la documentación de la API para [ActiveJob::QueueAdapters](https://api.rubyonrails.org/classes/ActiveJob/QueueAdapters.html).

### Setting the Backend

Puede configurar fácilmente su backend de cola:

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    # Be sure to have the adapter's gem in your Gemfile
    # and follow the adapter's specific installation
    # and deployment instructions.
    config.active_job.queue_adapter = :sidekiq
  end
end
```

También puede configurar su backend por trabajo.

```ruby
class GuestsCleanupJob < ApplicationJob
  self.queue_adapter = :resque
  #....
end

# Now your job will use `resque` as its backend queue adapter overriding what
# was configured in `config.active_job.queue_adapter`.
```

### Starting the Backend

Dado que los trabajos se ejecutan en paralelo a su aplicación Rails, la mayoría de las bibliotecas de colas
requieren que inicie un servicio de cola específico de la biblioteca (además de
iniciando su aplicación Rails) para que funcione el procesamiento del trabajo. Consulte la biblioteca
documentación para obtener instrucciones sobre cómo iniciar el backend de la cola.

Here is a noncomprehensive list of documentation:

- [Sidekiq](https://github.com/mperham/sidekiq/wiki/Active-Job)
- [Resque](https://github.com/resque/resque/wiki/ActiveJob)
- [Sneakers](https://github.com/jondot/sneakers/wiki/How-To:-Rails-Background-Jobs-with-ActiveJob)
- [Sucker Punch](https://github.com/brandonhilkert/sucker_punch#active-job)
- [Queue Classic](https://github.com/QueueClassic/queue_classic#active-job)
- [Delayed Job](https://github.com/collectiveidea/delayed_job#active-job)
- [Que](https://github.com/que-rb/que#additional-rails-specific-setup)

Queues
------

La mayoría de los adaptadores admiten varias colas. Con Active Job puede programar
el trabajo para ejecutar en una cola específica:

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :low_priority
  #....
end
```

Puede prefijar el nombre de la cola para todos sus trabajos usando
`config.active_job.queue_name_prefix` en` application.rb`:

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    config.active_job.queue_name_prefix = Rails.env
  end
end

# app/jobs/guests_cleanup_job.rb
class GuestsCleanupJob < ApplicationJob
  queue_as :low_priority
  #....
end

# Now your job will run on queue production_low_priority on your
# production environment and on staging_low_priority
# on your staging environment
```

También puede configurar el prefijo por trabajo.


```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :low_priority
  self.queue_name_prefix = nil
  #....
end

# Now your job's queue won't be prefixed, overriding what
# was configured in `config.active_job.queue_name_prefix`.
```

El delimitador de prefijo de nombre de cola predeterminado es '\ _'. Esto se puede cambiar configurando
`config.active_job.queue_name_delimiter` en `application.rb`:


```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    config.active_job.queue_name_prefix = Rails.env
    config.active_job.queue_name_delimiter = '.'
  end
end

# app/jobs/guests_cleanup_job.rb
class GuestsCleanupJob < ApplicationJob
  queue_as :low_priority
  #....
end

# Now your job will run on queue production.low_priority on your
# production environment and on staging.low_priority
# on your staging environment
```

Si desea tener más control sobre qué cola se ejecutará un trabajo, puede pasar un `: queue`
opción a `#set`:

```ruby
MyJob.set(queue: :another_queue).perform_later(record)
```

Para controlar la cola desde el nivel de trabajo, puede pasar un bloque a `#queue_as`.
El bloque se ejecutará en el contexto del trabajo (para que pueda acceder a `self.arguments`)
y debe devolver el nombre de la cola:

```ruby
class ProcessVideoJob < ApplicationJob
  queue_as do
    video = self.arguments.first
    if video.owner.premium?
      :premium_videojobs
    else
      :videojobs
    end
  end

  def perform(video)
    # Do process video
  end
end

ProcessVideoJob.perform_later(Video.last)
```

NOTE: Asegúrese de que su backend de cola "escuche" en su nombre de cola. Para algunos
backends que necesita para especificar las colas para escuchar.

Callbacks
---------

Active Job proporciona enlaces para activar la lógica durante el ciclo de vida de un trabajo. Me gusta
otras devoluciones de llamada en Rails, puede implementar las devoluciones de llamada como métodos ordinarios
y use un método de clase de estilo macro para registrarlos como devoluciones de llamada:

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :default

  around_perform :around_cleanup

  def perform
    # Do something later
  end

  private
    def around_cleanup
      # Do something before perform
      yield
      # Do something after perform
    end
end
```

Los métodos de clase de estilo macro también pueden recibir un bloque. Considere usar esto
style si el código dentro de su bloque es tan corto que cabe en una sola línea.
Por ejemplo, puede enviar métricas para cada trabajo en cola:

```ruby
class ApplicationJob < ActiveJob::Base
  before_enqueue { |job| $statsd.increment "#{job.class.name.underscore}.enqueue" }
end
```

### Available callbacks

* `before_enqueue`
* `around_enqueue`
* `after_enqueue`
* `before_perform`
* `around_perform`
* `after_perform`


Action Mailer
------------

Uno de los trabajos más comunes en una aplicación web moderna es enviar correos electrónicos al exterior.
del ciclo de solicitud-respuesta, por lo que el usuario no tiene que esperar. Trabajo activo
está integrado con Action Mailer para que pueda enviar correos electrónicos fácilmente de forma asincrónica:

```ruby
# If you want to send the email now use #deliver_now
UserMailer.welcome(@user).deliver_now

# If you want to send the email through Active Job use #deliver_later
UserMailer.welcome(@user).deliver_later
```

NOTE: El uso de la cola asincrónica de una tarea Rake (por ejemplo, para
enviar un correo electrónico usando `.deliver_later`) generalmente no funcionará porque Rake lo hará
probable final, lo que hace que se elimine el grupo de subprocesos en proceso, antes de cualquier/todos
de los correos electrónicos `.deliver_later` se procesan. Para evitar este problema, utilice
`.deliver_now` o ejecutar una cola persistente en desarrollo.

Internationalization
--------------------

Cada trabajo usa el conjunto `I18n.locale` cuando se creó el trabajo. Útil si envías
correos electrónicos de forma asincrónica:

```ruby
I18n.locale = :eo

UserMailer.welcome(@user).deliver_later # Email will be localized to Esperanto.
```


Supported types for arguments
----------------------------

ActiveJob admite los siguientes tipos de argumentos de forma predeterminada:

  - Tipos basicos (`NilClass`, `String`, `Integer`, `Float`, `BigDecimal`, `TrueClass`, `FalseClass`)
  - `Symbol`
  - `Date`
  - `Time`
  - `DateTime`
  - `ActiveSupport::TimeWithZone`
  - `ActiveSupport::Duration`
  - `Hash` (Las claves deben ser de `String` o tipo`Symbol`)
  - `ActiveSupport::HashWithIndifferentAccess`
  - `Array`
  - `Module`
  - `Class`

### GlobalID

Active Job admite GlobalID para parámetros. Esto hace posible pasar en vivo
Active Record objetos a su trabajo en lugar de pares de clase/id, que luego tiene
para deserializar manualmente. Antes, los trabajos se verían así:

```ruby
class TrashableCleanupJob < ApplicationJob
  def perform(trashable_class, trashable_id, depth)
    trashable = trashable_class.constantize.find(trashable_id)
    trashable.cleanup(depth)
  end
end
```

Ahora puedes simplemente hacer:

```ruby
class TrashableCleanupJob < ApplicationJob
  def perform(trashable, depth)
    trashable.cleanup(depth)
  end
end
```

Esto funciona con cualquier clase que se mezcle en `GlobalID::Identificación`, que
por defecto se ha mezclado con las clases Active Record.

### Serializers

Puede ampliar la lista de tipos de argumentos admitidos. Solo necesita definir su propio serializador:

```ruby
class MoneySerializer < ActiveJob::Serializers::ObjectSerializer
  # Checks if an argument should be serialized by this serializer.
  def serialize?(argument)
    argument.is_a? Money
  end

  # Converts an object to a simpler representative using supported object types.
  # The recommended representative is a Hash with a specific key. Keys can be of basic types only.
  # You should call `super` to add the custom serializer type to the hash.
  def serialize(money)
    super(
      "amount" => money.amount,
      "currency" => money.currency
    )
  end

  # Converts serialized value into a proper object.
  def deserialize(hash)
    Money.new(hash["amount"], hash["currency"])
  end
end
```

y agregue este serializador a la lista:

```ruby
Rails.application.config.active_job.custom_serializers << MoneySerializer
```

Exceptions
----------

Active Job proporciona una forma de detectar excepciones generadas durante la ejecución del
trabajo:

```ruby
class GuestsCleanupJob < ApplicationJob
  queue_as :default

  rescue_from(ActiveRecord::RecordNotFound) do |exception|
    # Do something with the exception
  end

  def perform
    # Do something later
  end
end
```

Si la excepción no se rescata dentro del trabajo, p. Ej. como se muestra arriba, el trabajo se denomina "fallido".

### Retrying or Discarding failed jobs

Un trabajo fallido no se reintentará, a menos que se configure de otra manera.

También es posible reintentar o descartar un trabajo si se genera una excepción durante la ejecución.
Por ejemplo:

```ruby
class RemoteServiceJob < ApplicationJob
  retry_on CustomAppException # defaults to 3s wait, 5 attempts

  discard_on ActiveJob::DeserializationError

  def perform(*args)
    # Might raise CustomAppException or ActiveJob::DeserializationError
  end
end
```

Para obtener más detalles, consulte la documentación de la API para [ActiveJob::Exceptions](https://api.rubyonrails.org/classes/ActiveJob/Exceptions/ClassMethods.html).

### Deserialization

GlobalID permite serializar objetos Active Record completos pasados ​​a `# perform`.

Si un registro aprobado se elimina después de que el trabajo se pone en cola pero antes del `# perform`
El método se llama Active Job generará un `ActiveJob::DeserializationError`
excepción.

Job Testing
--------------

Puede encontrar instrucciones detalladas sobre cómo probar sus trabajos en el
[testing guide](testing.html#testing-jobs).
