**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

La Línea de Comandos de Rails
=============================

Después de leer esta guía, sabrá:

* Cómo crear una aplicación Rails.
* Cómo generar modelos, controladores, migraciones de bases de datos y pruebas unitarias.
* Cómo iniciar un servidor de desarrollo.
* Cómo experimentar con objetos a través de un caparazón interactivo.

--------------------------------------------------------------------------------

NOTA: Este tutorial asume que tiene conocimientos básicos de Rails al leer el [Getting Started with Rails Guide](getting_started.html).

Command Line Basics
-------------------

Hay algunos comandos que son absolutamente críticos para el uso diario de Rails. En el orden de cuánto probablemente los usará están:

* `bin/rails console`
* `bin/rails server`
* `bin/rails test`
* `bin/rails generate`
* `bin/rails db:migrate`
* `bin/rails db:create`
* `bin/rails routes`
* `bin/rails dbconsole`
* `rails new app_name`

Puede obtener una lista de los comandos de rails disponibles, que a menudo dependerán de su directorio actual, escribiendo `rails --help`. Cada comando tiene una descripción y debería ayudarlo a encontrar lo que necesita.

```bash
$ rails --help
Usage: rails COMMAND [ARGS]

The most common rails commands are:
 generate    Generate new code (short-cut alias: "g")
 console     Start the Rails console (short-cut alias: "c")
 server      Start the Rails server (short-cut alias: "s")
 ...

Todos los comandos se pueden ejecutar con -h (o --help) para obtener más información.

Además de esos comandos, existen:
 about                               List versions of all Rails ...
 assets:clean[keep]                  Remove old compiled assets
 assets:clobber                      Remove compiled assets
 assets:environment                  Load asset compile environment
 assets:precompile                   Compile all the assets ...
 ...
 db:fixtures:load                    Loads fixtures into the ...
 db:migrate                          Migrate the database ...
 db:migrate:status                   Display status of migrations
 db:rollback                         Rolls the schema back to ...
 db:schema:cache:clear               Clears a db/schema_cache.yml file
 db:schema:cache:dump                Creates a db/schema_cache.yml file
 db:schema:dump                      Creates a db/schema.rb file ...
 db:schema:load                      Loads a schema.rb file ...
 db:seed                             Loads the seed data ...
 db:structure:dump                   Dumps the database structure ...
 db:structure:load                   Recreates the databases ...
 db:version                          Retrieves the current schema ...
 ...
 restart                             Restart app by touching ...
 tmp:create                          Creates tmp directories ...
```

Creemos una aplicación Rails simple para recorrer cada uno de estos comandos en contexto.

### `rails new`

Lo primero que queremos hacer es crear una nueva aplicación Rails ejecutando el comando `rails new` después de instalar Rails.

INFO: Puede instalar la gema rails escribiendo `gem install rails`, si aún no la tiene.

```bash
$ rails new commandsapp
     create
     create  README.md
     create  Rakefile
     create  config.ru
     create  .gitignore
     create  Gemfile
     create  app
     ...
     create  tmp/cache
     ...
        run  bundle install
```

¡Rails te preparará con lo que parece una gran cantidad de cosas para un comando tan pequeño! Ahora tiene toda la estructura de directorios de Rails con todo el código que necesita para ejecutar nuestra sencilla aplicación de inmediato.

Si desea evitar que se generen algunos archivos o componentes, puede agregar los siguientes argumentos a su comando `rails new`:

| Argumento               | Descripción                                                  |
| ----------------------- | ------------------------------------------------------------ |
| `--skip-gemfile`        | No cree un Gemfile                                           |
| `--skip-git`            | Omitir archivo .gitignore                                    |
| `--skip-keep`           | Omitir archivos .keep de control de código fuente            |
| `--skip-action-mailer`  | Omitir archivos de Action Mailer                             |
| `--skip-action-text`    | Omitir texto de acción joya                                  |
| `--skip-active-record`  | Omitir archivos Active Record                                |
| `--skip-active-storage` | Omitir archivos de Active Storage                            |
| `--skip-puma`           | Omitir archivos relacionados con Puma                        |
| `--skip-action-cable`   | Omitir archivos Action Cable                                 |
| `--skip-sprockets`      | Omitir archivos Sprockets                                    |
| `--skip-spring`         | No instale el precargador de la aplicación Spring            |
| `--skip-listen`         | No genere configuraciones que dependan de la gema de escucha |
| `--skip-javascript`     | Omitir archivos JavaScript                                   |
| `--skip-turbolinks`     | Omitir turbolinks gem                                        |
| `--skip-test`           | Omitir archivos de prueba                                    |
| `--skip-system-test`    | Omitir archivos de prueba del sistema                        |
| `--skip-bootsnap`       | Omitir gema de bootsnap                                      |

### `bin/rails server`

El comando `bin/rails server` lanza un servidor web llamado Puma que viene incluido con Rails. Lo utilizará siempre que desee acceder a su aplicación a través de un navegador web.

Sin más trabajo, `bin/rails server` ejecutará nuestra nueva y brillante aplicación Rails:


```bash
$ cd commandsapp
$ bin/rails server
=> Booting Puma
=> Rails 6.0.0 application starting in development
=> Run `bin/rails server --help` for more startup options
Puma starting in single mode...
* Version 3.12.1 (ruby 2.5.7-p206), codename: Llamas in Pajamas
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://localhost:3000
Use Ctrl-C to stop
```

Con solo tres comandos, preparamos un servidor Rails escuchando en el puerto 3000. Vaya a su navegador y abra [http://localhost: 3000](http://localhost:3000), verá una aplicación básica de Rails ejecutándose.

INFO: También puede usar el alias "s" para iniciar el servidor: `bin/rails s`.

El servidor se puede ejecutar en un puerto diferente usando la opción `-p`. El entorno de desarrollo predeterminado se puede cambiar usando `-e`.

```bash
$ bin/rails server -e production -p 4000
```

La opción `-b` vincula Rails a la IP especificada, por defecto es localhost. Puede ejecutar un servidor como un demonio pasando una opción `-d`.

### `bin/rails generat

El comando `bin/rails generate` usa plantillas para crear un montón de cosas. Ejecutar `bin/rails generate` por sí mismo da una lista de generadores disponibles:

INFO: También puede usar el alias "g" para invocar el comando del generador: `bin/rails g`.


```bash
$ bin/rails generate
Usage: rails generate GENERATOR [args] [options]

...
...

Please choose a generator below.

Rails:
  assets
  channel
  controller
  generator
  ...
  ...
```

NOTE: Puede instalar más generadores a través de gemas de generador, partes de complementos que sin duda instalará, ¡e incluso puede crear los suyos propios!

El uso de generadores le ahorrará una gran cantidad de tiempo al escribir **código repetitivo**, código que es necesario para que la aplicación funcione.

Hagamos nuestro propio controlador con el generador de controladores. Pero, ¿qué comando debemos usar? Preguntémosle al generador:

INFO: Todas las utilidades de la consola Rails tienen texto de ayuda. Como con la mayoría de las utilidades *nix, puede intentar agregar `--help` o `-h` al final, por ejemplo, `bin/rails server --help`.

```bash
$ bin/rails generate controller
Usage: bin/rails generate controller NAME [action action] [options]

...
...

Description:
    ...

    To create a controller within a module, specify the controller name as a path like 'parent_module/controller_name'.

    ...

Example:
    `bin/rails generate controller CreditCards open debit credit close`

    Credit card controller with URLs like /credit_cards/debit.
        Controller: app/controllers/credit_cards_controller.rb
        Test:       test/controllers/credit_cards_controller_test.rb
        Views:      app/views/credit_cards/debit.html.erb [...]
        Helper:     app/helpers/credit_cards_helper.rb
```

El generador del controlador está esperando parámetros en la forma de `generate controller ControllerName action1 action2`. Hagamos un controlador `Greetings` con una acción de **hola**, que nos dirá algo agradable.

```bash
$ bin/rails generate controller Greetings hello
     create  app/controllers/greetings_controller.rb
      route  get 'greetings/hello'
     invoke  erb
     create    app/views/greetings
     create    app/views/greetings/hello.html.erb
     invoke  test_unit
     create    test/controllers/greetings_controller_test.rb
     invoke  helper
     create    app/helpers/greetings_helper.rb
     invoke    test_unit
     invoke  assets
     invoke    scss
     create      app/assets/stylesheets/greetings.scss
```

¿Qué generó todo esto? Se aseguró de que hubiera un montón de directorios en nuestra aplicación y creó un archivo de controlador, un archivo de vista, un archivo de prueba funcional, un ayudante para la vista, un archivo JavaScript y un archivo de hoja de estilo.

Compruebe el controlador y modifíquelo un poco (en `app/controllers/greetings_controller.rb`):

```ruby
class GreetingsController < ApplicationController
  def hello
    @message = "Hello, how are you today?"
  end
end
```

Luego la vista, para mostrar nuestro mensaje (en `app/views/greetings/hello.html.erb`):

```erb
<h1>A Greeting for You!</h1>
<p><%= @message %></p>
```

Fire up your server using `bin/rails server`.

```bash
$ bin/rails server
=> Booting Puma...
```

La URL será [http://localhost:3000/greetings/hello](http://localhost:3000/greetings/hello).

INFO: Con una aplicación Rails normal y sencilla, sus URL generalmente seguirán el patrón de http://(host)/(controller)/(action), y una URL como http://(host)/(controller)  presionará la acción **index** de ese controlador.

Rails también viene con un generador para modelos de datos.

```bash
$ bin/rails generate model
Usage:
  bin/rails generate model NAME [field[:type][:index] field[:type][:index]] [options]

...

ActiveRecord options:
      [--migration], [--no-migration]        # Indicates when to generate migration
                                             # Default: true

...

Description:
    Stubs out a new model. Pass the model name, either CamelCased or
    under_scored, and an optional list of attribute pairs as arguments.

...
```

NOTE: Para obtener una lista de los tipos de campo disponibles para el parámetro `type`, consulte la  [API documentation](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_column)  para el método add_column para el módulo `SchemaStatements`. El parámetro `index` genera un índice correspondiente para la columna.

Pero en lugar de generar un modelo directamente (lo que haremos más adelante), configuremos un andamio. Un **andamio** en Rails es un conjunto completo de modelo, migración de base de datos para ese modelo, controlador para manipularlo, vistas para ver y manipular los datos y un conjunto de pruebas para cada uno de los anteriores.

Configuraremos un recurso simple llamado "HighScore" que hará un seguimiento de nuestra puntuación más alta en los videojuegos que jugamos.

```bash
$ bin/rails generate scaffold HighScore game:string score:integer
    invoke  active_record
    create    db/migrate/20190416145729_create_high_scores.rb
    create    app/models/high_score.rb
    invoke    test_unit
    create      test/models/high_score_test.rb
    create      test/fixtures/high_scores.yml
    invoke  resource_route
     route    resources :high_scores
    invoke  scaffold_controller
    create    app/controllers/high_scores_controller.rb
    invoke    erb
    create      app/views/high_scores
    create      app/views/high_scores/index.html.erb
    create      app/views/high_scores/edit.html.erb
    create      app/views/high_scores/show.html.erb
    create      app/views/high_scores/new.html.erb
    create      app/views/high_scores/_form.html.erb
    invoke    test_unit
    create      test/controllers/high_scores_controller_test.rb
    create      test/system/high_scores_test.rb
    invoke    helper
    create      app/helpers/high_scores_helper.rb
    invoke      test_unit
    invoke    jbuilder
    create      app/views/high_scores/index.json.jbuilder
    create      app/views/high_scores/show.json.jbuilder
    create      app/views/high_scores/_high_score.json.jbuilder
    invoke  assets
    invoke    scss
    create      app/assets/stylesheets/high_scores.scss
    invoke  scss
    create    app/assets/stylesheets/scaffolds.scss
```

El generador verifica que existan directorios para modelos, controladores, ayudantes, diseños, pruebas funcionales y unitarias, hojas de estilo, crea las vistas, controlador, modelo y migración de base de datos para HighScore (creando la tabla y campos `high_scores`), se encarga de la ruta para el **recurso** y nuevas pruebas para todo.

La migración requiere que ** migremos **, es decir, ejecutemos algún código Ruby (viviendo en ese `20130717151933_create_high_scores.rb`) para modificar el esquema de nuestra base de datos. ¿Qué base de datos? La base de datos SQLite3 que Rails creará para usted cuando ejecutemos el comando `bin/rails db: migrate`. Hablaremos más sobre ese comando a continuación.

```bash
$ bin/rails db:migrate
==  CreateHighScores: migrating ===============================================
-- create_table(:high_scores)
   -> 0.0017s
==  CreateHighScores: migrated (0.0019s) ======================================
```

INFO: Hablemos de pruebas unitarias. Las pruebas unitarias son código que prueba y realiza afirmaciones.
sobre el código. En las pruebas unitarias, tomamos una pequeña parte del código, digamos un método de un modelo,
y probar sus entradas y salidas. Las pruebas unitarias son tus amigas. Cuanto antes hagas
paz con el hecho de que su calidad de vida aumentará drásticamente cuando
prueba tu código, mejor. Seriamente. Por favor visita
[the testing guide](https://guides.rubyonrails.org/testing.html) for an in-depth
mira las pruebas unitarias.

Veamos la interfaz que Rails creó para nosotros.

```bash
$ bin/rails server
```

Vaya a su navegador y abra [http://localhost:3000/high_scores](http://localhost:3000/high_scores), ahora podemos crear nuevos puntajes altos (¡55,160 en Space Invaders!)


### `bin/rails console`

El comando `console` te permite interactuar con tu aplicación Rails desde la línea de comandos. En la parte inferior, `bin/rails console` usa IRB, por lo que si alguna vez la usaste, estarás como en casa. Esto es útil para probar ideas rápidas con código y cambiar datos del lado del servidor sin tocar el sitio web.

INFO: También puede utilizar el alias "c" para invocar la consola: `bin/rails c`.

Puede especificar el entorno en el que debe operar el comando `console`.

```bash
$ bin/rails console -e staging
```

Si desea probar algún código sin cambiar ningún dato, puede hacerlo invocando `bin/rails console --sandbox`.

```bash
$ bin/rails console --sandbox
Loading development environment in sandbox (Rails 5.1.0)
Any modifications you make will be rolled back on exit
irb(main):001:0>
```

#### The app and helper objects

Dentro de la `bin/rails console` tienes acceso a las instancias de `app` y `helper`.

Con el método `app` puede acceder a los ayudantes de ruta con nombre, así como realizar solicitudes.

```bash
>> app.root_path
=> "/"

>> app.get _
Started GET "/" for 127.0.0.1 at 2014-06-19 10:41:57 -0300
...
```

Con el método `helper` es posible acceder a Rails y a los ayudantes de su aplicación.


```bash
>> helper.time_ago_in_words 30.days.ago
=> "about 1 month"

>> helper.my_custom_helper
=> "my custom helper"
```

### `bin/rails dbconsole`

`bin/rails dbconsole` determina qué base de datos estás usando y te lleva a cualquier interfaz de línea de comandos que usarías con ella (¡y también determina los parámetros de la línea de comandos para darle!). Es compatible con MySQL (incluido MariaDB), PostgreSQL y SQLite3.

INFO: También puede usar el alias "db" para invocar la consola db: `bin/rails db`.

### `bin/rails runner`

`runner` ejecuta código Ruby en el contexto de Rails de forma no interactiva. Por ejemplo:

```bash
$ bin/rails runner "Model.long_running_method"
```

INFO: También puede usar el alias "r" para invocar al corredor: `bin/rails r`.

Puede especificar el entorno en el que debe operar el comando `runner` usando el modificador` -e`.

```bash
$ bin/rails runner -e staging "Model.long_running_method"
```

Incluso puede ejecutar código ruby ​​escrito en un archivo con runner.

```bash
$ bin/rails runner lib/code_to_be_run.rb
```

### `bin/rails destroy`

Piense en `destroy` como lo opuesto a `generate`. Descubrirá lo que generó y lo deshará.

INFO: También puede usar el alias "d" para invocar el comando de destrucción: `bin / rails d`.

```bash
$ bin/rails generate model Oops
      invoke  active_record
      create    db/migrate/20120528062523_create_oops.rb
      create    app/models/oops.rb
      invoke    test_unit
      create      test/models/oops_test.rb
      create      test/fixtures/oops.yml
```
```bash
$ bin/rails destroy model Oops
      invoke  active_record
      remove    db/migrate/20120528062523_create_oops.rb
      remove    app/models/oops.rb
      invoke    test_unit
      remove      test/models/oops_test.rb
      remove      test/fixtures/oops.yml
```

### `bin/rails about`

`bin/rails about` brinda información sobre los números de versión de Ruby, RubyGems, Rails, los subcomponentes de Rails, la carpeta de su aplicación, el nombre del entorno de Rails actual, el adaptador de base de datos de su aplicación y la versión del esquema. Es útil cuando necesita pedir ayuda, verificar si un parche de seguridad podría afectarlo o cuando necesita algunas estadísticas para una instalación existente de Rails.

```bash
$ bin/rails about
About your application's environment
Rails version             6.0.0
Ruby version              2.5.0 (x86_64-linux)
RubyGems version          2.7.3
Rack version              2.0.4
JavaScript Runtime        Node.js (V8)
Middleware:               Rack::Sendfile, ActionDispatch::Static, ActionDispatch::Executor, ActiveSupport::Cache::Strategy::LocalCache::Middleware, Rack::Runtime, Rack::MethodOverride, ActionDispatch::RequestId, ActionDispatch::RemoteIp, Sprockets::Rails::QuietAssets, Rails::Rack::Logger, ActionDispatch::ShowExceptions, WebConsole::Middleware, ActionDispatch::DebugExceptions, ActionDispatch::Reloader, ActionDispatch::Callbacks, ActiveRecord::Migration::CheckPending, ActionDispatch::Cookies, ActionDispatch::Session::CookieStore, ActionDispatch::Flash, Rack::Head, Rack::ConditionalGet, Rack::ETag
Application root          /home/foobar/commandsapp
Environment               development
Database adapter          sqlite3
Database schema version   20180205173523
```

### `bin/rails assets:`

Puede precompilar los activos en `app/assets` usando `bin/rails assets: precompile`, y eliminar los activos compilados más antiguos usando `bin/rails assets:clean`. El comando `assets:clean` permite implementar despliegues que aún pueden estar vinculados a un activo antiguo mientras se construyen los nuevos activos.

Si desea borrar completamente `public/assets`, puede usar `bin/rails assets: clobber`.

### `bin/rails db:`

Los comandos más comunes del espacio de nombres `db:` rails son `migrate` y `create`, y valdrá la pena probar todos los comandos de los rails de migración (`up`, `down`, `redo`, `reset `). `bin/rails db:version` es útil para solucionar problemas, ya que le indica la versión actual de la base de datos.

Puede encontrar más información sobre las migraciones en la guía  [Migrations](active_record_migrations.html). 

### `bin/rails notes`

`bin/rails notes` busca en su código comentarios que comiencen con una palabra clave específica. Puede consultar `bin/rails notes --help` para obtener información sobre el uso.

De forma predeterminada, buscará en los directorios `app`, `config`, `db`, `lib`, and `test`para anotaciones FIXME, OPTIMIZE y TODO en archivos con extensión `.builder`, `.rb`, `.rake`, `.yml`, `.yaml`, `.ruby`, `.css`, `.js`, y `.erb`.

```bash
$ bin/rails notes
app/controllers/admin/users_controller.rb:
  * [ 20] [TODO] any other way to do this?
  * [132] [FIXME] high priority for next deploy

lib/school.rb:
  * [ 13] [OPTIMIZE] refactor this code to make it faster
  * [ 17] [FIXME]
```

#### Annotations

Puede pasar anotaciones específicas utilizando el argumento `--annotations`. De forma predeterminada, buscará FIXME, OPTIMIZE y TODO.
Tenga en cuenta que las anotaciones distinguen entre mayúsculas y minúsculas.

```bash
$ bin/rails notes --annotations FIXME RELEASE
app/controllers/admin/users_controller.rb:
  * [101] [RELEASE] We need to look at this before next release
  * [132] [FIXME] high priority for next deploy

lib/school.rb:
  * [ 17] [FIXME]
```

#### Tags

Puede agregar más etiquetas predeterminadas para buscar usando `config.annotations.register_tags`. Recibe una lista de etiquetas.

```ruby
config.annotations.register_tags("DEPRECATEME", "TESTME")
```

```bash
$ bin/rails notes
app/controllers/admin/users_controller.rb:
  * [ 20] [TODO] do A/B testing on this
  * [ 42] [TESTME] this needs more functional tests
  * [132] [DEPRECATEME] ensure this method is deprecated in next release
```

#### Directories

Puede agregar más directorios predeterminados para buscar usando `config.annotations.register_directories`. Recibe una lista de nombres de directorio.

```ruby
config.annotations.register_directories("spec", "vendor")
```

```bash
$ bin/rails notes
app/controllers/admin/users_controller.rb:
  * [ 20] [TODO] any other way to do this?
  * [132] [FIXME] high priority for next deploy

lib/school.rb:
  * [ 13] [OPTIMIZE] Refactor this code to make it faster
  * [ 17] [FIXME]

spec/models/user_spec.rb:
  * [122] [TODO] Verify the user that has a subscription works

vendor/tools.rb:
  * [ 56] [TODO] Get rid of this dependency
```

#### Extensions

Puede agregar más extensiones de archivo predeterminadas para buscar utilizando `config.annotations.register_extensions`. Recibe una lista de extensiones con su expresión regular correspondiente para que coincida.

```ruby
config.annotations.register_extensions("scss", "sass") { |annotation| /\/\/\s*(#{annotation}):?\s*(.*)$/ }
```

```bash
$ bin/rails notes
app/controllers/admin/users_controller.rb:
  * [ 20] [TODO] any other way to do this?
  * [132] [FIXME] high priority for next deploy

app/assets/stylesheets/application.css.sass:
  * [ 34] [TODO] Use pseudo element for this class

app/assets/stylesheets/application.css.scss:
  * [  1] [TODO] Split into multiple components

lib/school.rb:
  * [ 13] [OPTIMIZE] Refactor this code to make it faster
  * [ 17] [FIXME]

spec/models/user_spec.rb:
  * [122] [TODO] Verify the user that has a subscription works

vendor/tools.rb:
  * [ 56] [TODO] Get rid of this dependency
```

### `bin/rails routes`

`bin/rails route` enumerará todas sus rutas definidas, lo cual es útil para rastrear problemas de enrutamiento en su aplicación, o para darle una buena descripción general de las URL en una aplicación con la que está tratando de familiarizarse.

### `bin/rails test`

INFORMACIÓN: Se ofrece una buena descripción de las pruebas unitarias en Rails en  [A Guide to Testing Rails Applications](testing.html)

Rails viene con un marco de prueba llamado minitest. Rails debe su estabilidad al uso de pruebas. Los comandos disponibles en el espacio de nombres `test:` ayudan a ejecutar las diferentes pruebas que esperemos que escriba.

### `bin/rails tmp:`

El directorio `Rails.root/tmp` es, como el directorio *nix / tmp, el lugar de almacenamiento de archivos temporales como archivos de identificación de procesos y acciones en caché.

Los comandos de espacio de nombres `tmp:` te ayudarán a limpiar y crear el directorio `Rails.root/tmp`:

* `bin/rails tmp:cache:clear` borra `tmp/cache`.
* `bin/rails tmp:sockets:clear` borra `tmp/sockets`.
* `bin/rails tmp:screenshots:clear` borra `tmp/screenshots`.
* `bin/rails tmp:clear` borra todos los archivos de caché, sockets y capturas de pantalla.
* `bin/rails tmp:create` crea directorios tmp para caché, sockets y pids.

### Miscellaneous

* `bin/rails initializers` imprime todos los inicializadores definidos en el orden en que los invoca Rails.
* `bin/rails middleware` enumera la pila de middleware de Rack habilitada para su aplicación.
* `bin/rails stats` es excelente para ver estadísticas en su código, mostrar cosas como KLOC (miles de líneas de código) y su relación de código a prueba.
* `bin/rails secret` le dará una clave pseudoaleatoria para usar en su sesión secreta.
* `bin/rails time:zones:all` enumera todas las zonas horarias que conoce Rails.

### Custom Rake Tasks

Las tareas de rake personalizadas tienen una extensión `.rake` y se colocan en
`Rails.root/lib/tasks`. Puede crear estas tareas de rastrillo personalizadas con el
Comando `bin/rails generate task`.

```ruby
desc "I am short, but comprehensive description for my cool task"
task task_name: [:prerequisite_task, :another_task_we_depend_on] do
  # All your magic here
  # Any valid Ruby code is allowed
end
```

Para pasar argumentos a su tarea de rake personalizada:

```ruby
task :task_name, [:arg_1] => [:prerequisite_1, :prerequisite_2] do |task, args|
  argument_1 = args.arg_1
end
```

Puede agrupar tareas colocándolas en espacios de nombres:

```ruby
namespace :db do
  desc "This task does nothing"
  task :nothing do
    # Seriously, nothing
  end
end
```

La invocación de las tareas se verá así:

```bash
$ bin/rails task_name
$ bin/rails "task_name[value 1]" # entire argument string should be quoted
$ bin/rails db:nothing
```

NOTE: Si necesita interactuar con los modelos de su aplicación, realizar consultas a la base de datos, etc., su tarea debería depender de la tarea `environment`, que cargará el código de su aplicación.

The Rails Advanced Command Line
-------------------------------

El uso más avanzado de la línea de comandos se centra en encontrar opciones útiles (incluso sorprendentes a veces) en las utilidades y adaptarlas a sus necesidades y flujo de trabajo específico. A continuación se enumeran algunos trucos bajo la manga de Rails.

### Rails with Databases and SCM

Al crear una nueva aplicación Rails, tiene la opción de especificar qué tipo de base de datos y qué tipo de sistema de gestión de código fuente va a utilizar su aplicación. Esto le ahorrará unos minutos y, ciertamente, muchas pulsaciones de teclas.

Veamos qué harán por nosotros una opción `--git` y una opción` --database = postgresql`:

```bash
$ mkdir gitapp
$ cd gitapp
$ git init
Initialized empty Git repository in .git/
$ rails new . --git --database=postgresql
      exists
      create  app/controllers
      create  app/helpers
...
...
      create  tmp/cache
      create  tmp/pids
      create  Rakefile
add 'Rakefile'
      create  README.md
add 'README.md'
      create  app/controllers/application_controller.rb
add 'app/controllers/application_controller.rb'
      create  app/helpers/application_helper.rb
...
      create  log/test.log
add 'log/test.log'
```

Tuvimos que crear el directorio **gitapp** e inicializar un repositorio git vacío antes de que Rails agregara los archivos que creó a nuestro repositorio. Veamos qué puso en la configuración de nuestra base de datos:

```bash
$ cat config/database.yml
# PostgreSQL. Versions 9.3 and up are supported.
#
# Install the pg driver:
#   gem install pg
# On macOS with Homebrew:
#   gem install pg -- --with-pg-config=/usr/local/bin/pg_config
# On macOS with MacPorts:
#   gem install pg -- --with-pg-config=/opt/local/lib/postgresql84/bin/pg_config
# On Windows:
#   gem install pg
#       Choose the win32 build.
#       Install PostgreSQL and put its /bin directory on your path.
#
# Configure Using Gemfile
# gem 'pg'
#
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: gitapp_development
...
...
```

También generó algunas líneas en nuestra configuración `database.yml` correspondientes a nuestra elección de PostgreSQL para la base de datos.

NOTE. El único problema con el uso de las opciones de SCM es que primero debe crear el directorio de su aplicación, luego inicializar su SCM, luego puede ejecutar el comando `rails new` para generar la base de su aplicación.
