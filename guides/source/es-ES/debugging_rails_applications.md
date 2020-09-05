**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Depuración de Applications Rails
================================

Esta guía presenta técnicas para depurar aplicaciones Ruby on Rails.

Después de leer esta guía, sabrá:

* El propósito de la depuración.
* Cómo rastrear problemas y problemas en su aplicación que sus pruebas no identifican.
* Las diferentes formas de depurar.
* Cómo analizar el seguimiento de la pila.

--------------------------------------------------------------------------------

View Helpers for Debugging
--------------------------

Una tarea común es inspeccionar el contenido de una variable. Rails ofrece tres formas diferentes de hacer esto:

* `debug`
* `to_yaml`
* `inspect`

### `debug`

El ayudante `debug` devolverá una etiqueta \ <pre> que muestra el objeto usando el formato YAML. Esto generará datos legibles por humanos a partir de cualquier objeto. Por ejemplo, si tiene este código en una vista:

```html+erb
<%= debug @article %>
<p>
  <b>Title:</b>
  <%= @article.title %>
</p>
```

Verás algo como esto:

```yaml
--- !ruby/object Article
attributes:
  updated_at: 2008-09-05 22:55:47
  body: It's a very helpful guide for debugging your Rails app.
  title: Rails debugging guide
  published: t
  id: "1"
  created_at: 2008-09-05 22:55:47
attributes_cache: {}


Title: Rails debugging guide
```

### `to_yaml

Alternativamente, llamar a `to_yaml` en cualquier objeto lo convierte en YAML. Puede pasar este objeto convertido al método auxiliar `simple_format` para formatear la salida. Así es como `debug` hace su magia.

```html+erb
<%= simple_format @article.to_yaml %>
<p>
  <b>Title:</b>
  <%= @article.title %>
</p>
```

El código anterior generará algo como esto:

```yaml
--- !ruby/object Article
attributes:
updated_at: 2008-09-05 22:55:47
body: It's a very helpful guide for debugging your Rails app.
title: Rails debugging guide
published: t
id: "1"
created_at: 2008-09-05 22:55:47
attributes_cache: {}

Title: Rails debugging guide
```

### `inspect`

Otro método útil para mostrar valores de objetos es `inspect`, especialmente cuando se trabaja con matrices o hashes. Esto imprimirá el valor del objeto como una cadena. Por ejemplo:

```html+erb
<%= [1, 2, 3, 4, 5].inspect %>
<p>
  <b>Title:</b>
  <%= @article.title %>
</p>
```

Rendirá:

```
[1, 2, 3, 4, 5]

Title: Rails debugging guide
```

The Logger
----------

También puede ser útil guardar información en archivos de registro en tiempo de ejecución. Rails mantiene un archivo de registro independiente para cada entorno de ejecución.

### What is the Logger?

Rails hace uso de la clase `ActiveSupport::Logger` para escribir información de registro. También se pueden sustituir otros registradores, como `Log4r`.

Puede especificar un registrador alternativo en `config/application.rb` o cualquier otro archivo de entorno, por ejemplo:

```ruby
config.logger = Logger.new(STDOUT)
config.logger = Log4r::Logger.new("Application Log")
```

O en la sección `Initializer`, agregue _cualquier_ de lo siguiente
```ruby
Rails.logger = Logger.new(STDOUT)
Rails.logger = Log4r::Logger.new("Application Log")
```

TIP: De forma predeterminada, cada registro se crea en `Rails.root/log/`  y el archivo de registro lleva el nombre del entorno en el que se ejecuta la aplicación.

### Log Levels

Los niveles de registro disponibles son: `:debug`, `:info`, `:warn`, `:error`, `:fatal`,
y `:unknown`, correspondiente a los números de nivel de registro de 0 a 5,
respectivamente. Para cambiar el nivel de registro predeterminado, use

```ruby
config.log_level = :warn # In any environment initializer, or
Rails.logger.level = 0 # at any time
```

Esto es útil cuando desea iniciar sesión en desarrollo o puesta en escena sin inundar su registro de producción con información innecesaria.

TIP: El nivel de registro de Rails predeterminado es `debug` en todos los entornos.

### Sending Messages

Para escribir en el registro actual, use el método `logger.(debug|info|warn|error|fatal|unknown)` desde un controlador, modelo o mailer:

```ruby
logger.debug "Person attributes hash: #{@person.attributes.inspect}"
logger.info "Processing the request..."
logger.fatal "Terminating application, raised unrecoverable error!!!"
```

A continuación, se muestra un ejemplo de un método instrumentado con registro adicional:


```ruby
class ArticlesController < ApplicationController
  # ...

  def create
    @article = Article.new(article_params)
    logger.debug "New article: #{@article.attributes.inspect}"
    logger.debug "Article should be valid: #{@article.valid?}"

    if @article.save
      logger.debug "The article was saved and now the user is going to be redirected..."
      redirect_to @article, notice: 'Article was successfully created.'
    else
      render :new
    end
  end

  # ...

  private
    def article_params
      params.require(:article).permit(:title, :body, :published)
    end
end
```

A continuación, se muestra un ejemplo del registro generado cuando se ejecuta esta acción del controlador:

```
Started POST "/articles" for 127.0.0.1 at 2018-10-18 20:09:23 -0400
Processing by ArticlesController#create as HTML
  Parameters: {"utf8"=>"✓", "authenticity_token"=>"XLveDrKzF1SwaiNRPTaMtkrsTzedtebPPkmxEFIU0ordLjICSnXsSNfrdMa4ccyBjuGwnnEiQhEoMN6H1Gtz3A==", "article"=>{"title"=>"Debugging Rails", "body"=>"I'm learning how to print in logs.", "published"=>"0"}, "commit"=>"Create Article"}
New article: {"id"=>nil, "title"=>"Debugging Rails", "body"=>"I'm learning how to print in logs.", "published"=>false, "created_at"=>nil, "updated_at"=>nil}
Article should be valid: true
   (0.0ms)  begin transaction
  ↳ app/controllers/articles_controller.rb:31
  Article Create (0.5ms)  INSERT INTO "articles" ("title", "body", "published", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?)  [["title", "Debugging Rails"], ["body", "I'm learning how to print in logs."], ["published", 0], ["created_at", "2018-10-19 00:09:23.216549"], ["updated_at", "2018-10-19 00:09:23.216549"]]
  ↳ app/controllers/articles_controller.rb:31
   (2.3ms)  commit transaction
  ↳ app/controllers/articles_controller.rb:31
The article was saved and now the user is going to be redirected...
Redirected to http://localhost:3000/articles/1
Completed 302 Found in 4ms (ActiveRecord: 0.8ms)
```

Agregar registros adicionales como este facilita la búsqueda de comportamientos inesperados o inusuales en sus registros. Si agrega registros adicionales, asegúrese de hacer un uso sensato de los niveles de registro para evitar llenar sus registros de producción con trivialidades inútiles.

### Verbose Query Logs

Al observar el resultado de la consulta de la base de datos en los registros, es posible que no quede claro de inmediato por qué se activan varias consultas de la base de datos cuando se llama a un solo método:


```
irb(main):001:0> Article.pamplemousse
  Article Load (0.4ms)  SELECT "articles".* FROM "articles"
  Comment Load (0.2ms)  SELECT "comments".* FROM "comments" WHERE "comments"."article_id" = ?  [["article_id", 1]]
  Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."article_id" = ?  [["article_id", 2]]
  Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."article_id" = ?  [["article_id", 3]]
=> #<Comment id: 2, author: "1", body: "Well, actually...", article_id: 1, created_at: "2018-10-19 00:56:10", updated_at: "2018-10-19 00:56:10">
```

Después de ejecutar `ActiveRecord::Base.verbose_query_logs = true` en la sesión de `bin/rails console` para habilitar los registros de consulta detallados y ejecutar el método nuevamente, resulta obvio qué línea única de código genera todas estas llamadas discretas a la base de datos:

```
irb(main):003:0> Article.pamplemousse
  Article Load (0.2ms)  SELECT "articles".* FROM "articles"
  ↳ app/models/article.rb:5
  Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."article_id" = ?  [["article_id", 1]]
  ↳ app/models/article.rb:6
  Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."article_id" = ?  [["article_id", 2]]
  ↳ app/models/article.rb:6
  Comment Load (0.1ms)  SELECT "comments".* FROM "comments" WHERE "comments"."article_id" = ?  [["article_id", 3]]
  ↳ app/models/article.rb:6
=> #<Comment id: 2, author: "1", body: "Well, actually...", article_id: 1, created_at: "2018-10-19 00:56:10", updated_at: "2018-10-19 00:56:10">
```

Debajo de cada declaración de base de datos, puede ver flechas que apuntan al nombre de archivo de origen específico (y número de línea) del método que resultó en una llamada a la base de datos. Esto puede ayudarlo a identificar y abordar los problemas de rendimiento causados ​​por consultas N + 1: consultas de base de datos única que generan múltiples consultas adicionales.

Los registros de consultas detallados están habilitados de forma predeterminada en los registros del entorno de desarrollo después de Rails 5.2.

WARNING: Recomendamos no utilizar esta configuración en entornos de producción. Se basa en el método `Kernel#caller` de Ruby, que tiende a asignar mucha memoria para generar trazas de pila de llamadas a métodos.
### Tagged Logging

Cuando se ejecutan aplicaciones multiusuario y multicuenta, suele ser útil
para poder filtrar los registros usando algunas reglas personalizadas. `TaggedLogging`
en Active Support lo ayuda a hacer exactamente eso al sellar las líneas de registro con subdominios, identificaciones de solicitud y cualquier otra cosa para ayudar a depurar dichas aplicaciones.

```ruby
logger = ActiveSupport::TaggedLogging.new(Logger.new(STDOUT))
logger.tagged("BCX") { logger.info "Stuff" }                            # Logs "[BCX] Stuff"
logger.tagged("BCX", "Jason") { logger.info "Stuff" }                   # Logs "[BCX] [Jason] Stuff"
logger.tagged("BCX") { logger.tagged("Jason") { logger.info "Stuff" } } # Logs "[BCX] [Jason] Stuff"
```

### Impact of Logs on Performance

El registro siempre tendrá un pequeño impacto en el rendimiento de su aplicación Rails,
particularmente al iniciar sesión en el disco. Además, hay algunas sutilezas:

El uso del nivel `:debug` tendrá una penalización de rendimiento mayor que `:fatal`,
ya que se está evaluando y escribiendo un número mucho mayor de cadenas en el
salida de registro (por ejemplo, disco).

Otro error potencial son demasiadas llamadas a "Logger" en su código:

```ruby
logger.debug "Person attributes hash: #{@person.attributes.inspect}"
```

En el ejemplo anterior, habrá un impacto en el rendimiento incluso si el
el nivel de salida no incluye depuración. La razón es que Ruby tiene que evaluar
estas cadenas, que incluye la creación de instancias del objeto `String` algo pesado
e interpolar las variables.

Por lo tanto, se recomienda pasar bloques a los métodos del registrador, ya que estos son
solo se evalúa si el nivel de salida es el mismo (o está incluido) en el nivel permitido
(es decir, carga diferida). El mismo código reescrito sería:

```ruby
logger.debug {"Person attributes hash: #{@person.attributes.inspect}"}
```

El contenido del bloque y, por lo tanto, la interpolación de cadenas, son solo
evaluado si la depuración está habilitada. Estos ahorros de rendimiento son solo
notable con una gran cantidad de registros, pero es una buena práctica emplearla.

INFO: Esta sección fue escrita por [Jon Cairns at a StackOverflow answer](https://stackoverflow.com/questions/16546730/logging-in-rails-is-there-any-performance-hit/16546935#16546935)
y tiene licencia bajo [cc by-sa 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
                      
Debugging with the `byebug` gem
---------------------------------

Cuando su código se comporta de manera inesperada, puede intentar imprimir en registros o
la consola para diagnosticar el problema. Desafortunadamente, hay momentos en que esto
Este tipo de seguimiento de errores no es eficaz para encontrar la causa raíz de un problema.
Cuando realmente necesita viajar a su código fuente en ejecución, el depurador
es tu mejor compañero.

El depurador también puede ayudarlo si desea aprender sobre el código fuente de Rails
pero no sé por dónde empezar. Simplemente depure cualquier solicitud a su aplicación y
Utilice esta guía para aprender cómo pasar del código que ha escrito al
código subyacente de Rails.

### Setup

Puede usar la gema `byebug` para establecer puntos de interrupción y recorrer el código en vivo en
Rieles. Para instalarlo, simplemente ejecute:

```bash
$ gem install byebug
```

Dentro de cualquier aplicación Rails, puede invocar el depurador llamando al
Método `byebug`.

He aquí un ejemplo:

```ruby
class PeopleController < ApplicationController
  def new
    byebug
    @person = Person.new
  end
end
```

### The Shell

Tan pronto como su aplicación llame al método `byebug`, el depurador será
comenzó en un shell de depuración dentro de la ventana de terminal donde lanzó su
servidor de aplicaciones, y se le colocará en el indicador del depurador `(byebug)`.
Antes del mensaje, el código alrededor de la línea que está a punto de ejecutarse será
mostrada y la línea actual será marcada por '=>', así:


```
[1, 10] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }

(byebug)
```

Si llegó allí mediante una solicitud del navegador, la pestaña del navegador que contiene la solicitud
se colgará hasta que el depurador haya finalizado y el seguimiento haya finalizado
procesando toda la solicitud.

Por ejemplo:

```bash
=> Booting Puma
=> Rails 6.0.0 application starting in development
=> Run `bin/rails server --help` for more startup options
Puma starting in single mode...
* Version 3.12.1 (ruby 2.5.7-p206), codename: Llamas in Pajamas
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://localhost:3000
Use Ctrl-C to stop
Started GET "/" for 127.0.0.1 at 2014-04-11 13:11:48 +0200
  ActiveRecord::SchemaMigration Load (0.2ms)  SELECT "schema_migrations".* FROM "schema_migrations"
Processing by ArticlesController#index as HTML

[3, 12] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }
(byebug)
```

Ahora es el momento de explorar su aplicación. Un buen lugar para comenzar es
pidiendo ayuda al depurador. Tipo: `help`

```
(byebug) help

  break      -- Sets breakpoints in the source code
  catch      -- Handles exception catchpoints
  condition  -- Sets conditions on breakpoints
  continue   -- Runs until program ends, hits a breakpoint or reaches a line
  debug      -- Spawns a subdebugger
  delete     -- Deletes breakpoints
  disable    -- Disables breakpoints or displays
  display    -- Evaluates expressions every time the debugger stops
  down       -- Moves to a lower frame in the stack trace
  edit       -- Edits source files
  enable     -- Enables breakpoints or displays
  finish     -- Runs the program until frame returns
  frame      -- Moves to a frame in the call stack
  help       -- Helps you using byebug
  history    -- Shows byebug's history of commands
  info       -- Shows several informations about the program being debugged
  interrupt  -- Interrupts the program
  irb        -- Starts an IRB session
  kill       -- Sends a signal to the current process
  list       -- Lists lines of source code
  method     -- Shows methods of an object, class or module
  next       -- Runs one or more lines of code
  pry        -- Starts a Pry session
  quit       -- Exits byebug
  restart    -- Restarts the debugged program
  save       -- Saves current byebug session to a file
  set        -- Modifies byebug settings
  show       -- Shows byebug settings
  source     -- Restores a previously saved byebug session
  step       -- Steps into blocks or methods one or more times
  thread     -- Commands to manipulate threads
  tracevar   -- Enables tracing of a global variable
  undisplay  -- Stops displaying all or some expressions when program stops
  untracevar -- Stops tracing a global variable
  up         -- Moves to a higher frame in the stack trace
  var        -- Shows variables and its values
  where      -- Displays the backtrace

(byebug)
```

Para ver las diez líneas anteriores, debe escribir `list-` (o `l-`).

```
(byebug) l-

[1, 10] in /PathTo/project/app/controllers/articles_controller.rb
   1  class ArticlesController < ApplicationController
   2    before_action :set_article, only: [:show, :edit, :update, :destroy]
   3
   4    # GET /articles
   5    # GET /articles.json
   6    def index
   7      byebug
   8      @articles = Article.find_recent
   9
   10     respond_to do |format|
```

De esta manera, puede moverse dentro del archivo y ver el código sobre la línea donde
agregó la llamada `byebug`. Finalmente, para ver de nuevo dónde se encuentra en el código, puede
escriba `list =`

```
(byebug) list=

[3, 12] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }
(byebug)
```

### The Context

Cuando comience a depurar su aplicación, se le colocará en diferentes
contextos a medida que recorre las diferentes partes de la pila.

El depurador crea un contexto cuando se alcanza un punto de parada o un evento. los
el contexto tiene información sobre el programa suspendido que habilita al depurador
para inspeccionar la pila de marcos, evaluar las variables desde la perspectiva del
programa depurado y conocer el lugar donde se detiene el programa depurado.

En cualquier momento puede llamar al comando `backtrace` (o su alias `where`) para imprimir
el backtrace de la aplicación. Esto puede ser muy útil para saber cómo consiguió
Dónde estás. Si alguna vez se preguntó cómo llegó a alguna parte de su código,
luego, "backtrace" proporcionará la respuesta.

```
(byebug) where
--> #0  ArticlesController.index
      at /PathToProject/app/controllers/articles_controller.rb:8
    #1  ActionController::BasicImplicitRender.send_action(method#String, *args#Array)
      at /PathToGems/actionpack-5.1.0/lib/action_controller/metal/basic_implicit_render.rb:4
    #2  AbstractController::Base.process_action(action#NilClass, *args#Array)
      at /PathToGems/actionpack-5.1.0/lib/abstract_controller/base.rb:181
    #3  ActionController::Rendering.process_action(action, *args)
      at /PathToGems/actionpack-5.1.0/lib/action_controller/metal/rendering.rb:30
...
```

El cuadro actual está marcado con `->`. Puedes moverte a donde quieras en este
trace (cambiando así el contexto) usando el comando `frame n`, donde _n_ es
el número de fotograma especificado. Si lo hace, `byebug` mostrará su nuevo
contexto.

```
(byebug) frame 2

[176, 185] in /PathToGems/actionpack-5.1.0/lib/abstract_controller/base.rb
   176:       # is the intended way to override action dispatching.
   177:       #
   178:       # Notice that the first argument is the method to be dispatched
   179:       # which is *not* necessarily the same as the action name.
   180:       def process_action(method_name, *args)
=> 181:         send_action(method_name, *args)
   182:       end
   183:
   184:       # Actually call the method associated with the action. Override
   185:       # this method if you wish to change how action methods are called,
(byebug)
```

Las variables disponibles son las mismas que si estuviera ejecutando la línea de código por
línea. Después de todo, eso es depurar.

También puede usar los comandos `up [n]` y `down [n]` para cambiar el contexto
_n_ fotogramas hacia arriba o hacia abajo de la pila, respectivamente. _n_ por defecto es uno. Arriba en esto
caso es hacia marcos de pila con números más altos, y hacia abajo es hacia marcos de números más bajos
apilar marcos.

### Threads

El depurador puede enumerar, detener, reanudar y cambiar entre subprocesos en ejecución mediante el uso de
el comando `thread` (o la abreviatura `th`). Este comando tiene un puñado de
opciones:

* `thread`: muestra el hilo actual.
* `thread list`: se usa para listar todos los hilos y sus estados. La corriente
el hilo está marcado con un signo más (+).
* `thread stop n`: detiene el hilo _n_.
* `thread resume n`: reanuda el hilo _n_.
* `thread switch n`: cambia el contexto del hilo actual a _n_.

Este comando es muy útil cuando está depurando subprocesos concurrentes y necesita
para verificar que no haya condiciones de carrera en su código.

### Inspecting Variables

Cualquier expresión se puede evaluar en el contexto actual. Para evaluar un
expresión, simplemente escríbala!

Este ejemplo muestra cómo puede imprimir las variables de instancia definidas dentro del
contexto actual:

```
[3, 12] in /PathTo/project/app/controllers/articles_controller.rb
    3:
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     byebug
=>  8:     @articles = Article.find_recent
    9:
   10:     respond_to do |format|
   11:       format.html # index.html.erb
   12:       format.json { render json: @articles }

(byebug) instance_variables
[:@_action_has_layout, :@_routes, :@_request, :@_response, :@_lookup_context,
 :@_action_name, :@_response_body, :@marked_for_same_origin_verification,
 :@_config]
```

Como puede haber descubierto, todas las variables a las que puede acceder desde un
se muestran el controlador. Esta lista se actualiza dinámicamente a medida que ejecuta el código.
Por ejemplo, ejecute la siguiente línea usando `next` (aprenderá más sobre esto
comando más adelante en esta guía).

```
(byebug) next

[5, 14] in /PathTo/project/app/controllers/articles_controller.rb
   5     # GET /articles.json
   6     def index
   7       byebug
   8       @articles = Article.find_recent
   9
=> 10      respond_to do |format|
   11        format.html # index.html.erb
   12        format.json { render json: @articles }
   13      end
   14    end
   15
(byebug)
```

Y luego pregunte de nuevo por las variables de instancia:

```
(byebug) instance_variables
[:@_action_has_layout, :@_routes, :@_request, :@_response, :@_lookup_context,
 :@_action_name, :@_response_body, :@marked_for_same_origin_verification,
 :@_config, :@articles]
```

Ahora se incluye `@artículos` en las variables de instancia, porque la línea que define
fue ejecutado.

TIP: También puede ingresar al modo ** irb ** con el comando `irb` (¡por supuesto!).
Esto iniciará una sesión irb dentro del contexto en el que la invocó.

El método `var` es la forma más conveniente de mostrar variables y sus valores.
Hagamos que `byebug` nos ayude con eso.

```
(byebug) help var

  [v]ar <subcommand>

  Shows variables and its values


  var all      -- Shows local, global and instance variables of self.
  var args     -- Information about arguments of the current scope
  var const    -- Shows constants of an object.
  var global   -- Shows global variables.
  var instance -- Shows instance variables of self or a specific object.
  var local    -- Shows local variables in current scope.

```

Esta es una excelente manera de inspeccionar los valores de las variables de contexto actuales. por
ejemplo, para comprobar que no tenemos variables locales definidas actualmente:


```
(byebug) var local
(byebug)
```

También puede inspeccionar un método de objeto de esta manera:
```
(byebug) var instance Article.new
@_start_transaction_state = nil
@aggregation_cache = {}
@association_cache = {}
@attributes = #<ActiveRecord::AttributeSet:0x007fd0682a9b18 @attributes={"id"=>#<ActiveRecord::Attribute::FromDatabase:0x007fd0682a9a00 @name="id", @value_be...
@destroyed = false
@destroyed_by_association = nil
@marked_for_destruction = false
@new_record = true
@readonly = false
@transaction_state = nil
```

También puede usar `display` para comenzar a ver variables. Esta es una buena forma de
rastreando los valores de una variable mientras la ejecución continúa.

```
(byebug) display @articles
1: @articles = nil
```

Las variables dentro de la lista mostrada se imprimirán con sus valores después
te mueves en la pila. Para dejar de mostrar una variable use `undisplay n` donde
_n_ es el número de variable (1 en el último ejemplo).

### Step by Step

Ahora debería saber dónde se encuentra en la traza en ejecución y poder imprimir el
variables disponibles. Pero continuemos y sigamos con la aplicación.
ejecución.

Use `step` (abreviado `s`) para continuar ejecutando su programa hasta el siguiente
punto de parada lógico y control de retorno al depurador. `next` es similar a
`step`, pero mientras `step` se detiene en la siguiente línea de código ejecutado, haciendo solo un
paso único, `siguiente` se mueve a la siguiente línea sin descender dentro de los métodos.

Por ejemplo, considere la siguiente situación:

```
Started GET "/" for 127.0.0.1 at 2014-04-11 13:39:23 +0200
Processing by ArticlesController#index as HTML

[1, 6] in /PathToProject/app/models/article.rb
   1: class Article < ApplicationRecord
   2:   def self.find_recent(limit = 10)
   3:     byebug
=> 4:     where('created_at > ?', 1.week.ago).limit(limit)
   5:   end
   6: end

(byebug)
```

Si usamos `next`, no profundizaremos en las llamadas a métodos. En cambio, byebug
vaya a la siguiente línea dentro del mismo contexto. En este caso, es la última línea
del método actual, por lo que `byebug` volverá a la siguiente línea de la persona que llama
método.

```
(byebug) next
[4, 13] in /PathToProject/app/controllers/articles_controller.rb
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     @articles = Article.find_recent
    8:
=>  9:     respond_to do |format|
   10:       format.html # index.html.erb
   11:       format.json { render json: @articles }
   12:     end
   13:   end

(byebug)
```

Si usamos `step` en la misma situación, `byebug` irá literalmente al siguiente
Instrucción Ruby que se ejecutará, en este caso, el método de la semana de Active Support.


```
(byebug) step

[49, 58] in /PathToGems/activesupport-5.1.0/lib/active_support/core_ext/numeric/time.rb
   49:
   50:   # Returns a Duration instance matching the number of weeks provided.
   51:   #
   52:   #   2.weeks # => 14 days
   53:   def weeks
=> 54:     ActiveSupport::Duration.weeks(self)
   55:   end
   56:   alias :week :weeks
   57:
   58:   # Returns a Duration instance matching the number of fortnights provided.
(byebug)
```

Esta es una de las mejores formas de encontrar errores en su código.

TIP: También puedes usar `step n` o `next n` para avanzar `n` pasos a la vez.

### Breakpoints

Un punto de interrupción hace que su aplicación se detenga cada vez que un determinado punto del programa
es alcanzado. El shell del depurador se invoca en esa línea.

Puede agregar puntos de interrupción dinámicamente con el comando `romper` (o simplemente `b`).
Hay 3 formas posibles de agregar puntos de interrupción manualmente:

* `break n`: establece el punto de interrupción en el número de línea _n_ en el archivo fuente actual.
* `break file: n [if expression]`: establece el punto de interrupción en el número de línea _n_ dentro
archivo llamado _file_. Si se da una _expresión_, debe evaluarse como _verdadero_ para
enciende el depurador.
* `break class(.|\#)method [if expression]`: establece el punto de interrupción en _method_ (. y
\ # para clase y método de instancia respectivamente) definido en _class_. los
_expression_ funciona de la misma manera que con el archivo: n.

Por ejemplo, en la situación anterior

```
[4, 13] in /PathToProject/app/controllers/articles_controller.rb
    4:   # GET /articles
    5:   # GET /articles.json
    6:   def index
    7:     @articles = Article.find_recent
    8:
=>  9:     respond_to do |format|
   10:       format.html # index.html.erb
   11:       format.json { render json: @articles }
   12:     end
   13:   end

(byebug) break 11
Successfully created breakpoint with id 1

```

Utilice `info breakpoints` para enumerar los puntos de interrupción. Si proporciona un número, indica
ese punto de ruptura. De lo contrario, enumera todos los puntos de interrupción.


```
(byebug) info breakpoints
Num Enb What
1   y   at /PathToProject/app/controllers/articles_controller.rb:11
```

Para eliminar puntos de interrupción: use el comando `delete n` para eliminar el punto de interrupción
número _n_. Si no se especifica ningún número, elimina todos los puntos de interrupción que están
actualmente activo.
```
(byebug) delete 1
(byebug) info breakpoints
No breakpoints.
```

También puede habilitar o deshabilitar puntos de interrupción:

* `enable breakpoints [n [m [...]]]`: permite una lista de puntos de interrupción específica o todos
puntos de interrupción para detener su programa. Este es el estado predeterminado cuando crea un
punto de ruptura.
* `disable breakpoints [n [m [...]]]`: asegúrese de que ciertos (o todos) los puntos de interrupción
ningún efecto en su programa.

### Catching Exceptions

El comando `catch exception-name` (o simplemente` cat exception-name`) se puede utilizar para
interceptar una excepción de tipo _exception-name_ cuando de otro modo no habría
manejador para ello.

Para enumerar todos los puntos de captura activos, use `catch`.

### Resuming Execution

Hay dos formas de reanudar la ejecución de una aplicación que está detenida en el
depurador:

* `continue [n]`: reanuda la ejecución del programa en la dirección donde su script por última vez
detenido; se omiten los puntos de interrupción establecidos en esa dirección. El argumento opcional
`n` le permite especificar un número de línea para establecer un punto de interrupción único que es
eliminado cuando se alcanza ese punto de interrupción.
* `finish [n]`: ejecutar hasta que regrese el marco de pila seleccionado. Si no hay marco
se da el número, la aplicación se ejecutará hasta que el marco seleccionado actualmente
devoluciones. El cuadro actualmente seleccionado comienza con el cuadro más reciente o 0 si
no se ha realizado ningún posicionamiento del cuadro (por ejemplo, arriba, abajo o cuadro). Si un marco
se le da un número, se ejecutará hasta que regrese el marco especificado.

### Editing

Dos comandos le permiten abrir código desde el depurador en un editor:

* `edit [file: n]`: edita el archivo llamado _file_ usando el editor especificado por el
Variable de entorno EDITOR. También se puede dar una línea específica _n_.

### Quitting

Para salir del depurador, use el comando `quit` (abreviado como `q`). O escriba `q!`
para omitir el mensaje "¿Realmente dejar de fumar?" (y / n) `y salir incondicionalmente.

Un simple abandono intenta terminar todos los hilos en vigor. Por lo tanto, su servidor
se detendrá y tendrá que volver a iniciarlo.

### Settings

`byebug` tiene algunas opciones disponibles para modificar su comportamiento:

```
(byebug) help set

  set <setting> <value>

  Modifies byebug settings

  Boolean values take "on", "off", "true", "false", "1" or "0". If you
  don't specify a value, the boolean setting will be enabled. Conversely,
  you can use "set no<setting>" to disable them.

  You can see these environment settings with the "show" command.

  List of supported settings:

  autosave       -- Automatically save command history record on exit
  autolist       -- Invoke list command on every stop
  width          -- Number of characters per line in byebug's output
  autoirb        -- Invoke IRB on every stop
  basename       -- <file>:<line> information after every stop uses short paths
  linetrace      -- Enable line execution tracing
  autopry        -- Invoke Pry on every stop
  stack_on_error -- Display stack trace when `eval` raises an exception
  fullpath       -- Display full file names in backtraces
  histfile       -- File where cmd history is saved to. Default: ./.byebug_history
  listsize       -- Set number of source lines to list by default
  post_mortem    -- Enable/disable post-mortem mode
  callstyle      -- Set how you want method call parameters to be displayed
  histsize       -- Maximum number of commands that can be stored in byebug history
  savefile       -- File where settings are saved to. Default: ~/.byebug_save
```

TIP: Puede guardar estas configuraciones en un archivo `.byebugrc` en su directorio personal.
El depurador lee estas configuraciones globales cuando se inicia. Por ejemplo:

```bash
set callstyle short
set listsize 25
```

Debugging with the `web-console` gem
------------------------------------

Web Console es un poco como "byebug", pero se ejecuta en el navegador. En cualquier pagina tu
están desarrollando, puede solicitar una consola en el contexto de una vista o un
controlador. La consola se representará junto a su contenido HTML.

### Console

Dentro de cualquier acción o vista del controlador, puede invocar la consola mediante
llamando al método `consola`.

Por ejemplo, en un controlador:

```ruby
class PostsController < ApplicationController
  def new
    console
    @post = Post.new
  end
end
```

O en una vista:

```html+erb
<% console %>

<h2>New Post</h2>
```

Esto generará una consola dentro de su vista. No necesitas preocuparte por el
ubicación de la llamada `console`; no se renderizará en el lugar de su
invocación, pero junto a su contenido HTML.

La consola ejecuta código Ruby puro: puede definir e instanciar
clases personalizadas, cree nuevos modelos e inspeccione variables.

NOTA: Solo se puede procesar una consola por solicitud. De lo contrario, `web-console`
generará un error en la segunda invocación de `console`.

### Inspecting Variables

Puede invocar `instance_variables` para enumerar todas las variables de instancia
disponible en su contexto. Si desea enumerar todas las variables locales, puede
haz eso con `local_variables`.

### Settings

* `config.web_console.whitelisted_ips`: Lista autorizada de IPv4 o IPv6
direcciones y redes (valores predeterminados: `127.0.0.1/8, ::1`).
* `config.web_console.whiny_requests`: Registra un mensaje cuando una consola renderiza
se evita (por defecto: `true`).

Dado que `web-console` evalúa el código Ruby plano de forma remota en el servidor, no intente
para usarlo en producción.


Debugging Memory Leaks
----------------------

Una aplicación Ruby (en Rails o no), puede perder memoria, ya sea en el código Ruby
o en el nivel de código C.

En esta sección, aprenderá cómo encontrar y reparar tales fugas usando herramientas
como Valgrind.

### Valgrind

[Valgrind](http://valgrind.org/) es una aplicación para detectar memoria basada en 
C fugas y condiciones de carrera.
                                 
Existen herramientas de Valgrind que pueden detectar automáticamente muchas gestiones de memoria
y subprocesos de errores, y perfile sus programas en detalle. Por ejemplo, si una C
extensión en el intérprete llama a `malloc()` pero no llama correctamente
`free()`, esta memoria no estará disponible hasta que la aplicación termine.

Para obtener más información sobre cómo instalar Valgrind y usarlo con Ruby, consulte
[Valgrind y Ruby](https://blog.evanweaver.com/2008/02/05/valgrind-and-ruby/)
por Evan Weaver.

### Find a Memory Leak

Hay un excelente artículo sobre la detección y reparación de fugas de memoria en Derailed, [which you can read here](https://github.com/schneems/derailed_benchmarks#is-my-app-leaking-memory).


Plugins for Debugging
---------------------

Hay algunos complementos de Rails para ayudarlo a encontrar errores y depurar su
solicitud. Aquí hay una lista de complementos útiles para depurar:

* [Query Trace](https://github.com/ruckus/active-record-query-trace/tree/master) Agrega consulta
rastreo de origen a sus registros.
* [Exception Notifier](https://github.com/smartinez87/exception_notification/tree/master)
Proporciona un objeto de correo y un conjunto predeterminado de plantillas para enviar correo electrónico.
notificaciones cuando ocurren errores en una aplicación Rails.
* [Beter Errors](https://github.com/charliesome/better_errors) Reemplaza el
página de error de Rails estándar con una nueva que contiene más información contextual,
como el código fuente y la inspección de variables.
* [RailsPanel](https://github.com/dejan/rails_panel) Extensión de Chrome para Rails
desarrollo que pondrá fin a su seguimiento de development.log. Tener toda la información
sobre las solicitudes de la aplicación Rails en el navegador, en el panel Herramientas para desarrolladores.
Proporciona información sobre db / renderizado / tiempos totales, lista de parámetros, vistas renderizadas y
más.
* [Pry](https://github.com/pry/pry) Una alternativa de IRB y una consola de desarrollo en tiempo de ejecución.

References
----------

* [byebug Homepage](https://github.com/deivid-rodriguez/byebug)
* [web-console Homepage](https://github.com/rails/web-console)
