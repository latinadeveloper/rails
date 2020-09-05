**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Configuración de Aplicaciones de Rails
======================================

Esta guía cubre las funciones de configuración e inicialización disponibles para las aplicaciones Rails.

Después de leer esta guía, sabrá:

* Cómo ajustar el comportamiento de sus aplicaciones Rails.
* Cómo agregar código adicional para que se ejecute en el momento de inicio de la aplicación.

--------------------------------------------------------------------------------

Locations for Initialization Code
---------------------------------

Rails ofrece cuatro puntos estándar para colocar el código de inicialización:

* `config / application.rb`
* Archivos de configuración específicos del entorno
* Inicializadores
* Después de los inicializadores


Running Code Before Rails
-------------------------

En el raro caso de que su aplicación necesite ejecutar algún código antes de que se cargue Rails, colóquelo encima de la llamada para `require "rails/all"` en `config/application.rb`.

Configuring Rails Components
----------------------------

En general, el trabajo de configurar Rails significa configurar los componentes de Rails, así como configurar Rails en sí. El archivo de configuración `config/application.rb` y los archivos de configuración específicos del entorno (como` config/environment/production.rb`) le permiten especificar las distintas configuraciones que desea transferir a todos los componentes.

Por ejemplo, puede agregar esta configuración al archivo `config/application.rb`:

```ruby
config.time_zone = 'Central Time (US & Canada)'
```

Esta es una configuración para Rails en sí. Si desea pasar la configuración a componentes individuales de Rails, puede hacerlo a través del mismo objeto `config` en` config / application.rb`:

```ruby
config.active_record.schema_format = :ruby
```

Rails utilizará esa configuración en particular para configurar Active Record.

### Rails General Configuration

Estos métodos de configuración deben llamarse en un objeto `Rails::Railtie`, como una subclase de` Rails::Engine` o `Rails::Application`.

* `config.after_initialize` toma un bloque que se ejecutará _ después de que Rails haya terminado de inicializar la aplicación. Eso incluye la inicialización del marco en sí, los motores y todos los inicializadores de la aplicación en `config/initializers`. Tenga en cuenta que este bloque _ se ejecutará para tareas de rake. Útil para configurar valores establecidos por otros inicializadores:

    ```ruby
    config.after_initialize do
      ActionView::Base.sanitized_allowed_tags.delete 'div'
    end
    ```

* `config.asset_host` establece el host para los activos. Útil cuando las CDN se utilizan para alojar activos o cuando desea evitar las restricciones de simultaneidad integradas en los navegadores que utilizan diferentes alias de dominio. Versión más corta de `config.action_controller.asset_host`.

* `config.autoload_once_paths` acepta una serie de rutas desde las cuales Rails cargará automáticamente constantes que no se borrarán por solicitud. Relevante si `config.cache_classes` es `false`, que es el caso en el modo de desarrollo por defecto. De lo contrario, toda la carga automática ocurre solo una vez. Todos los elementos de esta matriz también deben estar en `autoload_paths`. El valor predeterminado es una matriz vacía.

* `config.autoload_paths` acepta una serie de rutas desde las cuales Rails cargará automáticamente las constantes. El valor predeterminado son todos los directorios en `app`. Ya no se recomienda ajustar esto. Consulta [Autoloading and Reloading Constants](autoloading_and_reloading_constants.html#autoload-path-and-eager-load-path).

* `config.add_autoload_paths_to_load_path` dice si las rutas de carga automática deben agregarse a` $ LOAD_PATH`. Esta marca es `true` por defecto, pero se recomienda establecerla en `false` en el modo `:zeitwerk` antes, en `config/application.rb`. Zeitwerk usa rutas absolutas internamente, y las aplicaciones que se ejecutan en el modo `:zeitwerk` no necesitan `require_dependency`, por lo que los modelos, controladores, trabajos, etc.no necesitan estar en `$ LOAD_PATH`. Establecer esto en `false` evita que Ruby revise estos directorios al resolver llamadas `require` con rutas relativas, y ahorra trabajo de Bootsnap y RAM, ya que no necesita construir un índice para ellos.

* `config.cache_classes` controla si las clases y módulos de la aplicación deben recargarse o no si cambian. El valor predeterminado es `false` en el modo de desarrollo y `true` en el modo de producción. En el modo `test`, el valor predeterminado es `false` si Spring está instalado, `true` en caso contrario.

* `config.beginning_of_week` establece el comienzo de semana predeterminado para el
solicitud. Acepta un día de la semana válido como símbolo (por ejemplo, `:monday`).

* `config.cache_store` configura qué almacén de caché se utilizará para el almacenamiento en caché de Rails. Las opciones incluyen uno de los símbolos `:memory_store`, `:file_store`, `:mem_cache_store`, `:null_store`, `:redis_cache_store`, o un objeto que implemente la API de caché. El valor predeterminado es `:file_store`. Consulta [Cache Stores](caching_with_rails.html#cache-stores) para conocer las opciones de configuración por tienda.

* `config.colorize_logging` especifica si se deben usar o no códigos de color ANSI al registrar información. El valor predeterminado es `true`.

* `config.consider_all_requests_local` es una bandera. Si es `true`, cualquier error hará que se descargue información detallada de depuración en la respuesta HTTP, y el controlador `Rails::Info` mostrará el contexto de ejecución de la aplicación en `/rails/info/properties`. `true` de forma predeterminada en entornos de desarrollo y prueba, y `false` en modo de producción. Para un control más detallado, establezca esto en `false` e implemente `show_detailed_exceptions?` En los controladores para especificar qué solicitudes deben proporcionar información de depuración sobre errores.

* `config.console` le permite establecer la clase que se usará como consola cuando ejecute `bin/rails console`. Es mejor ejecutarlo en el bloque `console`:

    ```ruby
    console do
      # this block is called only when running console,
      # so we can safely require pry here
      require "pry"
      config.console = Pry
    end
    ```

* `config.disable_sandbox` controla si alguien puede iniciar o no una consola en modo sandbox. Esto es útil para evitar una sesión de larga duración de la consola sandbox, que podría hacer que un servidor de base de datos se quede sin memoria. El valor predeterminado es falso.

* `config.eager_load` cuando es` true`, ansioso carga todos los `config.eager_load_namespaces` registrados. Esto incluye su aplicación, motores, marcos de Rails y cualquier otro espacio de nombres registrado.

* `config.eager_load_namespaces` registra espacios de nombres que están ansiosos por cargar cuando` config.eager_load` es `true`. Todos los espacios de nombres de la lista deben responder al método `eager_load!`.

* `config.eager_load_paths` acepta una serie de rutas desde las cuales Rails cargará ansiosamente en el arranque si las clases de caché están habilitadas. Por defecto, todas las carpetas del directorio `app` de la aplicación.

* `config.enable_dependency_loading`: cuando es verdadero, habilita la carga automática, incluso si la aplicación está ansiosamente cargada y `config.cache_classes` se establece como verdadero. El valor predeterminado es falso.

* `config.encoding` configura la codificación de toda la aplicación. Por defecto es UTF-8.

* `config.exceptions_app` establece la aplicación de excepciones invocada por el middleware ShowException cuando ocurre una excepción. El valor predeterminado es `ActionDispatch::PublicExceptions.new(Rails.public_path)`.

* `config.debug_exception_response_format` establece el formato utilizado en las respuestas cuando se producen errores en el modo de desarrollo. El valor predeterminado es `:api` para aplicaciones solo API y `:default` para aplicaciones normales.

* `config.file_watcher` es la clase utilizada para detectar actualizaciones de archivos en el sistema de archivos cuando `config.reload_classes_only_on_change` es `true`. Rails se envía con `ActiveSupport::FileUpdateChecker`, el valor predeterminado, y` ActiveSupport::EventedFileUpdateChecker` (este depende de la gema [listen](https://github.com/guard/listen)). Las clases personalizadas deben cumplir con la API `ActiveSupport::FileUpdateChecker`.

* `config.filter_parameters` utilizado para filtrar los parámetros que
no desea que se muestre en los registros, como contraseñas o tarjetas de crédito
números. También filtra los valores sensibles de las columnas de la base de datos cuando se llama a "# inspeccionar" en un objeto de registro activo. Por defecto, Rails filtra las contraseñas agregando `Rails.application.config.filter_parameters + = [: contraseña]` en `config/initializers/filter_parameter_logging.rb`. El filtro de parámetros funciona mediante una expresión regular de coincidencia parcial.

* `config.force_ssl` obliga a que todas las solicitudes se sirvan a través de HTTPS y establece" https: // "como protocolo predeterminado al generar URL. La aplicación de HTTPS es manejada por el middleware `ActionDispatch::SSL`, que se puede configurar a través de `config.ssl_options` - ver su [documentación](https://api.rubyonrails.org/classes/ActionDispatch/SSL.html) para detalles.

* `config.log_formatter` define el formateador del registrador Rails. Esta opción tiene por defecto una instancia de `ActiveSupport::Logger::SimpleFormatter` para todos los modos. Si está configurando un valor para `config.logger`, debe pasar manualmente el valor de su formateador a su registrador antes de que se envuelva en una instancia de` ActiveSupport::TaggedLogging`, Rails no lo hará por usted.

* `config.log_level` define la verbosidad del registrador Rails. Esta opción
por defecto es `: debug` para todos los entornos. Los niveles de registro disponibles son: `:debug`,
`: info`,`: warn`, `: error`,`: fatal` y `:unknown`.

* `config.log_tags` acepta una lista de: métodos a los que responde el objeto` request`, un `Proc` que acepta el objeto `request` o algo que responde a `to_s`. Esto facilita etiquetar líneas de registro con información de depuración como subdominio e identificación de solicitud, ambos muy útiles para depurar aplicaciones de producción multiusuario.

* `config.logger` es el registrador que se utilizará para `Rails.logger` y cualquier registro de Rails relacionado como `ActiveRecord::Base.logger`. Por defecto, es una instancia de `ActiveSupport::TaggedLogging` que envuelve una instancia de `ActiveSupport::Logger` que genera un registro en el directorio `log/`. Puede suministrar un registrador personalizado, para obtener compatibilidad total debe seguir estas pautas:
  * Para admitir un formateador, debe asignar manualmente un formateador desde el valor `config.log_formatter` al registrador.
  * Para admitir registros etiquetados, la instancia de registro debe estar envuelta con `ActiveSupport::TaggedLogging`.
  * Para soportar el silenciamiento, el registrador debe incluir el módulo `ActiveSupport::LoggerSilence`. La clase `ActiveSupport::Logger` ya incluye estos módulos.

    ```ruby
    class MyLogger < ::Logger
      include ActiveSupport::LoggerSilence
    end

    mylogger           = MyLogger.new(STDOUT)
    mylogger.formatter = config.log_formatter
    config.logger      = ActiveSupport::TaggedLogging.new(mylogger)
    ```

* `config.middleware` le permite configurar el middleware de la aplicación. Esto se trata en profundidad en la sección [Configuración de middleware](#configuring-middleware) a continuación.

* `config.rake_eager_load` cuando es `true`, con ganas de cargar la aplicación al ejecutar tareas de Rake. El valor predeterminado es `false`.

* `config.reload_classes_only_on_change` habilita o deshabilita la recarga de clases solo cuando los archivos de seguimiento cambian. De forma predeterminada, rastrea todo en las rutas de carga automática y se establece en `true`. Si `config.cache_classes` es` true`, esta opción se ignora.

* `config.credentials.content_path` configura la ruta de búsqueda para las credenciales cifradas.

* `config.credentials.key_path` configura la ruta de búsqueda para la clave de cifrado.

* `secret_key_base` se utiliza para especificar una clave que permite que las sesiones de la aplicación se verifiquen con una clave segura conocida para evitar la manipulación. Las aplicaciones obtienen una clave generada aleatoriamente en entornos de prueba y desarrollo, otros entornos deben establecer una en `config/credentials.yml.enc`.

* `config.public_file_server.enabled` configura Rails para servir archivos estáticos desde el directorio público. Esta opción tiene el valor predeterminado de `true`, pero en el entorno de producción se establece en `false` porque el software del servidor (por ejemplo, NGINX o Apache) que se utiliza para ejecutar la aplicación debe entregar archivos estáticos en su lugar. Si está ejecutando o probando su aplicación en modo de producción usando WEBrick (no se recomienda usar WEBrick en producción) configure la opción en `true`. De lo contrario, no podrá utilizar el almacenamiento en caché de la página ni solicitar archivos que existen en el directorio público.

* `config.session_store` especifica qué clase usar para almacenar la sesión. Los valores posibles son `:cookie_store`, que es el predeterminado, `:mem_cache_store` y `:disabled`. El último le dice a Rails que no se ocupe de las sesiones. Por defecto, un almacén de cookies con el nombre de la aplicación como clave de sesión. También se pueden especificar tiendas de sesiones personalizadas:

    ```ruby
    config.session_store :my_custom_store
    ```

    Esta tienda personalizada debe definirse como `ActionDispatch::Session::MyCustomStore`.

* `config.time_zone` establece la zona horaria predeterminada para la aplicación y habilita el reconocimiento de la zona horaria para Active Record.

* `config.autoloader` establece el modo de carga automática. Esta opción toma el valor predeterminado de `:zeitwerk` si se especifica `6.0` en `config.load_defaults`. Las aplicaciones aún pueden usar el autocargador clásico estableciendo este valor en `:classic` después de cargar los valores predeterminados del marco:

    ```ruby
    config.load_defaults 6.0
    config.autoloader = :classic
    ```

### Configuring Assets

* `config.assets.enabled` una bandera que controla si el activo
la canalización está habilitada. Se establece en `true` de forma predeterminada.

* `config.assets.css_compressor` define el compresor CSS a utilizar. Está configurado por defecto por `sass-rails`. El valor alternativo único en este momento es `:yui`, que usa la gema `yui-compressor`.

* `config.assets.js_compressor` define el compresor de JavaScript a usar. Los valores posibles son `:closures`,`:uglifier` y `:yui` que requieren el uso de las gemas `closes-compiler`, `uglifier` o` yui-compressor` respectivamente.

* `config.assets.gzip` un indicador que permite la creación de una versión con gzip de los activos compilados, junto con los activos que no lo son. Establecido en `true` de forma predeterminada.

* `config.assets.paths` contiene las rutas que se utilizan para buscar activos. Agregar rutas a esta opción de configuración hará que esas rutas se utilicen en la búsqueda de activos.

* `config.assets.precompile` le permite especificar activos adicionales (distintos de `application.css` y `application.js`) que se precompilarán cuando se ejecute` rake assets:precompile`.

* `config.assets.unknown_asset_fallback` le permite modificar el comportamiento de la canalización de activos cuando un activo no está en la canalización, si utiliza sprockets-rails 3.2.0 o más reciente. El valor predeterminado es `false`.

* `config.assets.prefix` define el prefijo desde donde se sirven los activos. El valor predeterminado es `/assets`.

* `config.assets.manifest` define la ruta completa que se utilizará para el archivo de manifiesto del precompilador de activos. Por defecto, el archivo llamado `manifest- <random>.json` en el directorio `config.assets.prefix` dentro de la carpeta pública.

* `config.assets.digest` habilita el uso de huellas digitales SHA256 en nombres de activos. Establecido en `true` de forma predeterminada.

* `config.assets.debug` desactiva la concatenación y compresión de activos. Establecido en `true` de forma predeterminada en `development.rb`.

* `config.assets.version` es una cadena de opciones que se utiliza en la generación de hash SHA256. Esto se puede cambiar para forzar la recompilación de todos los archivos.

* `config.assets.compile` es un valor booleano que se puede usar para activar la compilación de Sprockets en vivo en producción.

* `config.assets.logger` acepta un registrador conforme a la interfaz de Log4r o la clase predeterminada Ruby `Logger`. Por defecto es el mismo configurado en `config.logger`. Establecer `config.assets.logger` en `false` desactivará el registro de activos servidos.

* `config.assets.quiet` deshabilita el registro de solicitudes de activos. Establecido en "true" de forma predeterminada en `development.rb`.

### Configuring Generators

Rails te permite alterar qué generadores se usan con el método `config.generators`. Este método toma un bloque:

```ruby
config.generators do |g|
  g.orm :active_record
  g.test_framework :test_unit
end
```

El conjunto completo de métodos que se pueden utilizar en este bloque es el siguiente:

* `assets` permite crear activos al generar un andamio. El valor predeterminado es `true`.
* `force_plural` permite nombres de modelos en plural. El valor predeterminado es `false`.
* `helper` define si se generan o no ayudantes. El valor predeterminado es `true`.
* `integration_tool` define qué herramienta de integración usar para generar pruebas de integración. El valor predeterminado es `:test_unit`.
* `system_tests` define qué herramienta de integración usar para generar pruebas del sistema. El valor predeterminado es `:test_unit`.
* `orm` define qué orm usar. El valor predeterminado es `false` y utilizará Active Record de forma predeterminada.
* `resource_controller` define qué generador usar para generar un controlador cuando se usa` bin/rails generate resource`. El valor predeterminado es `:controller`.
* `resource_route` define si se debe generar una definición de ruta de recursos
  o no. El valor predeterminado es `true`.
* `scaffold_controller` diferente de` resource_controller`, define qué generador usar para generar un controlador _scaffold_ cuando se usa `bin/rails generate scaffold`. El valor predeterminado es `:scaffold_controller`.
* `stylesheets` activa el gancho para las hojas de estilo en los generadores. Se usa en Rails para cuando se ejecuta el generador `scaffold`, pero este gancho también se puede usar en otras generaciones. El valor predeterminado es `true`.
* `stylesheet_engine` configura el motor de hojas de estilo (por ejemplo, sass) que se utilizará al generar activos. El valor predeterminado es `:css`.
* `scaffold_stylesheet` crea `scaffold.css` cuando se genera un recurso con scaffold. El valor predeterminado es `true`.
* `test_framework` define qué marco de prueba usar. El valor predeterminado es `false` y utilizará minitest de forma predeterminada.
* `template_engine` define qué motor de plantillas usar, como ERB o Haml. El valor predeterminado es `:erb`.

### Configuring Middleware

Cada aplicación Rails viene con un conjunto estándar de middleware que usa en este orden en el entorno de desarrollo:

* `ActionDispatch::HostAuthorization` previene contra la revinculación de DNS y otros ataques de encabezado de `Host`.
   Se incluye en el entorno de desarrollo por defecto con la siguiente configuración:

   ```ruby
   Rails.application.config.hosts = [
     IPAddr.new("0.0.0.0/0"), # All IPv4 addresses.
     IPAddr.new("::/0"),      # All IPv6 addresses.
     "localhost"              # The localhost reserved domain.
   ]
   ```

   En otros entornos, `Rails.application.config.hosts` está vacío y no
   Se realizarán las comprobaciones del encabezado `Host`. Si quieres protegerte del encabezado
   ataques a la producción, debe permitir manualmente los hosts permitidos
   con:

   ```ruby
   Rails.application.config.hosts << "product.com"
   ```

   El host de una solicitud se compara con las entradas `hosts` con el caso
   operador (`#===`), que permite a `hosts` admitir entradas de tipo `Regexp`,
   `Proc` e `IPAddr`, por nombrar algunos. Aquí hay un ejemplo con una expresión regular.

   ```ruby
   # Allow requests from subdomains like `www.product.com` and
   # `beta1.product.com`.
   Rails.application.config.hosts << /.*\.product\.com/
   ```

   Se admite un caso especial que le permite permitir todos los subdominios:

* `ActionDispatch::SSL` obliga a que todas las solicitudes se atiendan mediante HTTPS. Habilitado si `config.force_ssl` se establece en `true`. Las opciones pasadas a esto se pueden configurar configurando `config.ssl_options`.
* `ActionDispatch::Static` se usa para servir activos estáticos. Deshabilitado si `config.public_file_server.enabled` es `falso`. Establezca `config.public_file_server.index_name` si necesita servir un archivo de índice de directorio estático que no se llame `index`. Por ejemplo, para servir `main.html` en lugar de `index.html` para las solicitudes de directorio, establezca `config.public_file_server.index_name` en `"main"`.
* `ActionDispatch::Executor` permite la recarga de código seguro para subprocesos. Deshabilitado si `config.allow_concurrency` es `falso`, lo que hace que se cargue `Rack::Lock`. `Rack::Lock` envuelve la aplicación en mutex para que solo pueda ser llamada por un solo hilo a la vez.
* `ActiveSupport::Cache::Strategy::LocalCache` sirve como un caché respaldado en memoria básica. Este caché no es seguro para subprocesos y está diseñado solo para servir como caché de memoria temporal para un solo subproceso.
* `Rack::Runtime` establece un encabezado `X-Runtime`, que contiene el tiempo (en segundos) necesario para ejecutar la solicitud.
* `Rails::Rack::Logger` notifica a los registros que la solicitud ha comenzado. Una vez completada la solicitud, elimina todos los registros.
* `ActionDispatch::ShowExceptions` rescata cualquier excepción devuelta por la aplicación y muestra páginas de excepción agradables si la solicitud es local o si `config.consider_all_requests_local` se establece en `true`. Si `config.action_dispatch.show_exceptions` se establece en `false`, las excepciones se generarán independientemente.
* `ActionDispatch::RequestId` hace que un encabezado X-Request-Id único esté disponible para la respuesta y habilita el método `ActionDispatch::Request # uuid`.
* `ActionDispatch::RemoteIp` busca ataques de suplantación de IP y obtiene un` client_ip` válido de los encabezados de las solicitudes. Configurable con las opciones `config.action_dispatch.ip_spoofing_check` y` config.action_dispatch.trusted_proxies`.
* `Rack::Sendfile` intercepta las respuestas cuyo cuerpo está siendo servido desde un archivo y lo reemplaza con un encabezado X-Sendfile específico del servidor. Configurable con `config.action_dispatch.x_sendfile_header`.
* `ActionDispatch::Callbacks` ejecuta las devoluciones de llamada preparadas antes de atender la solicitud.
* `ActionDispatch::Cookies` establece cookies para la solicitud.
* `ActionDispatch::Session::CookieStore` es responsable de almacenar la sesión en cookies. Se puede usar un middleware alternativo para esto cambiando el `config.action_controller.session_store` a un valor alternativo. Además, las opciones pasadas a esto se pueden configurar usando `config.action_controller.session_options`.
* `ActionDispatch::Flash` configura las teclas `flash`. Solo disponible si `config.action_controller.session_store` se establece en un valor.
* `Rack::MethodOverride` permite que el método sea anulado si se establece `params[:_ method]`. Este es el middleware que admite los tipos de métodos HTTP PATCH, PUT y DELETE.
* `Rack::Head` convierte las solicitudes HEAD en solicitudes GET y las sirve como tal.

Además de este middleware habitual, puede agregar el suyo propio utilizando el método `config.middleware.use`:

```ruby
config.middleware.use Magical::Unicorns
```

Esto colocará el middleware `Magical::Unicorns` al final de la pila. Puede usar `insert_before` si desea agregar un middleware antes que otro.
```ruby
config.middleware.insert_before Rack::Head, Magical::Unicorns
```

O puede insertar un middleware en la posición exacta utilizando índices. Por ejemplo, si desea insertar el middleware `Magical::Unicorns` en la parte superior de la pila, puede hacerlo, así:
```ruby
config.middleware.insert_before 0, Magical::Unicorns
```

También hay `insert_after` que insertará un middleware tras otro:

```ruby
config.middleware.insert_after Rack::Head, Magical::Unicorns
```

Los middlewares también se pueden cambiar y reemplazar por otros:

```ruby
config.middleware.swap ActionController::Failsafe, Lifo::Failsafe
```

Los middlewares se pueden mover de un lugar a otro:

```ruby
config.middleware.move_before ActionDispatch::Flash, Magical::Unicorns
```

Esto moverá el middleware `Magical::Unicorns` antes
`ActionDispatch::Flash`. También puede moverlo después de:

```ruby
config.middleware.move_after ActionDispatch::Flash, Magical::Unicorns
```

También se pueden quitar de la pila por completo:

```ruby
config.middleware.delete Rack::MethodOverride
```

### Configuring i18n

Todas estas opciones de configuración se delegan a la biblioteca `I18n`.

* `config.i18n.available_locales` define las configuraciones regionales disponibles permitidas para la aplicación. Por defecto, todas las claves de configuración regional que se encuentran en los archivos de configuración regional, generalmente solo `:en` en una nueva aplicación.

* `config.i18n.default_locale` establece la configuración regional predeterminada de una aplicación utilizada para i18n. El valor predeterminado es `:en`.

* `config.i18n.enforce_available_locales` asegura que todas las configuraciones regionales pasadas a través de i18n deben declararse en la lista` available_locales`, generando una excepción `I18n::InvalidLocale` cuando se establece una configuración regional no disponible. El valor predeterminado es `true`. Se recomienda no deshabilitar esta opción a menos que sea muy necesario, ya que esto funciona como una medida de seguridad contra la configuración de una configuración regional no válida desde la entrada del usuario.

* `config.i18n.load_path` establece la ruta que usa Rails para buscar archivos de configuración regional. Por defecto es `config/locales/*.{Yml, rb}`.

* `config.i18n.raise_on_missing_translations` determina si se debe generar un error por traducciones faltantes
en controladores y vistas. Este valor predeterminado es `false`.

* `config.i18n.fallbacks` establece el comportamiento de reserva para las traducciones faltantes. Aquí hay 3 ejemplos de uso para esta opción:
  
  * Puede establecer la opción en `true` para usar la configuración regional predeterminada como alternativa, así:

    ```ruby
    config.i18n.fallbacks = true
    ```
  * O puede establecer una serie de configuraciones regionales como reserva, así:
  
    ```ruby
    config.i18n.fallbacks = [:tr, :en]
    ```

  * O puede establecer diferentes alternativas para las configuraciones regionales de forma individual. Por ejemplo, si desea utilizar `:tr` para `:az` y `:de`,`:en` para `:da` como alternativas, puede hacerlo así:
  
    ```ruby
    config.i18n.fallbacks = { az: :tr, da: [:de, :en] }
    #or
    config.i18n.fallbacks.map = { az: :tr, da: [:de, :en] }
    ```

### Configuring Active Model

* `config.active_model.i18n_customize_full_message` es un valor booleano que controla si el formato de error `full_message` puede anularse a nivel de atributo o modelo en los archivos de configuración regional. Esto es `false` por defecto.

### Configuring Active Record

`config.active_record` incluye una variedad de opciones de configuración:

* `config.active_record.logger` acepta un registrador conforme a la interfaz de Log4r o la clase Ruby Logger predeterminada, que luego se pasa a cualquier nueva conexión de base de datos realizada. Puede recuperar este registrador llamando a `logger` en una clase de modelo de Active Record o en una instancia de modelo de Active Record. Establezca en `nil` para deshabilitar el registro.

* `config.active_record.primary_key_prefix_type` le permite ajustar el nombre de las columnas de clave primaria. De forma predeterminada, Rails asume que las columnas de clave principal se denominan `id` (y no es necesario establecer esta opción de configuración). Hay otras dos opciones:
    * `:table_name` sería la clave principal para la clase de cliente` customerid`.
    * `:table_name_with_underscore` sería la clave principal para la clase de cliente `customer_id`.

* `config.active_record.table_name_prefix` le permite establecer una cadena global para que se anteponga a los nombres de las tablas. Si establece esto en `northwest_, la clase Cliente buscará  `northwest_customers como su tabla. El valor predeterminado es una cadena vacía.

* `config.active_record.table_name_suffix` le permite establecer una cadena global para agregarla a los nombres de las tablas. Si establece esto en `_northwest`, la clase Customer buscará `customers_northwest` como su tabla. El valor predeterminado es una cadena vacía.

* `config.active_record.schema_migrations_table_name` le permite establecer una cadena para ser utilizada como el nombre de la tabla de migraciones de esquema.

* `config.active_record.internal_metadata_table_name` le permite establecer una cadena para usar como el nombre de la tabla de metadatos interna.

* `config.active_record.protected_environments` le permite establecer una serie de nombres de entornos donde las acciones destructivas deben prohibirse.

* `config.active_record.pluralize_table_names` especifica si Rails buscará nombres de tablas en singular o plural en la base de datos. Si se establece en `true` (el valor predeterminado), la clase Cliente utilizará la tabla `customers`. Si se establece en falso, la clase Cliente utilizará la tabla `customer`.

* `config.active_record.default_timezone` determina si usar `Time.local` (si se establece en `:local`) o `Time.utc` (si se establece en `:utc`) al extraer fechas y horas de la base de datos. El valor predeterminado es `:utc`.

* `config.active_record.schema_format` controla el formato para descargar el esquema de la base de datos en un archivo. Las opciones son `:ruby` (el valor predeterminado) para una versión independiente de la base de datos que depende de las migraciones, o `:sql` para un conjunto de sentencias SQL (potencialmente dependientes de la base de datos).

* `config.active_record.error_on_ignored_order` especifica si se debe generar un error si se ignora el orden de una consulta durante una consulta por lotes. Las opciones son `true` (generar error) o `false` (advertir). El valor predeterminado es `false`.

* `config.active_record.timestamped_migrations` controla si las migraciones se numeran con números enteros en serie o con marcas de tiempo. El valor predeterminado es `true`, para usar marcas de tiempo, que se prefieren si hay varios desarrolladores trabajando en la misma aplicación.

* `config.active_record.lock_optimistically` controla si Active Record usará bloqueo optimista y es `true` por defecto.

* `config.active_record.cache_timestamp_format` controla el formato del valor de la marca de tiempo en la clave de caché. El valor predeterminado es `:usec`.

* `config.active_record.record_timestamps` es un valor booleano que controla si se produce o no la marca de tiempo de las operaciones `create` y `update` en un modelo. El valor predeterminado es `true`.

* `config.active_record.partial_writes` es un valor booleano y controla si se utilizan escrituras parciales o no (es decir, si las actualizaciones solo establecen atributos que están sucios). Tenga en cuenta que al usar escrituras parciales, también debe usar el bloqueo optimista `config.active_record.lock_optimistically` ya que las actualizaciones simultáneas pueden escribir atributos basados​en un estado de lectura posiblemente obsoleto. El valor predeterminado es `true`.

* `config.active_record.maintain_test_schema` es un valor booleano que controla si Active Record debe intentar mantener el esquema de la base de datos de prueba actualizado con `db/schema.rb` (o `db/structure.sql`) cuando Ejecute sus pruebas. El valor predeterminado es `true`.

* `config.active_record.dump_schema_after_migration` es una bandera que
  controla si debe ocurrir o no el volcado de esquema (`db/schema.rb` o
  `db/structure.sql`) cuando ejecuta migraciones. Esto se establece en `false` en
  `config/environment/production.rb` que es generado por Rails. los
  el valor predeterminado es `true` si no se establece esta configuración.
* `config.active_record.dump_schemas` controla qué esquemas de base de datos se descargarán al llamar a `db:structure:dump`.
  Las opciones son `: schema_search_path` (el valor predeterminado) que vuelca los esquemas enumerados en `schema_search_path`,
  `:all` que siempre vuelca todos los esquemas independientemente del `schema_search_path`,
  o una cadena de esquemas separados por comas.

* `config.active_record.belongs_to_required_by_default` es un valor booleano y
  controla si un registro falla en la validación si la asociación `belongs_to` no es
  presente.

* `config.active_record.warn_on_records_fetched_greater_than` permite establecer un
  umbral de advertencia para el tamaño del resultado de la consulta. Si el número de registros devuelto
  si una consulta supera el umbral, se registra una advertencia. Esto se puede utilizar para
  Identificar consultas que puedan estar causando un exceso de memoria.

* `config.active_record.index_nested_attribute_errors` permite errores para anidados
  Las relaciones `has_many` se mostrarán con un índice, así como el error.
  El valor predeterminado es `false`.

* `config.active_record.use_schema_cache_dump` permite a los usuarios obtener información de la caché del esquema
  de `db/schema_cache.yml` (generado por `bin/rails db:schema:cache:dump`), en lugar de
  tener que enviar una consulta a la base de datos para obtener esta información.
  El valor predeterminado es `true`.

* `config.active_record.collection_cache_versioning` habilita la misma clave de caché
  para ser reutilizado cuando el objeto se almacena en caché de tipo `ActiveRecord::Relation`
  cambios moviendo la información volátil (máximo actualizado en y recuento) de
  la clave de caché de la relación en la versión de caché para admitir el reciclaje de la clave de caché.
  El valor predeterminado es `false`.

El adaptador MySQL agrega una opción de configuración adicional:

* `ActiveRecord::ConnectionAdapters::Mysql2Adapter.emulate_booleans` controla si Active Record considerará todas las columnas `tinyint(1) `como booleanos. El valor predeterminado es `true`.

El adaptador de PostgreSQL agrega una opción de configuración adicional:

* `ActiveRecord::ConnectionAdapters::PostgreSQLAdapter.create_unlogged_tables`
  controla si las tablas de la base de datos creadas deben ser "unlogged", lo que puede acelerar
  mejora el rendimiento, pero agrega un riesgo de pérdida de datos si la base de datos falla. Es
  Se recomienda encarecidamente que no habilite esto en un entorno de producción.
  El valor predeterminado es `false` en todos los entornos.

El volcador de esquemas agrega dos opciones de configuración adicionales:

* `ActiveRecord::SchemaDumper.ignore_tables` acepta una matriz de tablas que _no_ deben incluirse en ningún archivo de esquema generado.

* `ActiveRecord::SchemaDumper.fk_ignore_pattern` permite establecer una regular diferente
  expresión que se utilizará para decidir si el nombre de una clave externa debe ser
  volcado a db / schema.rb o no. De forma predeterminada, los nombres de claves externas que comienzan con
  `fk_rails_` no se exportan al volcado del esquema de la base de datos.
  El valor predeterminado es `/ ^ fk_rails_ [0-9a-f] {10} $ /`.

### Configuring Action Controller

`config.action_controller` incluye una serie de opciones de configuración:

* `config.action_controller.asset_host` establece el host para los activos. Útil cuando las CDN se utilizan para alojar activos en lugar del servidor de aplicaciones en sí.

* `config.action_controller.perform_caching` configura si la aplicación debe realizar las funciones de almacenamiento en caché proporcionadas por el componente Action Controller o no. Establecido en `false` en modo de desarrollo, `true` en producción. Si no se especifica, el valor predeterminado será `true`.

* `config.action_controller.default_static_extension` configura la extensión utilizada para las páginas en caché. El valor predeterminado es `.html`.

* `config.action_controller.include_all_helpers` configura si todos los ayudantes de vista están disponibles en todas partes o tienen el alcance del controlador correspondiente. Si se establece en `false`, los métodos de "UsersHelper" solo están disponibles para las vistas representadas como parte de "UsersController". Si es "true", los métodos de "UsersHelper" están disponibles en todas partes. El comportamiento de configuración predeterminado (cuando esta opción no se establece explícitamente en `true` o `false`) es que todos los ayudantes de vista están disponibles para cada controlador.

* `config.action_controller.logger` acepta un registrador conforme a la interfaz de Log4r o la clase Ruby Logger predeterminada, que luego se utiliza para registrar información desde Action Controller. Establezca en `nil` para deshabilitar el registro.

* `config.action_controller.request_forgery_protection_token` establece el nombre del parámetro del token para RequestForgery. Llamar a `protect_from_forgery` lo establece en `:authentity_token` por defecto.

* `config.action_controller.allow_forgery_protection` habilita o deshabilita la protección CSRF. De forma predeterminada, es `false` en el modo de prueba y `true` en todos los demás modos.

* `config.action_controller.forgery_protection_origin_check` configura si el encabezado HTTP `Origin` debe ser verificado con el origen del sitio como una defensa CSRF adicional.

* `config.action_controller.per_form_csrf_tokens` configura si los tokens CSRF solo son válidos para el método/acción para el que fueron generados.

* `config.action_controller.default_protect_from_forgery` determina si se agrega protección contra falsificaciones en` ActionController::Base`. Esto es falso por defecto.

* `config.action_controller.relative_url_root` se puede usar para decirle a Rails que está [deploying to a subdirectory](configuring.html#deploy-to-a-subdirectory-related-url-root). El valor predeterminado es `ENV['RAILS_RELATIVE_URL_ROOT']`.

* `config.action_controller.permit_all_parameters` establece todos los parámetros para que la asignación masiva sea permitida por defecto. El valor predeterminado es `false`.

* `config.action_controller.action_on_unpermitted_parameters` habilita el registro o genera una excepción si se encuentran parámetros que no están explícitamente permitidos. Establezca en `:log` o `:raise` para habilitar. El valor predeterminado es `:log` en entornos de desarrollo y prueba, y `false` en todos los demás entornos.

* `config.action_controller.always_permitted_parameters` establece una lista de parámetros permitidos que están permitidos por defecto. Los valores predeterminados son `['controlador', 'acción']`.

* `config.action_controller.enable_fragment_cache_logging` determina si se registran las lecturas y escrituras de fragmentos de caché en formato detallado de la siguiente manera:

    ```
    Read fragment views/v1/2914079/v1/2914079/recordings/70182313-20160225015037000000/d0bdf2974e1ef6d31685c3b392ad0b74 (0.6ms)
    Rendered messages/_message.html.erb in 1.2 ms [cache hit]
    Write fragment views/v1/2914079/v1/2914079/recordings/70182313-20160225015037000000/3b4e249ac9d168c617e32e84b99218b5 (1.1ms)
    Rendered recordings/threads/_thread.html.erb in 1.5 ms [cache miss]
    ```

    De forma predeterminada, se establece en `false`, lo que da como resultado el siguiente resultado:
    
    ```
    Rendered messages/_message.html.erb in 1.2 ms [cache hit]
    Rendered recordings/threads/_thread.html.erb in 1.5 ms [cache miss]
    ```

### Configuring Action Dispatch

* `config.action_dispatch.session_store` establece el nombre de la tienda para los datos de la sesión. El valor predeterminado es `:cookie_store`; otras opciones válidas incluyen `:active_record_store`,`:mem_cache_store` o el nombre de su propia clase personalizada.

* `config.action_dispatch.default_headers` es un hash con encabezados HTTP que se establecen de forma predeterminada en cada respuesta. Por defecto, esto se define como:

    ```ruby
    config.action_dispatch.default_headers = {
      'X-Frame-Options' => 'SAMEORIGIN',
      'X-XSS-Protection' => '1; mode=block',
      'X-Content-Type-Options' => 'nosniff',
      'X-Download-Options' => 'noopen',
      'X-Permitted-Cross-Domain-Policies' => 'none',
      'Referrer-Policy' => 'strict-origin-when-cross-origin'
    }
    ```

* `config.action_dispatch.default_charset` especifica el juego de caracteres predeterminado para todas las representaciones. El valor predeterminado es `nil`.

* `config.action_dispatch.tld_length` establece la longitud del TLD (dominio de nivel superior) para la aplicación. El valor predeterminado es `1`.

* `config.action_dispatch.ignore_accept_header` se usa para determinar si ignorar los encabezados de aceptación de una solicitud. El valor predeterminado es `false`.

* `config.action_dispatch.x_sendfile_header` especifica el encabezado X-Sendfile específico del servidor. Esto es útil para el envío acelerado de archivos desde el servidor. Por ejemplo, se puede configurar en 'X-Sendfile' para Apache.

* `config.action_dispatch.http_auth_salt` establece el valor salt Auth HTTP. Defaults
a `'autenticación http'`.

* `config.action_dispatch.signed_cookie_salt` establece el valor de sal de las cookies firmadas.
El valor predeterminado es `'encrypted cookie''`.

* `config.action_dispatch.encrypted_cookie_salt` establece la sal de cookies encriptadas
  valor. Por defecto es `'encrypted cookie'`.

* `config.action_dispatch.encrypted_signed_cookie_salt` establece el
  valor de sal de cookies cifradas. El valor predeterminado es `'signed encrypted cookie'`.

* `config.action_dispatch.authenticated_encrypted_cookie_salt` establece el
  sal de cookie cifrada autenticada. El valor predeterminado es `` autenticado cifrado
  cookie'`.

* `config.action_dispatch.encrypted_cookie_cipher` establece el cifrado como
  utilizado para cookies encriptadas. Este valor predeterminado es "" aes-256-gcm "`.

* `config.action_dispatch.signed_cookie_digest` establece el resumen como
  utilizado para cookies firmadas. Este valor predeterminado es "" SHA1 "`.

* `config.action_dispatch.cookies_rotations` permite rotar
  secretos, cifrados y resúmenes de cookies cifradas y firmadas.

* `config.action_dispatch.use_authenticated_cookie_encryption` controla si
  Las cookies firmadas y cifradas utilizan el cifrado AES-256-GCM o
  el cifrado AES-256-CBC más antiguo. Su valor predeterminado es `true`.

* `config.action_dispatch.use_cookies_with_metadata` habilita la escritura
  cookies con el propósito y metadatos de caducidad incrustados. Su valor predeterminado es `true`.

* `config.action_dispatch.perform_deep_munge` configura si` deep_munge`
  el método debe realizarse en los parámetros. Consulta la [Security Guide](security.html#unsafe-query-generation).
  para más información. Su valor predeterminado es `true`.

* `config.action_dispatch.rescue_responses` configura qué excepciones se asignan a un estado HTTP. Acepta un hash y puede especificar pares de excepción / estado. Por defecto, esto se define como:

  ```ruby
  config.action_dispatch.rescue_responses = {
    'ActionController::RoutingError'               => :not_found,
    'AbstractController::ActionNotFound'           => :not_found,
    'ActionController::MethodNotAllowed'           => :method_not_allowed,
    'ActionController::UnknownHttpMethod'          => :method_not_allowed,
    'ActionController::NotImplemented'             => :not_implemented,
    'ActionController::UnknownFormat'              => :not_acceptable,
    'ActionController::InvalidAuthenticityToken'   => :unprocessable_entity,
    'ActionController::InvalidCrossOriginRequest'  => :unprocessable_entity,
    'ActionDispatch::Http::Parameters::ParseError' => :bad_request,
    'ActionController::BadRequest'                 => :bad_request,
    'ActionController::ParameterMissing'           => :bad_request,
    'Rack::QueryParser::ParameterTypeError'        => :bad_request,
    'Rack::QueryParser::InvalidParameterError'     => :bad_request,
    'ActiveRecord::RecordNotFound'                 => :not_found,
    'ActiveRecord::StaleObjectError'               => :conflict,
    'ActiveRecord::RecordInvalid'                  => :unprocessable_entity,
    'ActiveRecord::RecordNotSaved'                 => :unprocessable_entity
  }
  ```

Cualquier excepción que no esté configurada se asignará a 500 Internal Server Error.

* `config.action_dispatch.return_only_media_type_on_content_type` cambia el
  valor de retorno de `ActionDispatch::Response # content_type` al Content-Type
  encabezado sin modificación. El valor predeterminado es `false`.

* `ActionDispatch::Callbacks.before` toma un bloque de código para ejecutarse antes de la solicitud.

* `ActionDispatch::Callbacks.after` toma un bloque de código para ejecutarse después de la solicitud.

### Configuring Action View

`config.action_view` incluye una pequeña cantidad de opciones de configuración:

* `config.action_view.cache_template_loading` controla si las plantillas se deben recargar o no en cada solicitud. Por defecto es lo que esté configurado para `config.cache_classes`.

* `config.action_view.field_error_proc` proporciona un generador HTML para mostrar errores que provienen de Active Model. El valor predeterminado es

    ```ruby
    Proc.new do |html_tag, instance|
      %Q(<div class="field_with_errors">#{html_tag}</div>).html_safe
    end
    ```

* `config.action_view.default_form_builder` le dice a Rails qué creador de formularios debe
  utilizar de forma predeterminada. El valor predeterminado es `ActionView::Helpers::FormBuilder`. Si tu
  desea que su clase de constructor de formularios se cargue después de la inicialización (por lo que es
  recargado en cada solicitud en desarrollo), puede pasarlo como una `String`.

* `config.action_view.logger` acepta un registrador que se ajusta a la interfaz de Log4r o la clase Ruby Logger predeterminada, que luego se usa para registrar información desde la Vista de acción. Establezca en `nil` para deshabilitar el registro.

* `config.action_view.erb_trim_mode` da el modo de recorte que utilizará ERB. El valor predeterminado es `'-'`, que activa el recorte de los espacios de cola y la nueva línea cuando se usa `<%= -%>` or `<%= =%>`. Consulte la [documentación de Erubis](http://www.kuwata-lab.com/erubis/users-guide.06.html#topics-trimspaces) para obtener más información.

* `config.action_view.embed_authenticity_token_in_remote_forms` le permite
  establezca el comportamiento predeterminado para `autenticity_token` en formularios con `remote:
  true`. De forma predeterminada, se establece en `false`, lo que significa que los formularios remotos no
  incluir `authenticity_token`, que es útil cuando se almacena en caché de fragmentos
  la forma. Los formularios remotos obtienen la autenticidad de la etiqueta `meta`, por lo que incrustar
  no es necesario a menos que admita navegadores sin JavaScript. En tal caso
  puede pasar `authenticity_token: true` como una opción de formulario o establecer esto
  config ajuste a `true`.

* `config.action_view.prefix_partial_path_with_controller_namespace` determina si los parciales se buscan o no desde un subdirectorio en plantillas renderizadas desde controladores con espacio de nombres. Por ejemplo, considere un controlador llamado `Admin::ArticlesController` que muestra esta plantilla:

    ```erb
    <%= render @article %>
    ```
La configuración predeterminada es `true`, que usa el parcial en `/admin/articles/_article.erb`. Establecer el valor en `false` generaría `/articles/_article.erb`, que es el mismo comportamiento que la representación desde un controlador sin espacio de nombres como `ArticlesController`.

* `config.action_view.automatically_disable_submit_tag` determina si
  `submit_tag` debería deshabilitarse automáticamente al hacer clic, este valor predeterminado es `true`.

* `config.action_view.debug_missing_translation` determina si se envuelve la clave de traducción faltante en una etiqueta `<span>` o no. Este valor predeterminado es `true`.

* `config.action_view.form_with_generates_remote_forms` determina si `form_with` genera formularios remotos o no. Este valor predeterminado es `true`.

* `config.action_view.form_with_generates_ids` determina si `form_with` genera identificadores en las entradas. Este valor predeterminado es `false`.

* `config.action_view.default_enforce_utf8` determina si los formularios se generan con una etiqueta oculta que obliga a las versiones anteriores de Internet Explorer a enviar formularios codificados en UTF-8. Este valor predeterminado es `false`.

* `config.action_view.annotate_rendered_view_with_filenames` determina si anotar la vista renderizada con nombres de archivo de plantilla. Este valor predeterminado es `false`.

### Configuring Action Mailbox

`config.action_mailbox` proporciona las siguientes opciones de configuración:

* `config.action_mailbox.logger` contiene el registrador utilizado por Action Mailbox. Acepta un registrador conforme a la interfaz de Log4r o la clase Ruby Logger predeterminada. El predeterminado es `Rails.logger`.

  ```ruby
  config.action_mailbox.logger = ActiveSupport::Logger.new(STDOUT)
  ```

* `config.action_mailbox.incinerate_after` acepta un `ActiveSupport::Duration` que indica cuánto tiempo después de procesar los registros `ActionMailbox::InboundEmail` deben ser destruidos. El valor predeterminado es `30 días`.

   ```ruby
   # Incinerate inbound emails 14 days after processing.
   config.action_mailbox.incinerate_after = 14.days
   ```

* `config.action_mailbox.queues.incineration` acepta un símbolo que indica la cola de trabajos activos que se utilizarán para los trabajos de incineración. Su valor predeterminado es `:action_mailbox_incineration`.

* `config.action_mailbox.queues.routing` acepta un símbolo que indica la cola de trabajos activos que se utilizará para enrutar trabajos. Su valor predeterminado es `:action_mailbox_routing`.

### Configuring Action Mailer

Hay una serie de configuraciones disponibles en `config.action_mailer`:

* `config.action_mailer.logger` acepta un registrador conforme a la interfaz de Log4r o la clase predeterminada Ruby Logger, que luego se usa para registrar información de Action Mailer. Establezca en `nil` para deshabilitar el registro.

* `config.action_mailer.smtp_settings` permite la configuración detallada del método de entrega `:smtp`. Acepta un hash de opciones, que puede incluir cualquiera de estas opciones:
    * `:address` - Le permite usar un servidor de correo remoto. Simplemente cámbielo de su configuración predeterminada "localhost".
    * `:port` - En caso de que su servidor de correo no se ejecute en el puerto 25, puede cambiarlo.
    * `:domain`: si necesita especificar un dominio HELO, puede hacerlo aquí.
    * `:user_name` - Si su servidor de correo requiere autenticación, establezca el nombre de usuario en esta configuración.
    * `:password`: si su servidor de correo requiere autenticación, configure la contraseña en esta configuración.
    * `:authentication`: si su servidor de correo requiere autenticación, debe especificar el tipo de autenticación aquí. Este es un símbolo y uno de `:plain`, `:login`, `:cram_md5`.
    * `:enable_starttls_auto`: detecta si STARTTLS está habilitado en su servidor SMTP y comienza a usarlo. Su valor predeterminado es `true`.
    * `:openssl_verify_mode` - Al usar TLS, puede configurar cómo OpenSSL verifica el certificado. Esto es útil si necesita validar un certificado autofirmado y / o comodín. Puede ser una de las constantes de verificación de OpenSSL, `:none` o `:peer` - o la constante directamente `OpenSSL::SSL::VERIFY_NONE` o `OpenSSL::SSL::VERIFY_PEER`, respectivamente.
    * `:ssl/:tls`: permite que la conexión SMTP utilice SMTP / TLS (SMTPS: SMTP sobre una conexión TLS directa).

* `config.action_mailer.sendmail_settings` permite la configuración detallada del método de entrega de` sendmail`. Acepta un hash de opciones, que puede incluir cualquiera de estas opciones:
    * `:location`: la ubicación del ejecutable de sendmail. Por defecto es `/usr/sbin/sendmail`.
    * `:argumentos` - Los argumentos de la línea de comandos. El valor predeterminado es "-i".

* `config.action_mailer.raise_delivery_errors` especifica si generar un error si no se puede completar la entrega del correo electrónico. Su valor predeterminado es `true`.

* `config.action_mailer.delivery_method` define el método de entrega y su valor predeterminado es`:smtp`. Consulta la [sección de configuración en la guía de Action Mailer](action_mailer_basics.html # action-mailer-configuration) para obtener más información.

* `config.action_mailer.perform_deliveries` especifica si el correo se entregará realmente y es verdadero por defecto. Puede ser conveniente establecerlo en `false` para realizar pruebas.

* `config.action_mailer.default_options` configura los valores predeterminados de Action Mailer. Úselo para configurar opciones como `from` o `reply_to` para cada correo. Estos predeterminados a:

    ```ruby
    mime_version:  "1.0",
    charset:       "UTF-8",
    content_type: "text/plain",
    parts_order:  ["text/plain", "text/enriched", "text/html"]
    ```

    Assign a hash to set additional options:

    ```ruby
    config.action_mailer.default_options = {
      from: "noreply@example.com"
    }
    ```

* `config.action_mailer.observers` registra observadores que serán notificados cuando se entregue el correo.

    ```ruby
    config.action_mailer.observers = ["MailObserver"]
    ```

* `config.action_mailer.interceptors` registra los interceptores que serán llamados antes de que se envíe el correo.

    ```ruby
    config.action_mailer.interceptors = ["MailInterceptor"]
    ```

* `config.action_mailer.preview_interceptors` registra los interceptores que serán llamados antes de que se obtenga una vista previa del correo.

    ```ruby
    config.action_mailer.preview_interceptors = ["MyPreviewMailInterceptor"]
    ```

* `config.action_mailer.preview_path` especifica la ubicación de las vistas previas de correo.

    ```ruby
    config.action_mailer.preview_path = "#{Rails.root}/lib/mailer_previews"
    ```

* `config.action_mailer.show_previews` habilitar o deshabilitar las vistas previas de correo. Por defecto, esto es `true` en desarrollo.

    ```ruby
    config.action_mailer.show_previews = false
    ```

* `config.action_mailer.deliver_later_queue_name` especifica el nombre de la cola para
  mailers. Por defecto esto es`mailers`.

* `config.action_mailer.perform_caching` especifica si las plantillas de correo deben realizar el almacenamiento en caché de fragmentos o no. Si no se especifica, el valor predeterminado será `true`.

* `config.action_mailer.delivery_job`especifica el trabajo de entrega para el correo. Predeterminado a `ActionMailer::DeliveryJob`.

### Configuring Active Support

Hay algunas opciones de configuración disponibles en Active Support:

* `config.active_support.bare` habilita o deshabilita la carga de `active_support/all` al arrancar Rails. El valor predeterminado es `nil`, lo que significa que `active_support/all` está cargado.

* `config.active_support.test_order` establece el orden en el que se ejecutan los casos de prueba. Los valores posibles son `:random` y `:sorted`. El valor predeterminado es `:aleatorio`.

* `config.active_support.escape_html_entities_in_json` habilita o deshabilita el escape de entidades HTML en la serialización JSON. El valor predeterminado es `true`.

* `config.active_support.use_standard_json_time_format` habilita o deshabilita la serialización de fechas en formato ISO 8601. El valor predeterminado es `true`.

* `config.active_support.time_precision` establece la precisión de los valores de tiempo codificados en JSON. El valor predeterminado es `3`.

* `config.active_support.use_sha1_digests` especifica si se debe utilizar SHA-1 en lugar de MD5 para generar resúmenes no sensibles, como el encabezado ETag. El valor predeterminado es falso.

* `config.active_support.use_authenticated_message_encryption` especifica si se debe utilizar el cifrado autenticado AES-256-GCM como cifrado predeterminado para cifrar mensajes en lugar de AES-256-CBC. Esto es falso por defecto.

* `ActiveSupport :: Logger.silencer` se establece en `false` para deshabilitar la capacidad de silenciar el registro en un bloque. El valor predeterminado es `true`.

* `ActiveSupport :: Cache :: Store.logger` especifica el registrador que se utilizará en las operaciones del almacén de caché.

* `ActiveSupport :: Deprecation.behavior` establecedor alternativo a `config.active_support.deprecation` que configura el comportamiento de las advertencias de obsolescencia para Rails.

* `ActiveSupport :: Deprecation.disallowed_behavior` establecedor alternativo a `config.active_support.disallowed_deprecation` que configura el comportamiento de las advertencias de desaprobación no permitidas para Rails.

* `ActiveSupport :: Deprecation.disallowed_warnings` establecedor alternativo a `config.active_support.disallowed_deprecation_warnings` que configura las advertencias de desaprobación que la Aplicación considera no permitidas. Esto permite, por ejemplo, que las depreciaciones específicas se traten como fallas graves.

* `ActiveSupport :: Deprecation.silence` toma un bloque en el que se silencian todas las advertencias de obsolescencia.

* `ActiveSupport :: Deprecation.silenced` establece si se muestran o no las advertencias de obsolescencia. El valor predeterminado es `false`.

### Configuring Active Job

`config.active_job` proporciona las siguientes opciones de configuración:

* `config.active_job.queue_adapter` establece el adaptador para el backend de cola. El adaptador predeterminado es `: async`. Para obtener una lista actualizada de adaptadores integrados, consulte la [documentación de la API de ActiveJob::QueueAdapters](https://api.rubyonrails.org/classes/ActiveJob/QueueAdapters.html).

    ```ruby
    # Be sure to have the adapter's gem in your Gemfile
    # and follow the adapter's specific installation
    # and deployment instructions.
    config.active_job.queue_adapter = :sidekiq
    ```

* `config.active_job.default_queue_name` se puede usar para cambiar el nombre de la cola predeterminado. De forma predeterminada, esto es `default`.

   ```ruby
    config.active_job.default_queue_name = :medium_priority
    ```

* `config.active_job.queue_name_prefix` Le permite establecer un prefijo de nombre de cola opcional, que no esté en blanco, para todos los trabajos. De forma predeterminada, está en blanco y no se utiliza.
                                        
    La siguiente configuración pondría en cola el trabajo dado en la cola `production_high_priority` cuando se ejecuta en producción:

    ```ruby
    config.active_job.queue_name_prefix = Rails.env
    ```

    ```ruby
    class GuestsCleanupJob < ActiveJob::Base
      queue_as :high_priority
      #....
    end
    ```

* `config.active_job.queue_name_delimiter` tiene un valor predeterminado de `'_'`. Si se establece `queue_name_prefix`, entonces `queue_name_delimiter` se une al prefijo y al nombre de la cola sin prefijo.
                                           
   La siguiente configuración pondría en cola el trabajo proporcionado en la cola `video_server.low_priority`:

    ```ruby
    # prefix must be set for delimiter to be used
    config.active_job.queue_name_prefix = 'video_server'
    config.active_job.queue_name_delimiter = '.'
    ```

    ```ruby
    class EncoderJob < ActiveJob::Base
      queue_as :low_priority
      #....
    end
    ```

* `config.active_job.logger` acepta un registrador que se ajusta a la interfaz de Log4r o la clase Ruby Logger predeterminada, que luego se usa para registrar información de Active Job. Puede recuperar este registrador llamando a `logger` en una clase de trabajo activo o en una instancia de trabajo activo. Establezca en `nil` para deshabilitar el registro.

* `config.active_job.custom_serializers` permite configurar serializadores de argumentos personalizados. Por defecto es `[]`.

* `config.active_job.return_false_on_aborted_enqueue` cambia el valor de retorno de `#enqueue` a falso en lugar de la instancia del trabajo cuando se aborta la puesta en cola. El valor predeterminado es `false`.

* `config.active_job.log_arguments` controla si los argumentos de un trabajo están registrados. El valor predeterminado es `true`.

* `config.active_job.retry_jitter` controla la cantidad de" jitter "(variación aleatoria) aplicada al tiempo de retardo calculado al reintentar trabajos fallidos. El valor predeterminado es `0,15`.

### Configuring Action Cable

* `config.action_cable.url` acepta una cadena para la URL donde
 está alojando su servidor Action Cable. Usarías esta opción
si está ejecutando servidores Action Cable que están separados de su
aplicación principal.
* `config.action_cable.mount_path` acepta una cadena sobre dónde montar la acción
  Cable, como parte del proceso del servidor principal. El valor predeterminado es `/cable`.
Puede configurar esto como nil para no montar Action Cable como parte de su
servidor normal de Rails.

Puede encontrar opciones de configuración más detalladas en el
[Action Cable Overview](action_cable_overview.html#configuration).


### Configuring Active Storage

`config.active_storage` proporciona las siguientes opciones de configuración:

* `config.active_storage.variant_processor` acepta un símbolo `:mini_magick` o `:vips`, especificando si las transformaciones variantes se realizarán con MiniMagick o ruby-vips. El valor predeterminado es `:mini_magick`.

* `config.active_storage.analyzers` acepta una matriz de clases que indican los analizadores disponibles para los blobs de Active Storage. El valor predeterminado es `[ActiveStorage::Analyzer::ImageAnalyzer, ActiveStorage::Analyzer::VideoAnalyzer]`. El primero puede extraer el ancho y el alto de una mancha de imagen; este último puede extraer el ancho, alto, duración, ángulo y relación de aspecto de un blob de video.

* `config.active_storage.previewers` acepta una serie de clases que indican los previsualizadores de imágenes disponibles en los blobs de Active Storage. El valor predeterminado es `[ActiveStorage::Previewer::PopplerPDFPreviewer, ActiveStorage::Previewer::MuPDFPreviewer, ActiveStorage::Previewer::VideoPreviewer]`. `PopplerPDFPreviewer` y `MuPDFPreviewer`  pueden generar una miniatura desde la primera página de un blob PDF; `VideoPreviewer` del fotograma relevante de un blob de vídeo.

* `config.active_storage.paths` acepta un hash de opciones que indican la ubicación de los comandos del analizador / vista previa. El valor predeterminado es `{}`, lo que significa que los comandos se buscarán en la ruta predeterminada. Puede incluir cualquiera de estas opciones:
    * `:ffprobe` - La ubicación del ejecutable ffprobe.
    * `:mutool` - La ubicación del ejecutable mutool.
    * `:ffmpeg` - La ubicación del ejecutable ffmpeg.

   ```ruby
   config.active_storage.paths[:ffprobe] = '/usr/local/bin/ffprobe'
   ```

* `config.active_storage.variable_content_types` acepta una matriz de cadenas que indican los tipos de contenido que Active Storage puede transformar a través de ImageMagick. El valor predeterminado es `%w(image/png image/gif image/jpg image/jpeg image/pjpeg image/tiff image/bmp image/vnd.adobe.photoshop image/vnd.microsoft.icon image/webp)`.

* `config.active_storage.web_image_content_types` acepta una serie de cadenas consideradas como tipos de contenido de imágenes web en las que se pueden procesar variantes sin convertirlas al formato PNG de reserva. Si desea utilizar variantes de `WebP` en su aplicación, puede agregar `image/webp` a esta matriz. El valor predeterminado es `%w(image/png image/jpeg image/jpg image/gif)`.

* `config.active_storage.content_types_to_serve_as_binary` acepta una matriz de cadenas que indican los tipos de contenido que Active Storage siempre servirá como un archivo adjunto, en lugar de en línea. El valor predeterminado es `%w(texto/html
text/javascript image/svg+xml application/postscript application/x-shockwave-flash text/xml application/xml application/xhtml+xml application/mathml+xml text/cache-manifest)`.

* `config.active_storage.content_types_allowed_inline` acepta una matriz de cadenas que indican los tipos de contenido que Active Storage permite servir como en línea. El valor predeterminado es  `%w(image/png image/gif image/jpg image/jpeg image/vnd.adobe.photoshop image/vnd.microsoft.icon application/pdf)`.

* `config.active_storage.queues.analysis` acepta un símbolo que indica la cola de trabajos activos que se utilizarán para los trabajos de análisis. Cuando esta opción es `nil`, los trabajos de análisis se envían a la cola de trabajos activos predeterminada (consulte` config.active_job.default_queue_name`).

    ```ruby
      config.active_storage.queues.analysis = :low_priority
      ```

* `config.active_storage.queues.purge` acepta un símbolo que indica la cola de trabajos activos que se utilizará para depurar trabajos. Cuando esta opción es `nil`, los trabajos de depuración se envían a la cola de trabajos activos predeterminada (consulte `config.active_job.default_queue_name`).

   ```ruby
    config.active_storage.queues.purge = :low_priority
    ```

* `config.active_storage.queues.mirror` acepta un símbolo que indica la cola de trabajos activos que se utilizará para los trabajos de duplicación de carga directa. El valor predeterminado es `:active_storage_mirror`.

 ```ruby
  config.active_storage.queues.mirror = :low_priority
  ```

* `config.active_storage.logger` se puede usar para configurar el registrador usado por Active Storage. Acepta un registrador conforme a la interfaz de Log4r o la clase Ruby Logger predeterminada.

  ```ruby
  config.active_storage.logger = ActiveSupport::Logger.new(STDOUT)
  ```

* `config.active_storage.service_urls_expire_in` determina el vencimiento predeterminado de las URL generadas por:
  * `ActiveStorage :: Blob # url`
  * `ActiveStorage :: Blob # service_url_for_direct_upload`
  * `ActiveStorage :: Variant # url`

  El valor predeterminado es 5 minutos.

* `config.active_storage.routes_prefix` se puede usar para establecer el prefijo de ruta para las rutas servidas por Active Storage. Acepta una cadena que se antepondrá a las rutas generadas.

  ```ruby
  config.active_storage.routes_prefix = '/files'
  ```

  El valor predeterminado es `/ rails / active_storage`.

* `config.active_storage.replace_on_assign_to_many` determina si la asignación a una colección de adjuntos declarados con` has_many_attached` reemplaza los adjuntos existentes o los agrega. El valor predeterminado es `true`.

* `config.active_storage.track_variants` determina si las variantes se registran en la base de datos. El valor predeterminado es `true`.

* `config.active_storage.draw_routes` se puede usar para alternar la generación de rutas de almacenamiento activo. El valor predeterminado es `true`.

* `config.active_storage.resolve_model_to_route` se puede usar para cambiar globalmente cómo se entregan los archivos de Active Storage.

  Los valores permitidos son:
  * `: rails_storage_redirect`: Redirigir a URL de servicio de corta duración firmadas.
  * `: rails_storage_proxy`: Archivos proxy descargándolos.

  El valor predeterminado es `:rails_storage_redirect`.

### Results of `config.load_defaults`

`config.load_defaults` establece nuevos valores predeterminados hasta la versión aprobada e incluida. De modo que, al pasar, digamos, '6.0', también se obtienen los nuevos valores predeterminados de todas las versiones anteriores.


#### For '6.1', new defaults from previous versions below and:

- `config.active_record.has_many_inversing`: `true`
- `config.active_storage.track_variants`: `true`

#### For '6.0', new defaults from previous versions below and:

- `config.autoloader`: `:zeitwerk`
- `config.action_view.default_enforce_utf8`: `false`
- `config.action_dispatch.use_cookies_with_metadata`: `true`
- `config.action_dispatch.return_only_media_type_on_content_type`: `false`
- `config.action_mailer.delivery_job`: `"ActionMailer::MailDeliveryJob"`
- `config.active_job.return_false_on_aborted_enqueue`: `true`
- `config.active_storage.queues.analysis`: `:active_storage_analysis`
- `config.active_storage.queues.purge`: `:active_storage_purge`
- `config.active_storage.replace_on_assign_to_many`: `true`
- `config.active_record.collection_cache_versioning`: `true`

#### For '5.2', new defaults from previous versions below and:

- `config.active_record.cache_versioning`: `true`
- `config.action_dispatch.use_authenticated_cookie_encryption`: `true`
- `config.active_support.use_authenticated_message_encryption`: `true`
- `config.active_support.use_sha1_digests`: `true`
- `config.action_controller.default_protect_from_forgery`: `true`
- `config.action_view.form_with_generates_ids`: `true`

#### For '5.1', new defaults from previous versions below and:

- `config.assets.unknown_asset_fallback`: `false`
- `config.action_view.form_with_generates_remote_forms`: `true`

#### For '5.0':

- `config.action_controller.per_form_csrf_tokens`: `true`
- `config.action_controller.forgery_protection_origin_check`: `true`
- `ActiveSupport.to_time_preserves_timezone`: `true`
- `config.active_record.belongs_to_required_by_default`: `true`
- `config.ssl_options`: `{ hsts: { subdomains: true } }`

### Configuring a Database

Casi todas las aplicaciones de Rails interactuarán con una base de datos. Puede conectarse a la base de datos configurando una variable de entorno `ENV['DATABASE_URL']` o usando un archivo de configuración llamado `config/database.yml`.

Usando el archivo `config/database.yml` puede especificar toda la información necesaria para acceder a su base de datos:

```yaml
development:
  adapter: postgresql
  database: blog_development
  pool: 5
```

Esto se conectará a la base de datos llamada `blog_development` usando el adaptador `postgresql`. Esta misma información se puede almacenar en una URL y proporcionar a través de una variable de entorno como esta:

```ruby
> puts ENV['DATABASE_URL']
postgresql://localhost/blog_development?pool=5
```

El archivo `config/database.yml` contiene secciones para tres entornos diferentes en los que Rails puede ejecutarse de forma predeterminada:

* El entorno de `development` se utiliza en su computadora de desarrollo / local mientras interactúa manualmente con la aplicación.
* El entorno `test` se utiliza cuando se ejecutan pruebas automatizadas.
* El entorno de `production` se utiliza cuando implementas tu aplicación para que la use el mundo.

Si lo desea, puede especificar manualmente una URL dentro de su `config/database.yml`

```yaml
development:
  url: postgresql://localhost/blog_development?pool=5
```

El archivo `config/database.yml` puede contener etiquetas ERB` <% =%> `. Cualquier cosa en las etiquetas se evaluará como código Ruby. Puede usar esto para extraer datos de una variable de entorno o realizar cálculos para generar la información de conexión necesaria.


TIP: No es necesario que actualice las configuraciones de la base de datos manualmente. Si observa las opciones del generador de aplicaciones, verá que una de las opciones se llama `--database`. Esta opción le permite elegir un adaptador de una lista de las bases de datos relacionales más utilizadas. Incluso puede ejecutar el generador repetidamente: `cd .. && rails new blog --database = mysql`. Cuando confirme la sobrescritura del archivo `config/database.yml`, su aplicación se configurará para MySQL en lugar de SQLite. A continuación se muestran ejemplos detallados de las conexiones de bases de datos comunes.

### Connection Preference

Dado que hay dos formas de configurar su conexión (usando `config/database.yml` o usando una variable de entorno), es importante comprender cómo pueden interactuar.

Si tiene un archivo `config/database.yml` vacío pero su `ENV['DATABASE_URL'] `está presente, Rails se conectará a la base de datos a través de su variable de entorno:

```bash
$ cat config/database.yml

$ echo $DATABASE_URL
postgresql://localhost/my_database
```

Si tiene un `config/database.yml` pero no un `ENV['DATABASE_URL']`, entonces este archivo se usará para conectarse a su base de datos:

```bash
$ cat config/database.yml
development:
  adapter: postgresql
  database: my_database
  host: localhost

$ echo $DATABASE_URL
```

Si tiene configurados `config/database.yml` y `ENV['DATABASE_URL']`, Rails fusionará la configuración. Para entender mejor esto debemos ver algunos ejemplos.

Cuando se proporciona información de conexión duplicada, la variable de entorno tendrá prioridad:

```bash
$ cat config/database.yml
development:
  adapter: sqlite3
  database: NOT_my_database
  host: localhost

$ echo $DATABASE_URL
postgresql://localhost/my_database

$ bin/rails runner 'puts ActiveRecord::Base.configurations'
#<ActiveRecord::DatabaseConfigurations:0x00007fd50e209a28>

$ bin/rails runner 'puts ActiveRecord::Base.configurations.inspect'
#<ActiveRecord::DatabaseConfigurations:0x00007fc8eab02880 @configurations=[
  #<ActiveRecord::DatabaseConfigurations::UrlConfig:0x00007fc8eab020b0
    @env_name="development", @spec_name="primary",
    @config={"adapter"=>"postgresql", "database"=>"my_database", "host"=>"localhost"}
    @url="postgresql://localhost/my_database">
  ]
```

Aquí, el adaptador, el host y la base de datos coinciden con la información de `ENV['DATABASE_URL']`.

Si se proporciona información no duplicada, obtendrá todos los valores únicos, la variable de entorno aún tiene prioridad en casos de conflictos.

```bash
$ cat config/database.yml
development:
  adapter: sqlite3
  pool: 5

$ echo $DATABASE_URL
postgresql://localhost/my_database

$ bin/rails runner 'puts ActiveRecord::Base.configurations'
#<ActiveRecord::DatabaseConfigurations:0x00007fd50e209a28>

$ bin/rails runner 'puts ActiveRecord::Base.configurations.inspect'
#<ActiveRecord::DatabaseConfigurations:0x00007fc8eab02880 @configurations=[
  #<ActiveRecord::DatabaseConfigurations::UrlConfig:0x00007fc8eab020b0
    @env_name="development", @spec_name="primary",
    @config={"adapter"=>"postgresql", "database"=>"my_database", "host"=>"localhost", "pool"=>5}
    @url="postgresql://localhost/my_database">
  ]
```

Dado que el grupo no está en la información de conexión `ENV['DATABASE_URL']` proporcionada, su información se fusiona. Dado que el `adapter` está duplicado, la información de conexión `ENV['DATABASE_URL']` gana.

La única forma de no usar explícitamente la información de conexión en `ENV['DATABASE_URL']` es especificar una conexión URL explícita usando la subclave `"url"`:

```bash
$ cat config/database.yml
development:
  url: sqlite3:NOT_my_database

$ echo $DATABASE_URL
postgresql://localhost/my_database

$ bin/rails runner 'puts ActiveRecord::Base.configurations'
#<ActiveRecord::DatabaseConfigurations:0x00007fd50e209a28>

$ bin/rails runner 'puts ActiveRecord::Base.configurations.inspect'
#<ActiveRecord::DatabaseConfigurations:0x00007fc8eab02880 @configurations=[
  #<ActiveRecord::DatabaseConfigurations::UrlConfig:0x00007fc8eab020b0
    @env_name="development", @spec_name="primary",
    @config={"adapter"=>"sqlite3", "database"=>"NOT_my_database"}
    @url="sqlite3:NOT_my_database">
  ]
```

Aquí se ignora la información de conexión en `ENV['DATABASE_URL']`, observe el adaptador y el nombre de la base de datos diferentes.

Dado que es posible incrustar ERB en su `config/database.yml`, es una buena práctica mostrar explícitamente que está utilizando el `ENV['DATABASE_URL']` para conectarse a su base de datos. Esto es especialmente útil en producción, ya que no debe enviar secretos como la contraseña de su base de datos en su control de código fuente (como Git).

```bash
$ cat config/database.yml
production:
  url: <%= ENV['DATABASE_URL'] %>
```

Ahora el comportamiento es claro, que solo estamos usando la información de conexión en `ENV['DATABASE_URL']`.

#### Configuring an SQLite3 Database

Rails viene con soporte integrado para [SQLite3](http://www.sqlite.org), que es una aplicación de base de datos liviana sin servidor. Si bien un entorno de producción ajetreado puede sobrecargar SQLite, funciona bien para el desarrollo y las pruebas. Rails utiliza de forma predeterminada una base de datos SQLite al crear un nuevo proyecto, pero siempre puede cambiarlo más tarde.

Aquí está la sección del archivo de configuración predeterminado (`config/database.yml`) con información de conexión para el entorno de desarrollo:

```yaml
development:
  adapter: sqlite3
  database: db/development.sqlite3
  pool: 5
  timeout: 5000
```

NOTA: Rails usa una base de datos SQLite3 para el almacenamiento de datos de forma predeterminada porque es una base de datos de configuración cero que simplemente funciona. Rails también es compatible con MySQL (incluido MariaDB) y PostgreSQL "listo para usar", y tiene complementos para muchos sistemas de bases de datos. Si está utilizando una base de datos en un entorno de producción, lo más probable es que Rails tenga un adaptador para ella.

#### Configuring a MySQL or MariaDB Database

Si elige usar MySQL o MariaDB en lugar de la base de datos SQLite3 enviada, su `config/database.yml` se verá un poco diferente. Aquí está la sección de desarrollo:

```yaml
development:
  adapter: mysql2
  encoding: utf8mb4
  database: blog_development
  pool: 5
  username: root
  password:
  socket: /tmp/mysql.sock
```

Si su base de datos de desarrollo tiene un usuario root con una contraseña vacía, esta configuración debería funcionar para usted. De lo contrario, cambie el nombre de usuario y la contraseña en la sección de desarrollo según corresponda.

NOTE: Si su versión de MySQL es 5.5 o 5.6 y desea usar el juego de caracteres `utf8mb4` por defecto, configure su servidor MySQL para que admita el prefijo de clave más largo habilitando la variable de sistema `innodb_large_prefix`.

Los bloqueos de aviso están habilitados de forma predeterminada en MySQL y se utilizan para hacer que las migraciones de bases de datos sean seguras al mismo tiempo. Puede deshabilitar los bloqueos de aviso configurando `advisory_locks` en `false`:

```yaml
production:
  adapter: mysql2
  advisory_locks: false
```

#### Configuring a PostgreSQL Database

Si elige usar PostgreSQL, su `config/database.yml` se personalizará para usar bases de datos PostgreSQL:

```yaml
development:
  adapter: postgresql
  encoding: unicode
  database: blog_development
  pool: 5
```

Si está habilitado, Active Record creará hasta `1000` declaraciones preparadas por conexión de base de datos de forma predeterminada. Para modificar este comportamiento, puede establecer `statement_limit` en un valor diferente:

```yaml
production:
  adapter: postgresql
  statement_limit: 200
```

Cuantas más declaraciones preparadas se utilicen: más memoria necesitará su base de datos. Si su base de datos PostgreSQL está alcanzando los límites de memoria, intente reducir `statement_limit` o deshabilitar las declaraciones preparadas.

#### Configuring an SQLite3 Database for JRuby Platform

Si elige usar SQLite3 y está usando JRuby, su `config/database.yml` se verá un poco diferente. Aquí está la sección de desarrollo:


```yaml
development:
  adapter: jdbcsqlite3
  database: db/development.sqlite3
```

#### Configuring a MySQL or MariaDB Database for JRuby Platform

Si elige usar MySQL o MariaDB y está usando JRuby, su `config/database.yml` se verá un poco diferente. Aquí está la sección de desarrollo:

```yaml
development:
  adapter: jdbcmysql
  database: blog_development
  username: root
  password:
```

#### Configuring a PostgreSQL Database for JRuby Platform

Si elige utilizar PostgreSQL y está utilizando JRuby, su `config/database.yml` se verá un poco diferente. Aquí está la sección de desarrollo:

```yaml
development:
  adapter: jdbcpostgresql
  encoding: unicode
  database: blog_development
  username: blog
  password:
```

Change the username and password in the `development` section as appropriate.

#### Configuring Metadata Storage

De forma predeterminada, Rails almacenará información sobre su entorno y esquema de Rails
en una tabla interna llamada `ar_internal_metadata`.

Para desactivar esto por conexión, configure `use_metadata_table` en su base de datos
configuración. Esto es útil cuando se trabaja con una base de datos compartida y / o
usuario de la base de datos que no puede crear tablas.

```yaml
development:
  adapter: postgresql
  use_metadata_table: false
```

### Creating Rails Environments

Por defecto, Rails viene con tres entornos: "desarrollo", "prueba" y "producción". Si bien estos son suficientes para la mayoría de los casos de uso, hay circunstancias en las que desea más entornos.

Imagine que tiene un servidor que refleja el entorno de producción pero que solo se utiliza para realizar pruebas. Este tipo de servidor se denomina comúnmente "servidor provisional". Para definir un entorno llamado "staging" para este servidor, simplemente cree un archivo llamado `config/environment/staging.rb`. Utilice el contenido de cualquier archivo existente en `config/environment` como punto de partida y realice los cambios necesarios desde allí.

Ese entorno no es diferente a los predeterminados, inicie un servidor con `bin/rails server -e staging`, una consola con `bin/rails console -e staging`, `Rails.env.staging?` Funciona, etc.

### Deploy to a Subdirectory (relative URL root)

De forma predeterminada, Rails espera que su aplicación se ejecute en la raíz
(por ejemplo, `/`). Esta sección explica cómo ejecutar su aplicación dentro de un directorio.

Supongamos que queremos implementar nuestra aplicación en "/app1". Rails necesita saber
este directorio para generar las rutas apropiadas:

```ruby
config.relative_url_root = "/app1"
```

alternativamente, puede configurar el entorno `RAILS_RELATIVE_URL_ROOT`
variable.

Los carriles ahora antepondrán "/ app1" al generar enlaces.

#### Using Passenger

Passenger facilita la ejecución de su aplicación en un subdirectorio. Puede encontrar la configuración correspondiente en el [Manual del pasajero](https://www.phusionpassenger.com/library/deploy/apache/deploy/ruby/#deploying-an-app-to-a-sub-uri-or-subdirectorio).

#### Using a Reverse Proxy

La implementación de su aplicación utilizando un proxy inverso tiene claras ventajas sobre las implementaciones tradicionales. Le permiten tener más control sobre su servidor al colocar en capas los componentes requeridos por su aplicación.

Muchos servidores web modernos se pueden utilizar como servidor proxy para equilibrar elementos de terceros, como servidores de almacenamiento en caché o servidores de aplicaciones.

Uno de esos servidores de aplicaciones que puede usar es [Unicorn](https://bogomips.org/unicorn/) para ejecutarse detrás de un proxy inverso.

En este caso, necesitaría configurar el servidor proxy (NGINX, Apache, etc.) para aceptar conexiones desde su servidor de aplicaciones (Unicorn). De forma predeterminada, Unicorn escuchará las conexiones TCP en el puerto 8080, pero puede cambiar el puerto o configurarlo para usar sockets en su lugar.

Puede encontrar más información en el [Léame de Unicornio](https://bogomips.org/unicorn/README.html) y comprender la [filosofía](https://bogomips.org/unicorn/PHILOSOPHY.html) que hay detrás.

Una vez que haya configurado el servidor de aplicaciones, debe enviarle solicitudes mediante proxy configurando su servidor web de manera adecuada. Por ejemplo, su configuración de NGINX puede incluir:

```
upstream application_server {
  server 0.0.0.0:8080;
}

server {
  listen 80;
  server_name localhost;

  root /root/path/to/your_app/public;

  try_files $uri/index.html $uri.html @app;

  location @app {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://application_server;
  }

  # some other configuration
}
```

Asegúrese de leer la [documentación de NGINX](https://nginx.org/en/docs/) para obtener la información más actualizada.

Rails Environment Settings
--------------------------

Algunas partes de Rails también se pueden configurar externamente proporcionando variables de entorno. Varias partes de Rails reconocen las siguientes variables de entorno:

* `ENV[" RAILS_ENV"]` define el entorno de Rails (producción, desarrollo, prueba, etc.) en el que se ejecutará Rails.

* `ENV[" RAILS_RELATIVE_URL_ROOT"]` es utilizado por el código de enrutamiento para reconocer las URL cuando [implementa su aplicación en un subdirectorio](configuring.html#deploy-to-a-subdirectory-related-url-root).

* `ENV[" RAILS_CACHE_ID"]` y `ENV["RAILS_APP_VERSION"]` se utilizan para generar claves de caché expandidas en el código de caché de Rails. Esto le permite tener múltiples cachés separados de la misma aplicación.


Using Initializer Files
-----------------------

Después de cargar el marco y cualquier gema en su aplicación, Rails pasa a cargar inicializadores. Un inicializador es cualquier archivo Ruby almacenado en `config/initializers` en su aplicación. Puede usar inicializadores para mantener los ajustes de configuración que deben realizarse después de que se carguen todos los marcos y gemas, como las opciones para configurar los ajustes de estas partes.

NOTE: No hay garantía de que sus inicializadores se ejecuten después de todos los inicializadores de gemas, por lo que cualquier código de inicialización que dependa de que se haya inicializado una gema determinada debe ir a un bloque `config.after_initialize`.

NOTE: Puede utilizar subcarpetas para organizar sus inicializadores si lo desea, porque Rails examinará toda la jerarquía de archivos desde la carpeta de inicializadores hacia abajo.

TIP: Si bien Rails admite la numeración de los nombres de los archivos de inicialización con el fin de ordenar la carga, una mejor técnica es colocar cualquier código que deba cargarse en un orden específico dentro del mismo archivo. Esto reduce la pérdida de nombres de archivos, hace que las dependencias sean más explícitas y puede ayudar a que surjan nuevos conceptos dentro de su aplicación.

Initialization events
---------------------

Rails tiene 5 eventos de inicialización a los que se pueden conectar (enumerados en el orden en que se ejecutan):

* `before_configuration`: Esto se ejecuta tan pronto como la constante de la aplicación hereda de `Rails::Application`. Las llamadas `config` se evalúan antes de que esto suceda.

* `before_initialize`: Esto se ejecuta directamente antes de que ocurra el proceso de inicialización de la aplicación con el inicializador `:bootstrap_hook` cerca del comienzo del proceso de inicialización de Rails.

* `to_prepare`: se ejecuta después de que se ejecuten los inicializadores para todos los Railties (incluida la aplicación en sí), pero antes de la carga ansiosa y la pila de middleware está construida. Más importante aún, se ejecutará en cada solicitud en `development`, pero solo una vez (durante el arranque) en `production` y `test`.

* `before_eager_load`: Esto se ejecuta directamente antes de que ocurra la carga ansiosa, que es el comportamiento predeterminado para el entorno de `production` y no para el entorno de `development`.

* `after_initialize`: se ejecuta directamente después de la inicialización de la aplicación, después de que se ejecutan los inicializadores de la aplicación en `config/initializers`.

Para definir un evento para estos ganchos, use la sintaxis de bloque dentro de una subclase `Rails::Application`, `Rails::Railtie` o `Rails::Engine`:

```ruby
module YourApp
  class Application < Rails::Application
    config.before_initialize do
      # initialization code goes here
    end
  end
end
```

Alternativamente, también puede hacerlo a través del método `config` en el objeto` Rails.application`:

``` rubí
Rails.application.config.before_initialize hacer
  # código de inicialización va aquí
final
```

WARNING: Algunas partes de su aplicación, en particular el enrutamiento, aún no están configuradas en el punto donde se llama al bloque `after_initialize`.

### `Rails::Railtie#initializer`

Rails tiene varios inicializadores que se ejecutan al inicio y que se definen utilizando el método `initializer` de` Rails::Railtie`. Aquí hay un ejemplo del inicializador `set_helpers_path` de Action Controller:

```ruby
initializer "action_controller.set_helpers_path" do |app|
  ActionController::Helpers.helpers_path = app.helpers_paths
end
```

El método `initializer` toma tres argumentos, el primero es el nombre del inicializador y el segundo es un hash de opciones (no se muestra aquí) y el tercero es un bloque. La clave `:before` en el hash de opciones se puede especificar para especificar qué inicializador debe ejecutar este nuevo inicializador, y la clave `:after` especificará qué inicializador ejecutar este inicializador _después_.

Los inicializadores definidos mediante el método `initializer` se ejecutarán en el orden en el que están definidos, con la excepción de los que utilizan los métodos `:before` o `:after`.

ADVERTENCIA: Puede poner su inicializador antes o después de cualquier otro inicializador en la cadena, siempre que sea lógico. Supongamos que tiene 4 inicializadores llamados "uno" a "cuatro" (definidos en ese orden) y define "cuatro" para ir _antes_ "cuatro" pero _después_ "tres", eso no es lógico y Rails no podrá Determine su orden de inicializador.

El argumento de bloque del método `initializer` es la instancia de la aplicación en sí, por lo que podemos acceder a la configuración usando el método `config` como se hizo en el ejemplo.

Debido a que `Rails::Application` hereda de` Rails :: Railtie` (indirectamente), puede usar el método `initializer` en `config/application.rb` para definir inicializadores para la aplicación.

### Initializers

A continuación se muestra una lista completa de todos los inicializadores que se encuentran en Rails en el orden en que están definidos (y por lo tanto se ejecutan, a menos que se indique lo contrario).

* `load_environment_hook`: Sirve como marcador de posición para que `:load_environment_config` pueda definirse para ejecutarse antes.

* `load_active_support`: Requiere `active_support/dependencies` que establece la base para Active Support. Opcionalmente requiere `active_support/all` si` config.active_support.bare` no es veraz, que es el valor predeterminado.

* `initialize_logger`: Inicializa el registrador (un objeto` ActiveSupport::Logger`) para la aplicación y lo hace accesible en `Rails.logger`, siempre que ningún inicializador insertado antes de este punto haya definido `Rails.logger`.

* `initialize_cache`: Si `Rails.cache` no está configurado todavía, inicializa el caché haciendo referencia al valor en `config.cache_store` y almacena el resultado como `Rails.cache`. Si este objeto responde al método `middleware`, su middleware se inserta antes de `Rack::Runtime` en la pila de middleware.

* `set_clear_dependencies_hook`: Este inicializador, que se ejecuta solo si` cache_classes` se establece en `false`, usa` ActionDispatch::Callbacks.after` para eliminar las constantes a las que se ha hecho referencia durante la solicitud desde el espacio de objetos para que recargarse durante la siguiente solicitud.

* `initialize_dependency_mechanism`: Si `config.cache_classes` es verdadero, configura `ActiveSupport::Dependencies.mechanism` para `require` dependencias en lugar de `load`.

* `bootstrap_hook`: Ejecuta todos los bloques configurados ` before_initialize`.

* `i18n.callbacks`: En el entorno de desarrollo, configura una devolución de llamada` to_prepare` que llamará a `I18n.reload!` si alguna de las configuraciones regionales ha cambiado desde la última solicitud. En el modo de producción, esta devolución de llamada solo se ejecutará en la primera solicitud.

* `active_support.deprecation_behavior`: Configura informes de obsolescencia para entornos, de forma predeterminada en `:log` para desarrollo, `:notificar` para producción y`:stderr` para prueba. Si no se establece un valor para `config.active_support.deprecation`, este inicializador solicitará al usuario que configure esta línea en el archivo `config/environment` del entorno actual. Puede establecerse en una matriz de valores. Este inicializador también configura comportamientos para desaprobaciones no permitidas, por defecto en `:raise` para desarrollo y prueba y `:log` para producción. Las advertencias de desaprobación no permitidas se establecen de forma predeterminada en una matriz vacía.

* `active_support.initialize_time_zone`: Establece la zona horaria predeterminada para la aplicación según la configuración` config.time_zone`, que por defecto es "UTC".

* `active_support.initialize_beginning_of_week`: Establece el comienzo de semana predeterminado para la aplicación según la configuración de` config.beginning_of_week`, que por defecto es `:monday`.

* `active_support.set_configs`: Configura Active Support usando la configuración en `config.active_support` al `send` 'los nombres de los métodos como configuradores a `ActiveSupport` y pasando los valores.

* `action_dispatch.configure`: Configura el `ActionDispatch::Http::URL.tld_length` para que se establezca en el valor de `config.action_dispatch.tld_length`.

* `action_view.set_configs`: Configura la vista de acción usando la configuración en `config.action_view` al `send` 'los nombres de los métodos como definidores a `ActionView::Base` y pasando los valores.

* `action_controller.assets_config`: Inicializa el `config.actions_controller.assets_dir` en el directorio público de la aplicación si no está configurado explícitamente.

* `action_controller.set_helpers_path`: Establece el `helpers_path` del controlador de acción en el `helpers_path` de la aplicación.

* `action_controller.parameters_config`: Configura opciones de parámetros fuertes para` ActionController::Parameters`.

* `action_controller.set_configs`: Configura el controlador de acción usando la configuración en` config.action_controller` al `enviar` 'los nombres de los métodos como configuradores a `ActionController::Base` y pasando los valores.

* `action_controller.compile_config_methods`: Inicializa métodos para la configuración especificada para que sean más rápidos de acceder.

* `active_record.initialize_timezone`: Establece `ActiveRecord::Base.time_zone_aware_attributes` en `true`, así como también establece `ActiveRecord::Base.default_timezone` en UTC. Cuando los atributos se leen de la base de datos, se convertirán a la zona horaria especificada por `Time.zone`.

* `active_record.logger`: Establece `ActiveRecord::Base.logger`, si aún no lo está, en `Rails.logger`.

* `active_record.migration_error`: Configura el middleware para comprobar si hay migraciones pendientes.

* `active_record.check_schema_cache_dump`: Carga el volcado de caché del esquema si está configurado y disponible.

* `active_record.warn_on_records_fetched_greater_than`: habilita advertencias cuando las consultas devuelven un gran número de registros.

* `active_record.set_configs`: Configura Active Record usando la configuración en `config.active_record` enviando los nombres de los métodos como configuradores a `ActiveRecord::Base` y pasando los valores.

* `active_record.initialize_database`: Carga la configuración de la base de datos (por defecto) desde `config/database.yml` y establece una conexión para el entorno actual.

* `active_record.log_runtime`: Incluye `ActiveRecord::Railties::ControllerRuntime`, que es responsable de informar al registrador el tiempo que tardan las llamadas de Active Record en la solicitud.

* `active_record.set_reloader_hooks`: Restablece todas las conexiones recargables a la base de datos si `config.cache_classes` se establece en `false`.

* `active_record.add_watchable_files`: Agrega archivos `schema.rb` y `structure.sql` a los archivos observables.

* `active_job.logger`: Configura `activeJob::Base.logger` - si aún no está configurado -
  a `Rails.logger`.

* `active_job.set_configs`: Configura el trabajo activo usando la configuración en `config.active_job` enviando los nombres de los métodos como configuradores a `ActiveJob::Base` y pasando los valores.

* `action_mailer.logger`: Establece `ActionMailer::Base.logger`, si aún no lo está, en `Rails.logger`.

* `action_mailer.set_configs`: Configura Action Mailer usando la configuración en `config.action_mailer` al `enviar` 'los nombres de los métodos como establecedores a `ActionMailer::Base` y pasando los valores.

* `action_mailer.compile_config_methods`: Inicializa métodos para la configuración especificada para que sean más rápidos de acceder.

* `set_load_path`: Este inicializador se ejecuta antes de `bootstrap_hook`. Agrega las rutas especificadas por `config.load_paths` y todas las rutas de carga automática a `$ LOAD_PATH`.

* `set_autoload_paths`: Este inicializador se ejecuta antes de `bootstrap_hook`. Agrega todos los subdirectorios de `app` y las rutas especificadas por `config.autoload_paths`, `config.eager_load_paths` y `config.autoload_once_paths` a `ActiveSupport::Dependencies.autoload_paths`.

* `add_routing_paths`: Carga (por defecto) todos los archivos `config/routes.rb` (en la aplicación y railties, incluidos los motores) y configura las rutas para la aplicación.

* `add_locales`: Agrega los archivos en `config/locales` (de la aplicación, railties y motores) a `I18n.load_path`, haciendo disponibles las traducciones en estos archivos.

* `add_view_paths`: Agrega el directorio `app/views` de la aplicación, railties y motores a la ruta de búsqueda para ver los archivos de la aplicación.

* `load_environment_config`: Carga el archivo `config/environment` para el entorno actual.

* `prepend_helpers_path`: Agrega el directorio `app/helpers` de la aplicación, railties y motores a la ruta de búsqueda de ayudantes para la aplicación.

* `load_config_initializers`: Carga todos los archivos Ruby de `config/initializers` en la aplicación, railties y motores. Los archivos de este directorio se pueden usar para guardar los ajustes de configuración que deben realizarse después de que se carguen todos los marcos.

* `motors_blank_point`: proporciona un punto de inicialización para engancharse si desea hacer algo antes de que se carguen los motores. Después de este punto, se ejecutan todos los inicializadores de motor y railtie.

* `add_generator_templates`: Encuentra plantillas para generadores en `lib/\ templates` para la aplicación, railties y motores y las agrega a la configuración `config.generators.templates`, lo que hará que las plantillas estén disponibles para que todos los generadores puedan consultarlas.

* `sure_autoload_once_paths_as_subset`: Asegura que `config.autoload_once_paths` solo contenga rutas de `config.autoload_paths`. Si contiene rutas adicionales, se generará una excepción.

* `add_to_prepare_blocks`: El bloque para cada llamada `config.to_prepare` en la aplicación, un railtie o motor se agrega a las devoluciones de llamada `to_prepare` para Action Dispatch que se ejecutarán por solicitud en desarrollo, o antes de la primera solicitud en producción.

* `add_builtin_route`: Si la aplicación se está ejecutando en el entorno de desarrollo, esto agregará la ruta para `rails/info/properties` a las rutas de la aplicación. Esta ruta proporciona la información detallada, como la versión Rails y Ruby para `public/index.html` en una aplicación Rails predeterminada.

* `build_middleware_stack`: Construye la pila de middleware para la aplicación, devolviendo un objeto que tiene un método` call` que toma un objeto de entorno Rack para la solicitud.

* `eager_load!`: Si `config.eager_load` es `true`, ejecuta los hooks `config.before_eager_load` y luego llama a `eager_load! `que cargará todos los `config.eager_load_namespaces`.

* `finisher_hook`: Proporciona un gancho para después de que se completa la inicialización del proceso de la aplicación, así como para ejecutar todos los bloques` config.after_initialize` para la aplicación, railties y motores.

* `set_routes_reloader_hook`: Configura Action Dispatch para recargar el archivo de rutas usando `ActiveSupport::Callbacks.to_run`.

* `disable_dependency_loading`: Desactiva la carga automática de dependencias si `config.eager_load` se establece en `true`.

Las conexiones de la base de datos de Active Record son administradas por `ActiveRecord::ConnectionAdapters::ConnectionPool`, que asegura que un grupo de conexiones sincronice la cantidad de acceso de subprocesos a un número limitado de conexiones de base de datos. Este límite predeterminado es 5 y se puede configurar en `database.yml`.

```ruby
development:
  adapter: sqlite3
  database: db/development.sqlite3
  pool: 5
  timeout: 5000
```

Dado que la agrupación de conexiones se maneja dentro de Active Record de forma predeterminada, todos los servidores de aplicaciones (Thin, Puma, Unicorn, etc.) deberían comportarse de la misma manera. El grupo de conexiones de la base de datos está inicialmente vacío. A medida que aumenta la demanda de conexiones, las creará hasta que alcance el límite del grupo de conexiones.

Cualquier solicitud comprobará una conexión la primera vez que requiera acceso a la base de datos. Al final de la solicitud, volverá a verificar la conexión. Esto significa que la ranura de conexión adicional estará disponible nuevamente para la siguiente solicitud en la cola.

Si intenta utilizar más conexiones de las disponibles, Active Record bloqueará
usted y espere una conexión desde el grupo. Si no puede obtener una conexión,
Se producirá un error de tiempo de espera similar al que se indica a continuación.

```ruby
ActiveRecord::ConnectionTimeoutError - could not obtain a database connection within 5.000 seconds (waited 5.000 seconds)
```

Si obtiene el error anterior, es posible que desee aumentar el tamaño del
grupo de conexiones incrementando la opción `grupo` en` database.yml`

NOTE. Si está ejecutando en un entorno de subprocesos múltiples, podría existir la posibilidad de que varios subprocesos accedan a múltiples conexiones simultáneamente. Entonces, dependiendo de la carga de su solicitud actual, es muy posible que tenga varios subprocesos compitiendo por un número limitado de conexiones.

Custom configuration
--------------------

Puede configurar su propio código a través del objeto de configuración Rails con
configuración personalizada en el espacio de nombres `config.x` o en` config` directamente.
La diferencia clave entre estos dos es que debería usar `config.x` si
están definiendo la configuración _nested_ (ej: `config.x.nested.hi`), y solo
`config` para la configuración de _un nivel_ (por ejemplo,` config.hello`).

  ```ruby
  config.x.payment_processing.schedule = :daily
  config.x.payment_processing.retries  = 3
  config.super_debugger = true
  ```

Estos puntos de configuración están disponibles a través del objeto de configuración:


  ```ruby
  Rails.configuration.x.payment_processing.schedule # => :daily
  Rails.configuration.x.payment_processing.retries  # => 3
  Rails.configuration.x.payment_processing.not_set  # => nil
  Rails.configuration.super_debugger                # => true
  ```

También puede usar `Rails::Application.config_for` para cargar archivos de configuración completos:


  ```yaml
  # config/payment.yml:
  production:
    environment: production
    merchant_id: production_merchant_id
    public_key:  production_public_key
    private_key: production_private_key

  development:
    environment: sandbox
    merchant_id: development_merchant_id
    public_key:  development_public_key
    private_key: development_private_key

  # config/application.rb
  module MyApp
    class Application < Rails::Application
      config.payment = config_for(:payment)
    end
  end
  ```

  ```ruby
  Rails.configuration.payment['merchant_id'] # => production_merchant_id or development_merchant_id
  ```
`Rails::Application.config_for` supports a `shared` configuration to group common
configurations. The shared configuration will be merged into the environment
configuration.

  ```yaml
  # config/example.yml
  shared:
    foo:
      bar:
        baz: 1

  development:
    foo:
      bar:
        qux: 2
  ```


  ```ruby
  # development environment
  Rails.application.config_for(:example)[:foo][:bar] #=> { baz: 1, qux: 2 }
  ```

Search Engines Indexing
-----------------------

A veces, es posible que desee evitar que algunas páginas de su aplicación sean visibles
en sitios de búsqueda como Google, Bing, Yahoo o Duck Duck Go. Los robots que indexan
estos sitios analizarán primero el archivo `http://your-site.com/robots.txt` para
saber qué páginas está permitido indexar.

Rails crea este archivo para usted dentro de la carpeta `/public`. Por defecto, permite
motores de búsqueda para indexar todas las páginas de su aplicación. Si quieres bloquear
indexando en todas las páginas de su aplicación, use esto:

```
User-agent: *
Disallow: /
```

Para bloquear solo páginas específicas, es necesario utilizar una sintaxis más compleja. Aprender
en el [official documentation](https://www.robotstxt.org/robotstxt.html).
      
Evented File System Monitor
---------------------------

Si se carga [listen gem](https://github.com/guard/listen), Rails usa un
monitor del sistema de archivos para detectar cambios cuando `config.cache_classes` es
`false`:

```ruby
group :development do
  gem 'listen', '~> 3.2'
end
```

De lo contrario, en cada solicitud, Rails recorre el árbol de la aplicación para comprobar si
nada ha cambiado.

En Linux y macOS no se necesitan gemas adicionales, pero se requieren algunas
[for *BSD](https://github.com/guard/listen#on-bsd) y
[for Windows](https://github.com/guard/listen#on-windows).

Tenga en cuenta que [some setups are unsupported](https://github.com/guard/listen#issues--limitations).
