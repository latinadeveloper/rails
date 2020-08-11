**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Action Controller Overview
==========================

En esta guía, aprenderá cómo funcionan los controladores y cómo encajan en el ciclo de solicitud de su aplicación.

Después de leer esta guía, sabrá:

* Cómo seguir el flujo de una solicitud a través de un controlador.
* Cómo restringir los parámetros pasados ​​a su controlador.
* Cómo y por qué almacenar datos en la sesión o cookies.
* Cómo trabajar con filtros para ejecutar código durante el procesamiento de solicitudes.
* Cómo utilizar la autenticación HTTP incorporada de Action Controller.
* Cómo transmitir datos directamente al navegador del usuario.
* Cómo filtrar parámetros sensibles para que no aparezcan en el registro de la aplicación.
* Cómo lidiar con las excepciones que pueden surgir durante el procesamiento de la solicitud.

--------------------------------------------------------------------------------

What Does a Controller Do?
--------------------------

Action Controller es la C en [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller). Una vez que el enrutador ha determinado qué controlador se usara para una solicitud, el controlador es responsable de dar sentido a la solicitud y producir la salida adecuada. Afortunadamente, Action Controller hace la mayor parte del trabajo preliminar por usted y utiliza convenciones inteligentes para que esto sea lo más sencillo posible.

For most conventional [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) aplicaciones, el controlador recibirá la solicitud (esto es invisible para usted como desarrollador), buscará o guardará datos de un modelo y usará una vista para crear una salida HTML. Si su controlador necesita hacer las cosas de manera un poco diferente, eso no es un problema, esta es solo la forma más común de que funcione un controlador.

Por tanto, se puede pensar en un controlador como un intermediario entre modelos y vistas. Hace que los datos del modelo estén disponibles para la vista para que pueda mostrar esos datos al usuario y guarda o actualiza los datos del usuario en el modelo.

NOTE: Para obtener más detalles sobre el proceso de enrutamiento, consulte [Rails Routing from the Outside In](routing.html).

Controller Naming Convention
----------------------------

La convención de nomenclatura de los controladores en Rails favorece la pluralización de la última palabra en el nombre del controlador, aunque no es estrictamente necesario (por ejemplo, `ApplicationController`). Por ejemplo, `ClientsController` es preferible a `ClientController`, `SiteAdminsController` es preferible a `SiteAdminController` o `SitesAdminsController`, y así sucesivamente.

Seguir esta convención te permitirá usar los generadores de ruta predeterminados (por ejemplo, `resources`, etc.) sin necesidad de calificar cada`:route` o `:controller`, y mantendrá el uso de los ayudantes de ruta nombrados consistente en toda tu aplicación. Consulte la [Layouts & Rendering Guide](layouts_and_rendering.html) para obtener más detalles.

NOTE: La convención de nomenclatura del controlador difiere de la convención de nomenclatura de los modelos, que se espera que sean nombrados en forma singular.

Methods and Actions
-------------------

Un controlador es una clase Ruby que hereda de `ApplicationController` y tiene métodos como cualquier otra clase. Cuando su aplicación recibe una solicitud, el enrutamiento determinará qué controlador y acción ejecutar, luego Rails crea una instancia de ese controlador y ejecuta el método con el mismo nombre que la acción.

```ruby
class ClientsController < ApplicationController
  def new
  end
end
```

Como ejemplo, si un usuario va a `/clients/new` en su aplicación para agregar un nuevo cliente, Rails creará una instancia de `ClientsController` y llamará a su método `new`. Tenga en cuenta que el método vacío del ejemplo anterior funcionaría bien porque Rails representará por defecto la vista `new.html.erb` a ​​menos que la acción indique lo contrario. Al crear un nuevo `Client`, el método `new` puede hacer que un variable de instancia de `@client` sea accesible en la vista:

```ruby
def new
  @client = Client.new
end
```

[Layouts & Rendering Guide](layouts_and_rendering.html) explica esto con más detalle.

`ApplicationController` hereda de` ActionController::Base`, que define una serie de métodos útiles. Esta guía cubrirá algunos de estos, pero si tiene curiosidad por ver qué hay allí, puede verlos todos en la [API documentation](https://api.rubyonrails.org/classes/ActionController.html) o en la propia fuente.

Solo los métodos públicos se pueden llamar como acciones. Es una buena práctica reducir la visibilidad de los métodos (con "privado" o "protegido") que no pretenden ser acciones, como métodos auxiliares o filtros.

Parameters
----------

Probablemente desee acceder a los datos enviados por el usuario u otros parámetros en las acciones de su controlador. Hay dos tipos de parámetros posibles en una aplicación web. Los primeros son parámetros que se envían como parte de la URL, llamados parámetros de cadena de consulta. La cadena de consulta es todo después de "?" en la URL. El segundo tipo de parámetro generalmente se conoce como datos POST. Esta información generalmente proviene de un formulario HTML que ha sido completado por el usuario. Se denominan datos POST porque solo se pueden enviar como parte de una solicitud HTTP POST. Rails no hace ninguna distinción entre los parámetros de la cadena de consulta y los parámetros POST, y ambos están disponibles en el hash `params` de su controlador:

```ruby
class ClientsController < ApplicationController
  # This action uses query string parameters because it gets run
  # by an HTTP GET request, but this does not make any difference
  # to the way in which the parameters are accessed. The URL for
  # this action would look like this in order to list activated
  # clients: /clients?status=activated
  def index
    if params[:status] == "activated"
      @clients = Client.activated
    else
      @clients = Client.inactivated
    end
  end

  # This action uses POST parameters. They are most likely coming
  # from an HTML form which the user has submitted. The URL for
  # this RESTful request will be "/clients", and the data will be
  # sent as part of the request body.
  def create
    @client = Client.new(params[:client])
    if @client.save
      redirect_to @client
    else
      # This line overrides the default rendering behavior, which
      # would have been to render the "create" view.
      render "new"
    end
  end
end
```

### Hash and Array Parameters

El hash `params` no se limita a claves y valores unidimensionales. Puede contener matrices anidadas y hashes. Para enviar una matriz de valores, agregue un par de corchetes vacíos "[]" al nombre de la clave:

```
GET /clients?ids[]=1&ids[]=2&ids[]=3
```

NOTA: La URL real en este ejemplo se codificará como "/clients?ids%5b%5d=1&ids%5b%5d=2&ids%5b%5d=3" ya que los caracteres "[" and "]" no están permitidos en URLs. La mayoría de las veces no tiene que preocuparse por esto porque el navegador lo codificará por usted, y Rails lo decodificará automáticamente, pero si alguna vez tiene que enviar esas solicitudes al servidor manualmente, debe tener esto en cuenta. .


El valor de `params[:ids]` ahora será `["1", "2", "3"]`. Tenga en cuenta que los valores de los parámetros son siempre cadenas; Rails no intenta adivinar o lanzar el tipo.

NOTA: Los valores como `[nil]` o `[nil, nil, ...]` en `params` se reemplazan
con `[]` por razones de seguridad de forma predeterminada. Consulta la [Security Guide](security.html#unsafe-query-generation)
para más información.

Para enviar un hash, incluya el nombre de la clave entre corchetes:

```html
<form accept-charset="UTF-8" action="/clients" method="post">
  <input type="text" name="client[name]" value="Acme" />
  <input type="text" name="client[phone]" value="12345" />
  <input type="text" name="client[address][postcode]" value="12345" />
  <input type="text" name="client[address][city]" value="Carrot City" />
</form>
```


Cuando se envía este formulario, el valor de `params[:client]` será `{ "name" => "Acme", "phone" => "12345", "address" => { "postcode" => "12345", "city" => "Carrot City" } }`. Tenga en cuenta el hash anidado en `params[:client][:address]`.

El objeto `params` actúa como un Hash, pero le permite usar símbolos y cadenas indistintamente como claves.

### JSON parameters

Si está escribiendo una aplicación de servicio web, es posible que se sienta más cómodo aceptando parámetros en formato JSON. Si el encabezado "Content-Type" de su solicitud está configurado como "application/json", Rails cargará automáticamente sus parámetros en el hash `params`, al que puede acceder como lo haría normalmente.

Entonces, por ejemplo, si está enviando este contenido JSON:

```json
{ "company": { "name": "acme", "address": "123 Carrot Street" } }
```

Su controlador recibirá `params[:company]` como `{ "name" => "acme", "address" => "123 Carrot Street" }`.

Además, si ha activado `config.wrap_parameters` en su inicializador o llamado `wrap_parameters` en su controlador, puede omitir con seguridad el elemento raíz en el parámetro JSON. En este caso, los parámetros se clonarán y se ajustarán con una clave elegida en función del nombre de su controlador. Entonces, la solicitud JSON anterior se puede escribir como:

```json
{ "name": "acme", "address": "123 Carrot Street" }
```

Y, asumiendo que está enviando los datos a `CompaniesController`, entonces se incluirán dentro de la clave `:company` así:

Puede personalizar el nombre de la clave o los parámetros específicos que desea ajustar consultando la [API documentation](https://api.rubyonrails.org/classes/ActionController/ParamsWrapper.html)

NOTE: El soporte para analizar parámetros XML se ha extraído en una gema llamada `actionpack-xml_parser`.

### Routing Parameters

El hash `params` siempre contendrá las claves `:controller` y `:action`, pero debes usar los métodos `controller_name` y `action_name` en su lugar para acceder a estos valores. Cualquier otro parámetro definido por el enrutamiento, como `:id`, también estará disponible. Como ejemplo, considere una lista de clientes donde la lista puede mostrar clientes activos o inactivos. Podemos agregar una ruta que capture el parámetro `:status` en una URL "pretty":

```ruby
get '/clients/:status', to: 'clients#index', foo: 'bar'
```

En este caso, cuando un usuario abre la URL `/clients/active`, `params[:status] `se configurará como" activo ". Cuando se utiliza esta ruta, `params[:foo]` también se establecerá en "bar", como si se pasara en la cadena de consulta. Su controlador también recibirá `params[:action]` como "índice" y `params[:controller]` como "clients".

### `default_url_options`

Puede establecer parámetros predeterminados globales para la generación de URL definiendo un método llamado `default_url_options` en su controlador. Dicho método debe devolver un hash con los valores predeterminados deseados, cuyas claves deben ser símbolos:

```ruby
class ApplicationController < ActionController::Base
  def default_url_options
    { locale: I18n.locale }
  end
end
```

Estas opciones se usarán como punto de partida al generar URL, por lo que es posible que sean anuladas por las opciones pasadas a las llamadas `url_for`.

Si define `default_url_options` en` ApplicationController`, como en el ejemplo anterior, estos valores predeterminados se utilizarán para toda la generación de URL. El método también se puede definir en un controlador específico, en cuyo caso solo afecta a las URL generadas allí.

En una solicitud determinada, el método no se llama realmente para cada URL generada; por razones de rendimiento, el hash devuelto se almacena en caché, hay como máximo una invocación por solicitud.

### Strong Parameters

Con parámetros fuertes, los parámetros del controlador de acción están prohibidos
utilizarse en asignaciones masivas de modelos activos hasta que hayan sido
permitido. Esto significa que tendrá que tomar una decisión consciente sobre
qué atributos permitir para la actualización masiva. Esta es una mejor seguridad
práctica para ayudar a evitar permitir accidentalmente a los usuarios actualizar información
atributos del modelo.

Además, los parámetros se pueden marcar según sea necesario y fluirán a través de un
flujo de subida/rescate predefinido que dará como resultado una solicitud incorrecta devuelve 400
si no se pasan todos los parámetros requeridos.

```ruby
class PeopleController < ActionController::Base
  # This will raise an ActiveModel::ForbiddenAttributesError exception
  # because it's using mass assignment without an explicit permit
  # step.
  def create
    Person.create(params[:person])
  end

  # This will pass with flying colors as long as there's a person key
  # in the parameters, otherwise it'll raise an
  # ActionController::ParameterMissing exception, which will get
  # caught by ActionController::Base and turned into a 400 Bad
  # Request error.
  def update
    person = current_account.people.find(params[:id])
    person.update!(person_params)
    redirect_to person
  end

  private
    # Using a private method to encapsulate the permissible parameters
    # is just a good pattern since you'll be able to reuse the same
    # permit list between create and update. Also, you can specialize
    # this method with per-user checking of permissible attributes.
    def person_params
      params.require(:person).permit(:name, :age)
    end
end
```

#### Permitted Scalar Values

Dado

```ruby
params.permit(:id)
```

se permitirá la inclusión de la clave `:id` si aparece en `params` y
tiene asociado un valor escalar permitido. De lo contrario, la clave va
filtrarse, por lo que matrices, hashes o cualquier otro objeto no se pueden
inyectado.

Los tipos escalares permitidos son `String`, `Symbol`, `NilClass`,
`Numeric`, `TrueClass`, `FalseClass`, `Date`, `Time`, `DateTime`,
`StringIO`, `IO`, `ActionDispatch::Http::UploadedFile`, y
`Rack::Test::UploadedFile`.

Para declarar que el valor en `params` debe ser una matriz de permitidos
valores escalares, asigne la clave a una matriz vacía:

```ruby
params.permit(id: [])
```

A veces no es posible o conveniente declarar las claves válidas de
un parámetro hash o su estructura interna. Simplemente mapee a un hash vacío:

```ruby
params.permit(preferences: {})
```

pero tenga cuidado porque esto abre la puerta a entradas arbitrarias. En esto
caso, `permit` asegura que los valores en la estructura devuelta estén permitidos
escalares y filtra cualquier otra cosa.

Para permitir un hash completo de parámetros, el método `permit!` puede ser
usado:

```ruby
params.require(:log_entry).permit!
```

Esto marca el hash de los parámetros `:log_entry` y cualquier sub-hash del mismo como
permitido y no comprueba los escalares permitidos, se acepta cualquier cosa.
Se debe tener mucho cuidado al usar `permit!`, Ya que permitirá que todas las
y los atributos del modelo futuro que se asignarán en masa.

#### Nested Parameters

También puede usar `permit` en parámetros anidados, como:

```ruby
params.permit(:name, { emails: [] },
              friends: [ :name,
                         { family: [ :name ], hobbies: [] }])
```

Esta declaración permite los atributos `name`,`emails` y `friends`.
Se espera que los `emails` sean una variedad de
valores escalares, y que `friends` será un conjunto de recursos con
atributos específicos: deben tener un atributo `name` (cualquier valores escalares permitidos), un atributo `hobbies` como una matriz de
valores escalares permitidos, y un atributo `family` que está restringido
a tener un `name` (cualquier valor escalar también esta permitido aquí ).

#### More Examples

Es posible que desee utilizar también los atributos permitidos en su "nuevo"
acción. Esto plantea el problema de que no puede utilizar "require" en el
clave raíz porque, normalmente, no existe cuando se llama a `new`:

```ruby
# using `fetch` you can supply a default and use
# the Strong Parameters API from there.
params.fetch(:blog, {}).permit(:title, :author)
```

El método de clase de modelo `accept_nested_attributes_for` le permite
actualizar y destruir registros asociados. Esto se basa en el `id` y ` _destroy`
parámetros:

```ruby
# permit :id and :_destroy
params.require(:author).permit(:name, books_attributes: [:title, :id, :_destroy])
```

Los hash con claves enteras se tratan de forma diferente y puedes declarar
los atributos como si fueran hijos directos. Obtienes este tipo de
parámetros cuando se usa `accept_nested_attributes_for` en combinación
con una asociación `has_many`:

```ruby
# To permit the following data:
# {"book" => {"title" => "Some Book",
#             "chapters_attributes" => { "1" => {"title" => "First Chapter"},
#                                        "2" => {"title" => "Second Chapter"}}}}

params.require(:book).permit(:title, chapters_attributes: [:title])
```

Imagina un escenario en el que tienes parámetros que representan un producto
nombre y un hash de datos arbitrarios asociados con ese producto, y
desea permitir el atributo de nombre del producto y también el
hash de datos:

```ruby
def product_params
  params.require(:product).permit(:name, data: {})
end
```

#### Outside the Scope of Strong Parameters

La API de parámetros sólidos se diseñó con los casos de uso más comunes. 
No pretende ser una solución milagrosa para manejar todos sus
problemas de parámetros filtrados. Sin embargo, puede mezclar fácilmente la API con su
propio código para adaptarse a su situación.

Session
-------

Su aplicación tiene una sesión para cada usuario en la que puede almacenar pequeñas cantidades de datos que se conservarán entre las solicitudes. La sesión solo está disponible en el controlador y la vista y puede utilizar uno de varios mecanismos de almacenamiento diferentes:

* `ActionDispatch::Session::CookieStore` - Almacena todo en el cliente.
* `ActionDispatch::Session::CacheStore` - Almacena los datos en la caché de Rails.
* `ActionDispatch::Session::ActiveRecordStore` - Almacena los datos en una base de datos usando Active Record. (requiere la gem `activerecord-session_store`).
* `ActionDispatch::Session::MemCacheStore`: Almacena los datos en un clúster Memcached (esta es una implementación heredada; considere usar CacheStore en su lugar).

Todas las tiendas de sesiones usan una cookie para almacenar una ID única para cada sesión (debe usar una cookie, Rails no le permitirá pasar la ID de la sesión en la URL ya que esto es menos seguro).

Para la mayoría de las tiendas, esta ID se usa para buscar los datos de la sesión en el servidor, p. Ej. en una tabla de base de datos. Hay una excepción, y ese es el almacén de sesiones predeterminado y recomendado, el CookieStore, que almacena todos los datos de la sesión en la propia cookie (el ID todavía está disponible para usted si lo necesita). Esto tiene la ventaja de ser muy ligero y no requiere ninguna configuración en una nueva aplicación para poder usar la sesión. Los datos de las cookies están firmados criptográficamente para que sean a prueba de manipulaciones. Y también está encriptado para que cualquier persona con acceso no pueda leer su contenido. (Rails no lo aceptará si ha sido editado).

CookieStore puede almacenar alrededor de 4kB de datos, mucho menos que los demás, pero esto suele ser suficiente. No se recomienda almacenar grandes cantidades de datos en la sesión, independientemente del almacén de sesiones que utilice su aplicación. En especial, debe evitar almacenar objetos complejos (cualquier cosa que no sean objetos básicos de Ruby, el ejemplo más común son las instancias de modelo) en la sesión, ya que es posible que el servidor no pueda volver a ensamblarlos entre solicitudes, lo que resultará en un error.

Si sus sesiones de usuario no almacenan datos críticos o no necesitan estar disponibles por períodos prolongados (por ejemplo, si solo usa el flash para mensajería), puede considerar usar `ActionDispatch::Session::CacheStore`. Esto almacenará sesiones usando la implementación de caché que ha configurado para su aplicación. La ventaja de esto es que puede usar su infraestructura de caché existente para almacenar sesiones sin requerir ninguna configuración o administración adicional. La desventaja, por supuesto, es que las sesiones serán efímeras y podrían desaparecer en cualquier momento.

Obtenga más información sobre el almacenamiento de sesiones en el [Security Guide](security.html).

Si necesita un mecanismo de almacenamiento de sesión diferente, puede cambiarlo en un inicializador:

```ruby
# Use the database for sessions instead of the cookie-based default,
# which shouldn't be used to store highly confidential information
# (create the session table with "rails g active_record:session_migration")
# Rails.application.config.session_store :active_record_store
```

Rails configura una clave de sesión (el nombre de la cookie) al firmar los datos de la sesión. Estos también se pueden cambiar en un inicializador:

```ruby
# Be sure to restart your server when you modify this file.
Rails.application.config.session_store :cookie_store, key: '_your_app_session'
```

También puede pasar una clave `:domain` y especificar el nombre de dominio para la cookie:

```ruby
# Be sure to restart your server when you modify this file.
Rails.application.config.session_store :cookie_store, key: '_your_app_session', domain: ".example.com"
```

Rails configura (para CookieStore) una clave secreta que se utiliza para firmar los datos de la sesión en `config/credentials.yml.enc`. Esto se puede cambiar con `bin/rails credentials:edit`.

```yaml
# aws:
#   access_key_id: 123
#   secret_access_key: 345

# Used as the base secret for all MessageVerifiers in Rails, including the one protecting cookies.
secret_key_base: 492f...
```

NOTE: Cambiar la base_clave_secreta cuando se usa `CookieStore` invalidará todas las sesiones existentes.

### Accessing the Session

En su controlador puede acceder a la sesión a través del método de instancia `session`.

NOTE: Las sesiones se cargan de forma diferida. Si no accede a las sesiones en el código de su acción, no se cargarán. Por lo tanto, nunca necesitará deshabilitar las sesiones, simplemente no acceder a ellas hará el trabajo.

Los valores de sesión se almacenan utilizando pares clave/valor como un hash:

```ruby
class ApplicationController < ActionController::Base

  private

  # Finds the User with the ID stored in the session with the key
  # :current_user_id This is a common way to handle user login in
  # a Rails application; logging in sets the session value and
  # logging out removes it.
  def current_user
    @_current_user ||= session[:current_user_id] &&
      User.find_by(id: session[:current_user_id])
  end
end
```

Para almacenar algo en la sesión, simplemente asígnelo a la clave como un hash:

```ruby
class LoginsController < ApplicationController
  # "Create" a login, aka "log the user in"
  def create
    if user = User.authenticate(params[:username], params[:password])
      # Save the user ID in the session so it can be used in
      # subsequent requests
      session[:current_user_id] = user.id
      redirect_to root_url
    end
  end
end
```

Para eliminar algo de la sesión, elimine el par clave/valor:

```ruby
class LoginsController < ApplicationController
  # "Delete" a login, aka "log the user out"
  def destroy
    # Remove the user id from the session
    session.delete(:current_user_id)
    # Clear the memoized current user
    @_current_user = nil
    redirect_to root_url
  end
end
```

Para restablecer toda la sesión, use `reset_session`.

### The Flash

El flash es una parte especial de la sesión que se borra con cada solicitud. Esto significa que los valores almacenados allí solo estarán disponibles en la próxima solicitud, lo que es útil para pasar mensajes de error, etc.

Se accede a ella de la misma forma que a la sesión, como un hash (es una instancia de [FlashHash](https://api.rubyonrails.org/classes/ActionDispatch/Flash/FlashHash.html)).

Usemos el acto de cerrar la sesión como ejemplo. El controlador puede enviar un mensaje que se mostrará al usuario en la siguiente solicitud:

```ruby
class LoginsController < ApplicationController
  def destroy
    session.delete(:current_user_id)
    flash[:notice] = "You have successfully logged out."
    redirect_to root_url
  end
end
```

Tenga en cuenta que también es posible asignar un mensaje flash como parte de la redirección. Puede asignar `:notice`,`:alert` o el propósito general `:flash`:

```ruby
redirect_to root_url, notice: "You have successfully logged out."
redirect_to root_url, alert: "You're stuck here!"
redirect_to root_url, flash: { referral_code: 1234 }
```

La acción `destroy` redirige a la aplicación `root_url`, donde se mostrará el mensaje. Tenga en cuenta que depende totalmente de la siguiente acción decidir qué hará, si es que hará algo, con lo que la acción anterior puso en el flash. Es convencional mostrar cualquier alerta de error o avisos del flash en el diseño de la aplicación:

```erb
<html>
  <!-- <head/> -->
  <body>
    <% flash.each do |name, msg| -%>
      <%= content_tag :div, msg, class: name %>
    <% end -%>

    <!-- more content -->
  </body>
</html>
```

De esta manera, si una acción establece un aviso o un mensaje de alerta, el diseño lo mostrará automáticamente.

Puede pasar cualquier cosa que la sesión pueda almacenar; no está limitado a avisos y alertas:

```erb
<% if flash[:just_signed_up] %>
  <p class="welcome">Welcome to our site!</p>
<% end %>
```

Si desea que un valor flash se transfiera a otra solicitud, use el método `keep`:

```ruby
class MainController < ApplicationController
  # Let's say this action corresponds to root_url, but you want
  # all requests here to be redirected to UsersController#index.
  # If an action sets the flash and redirects here, the values
  # would normally be lost when another redirect happens, but you
  # can use 'keep' to make it persist for another request.
  def index
    # Will persist all flash values.
    flash.keep

    # You can also use a key to keep only some kind of value.
    # flash.keep(:notice)
    redirect_to users_url
  end
end
```

#### `flash.now`

De forma predeterminada, agregar valores a la memoria flash los hará disponibles para la próxima solicitud, pero a veces es posible que desee acceder a esos valores en la misma solicitud. Por ejemplo, si la acción `create` falla al guardar un recurso y renderiza la plantilla `new` directamente, eso no dará como resultado una nueva solicitud, pero es posible que aún desee mostrar un mensaje usando el flash. Para hacer esto, puede usar `flash.now` de la misma manera que usa el `flash` normal:

```ruby
class ClientsController < ApplicationController
  def create
    @client = Client.new(client_params)
    if @client.save
      # ...
    else
      flash.now[:error] = "Could not save client"
      render action: "new"
    end
  end
end
```

Cookies
-------

Su aplicación puede almacenar pequeñas cantidades de datos en el cliente, llamadas cookies, que se mantendrán en las solicitudes e incluso en las sesiones. Rails proporciona un fácil acceso a las cookies a través del método `cookies`, que, al igual que la `sesión`, funciona como un hash:

```ruby
class CommentsController < ApplicationController
  def new
    # Auto-fill the commenter's name if it has been stored in a cookie
    @comment = Comment.new(author: cookies[:commenter_name])
  end

  def create
    @comment = Comment.new(comment_params)
    if @comment.save
      flash[:notice] = "Thanks for your comment!"
      if params[:remember_name]
        # Remember the commenter's name.
        cookies[:commenter_name] = @comment.author
      else
        # Delete cookie for the commenter's name cookie, if any.
        cookies.delete(:commenter_name)
      end
      redirect_to @comment.article
    else
      render action: "new"
    end
  end
end
```

Tenga en cuenta que si bien para los valores de sesión establece la clave en `nil`, para eliminar un valor de cookie debe usar `cookies.delete(:key)`.

Rails también proporciona un tarro de galletas firmado y un tarro de galletas encriptado para almacenar
informacion delicada. El tarro de galletas firmado agrega una firma criptográfica en el
valores de cookies para proteger su integridad. El tarro de galletas cifrado cifra el
valores además de firmarlos, para que el usuario final no pueda leerlos.
Referirse a [API documentation](https://api.rubyonrails.org/classes/ActionDispatch/Cookies.html)
for more details.

Estos tarros de cookies especiales utilizan un serializador para serializar los valores asignados en
strings y deserializa en objetos Ruby en read.

Puede especificar qué serializador usar:

```ruby
Rails.application.config.action_dispatch.cookies_serializer = :json
```

El serializador predeterminado para nuevas aplicaciones es `:json`. Por compatibilidad con
aplicaciones antiguas con cookies existentes, `:marshal` se usa cuando `serializer`
no se especifica la opción.

También puede establecer esta opción en `:hybrid`, en cuyo caso Rails lo haría de forma transparente
deserializar las cookies existentes (serializadas por `Marshal`) al leerlas y reescribirlas en
el formato `JSON`. Esto es útil para migrar aplicaciones existentes a la
Serializador `:json`.

También es posible pasar un serializador personalizado que responda a `load` y
`dump`:

```ruby
Rails.application.config.action_dispatch.cookies_serializer = MyCustomSerializer
```

Cuando utilice el serializador `:json` o `:hybrid`, debe tener en cuenta que no todos
Los objetos Ruby se pueden serializar como JSON. Por ejemplo, objetos `Date` y` Time`
se serializarán como cadenas, y los `Hash`es tendrán sus claves en cadena.

```ruby
class CookiesController < ApplicationController
  def set_cookie
    cookies.encrypted[:expiration_date] = Date.tomorrow # => Thu, 20 Mar 2014
    redirect_to action: 'read_cookie'
  end

  def read_cookie
    cookies.encrypted[:expiration_date] # => "2014-03-20"
  end
end
```

Es aconsejable que solo almacene datos simples (cadenas y números) en las cookies.
Si tiene que almacenar objetos complejos, necesitará manejar la conversión
manualmente al leer los valores en solicitudes posteriores.

Si usa la tienda de sesión de cookies, esto se aplicaría a la `session` y
hash `flash` también.

Rendering XML and JSON data
---------------------------

ActionController hace que sea extremadamente fácil renderizar datos `XML` o `JSON`. Si ha generado un controlador usando scaffolding, se vería así:

```ruby
class UsersController < ApplicationController
  def index
    @users = User.all
    respond_to do |format|
      format.html # index.html.erb
      format.xml  { render xml: @users }
      format.json { render json: @users }
    end
  end
end
```

Puede notar en el código anterior que estamos usando `render xml: @ users`, no `render xml: @users.to_xml`. Si el objeto no es una cadena, Rails invocará automáticamente `to_xml` por nosotros.

Filters
-------

Los filtros son métodos que se ejecutan "before", "after" o "around" de una acción del controlador.

Los filtros se heredan, por lo que si establece un filtro en `ApplicationController`, se ejecutará en todos los controladores de su aplicación.

Los filtros "antes" pueden detener el ciclo de solicitud. Un filtro "before" común es aquel que requiere que un usuario inicie sesión para que se ejecute una acción. Puede definir el método de filtro de esta manera:

```ruby
class ApplicationController < ActionController::Base
  before_action :require_login

  private

  def require_login
    unless logged_in?
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url # halts request cycle
    end
  end
end
```

El método simplemente almacena un mensaje de error en la memoria flash y redirige al formulario de inicio de sesión si el usuario no ha iniciado sesión. Si un filtro "antes" se procesa o redirecciona, la acción no se ejecutará. Si hay filtros adicionales programados para ejecutarse después de ese filtro, también se cancelan.

```ruby
class LoginsController < ApplicationController
  skip_before_action :require_login, only: [:new, :create]
end
```

Ahora, las acciones `new` y `create` de `LoginsController` funcionarán como antes sin requerir que el usuario inicie sesión. La opción`: only` se usa para omitir este filtro solo para estas acciones, y también hay una opción `: except` que funciona al revés. Estas opciones también se pueden usar al agregar filtros, por lo que puede agregar un filtro que solo se ejecuta para las acciones seleccionadas en primer lugar.

NOTE: Llamar al mismo filtro varias veces con diferentes opciones no funcionará,
ya que la última definición de filtro sobrescribirá las anteriores.

### After Filters and Around Filters

Además de los filtros "before", también puede ejecutar filtros después de que se haya ejecutado una acción, o tanto antes como después.

Los filtros "after" son similares a los filtros "before", pero debido a que la acción ya se ha ejecutado, tienen acceso a los datos de respuesta que están a punto de enviarse al cliente. Obviamente, los filtros "después" no pueden detener la ejecución de la acción. Tenga en cuenta que los filtros "posteriores" se ejecutan solo después de una acción exitosa, pero no cuando se genera una excepción en el ciclo de solicitud.

Los filtros "around" son responsables de ejecutar sus acciones asociadas al ceder, de forma similar a como funcionan los middlewares de Rack.

Por ejemplo, en un sitio web donde los cambios tienen un flujo de trabajo de aprobación, un administrador podría obtener una vista previa de ellos fácilmente, simplemente aplíquelos dentro de una transacción:

```ruby
class ChangesController < ApplicationController
  around_action :wrap_in_transaction, only: :show

  private

  def wrap_in_transaction
    ActiveRecord::Base.transaction do
      begin
        yield
      ensure
        raise ActiveRecord::Rollback
      end
    end
  end
end
```

Tenga en cuenta que un filtro "around" también envuelve el renderizado. En particular, si en el ejemplo anterior, la propia vista lee de la base de datos (por ejemplo, a través de un alcance), lo hará dentro de la transacción y, por lo tanto, presentará los datos para obtener una vista previa.

Puede optar por no ceder y generar la respuesta usted mismo, en cuyo caso la acción no se ejecutará.

### Other Ways to Use Filters

Si bien la forma más común de usar filtros es creando métodos privados y usando *_action para agregarlos, hay otras dos formas de hacer lo mismo.

La primera es usar un bloque directamente con los métodos *\ _action . El bloque recibe el controlador como argumento. El filtro `require_login` de arriba podría reescribirse para usar un bloque:


```ruby
class ApplicationController < ActionController::Base
  before_action do |controller|
    unless controller.send(:logged_in?)
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url
    end
  end
end
```

Tenga en cuenta que el filtro en este caso usa `send` porque el método `log_in? `Es privado y el filtro no se ejecuta en el alcance del controlador. Esta no es la forma recomendada de implementar este filtro en particular, pero en casos más simples podría ser útil.

Específicamente para `around_action`, el bloque también cede en la `action`:

```ruby
around_action { |_controller, action| time(&action) }
```

La segunda forma es usar una clase (en realidad, cualquier objeto que responda a los métodos correctos servirá) para manejar el filtrado. Esto es útil en casos que son más complejos y no se pueden implementar de forma legible y reutilizable utilizando los otros dos métodos. Como ejemplo, puede volver a escribir el filtro de inicio de sesión para usar una clase:

```ruby
class ApplicationController < ActionController::Base
  before_action LoginFilter
end

class LoginFilter
  def self.before(controller)
    unless controller.send(:logged_in?)
      controller.flash[:error] = "You must be logged in to access this section"
      controller.redirect_to controller.new_login_url
    end
  end
end
```

Nuevamente, este no es un ejemplo ideal para este filtro, porque no se ejecuta en el alcance del controlador, pero hace que el controlador se pase como argumento. La clase de filtro debe implementar un método con el mismo nombre que el filtro, por lo que para el filtro `before_action` la clase debe implementar un método `before`, y así sucesivamente. El método `around` debe `yield` para ejecutar la acción.

Request Forgery Protection
--------------------------

La falsificación de solicitudes entre sitios es un tipo de ataque en el que un sitio engaña a un usuario para que realice solicitudes en otro sitio, posiblemente agregando, modificando o eliminando datos en ese sitio sin el conocimiento o permiso del usuario.

El primer paso para evitar esto es asegurarse de que solo se pueda acceder a todas las acciones "destructivas" (crear, actualizar y destruir) con solicitudes que no sean GET. Si sigue las convenciones RESTful, ya lo está haciendo. Sin embargo, un sitio malicioso aún puede enviar una solicitud que no sea GET a su sitio con bastante facilidad, y ahí es donde entra la protección contra la falsificación de solicitudes. Como su nombre lo indica, protege de las solicitudes falsificadas.

La forma en que se hace esto es agregar un token no adivinable que solo su servidor conoce en cada solicitud. De esta forma, si llega una solicitud sin el token adecuado, se le negará el acceso.

Si genera un formulario como este:

```erb
<%= form_with model: @user, local: true do |form| %>
  <%= form.text_field :username %>
  <%= form.text_field :password %>
<% end %>
```

Verá cómo se agrega el token como un campo oculto:

```html
<form accept-charset="UTF-8" action="/users/1" method="post">
<input type="hidden"
       value="67250ab105eb5ad10851c00a5621854a23af5489"
       name="authenticity_token"/>
<!-- fields -->
</form>
```

Rails agrega este token a cada formulario que se genera usando los [form helpers](form_helpers.html), por lo que la mayoría de las veces no tiene que preocuparse por eso. Si está escribiendo un formulario manualmente o necesita agregar el token por otro motivo, está disponible a través del método `form_authenticity_token`:

El `form_authenticity_token` genera un token de autenticación válido. Eso es útil en lugares donde Rails no lo agrega automáticamente, como en las llamadas Ajax personalizadas.

La [Security Guide](security.html) tiene más información sobre este y muchos otros problemas relacionados con la seguridad que debe tener en cuenta al desarrollar una aplicación web.

The Request and Response Objects
--------------------------------

En cada controlador hay dos métodos de acceso que apuntan a la solicitud y los objetos de respuesta asociados con el ciclo de solicitud que se encuentra actualmente en ejecución. El método `request` contiene una instancia de` ActionDispatch :: Request` y el método `response` devuelve un objeto de respuesta que representa lo que se enviará de vuelta al cliente.

### The `request` Object

El objeto de solicitud contiene mucha información útil sobre la solicitud que proviene del cliente. Para obtener una lista completa de los métodos disponibles, consulte la [documentación de la API de Rails] (https://api.rubyonrails.org/classes/ActionDispatch/Request.html) y la [Documentación de rack] (https: //www.rubydoc .info / github / rack / rack / Rack / Request). Entre las propiedades a las que puede acceder en este objeto se encuentran:

| Property of `request`                     | Purpose                                                                          |
| ----------------------------------------- | -------------------------------------------------------------------------------- |
| host                                      | The hostname used for this request.                                              |
| domain(n=2)                               | The hostname's first `n` segments, starting from the right (the TLD).            |
| format                                    | The content type requested by the client.                                        |
| method                                    | The HTTP method used for the request.                                            |
| get?, post?, patch?, put?, delete?, head? | Returns true if the HTTP method is GET/POST/PATCH/PUT/DELETE/HEAD.               |
| headers                                   | Returns a hash containing the headers associated with the request.               |
| port                                      | The port number (integer) used for the request.                                  |
| protocol                                  | Returns a string containing the protocol used plus "://", for example "http://". |
| query_string                              | The query string part of the URL, i.e., everything after "?".                    |
| remote_ip                                 | The IP address of the client.                                                    |
| url                                       | The entire URL used for the request.                                             |

#### `path_parameters`, `query_parameters`, and `request_parameters`

Rails recopila todos los parámetros enviados junto con la solicitud en el hash `params`, ya sea que se envíen como parte de la cadena de consulta o el cuerpo de la publicación. El objeto de solicitud tiene tres descriptores de acceso que le dan acceso a estos parámetros según su procedencia. El hash `query_parameters` contiene parámetros que se enviaron como parte de la cadena de consulta, mientras que el hash `request_parameters` contiene parámetros enviados como parte del cuerpo de la publicación. El hash `path_parameters` contiene parámetros que fueron reconocidos por el enrutamiento como parte de la ruta que conduce a este controlador y acción en particular.

### The `response` Object

Por lo general, el objeto de respuesta no se usa directamente, sino que se crea durante la ejecución de la acción y la presentación de los datos que se envían de vuelta al usuario, pero a veces, como en un filtro posterior, puede ser útil acceder a la respuesta. directamente. Algunos de estos métodos de acceso también tienen definidores, lo que le permite cambiar sus valores. Para obtener una lista completa de los métodos disponibles, consulte la [Rails API documentation](https://api.rubyonrails.org/classes/ActionDispatch/Response.html) y [Rack Documentation](https://www.rubydoc.info/github/rack/rack/Rack/Response).

| Property of `response` | Purpose                                                                                             |
| ---------------------- | --------------------------------------------------------------------------------------------------- |
| body                   | This is the string of data being sent back to the client. This is most often HTML.                  |
| status                 | The HTTP status code for the response, like 200 for a successful request or 404 for file not found. |
| location               | The URL the client is being redirected to, if any.                                                  |
| content_type           | The content type of the response.                                                                   |
| charset                | The character set being used for the response. Default is "utf-8".                                  |
| headers                | Headers used for the response.                                                                      |

#### Setting Custom Headers

Si desea configurar encabezados personalizados para una respuesta, entonces `response.headers` es el lugar para hacerlo. El atributo de encabezados es un hash que asigna los nombres de los encabezados a sus valores, y Rails establecerá algunos de ellos automáticamente. Si desea agregar o cambiar un encabezado, simplemente asígnelo a `response.headers` de esta manera:

```ruby
response.headers["Content-Type"] = "application/pdf"
```

NOTE: En el caso anterior, tendría más sentido usar el configurador `content_type` directamente.

HTTP Authentications
--------------------

Rails viene con tres mecanismos de autenticación HTTP integrados:

* Autenticación básica
* Autenticación implícita
* Autenticación de token

### HTTP Basic Authentication

La autenticación básica HTTP es un esquema de autenticación que es compatible con la mayoría de los navegadores y otros clientes HTTP. Como ejemplo, considere una sección de administración que solo estará disponible ingresando un nombre de usuario y una contraseña en la ventana de diálogo básico HTTP del navegador. Usar la autenticación incorporada es bastante fácil y solo requiere que use un método, `http_basic_authenticate_with`.

```ruby
class AdminsController < ApplicationController
  http_basic_authenticate_with name: "humbaba", password: "5baa61e4"
end
```

Con esto en su lugar, puede crear controladores de espacio de nombres que hereden de `AdminsController`. De esta forma, el filtro se ejecutará para todas las acciones en esos controladores, protegiéndolos con autenticación básica HTTP.

### HTTP Digest Authentication

La autenticación implícita HTTP es superior a la autenticación básica, ya que no requiere que el cliente envíe una contraseña sin cifrar a través de la red (aunque la autenticación básica HTTP es segura a través de HTTPS). Usar la autenticación implícita con Rails es bastante fácil y solo requiere usar un método, `authenticate_or_request_with_http_digest`.

```ruby
class AdminsController < ApplicationController
  USERS = { "lifo" => "world" }

  before_action :authenticate

  private
    def authenticate
      authenticate_or_request_with_http_digest do |username|
        USERS[username]
      end
    end
end
```

Como se ve en el ejemplo anterior, el bloque `authenticate_or_request_with_http_digest` toma solo un argumento: el nombre de usuario. Y el bloque devuelve la contraseña. Devolver `false` o `nil` de `authenticate_or_request_with_http_digest` provocará un error de autenticación.

### HTTP Token Authentication

La autenticación de token HTTP es un esquema para permitir el uso de tokens de portador en el encabezado HTTP  `Authorization`. Hay muchos formatos de token disponibles y describirlos está fuera del alcance de este documento.

Como ejemplo, suponga que desea utilizar un token de autenticación que se ha emitido con anterioridad para realizar la autenticación y el acceso. Implementar la autenticación de token con Rails es bastante fácil y solo requiere el uso de un método, `authenticate_or_request_with_http_token`.

```ruby
class PostsController < ApplicationController
  TOKEN = "secret"

  before_action :authenticate

  private
    def authenticate
      authenticate_or_request_with_http_token do |token, options|
        ActiveSupport::SecurityUtils.secure_compare(token, TOKEN)
      end
    end
end
```

Como se ve en el ejemplo anterior, el bloque `authenticate_or_request_with_http_token` toma dos argumentos: el token y un` Hash` que contiene las opciones que se analizaron del encabezado HTTP `Authorization`. El bloque debe devolver `true` si la autenticación es exitosa. Si devuelve `false` o `nil`, se producirá un error de autenticación.

Streaming and File Downloads
----------------------------

A veces, es posible que desee enviar un archivo al usuario en lugar de representar una página HTML. Todos los controladores en Rails tienen los métodos `send_data` y `send_file`, que transmitirán datos al cliente. `send_file` es un método conveniente que le permite proporcionar el nombre de un archivo en el disco y transmitirá el contenido de ese archivo por usted.

```ruby
require "prawn"
class ClientsController < ApplicationController
  # Generates a PDF document with information on the client and
  # returns it. The user will get the PDF as a file download.
  def download_pdf
    client = Client.find(params[:id])
    send_data generate_pdf(client),
              filename: "#{client.name}.pdf",
              type: "application/pdf"
  end

  private
    def generate_pdf(client)
      Prawn::Document.new do
        text client.name, align: :center
        text "Address: #{client.address}"
        text "Email: #{client.email}"
      end.render
    end
end
```

La acción `download_pdf` en el ejemplo anterior llamará a un método privado que en realidad genera el documento PDF y lo devuelve como una cadena. Esta cadena luego se transmitirá al cliente como una descarga de archivo y se le sugerirá un nombre de archivo al usuario. A veces, al transmitir archivos al usuario, es posible que no desee que descarguen el archivo. Tome imágenes, por ejemplo, que se pueden incrustar en páginas HTML. Para decirle al navegador que un archivo no está destinado a ser descargado, puede establecer la opción `:disposition` en" en línea ". El valor opuesto y predeterminado para esta opción es "adjunto".

### Sending Files

Si desea enviar un archivo que ya existe en el disco, use el método `send_file`.

```ruby
class ClientsController < ApplicationController
  # Stream a file that has already been generated and stored on disk.
  def download_pdf
    client = Client.find(params[:id])
    send_file("#{Rails.root}/files/clients/#{client.id}.pdf",
              filename: "#{client.name}.pdf",
              type: "application/pdf")
  end
end
```

Esto leerá y transmitirá el archivo 4kB en ese momento, evitando cargar el archivo completo en la memoria a la vez. Puede desactivar la transmisión con la opción `:stream` o ajustar el tamaño del bloque con la opción `:buffer_size`.

Si no se especifica `:type`, se adivinará a partir de la extensión de archivo especificada en `:filename`. Si el tipo de contenido no está registrado para la extensión, se utilizará `application/octet-stream`.

WARNING: Tenga cuidado cuando utilice datos provenientes del cliente (parámetros, cookies, etc.) para ubicar el archivo en el disco, ya que este es un riesgo de seguridad que podría permitir que alguien obtenga acceso a archivos para los que no está destinado.

TIP: No se recomienda que transmita archivos estáticos a través de Rails si, en cambio, puede mantenerlos en una carpeta pública en su servidor web. Es mucho más eficiente permitir que el usuario descargue el archivo directamente usando Apache u otro servidor web, evitando que la solicitud pase innecesariamente por toda la pila de Rails.

### RESTful Downloads

Si bien `send_data` funciona bien, si está creando una aplicación RESTful, generalmente no es necesario tener acciones separadas para la descarga de archivos. En terminología REST, el archivo PDF del ejemplo anterior puede considerarse simplemente otra representación del recurso del cliente. Rails proporciona una forma sencilla y bastante elegante de realizar "RESTful downloads". A continuación, le indicamos cómo puede reescribir el ejemplo para que la descarga del PDF sea parte de la acción `show`, sin transmisión:

```ruby
class ClientsController < ApplicationController
  # The user can request to receive this resource as HTML or PDF.
  def show
    @client = Client.find(params[:id])

    respond_to do |format|
      format.html
      format.pdf { render pdf: generate_pdf(@client) }
    end
  end
end
```

Para que este ejemplo funcione, debe agregar el tipo PDF MIME a Rails. Esto se puede hacer agregando la siguiente línea al archivo `config/initializers/mime_types.rb`:

```ruby
Mime::Type.register "application/pdf", :pdf
```

NOTE: Los archivos de configuración no se vuelven a cargar en cada solicitud, por lo que debe reiniciar el servidor para que los cambios surtan efecto.

Ahora el usuario puede solicitar obtener una versión PDF de un cliente simplemente agregando ".pdf" a la URL:

```bash
GET /clients/1.pdf
```

### Live Streaming of Arbitrary Data

Rails le permite transmitir más que solo archivos. De hecho, puedes transmitir cualquier cosa
le gustaría en un objeto de respuesta. El módulo `ActionController::Live` permite
que cree una conexión persistente con un navegador. Con este módulo,
Ser capaz de enviar datos arbitrarios al navegador en momentos específicos.

#### Incorporating Live Streaming

Incluir `ActionController::Live` dentro de su clase de controlador proporcionará
todas las acciones dentro del controlador la capacidad de transmitir datos. Puedes mezclar
el módulo así:

```ruby
class MyController < ActionController::Base
  include ActionController::Live

  def stream
    response.headers['Content-Type'] = 'text/event-stream'
    100.times {
      response.stream.write "hello world\n"
      sleep 1
    }
  ensure
    response.stream.close
  end
end
```

El código anterior mantendrá una conexión persistente con el navegador y enviará 100
mensajes de `"hello world\n"`, cada uno con un segundo de diferencia.

Hay un par de cosas a tener en cuenta en el ejemplo anterior. Tenemos que hacer
asegúrese de cerrar el flujo de respuesta. Olvidando cerrar el arroyo se irá
el enchufe abierto para siempre. También tenemos que establecer el tipo de contenido en `text/event-stream`
antes de escribir en el flujo de respuesta. Esto se debe a que los encabezados no se pueden escribir.
después de que se haya confirmado la respuesta (cuando `response.committed?` devuelve un verdadero
value), que ocurre cuando `write` o `confirm` el flujo de respuesta.

#### Example Usage

Supongamos que está creando una máquina de karaoke y un usuario quiere obtener la
letra de una canción en particular. Cada `Song` tiene un número particular de líneas y
cada línea toma tiempo `num_beats` para terminar de cantar.

Si quisiéramos devolver la letra en estilo Karaoke (solo enviando la línea cuando
el cantante ha terminado la línea anterior), entonces podríamos usar `ActionController::Live`
como sigue:

```ruby
class LyricsController < ActionController::Base
  include ActionController::Live

  def show
    response.headers['Content-Type'] = 'text/event-stream'
    song = Song.find(params[:id])

    song.each do |line|
      response.stream.write line.lyrics
      sleep line.num_beats
    end
  ensure
    response.stream.close
  end
end
```

El código anterior envía la siguiente línea solo después de que el cantante haya completado la anterior.
línea.

#### Streaming Considerations

La transmisión de datos arbitrarios es una herramienta extremadamente poderosa. Como se muestra en la anterior
ejemplos, puede elegir cuándo y qué enviar a través de un flujo de respuesta. Sin embargo,
también debe tener en cuenta las siguientes cosas:

* Cada flujo de respuesta crea un nuevo hilo y copia el hilo local
  variables del hilo original. Tener demasiadas variables locales de subproceso puede
  impactar negativamente en el rendimiento. Del mismo modo, una gran cantidad de subprocesos también pueden
  obstaculizar el rendimiento.
* No cerrar el flujo de respuesta dejará abierto el socket correspondiente.
  Siempre. Asegúrese de llamar a `close`siempre que utilice un flujo de respuesta.
* Los servidores WEBrick almacenan en búfer todas las respuestas, por lo que incluyen `ActionController::Live`
  no trabajará. Debe utilizar un servidor web que no almacena automáticamente en búfer
  respuestas.

Log Filtering
-------------

Rails mantiene un archivo de registro para cada entorno en la carpeta `log`. Estos son extremadamente útiles al depurar lo que realmente está sucediendo en su aplicación, pero en una aplicación en vivo es posible que no desee que toda la información se almacene en el archivo de registro.

### Parameters Filtering

Puede filtrar los parámetros de solicitud confidenciales de sus archivos de registro agregándolos a `config.filter_parameters` en la configuración de la aplicación. Estos parámetros se marcarán como [FILTERED] en el registro.

```ruby
config.filter_parameters << :password
```

NOTE: Los parámetros proporcionados se filtrarán mediante una expresión regular de coincidencia parcial. Rails agrega por defecto `:password` en el inicializador apropiado (`initializers/filter_parameter_logging.rb`) y se preocupa por los parámetros típicos de la aplicación `password` y `password_confirmation`.

### Redirects Filtering

A veces es deseable filtrar de los archivos de registro algunas ubicaciones confidenciales a las que redirige su aplicación.
Puedes hacerlo usando la opción de configuración `config.filter_redirect`:

```ruby
config.filter_redirect << 's3.amazonaws.com'
```

Puede configurarlo en una Cadena, una Regexp o una matriz de ambos.

```ruby
config.filter_redirect.concat ['s3.amazonaws.com', /private_path/]
```

Las URL coincidentes se marcarán como '[FILTERED]'.

Rescue
------

Lo más probable es que su aplicación contenga errores o genere una excepción que debe manejarse. Por ejemplo, si el usuario sigue un enlace a un recurso que ya no existe en la base de datos, Active Record lanzará la excepción `ActiveRecord::RecordNotFound`.

El manejo de excepciones predeterminado de Rails muestra un mensaje de "500 Server Error" para todas las excepciones. Si la solicitud se realizó localmente, se muestra un buen seguimiento y algo de información adicional para que pueda averiguar qué salió mal y solucionarlo. Si la solicitud fue remota, Rails simplemente mostrará un simple mensaje de "500 Server Error" al usuario, o un "404 Not Found" si hubo un error de enrutamiento o no se pudo encontrar un registro. A veces, es posible que desee personalizar cómo se detectan estos errores y cómo se muestran al usuario. Hay varios niveles de manejo de excepciones disponibles en una aplicación Rails:

### The Default 500 and 404 Templates

De forma predeterminada, una aplicación de producción generará un mensaje de error 404 o 500, en el entorno de desarrollo se generan todas las excepciones no controladas. Estos mensajes están contenidos en archivos HTML estáticos en la carpeta pública, en `404.html` y` 500.html` respectivamente. Puede personalizar estos archivos para agregar información y estilo adicionales, pero recuerde que son HTML estático; es decir, no puede usar ERB, SCSS, CoffeeScript o diseños para ellos.

### `rescue_from`

Si quieres hacer algo un poco más elaborado al detectar errores, puedes usar `rescue_from`, que maneja excepciones de cierto tipo (o múltiples tipos) en un controlador completo y sus subclases.

Cuando ocurre una excepción que es capturada por una directiva `rescue_from`, el objeto de excepción se pasa al manejador. El controlador puede ser un método o un objeto `Proc` pasado a la opción `:with`. También puede utilizar un bloque directamente en lugar de un objeto `Proc` explícito.

Así es como puede usar `rescue_from` para interceptar todos los errores de` ActiveRecord::RecordNotFound` y hacer algo con ellos.

```ruby
class ApplicationController < ActionController::Base
  rescue_from ActiveRecord::RecordNotFound, with: :record_not_found

  private
    def record_not_found
      render plain: "404 Not Found", status: 404
    end
end
```

Por supuesto, este ejemplo es cualquier cosa menos elaborado y no mejora en absoluto el manejo de excepciones predeterminado, pero una vez que pueda detectar todas esas excepciones, podrá hacer lo que quiera con ellas. Por ejemplo, puede crear clases de excepción personalizadas que se lanzarán cuando un usuario no tenga acceso a una determinada sección de su aplicación:

```ruby
class ApplicationController < ActionController::Base
  rescue_from User::NotAuthorized, with: :user_not_authorized

  private
    def user_not_authorized
      flash[:error] = "You don't have access to this section."
      redirect_back(fallback_location: root_path)
    end
end

class ClientsController < ApplicationController
  # Check that the user has the right authorization to access clients.
  before_action :check_authorization

  # Note how the actions don't have to worry about all the auth stuff.
  def edit
    @client = Client.find(params[:id])
  end

  private
    # If the user is not authorized, just throw the exception.
    def check_authorization
      raise User::NotAuthorized unless current_user.admin?
    end
end
```

WARNING: Usar `rescue_from` con `Exception` o `StandardError` causaría efectos secundarios graves ya que evita que Rails maneje las excepciones correctamente. Como tal, no se recomienda hacerlo a menos que exista una razón de peso.

NOTE: Cuando se ejecuta en el entorno de producción, todos
Los errores de `ActiveRecord::RecordNotFound` generan la página de error 404. A menos que necesites
un comportamiento personalizado que no necesita para manejar esto.

NOTE: Ciertas excepciones solo se pueden rescatar desde la clase `ApplicationController`, ya que se generan antes de que se inicialice el controlador y se ejecute la acción.

Force HTTPS protocol
--------------------

Si desea asegurarse de que la comunicación con su controlador solo sea posible
vía HTTPS, debe hacerlo habilitando el middleware `ActionDispatch::SSL` a través de
`config.force_ssl` en la configuración de su entorno.
