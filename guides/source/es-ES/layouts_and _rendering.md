**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Layouts and Rendering in Rails
==============================

Esta guía cubre las características de diseño básicas de Action Controller y Action View.

Después de leer esta guía, sabrá:

* Cómo usar los diversos métodos de renderizado integrados en Rails.
* Cómo crear diseños con múltiples secciones de contenido.
* Cómo usar parciales para SECAR tus vistas.
* Cómo utilizar diseños anidados (sub-plantillas).

--------------------------------------------------------------------------------

Overview: How the Pieces Fit Together
-------------------------------------

Esta guía se centra en la interacción entre el Controlador y la Vista en el triángulo Modelo-Vista-Controlador. Como sabe, el controlador es responsable de organizar todo el proceso de manejo de una solicitud en Rails, aunque normalmente entrega cualquier código pesado al modelo. Pero luego, cuando llega el momento de enviar una respuesta al usuario, el Controlador entrega las cosas a la Vista. Es ese traspaso el tema de esta guía.

A grandes rasgos, esto implica decidir qué se debe enviar como respuesta y llamar a un método apropiado para crear esa respuesta. Si la respuesta es una vista completa, Rails también hace un trabajo adicional para envolver la vista en un diseño y posiblemente extraer vistas parciales. Verá todos esos caminos más adelante en esta guía.

Creating Responses
------------------

Desde el punto de vista del controlador, hay tres formas de crear una respuesta HTTP:

* Llame a `render` para crear una respuesta completa para enviar de vuelta al navegador
* Llame a `redirect_to` para enviar un código de estado de redirección HTTP al navegador
* Llame a `head` para crear una respuesta que consta únicamente de encabezados HTTP para enviar de vuelta al navegador

### Rendering by Default: Convention Over Configuration in Action

Has escuchado que Rails promueve la "convención sobre la configuración". La representación predeterminada es un excelente ejemplo de esto. De forma predeterminada, los controladores en Rails representan automáticamente vistas con nombres que corresponden a rutas válidas. Por ejemplo, si tiene este código en su clase `BooksController`:


```ruby
class BooksController < ApplicationController
end
```

Y lo siguiente en su archivo de rutas:

```ruby
resources :books
```

Y tienes un archivo de vista`app/views/books/index.html.erb`:


```html+erb
<h1>Books are coming soon!</h1>
```

Rails mostrará automáticamente `app/views/books/index.html.erb` cuando navegue a`/books` y verá "¡Pronto vendrán libros!" en tu pantalla

Sin embargo, una próxima pantalla es mínimamente útil, por lo que pronto creará su modelo `Book` y agregará la acción de índice a `BooksController`:

```ruby
class BooksController < ApplicationController
  def index
    @books = Book.all
  end
end
```

Tenga en cuenta que no tenemos una representación explícita al final de la acción de índice de acuerdo con el principio de "convención sobre configuración". La regla es que si no renderizas algo explícitamente al final de una acción del controlador, Rails buscará automáticamente la plantilla `action_name.html.erb` en la ruta de la vista del controlador y la renderizará. Entonces, en este caso, Rails representará el archivo `app / views / books / index.html.erb`.

Si queremos mostrar las propiedades de todos los libros en nuestra vista, podemos hacerlo con una plantilla ERB como esta:

```html+erb
<h1>Listing Books</h1>

<table>
  <thead>
    <tr>
      <th>Title</th>
      <th>Content</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @books.each do |book| %>
      <tr>
        <td><%= book.title %></td>
        <td><%= book.content %></td>
        <td><%= link_to "Show", book %></td>
        <td><%= link_to "Edit", edit_book_path(book) %></td>
        <td><%= link_to "Destroy", book, method: :delete, data: { confirm: "Are you sure?" } %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to "New book", new_book_path %>
```

NOTE: La representación real se realiza mediante clases anidadas del módulo. [`ActionView::Template::Handlers`](https://api.rubyonrails.org/classes/ActionView/Template/Handlers.html). Esta guía no profundiza en ese proceso, pero es importante saber que la extensión del archivo en su vista controla la elección del manejador de plantillas.

### Using `render`

En la mayoría de los casos, el método `ActionController::Base#render` hace el trabajo pesado de representar el contenido de su aplicación para que lo use un navegador. Hay una variedad de formas de personalizar el comportamiento de `render`. Puede representar la vista predeterminada para una plantilla de Rails, o una plantilla específica, o un archivo, o código en línea, o nada en absoluto. Puede representar texto, JSON o XML. También puede especificar el tipo de contenido o el estado HTTP de la respuesta renderizada.

TIP: si desea ver los resultados exactos de una llamada a `render` sin necesidad de inspeccionarla en un navegador, puede llamar a `render_to_string`. Este método toma exactamente las mismas opciones que `render`, pero devuelve una cadena en lugar de enviar una respuesta al navegador.

#### Rendering an Action's View

Si desea renderizar la vista que corresponde a una plantilla diferente dentro del mismo controlador, puede usar `render` con el nombre de la vista:

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render "edit"
  end
end
```

Si falla la llamada a `update`, al llamar a la acción `update` en este controlador se mostrará la plantilla `edit.html.erb` que pertenece al mismo controlador.

Si lo prefiere, puede utilizar un símbolo en lugar de una cadena para especificar la acción a representar:

```ruby
def update
  @book = Book.find(params[:id])
  if @book.update(book_params)
    redirect_to(@book)
  else
    render :edit
  end
end
```

#### Rendering an Action's Template from Another Controller

¿Qué sucede si desea renderizar una plantilla desde un controlador completamente diferente del que contiene el código de acción? También puede hacerlo con `render`, que acepta la ruta completa (relativa a`app/views`) de la plantilla para renderizar. Por ejemplo, si está ejecutando código en un "AdminProductsController" que se encuentra en `app / controllers / admin`, puede representar los resultados de una acción en una plantilla en `app/views/products` de esta manera:

```ruby
render "products/show"
```

Rails sabe que esta vista pertenece a un controlador diferente debido al carácter de barra inclinado en la cadena. Si desea ser explícito, puede utilizar la opción `:template` (que era necesaria en Rails 2.2 y versiones anteriores):

```ruby
render template: "products/show"
```

#### Wrapping it up

Las tres formas anteriores de renderizar (renderizar otra plantilla dentro del controlador, renderizar una plantilla dentro de otro controlador y renderizar un archivo arbitrario en el sistema de archivos) son en realidad variantes de la misma acción.

De hecho, en la clase BooksController, dentro de la acción de actualización donde queremos renderizar la plantilla de edición si el libro no se actualiza con éxito, todas las siguientes llamadas de renderización mostrarán la plantilla `edit.html.erb` en el` directorio de vistas / libros:

```ruby
render :edit
render action: :edit
render "edit"
render action: "edit"
render "books/edit"
render template: "books/edit"
```

El que use es realmente una cuestión de estilo y convención, pero la regla general es usar el más simple que tenga sentido para el código que está escribiendo.

#### Using `render` with `:inline`

El método `render` puede prescindir de una vista por completo, si está dispuesto a utilizar la opción `:inline` para proporcionar ERB como parte de la llamada al método. Esto es perfectamente válido:

```ruby
render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>"
```

WARNING: rara vez hay una buena razón para usar esta opción. Mezclar ERB en sus controladores anula la orientación MVC de Rails y dificultará que otros desarrolladores sigan la lógica de su proyecto. En su lugar, utilice una vista erb separada.

Por defecto, el renderizado en línea usa ERB. Puede forzarlo a usar Builder en su lugar con la opción `:type`:

```ruby
render inline: "xml.p {'Horrid coding practice!'}", type: :builder
```

#### Rendering Text

Puede enviar texto sin formato, sin ningún tipo de marcado, de vuelta al navegador utilizando
la opción `:plain` para `render`:

```ruby
render plain: "OK"
```

TIP: La representación de texto puro es más útil cuando responde a Ajax o
solicitudes de serviciode web que esperan algo diferente a lo apropiado


NOTE: De manera predeterminada, si usa la opción `:plain`, el texto se representa sin
utilizando el diseño actual. Si quieres que Rails ponga el texto en el actual
layout, debe agregar la opción `layout: true` y usar el `.text.erb`
extensión para el archivo de diseño.

#### Rendering HTML

Puede enviar una cadena HTML al navegador utilizando la opción `:html` para
`render`:


```ruby
render html: helpers.tag.strong('Not Found')
```

TIP: esto es útil cuando representa un pequeño fragmento de código HTML.
Sin embargo, es posible que desee considerar moverlo a un archivo de plantilla si el marcado
es complejo.

NOTE: Al usar la opción `html:`, las entidades HTML se escaparán si la cadena no está compuesta con API compatibles con `html_safe`.

#### Rendering JSON

JSON es un formato de datos JavaScript utilizado por muchas bibliotecas Ajax. Rails tiene soporte incorporado para convertir objetos a JSON y renderizar ese JSON en el navegador:

```ruby
render json: @product
```

TIP: No es necesario llamar a `to_json` en el objeto que desea renderizar. Si usa la opción `:json`, `render` llamará automáticamente a `to_json` por usted.

#### Rendering XML

Rails también tiene soporte incorporado para convertir objetos a XML y hacer que ese XML vuelva a la persona que llama:

```ruby
render xml: @product
```

TIP: No necesita llamar a `to_xml` en el objeto que desea representar. Si usa la opción `:xml`, `render` llamará automáticamente a `to_xml` por usted.

#### Rendering Vanilla JavaScript

Los rieles pueden representar JavaScript vainilla:

```ruby
render js: "alert('Hello Rails');"
```

Esto enviará la cadena suministrada al navegador con un tipo MIME de `text/javascript`.

#### Rendering raw body

Puede enviar un contenido sin procesar al navegador, sin configurar ningún contenido.
type, usando la opción `:body` para` render`:

```ruby
render body: "raw"
```

TIP: esta opción debe usarse solo si no le importa el tipo de contenido de
la respuesta. El uso de `:plain` o `:html` podría ser más apropiado para la mayoría de
hora.

NOTE: A menos que se anule, su respuesta devuelta de esta opción de procesamiento será
`text/plain`, ya que ese es el tipo de contenido predeterminado de la respuesta Action Dispatch.

#### Rendering raw file

Los rieles pueden representar un archivo sin formato desde una ruta absoluta. Esto es útil para
renderización condicional de archivos estáticos como páginas de error.

```ruby
render file: "#{Rails.root}/public/404.html", layout: false
```

Esto representa el archivo sin formato (no es compatible con ERB u otros controladores). Por
por defecto se representa dentro del diseño actual.

WARNING: El uso de la opción `:file` en combinación con la entrada de los usuarios puede provocar problemas de seguridad
ya que un atacante podría usar esta acción para acceder a archivos confidenciales de seguridad en su sistema de archivos.

TIP: `send_file` suele ser una opción mejor y más rápida si no se requiere un diseño.

#### Rendering objects

Los rieles pueden representar objetos que respondan a `:render_in`.

```ruby
render MyComponent.new
```
Esto llama a `render_in` en el objeto proporcionado con el contexto de vista actual.

#### Options for `render`

Las llamadas al método `render` generalmente aceptan seis opciones:

* `:content_type`
* `:layout`
* `:location`
* `:status`
* `:formats`
* `:variants`

##### The `:content_type` Option

Por defecto, Rails servirá los resultados de una operación de renderizado con el tipo de contenido MIME de `text / html` (o` application / json` si usa la opción `: json`, o` application / xml` para el ` : opción xml`). Hay momentos en los que le gustaría cambiar esto, y puede hacerlo configurando la opción `:content_type`:

```ruby
render template: "feed", content_type: "application/rss"
```

##### The `:layout` Option

Con la mayoría de las opciones para "renderizar", el contenido renderizado se muestra como parte del diseño actual. Aprenderá más sobre los diseños y cómo usarlos más adelante en esta guía.

Puedes usar la opción `:layout` para decirle a Rails que use un archivo específico como diseño para la acción actual:

```ruby
render layout: "special_layout"
```

También puedes decirle a Rails que renderice sin ningún diseño:

```ruby
render layout: false
```

##### The `:location` Option

Puede usar la opción `:location` para establecer el encabezado HTTP `Location`:

```ruby
render xml: photo, location: photo_url(photo)
```

##### The `:status` Option

Rails generará automáticamente una respuesta con el código de estado HTTP correcto (en la mayoría de los casos, esto es `200 OK`). Puede usar la opción `:status` para cambiar esto:

```ruby
render status: 500
render status: :forbidden
```

Rails comprende tanto los códigos de estado numéricos como los símbolos correspondientes que se muestran a continuación.


| Response Class      | HTTP Status Code | Symbol                           |
| ------------------- | ---------------- | -------------------------------- |
| **Informational**   | 100              | :continue                        |
|                     | 101              | :switching_protocols             |
|                     | 102              | :processing                      |
| **Success**         | 200              | :ok                              |
|                     | 201              | :created                         |
|                     | 202              | :accepted                        |
|                     | 203              | :non_authoritative_information   |
|                     | 204              | :no_content                      |
|                     | 205              | :reset_content                   |
|                     | 206              | :partial_content                 |
|                     | 207              | :multi_status                    |
|                     | 208              | :already_reported                |
|                     | 226              | :im_used                         |
| **Redirection**     | 300              | :multiple_choices                |
|                     | 301              | :moved_permanently               |
|                     | 302              | :found                           |
|                     | 303              | :see_other                       |
|                     | 304              | :not_modified                    |
|                     | 305              | :use_proxy                       |
|                     | 307              | :temporary_redirect              |
|                     | 308              | :permanent_redirect              |
| **Client Error**    | 400              | :bad_request                     |
|                     | 401              | :unauthorized                    |
|                     | 402              | :payment_required                |
|                     | 403              | :forbidden                       |
|                     | 404              | :not_found                       |
|                     | 405              | :method_not_allowed              |
|                     | 406              | :not_acceptable                  |
|                     | 407              | :proxy_authentication_required   |
|                     | 408              | :request_timeout                 |
|                     | 409              | :conflict                        |
|                     | 410              | :gone                            |
|                     | 411              | :length_required                 |
|                     | 412              | :precondition_failed             |
|                     | 413              | :payload_too_large               |
|                     | 414              | :uri_too_long                    |
|                     | 415              | :unsupported_media_type          |
|                     | 416              | :range_not_satisfiable           |
|                     | 417              | :expectation_failed              |
|                     | 421              | :misdirected_request             |
|                     | 422              | :unprocessable_entity            |
|                     | 423              | :locked                          |
|                     | 424              | :failed_dependency               |
|                     | 426              | :upgrade_required                |
|                     | 428              | :precondition_required           |
|                     | 429              | :too_many_requests               |
|                     | 431              | :request_header_fields_too_large |
|                     | 451              | :unavailable_for_legal_reasons   |
| **Server Error**    | 500              | :internal_server_error           |
|                     | 501              | :not_implemented                 |
|                     | 502              | :bad_gateway                     |
|                     | 503              | :service_unavailable             |
|                     | 504              | :gateway_timeout                 |
|                     | 505              | :http_version_not_supported      |
|                     | 506              | :variant_also_negotiates         |
|                     | 507              | :insufficient_storage            |
|                     | 508              | :loop_detected                   |
|                     | 510              | :not_extended                    |
|                     | 511              | :network_authentication_required |

NOTE: Si intenta renderizar contenido junto con un código de estado sin contenido
(100-199, 204, 205 o 304), se eliminará de la respuesta.

##### The `:formats` Option

Rails usa el formato especificado en la solicitud (o `:html` por defecto). Usted puede
cambie esto pasando la opción `:formats` con un símbolo o una matriz:

```ruby
render formats: :xml
render formats: [:json, :xml]
```

Si no existe una plantilla con el formato especificado, se genera un error `ActionView::MissingTemplate`.

##### The `:variants` Option

Esto le dice a Rails que busque variaciones de la plantilla del mismo formato.
Puede especificar una lista de variantes pasando la opción `:variants` con un símbolo o una matriz.

Un ejemplo de uso sería este.

```ruby
# called in HomeController#index
render variants: [:mobile, :desktop]
```

Con este conjunto de variantes, Rails buscará el siguiente conjunto de plantillas y utilizará la primera que exista.

- `app/views/home/index.html+mobile.erb`
- `app/views/home/index.html+desktop.erb`
- `app/views/home/index.html.erb`

Si no existe una plantilla con el formato especificado, se genera un error `ActionView::MissingTemplate`.

En lugar de configurar la variante en la llamada de renderizado, también puede configurarla en el objeto de solicitud en la acción de su controlador.

```ruby
def index
  request.variant = determine_variant
end

private

def determine_variant
  variant = nil
  # some code to determine the variant(s) to use
  variant = :mobile if session[:use_mobile]

  variant
end
```

#### Finding Layouts

Para encontrar el diseño actual, Rails primero busca un archivo en `app/views/layouts` con el mismo nombre base que el controlador. Por ejemplo, las acciones de renderizado de la clase `PhotosController` usarán `app/views/layouts/photos.html.erb` (o `app/views/layouts/photos.builder`). Si no existe un diseño específico del controlador, Rails usará `app/views/layouts/application.html.erb` o `app/views/layouts/application.builder`. Si no hay un diseño `.erb`, Rails usará un diseño `.builder` si existe. Rails también proporciona varias formas de asignar diseños específicos de manera más precisa a controladores y acciones individuales.

##### Specifying Layouts for Controllers

Puede anular las convenciones de diseño predeterminadas en sus controladores utilizando la declaración `layout`. Por ejemplo:

```ruby
class ProductsController < ApplicationController
  layout "inventory"
  #...
end
```

Con esta declaración, todas las vistas generadas por `ProductsController` usarán `app/views/layouts/Inventory.html.erb` como su diseño.

Para asignar un diseño específico para toda la aplicación, use una declaración de diseño en su clase ApplicationController:

```ruby
class ApplicationController < ActionController::Base
  layout "main"
  #...
end
```

Con esta declaración, todas las vistas en toda la aplicación usarán `app/views/layouts/main.html.erb` para su diseño.

##### Choosing Layouts at Runtime

Puede utilizar un símbolo para aplazar la elección del diseño hasta que se procese una solicitud:

```ruby
class ProductsController < ApplicationController
  layout :products_layout

  def show
    @product = Product.find(params[:id])
  end

  private
    def products_layout
      @current_user.special? ? "special" : "products"
    end

end
```

Ahora, si el usuario actual es un usuario especial, obtendrá un diseño especial al ver un producto.

Incluso puede utilizar un método en línea, como Proc, para determinar el diseño. Por ejemplo, si pasa un objeto Proc, el bloque que le da a Proc recibirá la instancia del `controller`, por lo que el diseño se puede determinar en función de la solicitud actual:

```ruby
class ProductsController < ApplicationController
  layout Proc.new { |controller| controller.request.xhr? ? "popup" : "application" }
end
```

##### Conditional Layouts

Los diseños especificados en el nivel del controlador admiten las opciones `:only` y `:except`. Estas opciones toman un nombre de método o una matriz de nombres de métodos, correspondientes a los nombres de métodos dentro del controlador:

```ruby
class ProductsController < ApplicationController
  layout "product", except: [:index, :rss]
end
```

Con esta declaración, el diseño `product` se usaría para todo menos los métodos `rss` e `index`.

##### Layout Inheritance

Las declaraciones de diseño caen en cascada hacia abajo en la jerarquía, y las declaraciones de diseño más específicas siempre anulan las más generales. Por ejemplo:


* `application_controller.rb`

    ```ruby
    class ApplicationController < ActionController::Base
      layout "main"
    end
    ```

* `articles_controller.rb`

    ```ruby
    class ArticlesController < ApplicationController
    end
    ```

* `special_articles_controller.rb`

    ```ruby
    class SpecialArticlesController < ArticlesController
      layout "special"
    end
    ```

* `old_articles_controller.rb`

    ```ruby
    class OldArticlesController < SpecialArticlesController
      layout false

      def show
        @article = Article.find(params[:id])
      end

      def index
        @old_articles = Article.older
        render layout: "old"
      end
      # ...
    end
    ```

En esta aplication:

* En general, las vistas se renderizarán en el diseño `main`
* `ArticlesController#index` usará el diseño `main`
* `SpecialArticlesController#index` usará el diseño `special`
* `OldArticlesController#show` no usará ningún diseño
* `OldArticlesController#index` usará el diseño `old`

##### Template Inheritance

Similar a la lógica de Herencia de Diseño, si una plantilla o parcial no se encuentra en la ruta convencional, el controlador buscará una plantilla o parcial para representar en su cadena de herencia. Por ejemplo:


```ruby
# in app/controllers/application_controller
class ApplicationController < ActionController::Base
end

# in app/controllers/admin_controller
class AdminController < ApplicationController
end

# in app/controllers/admin/products_controller
class Admin::ProductsController < AdminController
  def index
  end
end
```

El orden de búsqueda para una acción `admin/products#index` será:

* `app/views/admin/products/`
* `app/views/admin/`
* `app/views/application/`

Esto hace que `app/views/application/` sea un gran lugar para sus parciales compartidos, que luego se pueden representar en su ERB como tal:

```erb
<%# app/views/admin/products/index.html.erb %>
<%= render @products || "empty_list" %>

<%# app/views/application/_empty_list.html.erb %>
There are no items in this list <em>yet</em>.
```

#### Avoiding Double Render Errors

Tarde o temprano, la mayoría de los desarrolladores de Rails verán el mensaje de error "Solo se puede renderizar o redirigir una vez por acción". Si bien esto es molesto, es relativamente fácil de solucionar. Por lo general, ocurre debido a un malentendido fundamental de la forma en que funciona "render".

Por ejemplo, aquí hay un código que activará este error:

```ruby
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show"
  end
  render action: "regular_show"
end
```

Si `@book.special?` Se evalúa como `true`, Rails iniciará el proceso de renderizado para volcar la variable `@book` en la vista `special_show`. Pero esto _no_ detendrá la ejecución del resto del código en la acción `show`, y cuando Rails llegue al final de la acción, comenzará a representar la vista `regular_show` y arrojará un error. La solución es simple: asegúrese de tener solo una llamada a `render` o `redirect` en una única ruta de código. Una cosa que puede ayudar es "y volver". Aquí hay una versión parcheada del método:

```ruby
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show" and return
  end
  render action: "regular_show"
end
```

Asegúrese de usar `and return` en lugar de` && return` porque `&& return` no funcionará debido a la precedencia del operador en el lenguaje Ruby.

Tenga en cuenta que el renderizado implícito realizado por ActionController detecta si se ha llamado a `render`, por lo que lo siguiente funcionará sin errores:

```ruby
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show"
  end
end
```

Esto renderizará un libro con `special?` Establecido con la plantilla `special_show`, mientras que otros libros se renderizarán con la plantilla `show` predeterminada.

### Using `redirect_to`

Otra forma de manejar la devolución de respuestas a una solicitud HTTP es con `redirect_to`. Como ha visto, `render` le dice a Rails qué vista (u otro activo) usar para construir una respuesta. El método `redirect_to` hace algo completamente diferente: le dice al navegador que envíe una nueva solicitud para una URL diferente. Por ejemplo, puede redirigir desde donde se encuentre en su código al índice de fotos en su aplicación con esta llamada:

```ruby
redirect_to photos_url
```

Puede utilizar `redirect_back` para devolver al usuario a la página de la que acaba de llegar.
Esta ubicación se extrae del encabezado `HTTP_REFERER` que no está garantizado
para ser configurado por el navegador, por lo que debe proporcionar el `fallback_location`
utilizar en este caso.

```ruby
redirect_back(fallback_location: root_path)
```

NOTE: `redirect_to` y `redirect_back` no se detienen y regresan inmediatamente de la ejecución del método, simplemente establecen respuestas HTTP. Se ejecutarán las declaraciones que se produzcan después de ellas en un método. Puede detener mediante un "retorno" explícito o algún otro mecanismo de detención, si es necesario.

#### Getting a Different Redirect Status Code

Rails usa el código de estado HTTP 302, una redirección temporal, cuando llamas a `redirect_to`. Si desea usar un código de estado diferente, quizás 301, una redirección permanente, puede usar la opción `:status`:

```ruby
redirect_to photos_path, status: 301
```

Al igual que la opción `:status` para`  ender`, `:status` para `redirect_to` acepta designaciones de encabezado tanto numéricas como simbólicas.

#### The Difference Between `render` and `redirect_to`

A veces, los desarrolladores sin experiencia piensan en `redirect_to` como una especie de comando `goto`, que mueve la ejecución de un lugar a otro en su código Rails. Esto no es correcto. Su código deja de ejecutarse y espera una nueva solicitud del navegador. Simplemente sucede que le ha dicho al navegador qué solicitud debe realizar a continuación, enviando un código de estado HTTP 302.

Considere estas acciones para ver la diferencia:

```ruby
def index
  @books = Book.all
end

def show
  @book = Book.find_by(id: params[:id])
  if @book.nil?
    render action: "index"
  end
end
```

Con el código en esta forma, probablemente habrá un problema si la variable `@book` es `nil`. Recuerde, una `render :action` no ejecuta ningún código en la acción de destino, por lo que nada configurará la variable `@books` que probablemente requiera la vista `index`. Una forma de solucionar esto es redirigir en lugar de renderizar:

```ruby
def index
  @books = Book.all
end

def show
  @book = Book.find_by(id: params[:id])
  if @book.nil?
    redirect_to action: :index
  end
end
```

Con este código, el navegador hará una nueva solicitud para la página de índice, se ejecutará el código del método `index` y todo estará bien.

El único inconveniente de este código es que requiere un viaje de ida y vuelta al navegador: el navegador solicitó la acción de mostrar con `/books/1` y el controlador encuentra que no hay libros, por lo que el controlador envía una respuesta de redirección 302 a el navegador le dice que vaya a `/books/`, el navegador cumple y envía una nueva solicitud al controlador solicitando ahora la acción `index`, el controlador luego obtiene todos los libros en la base de datos y muestra la plantilla de índice, enviándolo de nuevo al navegador que luego lo muestra en su pantalla.

Si bien en una aplicación pequeña, esta latencia adicional puede no ser un problema, es algo en lo que pensar si el tiempo de respuesta es una preocupación. Podemos demostrar una forma de manejar esto con un ejemplo artificial:

```ruby
def index
  @books = Book.all
end

def show
  @book = Book.find_by(id: params[:id])
  if @book.nil?
    @books = Book.all
    flash.now[:alert] = "Your book was not found"
    render "index"
  end
end
```

Esto detectaría que no hay libros con el ID especificado, completará la variable de instancia `@ books` con todos los libros en el modelo y luego representará directamente la plantilla` index.html.erb`, devolviéndola al navegador con una mensaje de alerta flash para decirle al usuario lo sucedido.

### Using `head` To Build Header-Only Responses

El método `head` se puede usar para enviar respuestas con solo encabezados al navegador. El método `head` acepta un número o símbolo (ver [reference table](#the-status-option)) que representa un código de estado HTTP. El argumento de opciones se interpreta como un hash de nombres y valores de encabezado. Por ejemplo, puede devolver solo un encabezado de error:

```ruby
head :bad_request
```

Esto produciría el siguiente encabezado:

```
HTTP/1.1 400 Bad Request
Connection: close
Date: Sun, 24 Jan 2010 12:15:53 GMT
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
X-Runtime: 0.013483
Set-Cookie: _blog_session=...snip...; path=/; HttpOnly
Cache-Control: no-cache
```

O puede usar otros encabezados HTTP para transmitir otra información:

```ruby
head :created, location: photo_path(@photo)
```

Que produciría:

```
HTTP/1.1 201 Created
Connection: close
Date: Sun, 24 Jan 2010 12:16:44 GMT
Transfer-Encoding: chunked
Location: /photos/1
Content-Type: text/html; charset=utf-8
X-Runtime: 0.083496
Set-Cookie: _blog_session=...snip...; path=/; HttpOnly
Cache-Control: no-cache
```

Structuring Layouts
-------------------

Cuando Rails representa una vista como respuesta, lo hace combinando la vista con el diseño actual, utilizando las reglas para encontrar el diseño actual que se trataron anteriormente en esta guía. Dentro de un diseño, tiene acceso a tres herramientas para combinar diferentes bits de salida para formar la respuesta general:

* Asset tags
* `yield` and `content_for`
* Partials

### Asset Tag Helpers

Los ayudantes de etiquetas de activos proporcionan métodos para generar HTML que vinculan vistas a feeds, JavaScript, hojas de estilo, imágenes, videos y audios. Hay seis ayudantes de etiquetas de activos disponibles en Rails:

* `auto_discovery_link_tag`
* `javascript_include_tag`
* `stylesheet_link_tag`
* `image_tag`
* `video_tag`
* `audio_tag`


Puede usar estas etiquetas en diseños u otras vistas, aunque `auto_discovery_link_tag`,` javascript_include_tag` y `stylesheet_link_tag`, se usan más comúnmente en la sección `<head>` de un diseño.

WARNING: Los asistentes de etiquetas de activos _no_ verifican la existencia de los activos en las ubicaciones especificadas; simplemente asumen que sabes lo que estás haciendo y generan el enlace.

#### Linking to Feeds with the `auto_discovery_link_tag`

El ayudante `auto_discovery_link_tag` crea HTML que la mayoría de los navegadores y lectores de feeds pueden utilizar para detectar la presencia de feeds RSS, Atom o JSON. Toma el tipo de enlace (`:rss`, `:atom` o `: json`), un hash de opciones que se pasan a url_for y un hash de opciones para la etiqueta:

```erb
<%= auto_discovery_link_tag(:rss, {action: "feed"},
  {title: "RSS Feed"}) %>
```

Hay tres opciones de etiqueta disponibles para el `auto_discovery_link_tag`:

* `:rel` especifica el valor `rel` en el enlace. El valor predeterminado es "alternativo".
* `:type` especifica un tipo MIME explícito. Rails generará un tipo MIME apropiado automáticamente.
* `:title` especifica el título del enlace. El valor predeterminado es el valor `:type` en mayúsculas, por ejemplo, "ATOM" o "RSS".

#### Linking to JavaScript Files with the `javascript_include_tag`

El ayudante `javascript_include_tag` devuelve una etiqueta HTML `script` para cada fuente proporcionada.

Si está utilizando Rails con [Asset Pipeline](asset_pipeline.html)habilitado, este asistente generará un enlace a `/assets/javascripts/` en lugar de a `public/javascripts` que se usaba en versiones anteriores de Rails. Este enlace luego es servido por la canalización de activos.

Un archivo JavaScript dentro de una aplicación Rails o motor Rails va en una de estas tres ubicaciones: `app/assets`,`lib/assets` o `vendor/assets`. Estas ubicaciones se explican en detalle en la [Asset Organization section in the Asset Pipeline Guide](asset_pipeline.html#asset-organization).

Puede especificar una ruta completa relativa a la raíz del documento, o una URL, si lo prefiere. Por ejemplo, para enlazar a un archivo JavaScript que está dentro de un directorio llamado `javascripts` dentro de uno de `app/assets`, `lib/assets` o `vendor/assets`, deberías hacer esto:

```erb
<%= javascript_include_tag "main" %>
```

Rails generará una etiqueta `script` como esta:

```html
<script src='/assets/main.js'></script>
```

Luego, la gem Sprockets atiende la solicitud a este activo.

Para incluir varios archivos como `app/assets/javascripts/main.js` y `app/assets/javascripts/column.js` al mismo tiempo:

```erb
<%= javascript_include_tag "main", "columns" %>
```
Para incluir `app/assets/javascripts/main.js` y `app/assets/javascripts/photos/columns.js`:

```erb
<%= javascript_include_tag "main", "/photos/columns" %>
```

Para incluir `http://example.com/main.js`:

```erb
<%= javascript_include_tag "http://example.com/main.js" %>
```

#### Linking to CSS Files with the `stylesheet_link_tag`

El ayudante `stylesheet_link_tag` devuelve una etiqueta HTML `<link>` para cada fuente proporcionada.

Si está utilizando Rails con el "Asset Pipeline" habilitado, este asistente generará un enlace a `/assets/stylesheets/`. Este enlace luego es procesado por la gema Sprockets. Un archivo de hoja de estilo se puede almacenar en una de estas tres ubicaciones: `app/assets`, `lib/assets` o `vendor/assets`.

Puede especificar una ruta completa relativa a la raíz del documento o una URL. Por ejemplo, para vincular a un archivo de hoja de estilo que está dentro de un directorio llamado `stylesheets` dentro de uno de `app/assets`, `lib/assets` o` vendor/assets`, deberías hacer esto:

```erb
<%= stylesheet_link_tag "main" %>
```

Para incluir `app/assets/stylesheets/main.css` y `app/assets/stylesheets/columns.css`:


```erb
<%= stylesheet_link_tag "main", "columns" %>
```

Para incluir `app/assets/stylesheets/main.css` y `app/assets/stylesheets/photos/columns.css`:

```erb
<%= stylesheet_link_tag "main", "photos/columns" %>
```

Para incluir   `http://example.com/main.css`:

```erb
<%= stylesheet_link_tag "http://example.com/main.css" %>
```

De forma predeterminada, la `stylesheet_link_tag` crea enlaces con`media="screen" rel="stylesheet"`. Puede anular cualquiera de estos valores predeterminados especificando una opción adecuada (`:media`,`:rel`):

```erb
<%= stylesheet_link_tag "main_print", media: "print" %>
```

#### Linking to Images with the `image_tag`

El ayudante `image_tag` crea una etiqueta HTML`<img/>` en el archivo especificado. Por defecto, los archivos se cargan desde "public/images".

WARNING: Tenga en cuenta que debe especificar la extensión de la imagen.

```erb
<%= image_tag "header.png" %>
```

Puede proporcionar una ruta a la imagen si lo desea:

```erb
<%= image_tag "icons/delete.gif" %>
```

Puede proporcionar un hash de opciones HTML adicionales:

```erb
<%= image_tag "icons/delete.gif", {height: 45} %>
```

Puede proporcionar texto alternativo para la imagen que se utilizará si el usuario tiene imágenes desactivadas en su navegador. Si no especifica un texto alternativo explícitamente, el nombre predeterminado del archivo es el del archivo, en mayúsculas y sin extensión. Por ejemplo, estas dos etiquetas de imagen devolverían el mismo código:

```erb
<%= image_tag "home.gif" %>
<%= image_tag "home.gif", alt: "Home" %>
```

También puede especificar una etiqueta de tamaño especial, en el formato "{ancho} x {alto}":

```erb
<%= image_tag "home.gif", size: "50x20" %>
```

Además de las etiquetas especiales anteriores, puede proporcionar un hash final de las opciones HTML estándar, como `:class`, `:id` o `:name`:

```erb
<%= image_tag "home.gif", alt: "Go Home",
                          id: "HomeImage",
                          class: "nav_bar" %>
```

#### Linking to Videos with the `video_tag`

El ayudante `video_tag` crea una etiqueta HTML 5 `<video>` en el archivo especificado. De forma predeterminada, los archivos se cargan desde `public/videos`.

```erb
<%= video_tag "movie.ogg" %>
```

Produce

```erb
<video src="/videos/movie.ogg" />
```

Como una `image_tag`, puede proporcionar una ruta, ya sea absoluta o relativa al directorio `public/videos`. Además, puede especificar la opción `size: "#{width}x#{height}"`  como una `image_tag`. Las etiquetas de video también pueden tener cualquiera de las opciones HTML especificadas al final (`id`, `class` et al).

La etiqueta de video también admite todas las opciones HTML `<video>` mediante el hash de opciones HTML, que incluyen:

* `poster:" image_name.png"`, proporciona una imagen para colocar en lugar del video antes de que comience a reproducirse.
* `autoplay: true`, comienza a reproducir el video al cargar la página.
* `loop: true`, repite el video una vez que llega al final.
* `controls: true`, proporciona controles proporcionados por el navegador para que el usuario interactúe con el video.
* `autobuffer: true`, el video precargará el archivo para el usuario al cargar la página.

También puede especificar varios videos para reproducir pasando una serie de videos a la etiqueta `video_tag`:

``erb
<%= video_tag ["trailer.ogg", "movie.ogg"] %>
```

Esto producirá:

```erb
<video>
  <source src="/videos/trailer.ogg">
  <source src="/videos/movie.ogg">
</video>
```

#### Linking to Audio Files with the `audio_tag`

El ayudante `audio_tag` crea una etiqueta HTML 5 `<audio>` en el archivo especificado. De forma predeterminada, los archivos se cargan desde `public/audios`.

```erb
<%= audio_tag "music.mp3" %>
```

Puede proporcionar una ruta al archivo de audio si lo desea:

```erb
<%= audio_tag "music/first_song.mp3" %>
```

También puede proporcionar un hash de opciones adicionales, como `:id`,`: class`, etc.

Al igual que el `video_tag`, el `audio_tag` tiene opciones especiales:

* `autoplay: true`, comienza a reproducir el audio al cargar la página
* `controls: true`, proporciona controles proporcionados por el navegador para que el usuario interactúe con el audio.
* `autobuffer: true`, el audio precargará el archivo para el usuario al cargar la página.

### Understanding `yield`

Dentro del contexto de un diseño, `yield` identifica una sección donde se debe insertar el contenido de la vista. La forma más sencilla de utilizar esto es tener un único "rendimiento", en el que se inserta todo el contenido de la vista que se está renderizando actualmente:

```html+erb
<html>
  <head>
  </head>
  <body>
  <%= yield %>
  </body>
</html>
```

También puede crear un diseño con varias regiones productivas:

```html+erb
<html>
  <head>
  <%= yield :head %>
  </head>
  <body>
  <%= yield %>
  </body>
</html>
```

El cuerpo principal de la vista siempre se representará en el `yield` sin nombre. Para convertir el contenido en un `yield` con nombre, utiliza el método `content_for`.

### Using the `content_for` Method

El método `content_for` le permite insertar contenido en un bloque llamado` yield` en su diseño. Por ejemplo, esta vista funcionaría con el diseño que acaba de ver:

```html+erb
<% content_for :head do %>
  <title>A simple page</title>
<% end %>

<p>Hello, Rails!</p>
```

El resultado de convertir esta página en el diseño proporcionado sería este HTML:

```html+erb
<html>
  <head>
  <title>A simple page</title>
  </head>
  <body>
  <p>Hello, Rails!</p>
  </body>
</html>
```

El método `content_for` es muy útil cuando su diseño contiene regiones distintas, como barras laterales y pies de página, que deben insertar sus propios bloques de contenido. También es útil para insertar etiquetas que cargan archivos JavaScript o css específicos de la página en el encabezado de un diseño genérico.

### Using Partials

Las plantillas parciales, generalmente llamadas "partials", son otro dispositivo para dividir el proceso de renderizado en partes más manejables. Con un parcial, puede mover el código para representar una parte particular de una respuesta a su propio archivo.

#### Naming Partials

Para renderizar un parcial como parte de una vista, usa el método `render` dentro de la vista:

```ruby
<%= render "menu" %>
```

Esto generará un archivo llamado `_menu.html.erb` en ese punto dentro de la vista que se está procesando. Tenga en cuenta el carácter de subrayado inicial: los parciales se nombran con un carácter de subrayado inicial para distinguirlos de las vistas normales, aunque se mencionan sin el subrayado. Esto es cierto incluso cuando está extrayendo un parcial de otra carpeta:

```ruby
<%= render "shared/menu" %>
```

Ese código extraerá el parcial de `app/views/shared/_menu.html.erb`.

#### Using Partials to Simplify Views

Una forma de usar parciales es tratarlos como el equivalente de subrutinas: como una forma de mover detalles fuera de una vista para que pueda comprender lo que está sucediendo más fácilmente. Por ejemplo, es posible que tenga una vista similar a esta:

```erb
<%= render "shared/ad_banner" %>

<h1>Products</h1>

<p>Here are a few of our fine products:</p>
...

<%= render "shared/footer" %>
```

Aquí, los parciales `_ad_banner.html.erb` y `_footer.html.erb` podrían contener
contenido compartido por muchas páginas de su aplicación. No necesitas ver
los detalles de estas secciones cuando se concentra en una página en particular.

Como se vio en las secciones anteriores de esta guía, `yield` es una herramienta muy poderosa
para limpiar sus diseños. Tenga en cuenta que es Ruby puro, por lo que puede usar
casi en todas partes. Por ejemplo, podemos usarlo para SECAR el diseño del formulario
definiciones para varios recursos similares:

* `users/index.html.erb`

    ```html+erb
    <%= render "shared/search_filters", search: @q do |form| %>
      <p>
        Name contains: <%= form.text_field :name_contains %>
      </p>
    <% end %>
    ```

* `roles/index.html.erb`

    ```html+erb
    <%= render "shared/search_filters", search: @q do |form| %>
      <p>
        Title contains: <%= form.text_field :title_contains %>
      </p>
    <% end %>
    ```

* `shared/_search_filters.html.erb`

    ```html+erb
    <%= form_with model: search do |form| %>
      <h1>Search form:</h1>
      <fieldset>
        <%= yield form %>
      </fieldset>
      <p>
        <%= form.submit "Search" %>
      </p>
    <% end %>
    ```
  
TIP: Para el contenido que se comparte entre todas las páginas de su aplicación, puede utilizar parciales directamente desde los diseños.

#### Partial Layouts

Un parcial puede usar su propio archivo de diseño, al igual que una vista puede usar un diseño. Por ejemplo, puede llamar a un parcial como este:

```erb
<%= render partial: "link_area", layout: "graybar" %>
```

Esto buscaría un parcial llamado `_link_area.html.erb` y lo renderizaría usando el diseño `_graybar.html.erb`. Tenga en cuenta que los diseños para parciales siguen el mismo nombre de subrayado inicial que los parciales regulares y se colocan en la misma carpeta con el parcial al que pertenecen (no en la carpeta maestra de diseños).

También tenga en cuenta que se requiere especificar explícitamente `:partial` cuando se pasan opciones adicionales como`:layout`.

#### Passing Local Variables

También puede pasar variables locales a parciales, haciéndolas aún más poderosas y flexibles. Por ejemplo, puede utilizar esta técnica para reducir la duplicación entre las páginas nuevas y de edición, mientras mantiene un poco de contenido distinto:


* `new.html.erb`

    ```html+erb
    <h1>New zone</h1>
    <%= render partial: "form", locals: {zone: @zone} %>
    ```

* `edit.html.erb`

    ```html+erb
    <h1>Editing zone</h1>
    <%= render partial: "form", locals: {zone: @zone} %>
    ```

* `_form.html.erb`

    ```html+erb
    <%= form_with model: zone do |form| %>
      <p>
        <b>Zone name</b><br>
        <%= form.text_field :name %>
      </p>
      <p>
        <%= form.submit %>
      </p>
    <% end %>
    ```
Aunque el mismo parcial se representará en ambas vistas, el asistente de envío de la Vista de acción devolverá "Crear zona" para la nueva acción y "Actualizar zona" para la acción de edición.

Para pasar una variable local a un parcial solo en casos específicos, use `local_assigns`.


* `index.html.erb`

  ```erb
  <%= render user.articles %>
  ```

* `show.html.erb`

  ```erb
  <%= render article, full: true %>
  ```

* `_article.html.erb`

  ```erb
  <h2><%= article.title %></h2>

  <% if local_assigns[:full] %>
    <%= simple_format article.body %>
  <% else %>
    <%= truncate article.body %>
  <% end %>
  ```

De esta forma es posible utilizar el parcial sin necesidad de declarar todas las variables locales.

Cada parcial también tiene una variable local con el mismo nombre que el parcial (menos el subrayado inicial). Puede pasar un objeto a esta variable local a través de la opción `:object`:

```erb
<%= render partial: "customer", object: @new_customer %>
```

Dentro del parcial `customer`, la variable `customer` se referirá a `@new_customer` desde la vista principal.

Si tiene una instancia de un modelo para convertir en parcial, puede usar una sintaxis abreviada:

```erb
<%= render @customer %>
```

Suponiendo que la variable de instancia `@customer` contiene una instancia del modelo `Customer`, esto usará `_customer.html.erb` para representarlo y pasará la variable local `customer` al parcial que se referirá al ` Variable de instancia de @ customer en la vista principal.

* `index.html.erb`

    ```html+erb
    <h1>Products</h1>
    <%= render partial: "product", collection: @products %>
    ```

* `_product.html.erb`

    ```html+erb
    <p>Product Name: <%= product.name %></p>
    ```

Cuando se llama a un parcial con una colección pluralizada, las instancias individuales del parcial tienen acceso al miembro de la colección que se representa mediante una variable con el nombre del parcial. En este caso, el parcial es `_product`, y dentro del parcial `_product`, puede hacer referencia a `product` para obtener la instancia que se está renderizando.

También hay una abreviatura para esto. Suponiendo que `@products` es una colección de instancias de `Product`, puedes simplemente escribir esto en el `index.html.erb` para producir el mismo resultado:

```html+erb
<h1>Products</h1>
<%= render @products %>
```

Rails determina el nombre del parcial a usar mirando el nombre del modelo en la colección. De hecho, incluso puede crear una colección heterogénea y renderizarla de esta manera, y Rails elegirá el parcial adecuado para cada miembro de la colección:


* `index.html.erb`

    ```html+erb
    <h1>Contacts</h1>
    <%= render [customer1, employee1, customer2, employee2] %>
    ```

* `customers/_customer.html.erb`

    ```html+erb
    <p>Customer: <%= customer.name %></p>
    ```

* `employees/_employee.html.erb`

    ```html+erb
    <p>Employee: <%= employee.name %></p>
    ```

En este caso, Rails utilizará los parciales de cliente o empleado según corresponda para cada miembro de la colección.

En el caso de que la colección esté vacía, `render` devolverá nil, por lo que debería ser bastante sencillo proporcionar contenido alternativo.

```html+erb
<h1>Products</h1>
<%= render(@products) || "There are no products available." %>
```

#### Local Variables

Para usar un nombre de variable local personalizado dentro del parcial, especifique la opción `:as` en la llamada al parcial:

```erb
<%= render partial: "product", collection: @products, as: :item %>
```

Con este cambio, puede acceder a una instancia de la colección `@products` como la variable local `item` dentro del parcial.

También puede pasar variables locales arbitrarias a cualquier parcial que esté renderizando con la opción `locals: {}`:

```erb
<%= render partial: "product", collection: @products,
           as: :item, locals: {title: "Products Page"} %>
```

En este caso, el parcial tendrá acceso a una variable local `title` con el valor" Página de productos ".

TIP: Rails también hace que una variable de contador esté disponible dentro de un parcial llamado por la colección, llamado así por el título del parcial seguido de "_counter". Por ejemplo, al renderizar una colección `@products`, el `_product.html.erb` parcial puede acceder a la variable `product_counter` que indexa el número de veces que se ha renderizado dentro de la vista adjunta. Tenga en cuenta que también se aplica cuando se cambió el nombre parcial utilizando la opción `as:`. Por ejemplo, la variable de contador para el código anterior sería `item_counter`.

También puede especificar un segundo parcial para ser renderizado entre instancias del parcial principal usando la opción `:spacer_template`:

#### Spacer Templates

```erb
<%= render partial: @products, spacer_template: "product_ruler" %>
```

Rails renderizará el `_product_ruler` parcial (sin que se le pasen datos) entre cada par de parciales de `_product`.

#### Collection Partial Layouts

Al renderizar colecciones, también es posible usar la opción `:layout`:

```erb
<%= render partial: "product", collection: @products, layout: "special_layout" %>
```

El diseño se renderizará junto con el parcial de cada elemento de la colección. Las variables object actual y object_counter también estarán disponibles en el diseño, de la misma manera que están dentro del parcial.

### Using Nested Layouts

Es posible que su aplicación requiera un diseño que difiera ligeramente del diseño de su aplicación normal para admitir un controlador en particular. En lugar de repetir el diseño principal y editarlo, puede lograr esto utilizando diseños anidados (a veces llamados sub-plantillas). Aquí hay un ejemplo:

Suponga que tiene el siguiente diseño de `ApplicationController`:


* `app/views/layouts/application.html.erb`

    ```html+erb
    <html>
    <head>
      <title><%= @page_title or "Page Title" %></title>
      <%= stylesheet_link_tag "layout" %>
      <style><%= yield :stylesheets %></style>
    </head>
    <body>
      <div id="top_menu">Top menu items here</div>
      <div id="menu">Menu items here</div>
      <div id="content"><%= content_for?(:content) ? yield(:content) : yield %></div>
    </body>
    </html>
    ```

En las páginas generadas por `NewsController`, desea ocultar el menú superior y agregar un menú derecho:


* `app/views/layouts/news.html.erb`

    ```html+erb
    <% content_for :stylesheets do %>
      #top_menu {display: none}
      #right_menu {float: right; background-color: yellow; color: black}
    <% end %>
    <% content_for :content do %>
      <div id="right_menu">Right menu items here</div>
      <%= content_for?(:news_content) ? yield(:news_content) : yield %>
    <% end %>
    <%= render template: "layouts/application" %>
    ```

Eso es. Las vistas de Noticias usarán el nuevo diseño, ocultando el menú superior y agregando un nuevo menú derecho dentro del div "contenido".

Hay varias formas de obtener resultados similares con diferentes esquemas de sub-plantillas utilizando esta técnica. Tenga en cuenta que no hay límite en los niveles de anidación. Se puede usar el método `ActionView ::render` a través de` render template: 'layouts/news'` para basar un nuevo diseño en el diseño de Noticias. Si está seguro de que no subtemplará el diseño `News`, puede reemplazar el `content_for?(:news_content) ? yield(:news_content) :yield` con simplemente `yield`.
