**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Descripción General de la Action View
=====================================

Después de leer esta guía, sabrá:

* Qué es Action View y cómo usarlo con Rails.
* La mejor manera de usar plantillas, parciales y diseños.
* Qué ayudantes proporciona Action View.
* Cómo utilizar vistas localizadas.

--------------------------------------------------------------------------------

What is Action View?
--------------------

En Rails, las solicitudes web son manejadas por [Action Controller](action_controller_overview.html) y Action View. Por lo general, Action Controller se ocupa de comunicarse con la base de datos y realizar acciones CRUD cuando sea necesario. Action View es responsable de compilar la respuesta.

Las plantillas de Action View se escriben usando Ruby incrustado en etiquetas mezcladas con HTML. Para evitar saturar las plantillas con código repetitivo, una serie de clases auxiliares proporcionan un comportamiento común para formularios, fechas y cadenas. También es fácil agregar nuevos ayudantes a su aplicación a medida que evoluciona.

NOTE: Algunas características de la Action View están vinculadas al Active Record, pero eso no significa que la Action View dependa del Registro activo. Action View es un paquete independiente que se puede utilizar con cualquier tipo de bibliotecas Ruby.

Using Action View with Rails
----------------------------

Para cada controlador hay un directorio asociado en el directorio `app/views` que contiene los archivos de plantilla que componen las vistas asociadas con ese controlador. Estos archivos se utilizan para mostrar la vista que resulta de cada acción del controlador.

Echemos un vistazo a lo que hace Rails por defecto cuando crea un nuevo recurso usando el generador de andamios:

```bash
$ bin/rails generate scaffold article
      [...]
      invoke  scaffold_controller
      create    app/controllers/articles_controller.rb
      invoke    erb
      create      app/views/articles
      create      app/views/articles/index.html.erb
      create      app/views/articles/edit.html.erb
      create      app/views/articles/show.html.erb
      create      app/views/articles/new.html.erb
      create      app/views/articles/_form.html.erb
      [...]
```

Hay una convención de nomenclatura para vistas en Rails. Normalmente, las vistas comparten su nombre con la acción del controlador asociada, como puede ver arriba.
Por ejemplo, la acción del controlador de índice de `articles_controller.rb` usará el archivo de visualización `index.html.erb` en el directorio `app/views/articles`.
El HTML completo devuelto al cliente se compone de una combinación de este archivo ERB, una plantilla de diseño que lo envuelve y todos los parciales a los que la vista puede hacer referencia. En esta guía encontrará documentación más detallada sobre cada uno de estos tres componentes.

Templates, Partials, and Layouts
-------------------------------

Como se mencionó, la salida final de HTML es una composición de tres elementos de Rails: `Templates`, `Partials` y `Layouts`.
A continuación se muestra una breve descripción general de cada uno de ellos.

### Templates

Las plantillas de Action View se pueden escribir de varias formas. Si el archivo de plantilla tiene una extensión `.erb`, entonces usa una mezcla de ERB (Embedded Ruby) y HTML. Si el archivo de plantilla tiene una extensión `.builder`, se utiliza la biblioteca `Builder::XmlMarkup`.

Rails admite varios sistemas de plantillas y utiliza una extensión de archivo para distinguirlos. Por ejemplo, un archivo HTML que utilice el sistema de plantillas ERB tendrá la extensión `.html.erb`.

#### ERB

Dentro de una plantilla ERB, el código Ruby se puede incluir usando etiquetas `<% %>` y `<%= %>`. Las etiquetas `<% %>` se utilizan para ejecutar código Ruby que no devuelve nada, como condiciones, bucles o bloques, y las etiquetas `<%= %>` se utilizan cuando se desea salida.

Considere el siguiente bucle para nombres:

```html+erb
<h1>Names of all the people</h1>
<% @people.each do |person| %>
  Name: <%= person.name %><br>
<% end %>
```

El bucle se configura usando etiquetas de incrustación regulares (`<% %>`) y el nombre se inserta usando las etiquetas de incrustación de salida (`<% =%>`). Tenga en cuenta que esto no es solo una sugerencia de uso: las funciones de salida regulares como `print` y `Puts` no se mostrarán en la vista con plantillas ERB. Entonces esto estaría mal:

```html+erb
<%# WRONG %>
Hi, Mr. <% puts "Frodo" %>
```

Para suprimir los espacios en blanco iniciales y finales, puede usar `<% -` `-%>` indistintamente con `<%` y `%>`.

A continuación se muestran algunos ejemplos básicos:

```ruby
xml.em("emphasized")
xml.em { xml.b("emph & bold") }
xml.a("A Link", "href" => "https://rubyonrails.org")
xml.target("name" => "compile", "option" => "fast")
```

que produciría:

```html
<em>emphasized</em>
<em><b>emph &amp; bold</b></em>
<a href="https://rubyonrails.org">A link</a>
<target option="fast" name="compile" />
```

Cualquier método con un bloque se tratará como una etiqueta de marcado XML con marcado anidado en el bloque. Por ejemplo, lo siguiente:

```ruby
xml.div {
  xml.h1(@person.name)
  xml.p(@person.bio)
}
```

produciría algo como:

```html
<div>
  <h1>David Heinemeier Hansson</h1>
  <p>A product of Danish Design during the Winter of '79...</p>
</div>
```

A continuación se muestra un ejemplo de RSS completo que se usa realmente en Basecamp:

```ruby
xml.rss("version" => "2.0", "xmlns:dc" => "http://purl.org/dc/elements/1.1/") do
  xml.channel do
    xml.title(@feed_title)
    xml.link(@url)
    xml.description "Basecamp: Recent items"
    xml.language "en-us"
    xml.ttl "40"

    for item in @recent_items
      xml.item do
        xml.title(item_title(item))
        xml.description(item_description(item)) if item_description(item)
        xml.pubDate(item_pubDate(item))
        xml.guid(@person.firm.account.url + @recent_items.url(item))
        xml.link(@person.firm.account.url + @recent_items.url(item))
        xml.tag!("dc:creator", item.author_name) if item_has_creator?(item)
      end
    end
  end
end
```

#### Jbuilder
[Jbuilder](https://github.com/rails/jbuilder) es una joya que es
mantenida por el equipo de Rails e incluido en Rails predeterminado `Gemfile`.
Es similar a Builder, pero ese se usa para generar JSON, en lugar de XML.

Si no lo tiene, puede agregar lo siguiente a su `Gemfile`:

```ruby
gem 'jbuilder'
```

Un objeto Jbuilder llamado `json` se pone automáticamente a disposición de las plantillas con
una extensión `.jbuilder`.

A continuación se muestra un ejemplo básico:

```ruby
json.name("Alex")
json.email("alex@example.com")
```

produciría:

```json
{
  "name": "Alex",
  "email": "alex@example.com"
}
```

Consulte la[Jbuilder documentation](https://github.com/rails/jbuilder#jbuilder) para
más ejemplos e información.

#### Template Caching

Por defecto, Rails compilará cada plantilla a un método para representarla. Cuando modifique una plantilla, Rails verificará el tiempo de modificación del archivo y lo volverá a compilar en modo de desarrollo.

### Partials

Las plantillas parciales, generalmente llamadas "partials", son otro dispositivo para dividir el proceso de renderizado en fragmentos más manejables. Con parciales, puede extraer fragmentos de código de sus plantillas para separar archivos y también reutilizarlos en todas sus plantillas.

#### Naming Partials

Para renderizar un parcial como parte de una vista, utiliza el método `render` dentro de la vista:

```erb
<%= render "menu" %>
```

Esto generará un archivo llamado `_menu.html.erb` en ese punto dentro de la vista que se está procesando. Tenga en cuenta el carácter de guión bajo: los parciales se nombran con un guión bajo para distinguirlos de las vistas normales, a pesar de que se mencionan sin el guión bajo. Esto es válido incluso cuando extrae un parcial de otra carpeta:

```erb
<%= render "shared/menu" %>
```

Ese código extraerá el parcial de `app/views/shared/_menu.html.erb`.

#### Using Partials to simplify Views

Una forma de usar parciales es tratarlos como el equivalente de subrutinas; una forma de mover detalles fuera de una vista para que pueda comprender lo que está sucediendo más fácilmente. Por ejemplo, puede tener una vista similar a esta:

```html+erb
<%= render "shared/ad_banner" %>
<h1>Products</h1>

<p>Here are a few of our fine products:</p>
<% @products.each do |product| %>
  <%= render partial: "product", locals: { product: product } %>
<% end %>

<%= render "shared/footer" %>
```

Aquí, los parciales `_ad_banner.html.erb` y `_footer.html.erb` podrían incluir contenido que se comparte entre muchas páginas de su aplicación. No necesita ver los detalles de estas secciones cuando se concentra en una página en particular.

#### `render` without `partial` and `locals` options

En el ejemplo anterior, `render` toma 2 opciones: `partial` y `locals`. Pero si
estas son las únicas opciones que desea pasar, puede omitir usando estas opciones.
Por ejemplo, en lugar de:

```erb
<%= render partial: "product", locals: { product: @product } %>
```

También puedes hacer:

```erb
<%= render "product", product: @product %>
```

#### The `as` and `object` options

Por defecto, `ActionView::Partials::PartialRenderer` tiene su objeto en una variable local con el mismo nombre que la plantilla. Entonces, dado:

```erb
<%= render partial: "product" %>
```

dentro de `_product` parcial obtendremos `@product` en la variable local `product`,
como si hubiéramos escrito:

```erb
<%= render partial: "product", locals: { product: @product } %>
```

La opción `object` puede usarse para especificar directamente qué objeto se representa en el parcial; útil cuando el objeto de la plantilla está en otra parte (por ejemplo, en una variable de instancia diferente o en una variable local).

Por ejemplo, en lugar de:

```erb
<%= render partial: "product", locals: { product: @item } %>
```

nosotros haríamos:

```erb
<%= render partial: "product", object: @item %>
```

Con la opción `as` podemos especificar un nombre diferente para dicha variable local. Por ejemplo, si quisiéramos que fuera `item` en lugar de `product`, haríamos:

```erb
<%= render partial: "product", object: @item, as: "item" %>
```

Esto es equivalente a

`` erb
<% = render parcial: "producto", locales: {item: @item}%>
`` `

#### Colecciones de renderizado

Es muy común que una plantilla necesite iterar sobre una colección y representar una sub-plantilla para cada uno de los elementos. Este patrón se ha implementado como un método único que acepta una matriz y representa un parcial para cada uno de los elementos de la matriz.

Entonces este ejemplo para renderizar todos los productos:

```erb
<% @products.each do |product| %>
  <%= render partial: "product", locals: { product: product } %>
<% end %>
```

se puede reescribir en una sola línea:

```erb
<%= render partial: "product", collection: @products %>
```

Cuando se llama a un parcial con una colección, las instancias individuales del parcial tienen acceso al miembro de la colección que se representa mediante una variable con el nombre del parcial. En este caso, el parcial es `_product`, y dentro de él puede hacer referencia a `product` para obtener el miembro de la colección que se está procesando.

Puede usar una sintaxis abreviada para representar colecciones. Suponiendo que `@productos` es una colección de instancias de `Product`, simplemente puede escribir lo siguiente para producir el mismo resultado:

```erb
<%= render @products %>
```

Rails determina el nombre del parcial a utilizar mirando el nombre del modelo en la colección, `Product` en este caso. De hecho, incluso puede renderizar una colección compuesta por instancias de diferentes modelos utilizando esta abreviatura, y Rails elegirá el parcial adecuado para cada miembro de la colección.

#### Spacer Templates

También puede especificar un segundo parcial para ser renderizado entre instancias del parcial principal usando la opción `:spacer_template`:

```erb
<%= render partial: @products, spacer_template: "product_ruler" %>
```

Rails renderizará el `_product_ruler` parcial (sin que se le pasen datos) entre cada par de parciales de `_product`.

### Layouts

Los diseños se pueden utilizar para representar una plantilla de vista común en torno a los resultados de las acciones del controlador Rails. Normalmente, una aplicación Rails tendrá un par de diseños en los que se renderizarán las páginas. Por ejemplo, un sitio podría tener un diseño para un usuario conectado y otro para el lado de marketing o ventas del sitio. El diseño del usuario registrado puede incluir navegación de nivel superior que debería estar presente en muchas acciones del controlador. El diseño de ventas de una aplicación SaaS puede incluir navegación de nivel superior para cosas como las páginas "Precios" y "Contáctenos". Es de esperar que cada diseño tenga un aspecto diferente. Puede leer sobre los diseños con más detalle en la guía [Layouts and Rendering in Rails](layouts_and_rendering.html).

Partial Layouts
---------------

Los parciales pueden tener sus propios diseños aplicados. Estos diseños son diferentes de los aplicados a una acción de controlador, pero funcionan de manera similar.

Digamos que estamos mostrando un artículo en una página que debería estar envuelto en un `div` para fines de visualización. En primer lugar, crearemos un nuevo `Article`:

```ruby
Article.create(body: 'Partial Layouts are cool!')
```

En la plantilla `show`, renderizaremos el parcial de `_article` envuelto en el diseño de `box`:

**articles/show.html.erb**

```erb
<%= render partial: 'article', layout: 'box', locals: { article: @article } %>
```

El diseño `box` simplemente envuelve el parcial `_article` en un `div`:

**articles/_box.html.erb**

```html+erb
<div class='box'>
  <%= yield %>
</div>
```

Tenga en cuenta que el diseño parcial tiene acceso a la variable local `article` que se pasó a la llamada `render`. Sin embargo, a diferencia de los diseños de toda la aplicación, los diseños parciales todavía tienen el prefijo de subrayado.

También puede representar un bloque de código dentro de un diseño parcial en lugar de llamar a `yield`. Por ejemplo, si no tuviéramos el `_article` parcial, podríamos hacer esto en su lugar:

**articles/show.html.erb**

```html+erb
<% render(layout: 'box', locals: { article: @article }) do %>
  <div>
    <p><%= article.body %></p>
  </div>
<% end %>
```

Suponiendo que usamos el mismo parcial `_box` de arriba, esto produciría el mismo resultado que en el ejemplo anterior.

View Paths
----------

Al presentar una respuesta, el controlador debe resolver dónde
se ubican las vistas. Por defecto, solo se ve dentro del directorio `app/views`.

Podemos agregar otras ubicaciones y darles cierta prioridad al resolver
rutas utilizando los métodos `prepend_view_path` y` append_view_path`.

### Prepend view path

Esto puede ser útil, por ejemplo, cuando queremos colocar vistas dentro de un
directorio para subdominios.

Podemos hacer esto usando:

```ruby
prepend_view_path "app/views/#{request.subdomain}"
```

Luego, la Vista de acción se verá primero en este directorio al resolver las vistas.

### Append view path

Del mismo modo, podemos agregar rutas:

```ruby
append_view_path "app/views/direct"
```

Esto agregará `app/views/direct` al final de las rutas de búsqueda.

Overview of helpers provided by Action View
-------------------------------------------

WIP: no todos los ayudantes se enumeran aquí. Para obtener una lista completa, consulte la [API documentation](https://api.rubyonrails.org/classes/ActionView/Helpers.html)

El siguiente es solo un breve resumen general de los ayudantes disponibles en la Vista de acción. Se recomienda que revise la  [API Documentation](https://api.rubyonrails.org/classes/ActionView/Helpers.html), que cubre todos los ayudantes con más detalle, pero esto debería servir como un buen punto de partida.

### AssetTagHelper

Este módulo proporciona métodos para generar HTML que vincula vistas a activos como imágenes, archivos JavaScript, hojas de estilo y feeds.

De forma predeterminada, Rails se vincula a estos activos en el host actual en la carpeta pública, pero puede indicarle a Rails que se vincule a los activos desde un servidor de activos dedicado configurando `config.action_controller.asset_host` en la configuración de la aplicación, generalmente en` config/ambientes/production.rb`. Por ejemplo, supongamos que su host de activos es `assets.example.com`:

```ruby
config.action_controller.asset_host = "assets.example.com"
image_tag("rails.png") # => <img src="http://assets.example.com/images/rails.png" />
```

#### auto_discovery_link_tag

Devuelve una etiqueta de enlace que los navegadores y lectores de feeds pueden usar para detectar automáticamente un feed RSS, Atom o JSON.

```ruby
auto_discovery_link_tag(:rss, "http://www.example.com/feed.rss", { title: "RSS Feed" }) # =>
  <link rel="alternate" type="application/rss+xml" title="RSS Feed" href="http://www.example.com/feed.rss" />
```

#### image_path

Calcula la ruta a un recurso de imagen en el directorio `app/assets/images`. Se pasarán las rutas completas desde la raíz del documento. Utilizado internamente por `image_tag` para construir la ruta de la imagen.

```ruby
image_path("edit.png") # => /assets/edit.png
```

La huella digital se agregará al nombre del archivo si config.assets.digest se establece en verdadero.

```ruby
image_path("edit.png") # => /assets/edit-2d1a2db63fc738690021fedb5a65b68e.png
```

#### image_url

Calcula la URL de un recurso de imagen en el directorio `app/assets/images`. Esto llamará a `image_path` internamente y se fusionará con su host actual o su host activo.

```ruby
image_url("edit.png") # => http://www.example.com/assets/edit.png
```

#### image_tag

Devuelve una etiqueta de imagen HTML para la fuente. La fuente puede ser una ruta completa o un archivo que existe en su directorio `app/assets/images`.

```ruby
image_tag("icon.png") # => <img src="/assets/icon.png" />
```

#### javascript_include_tag

Devuelve una etiqueta de secuencia de comandos HTML para cada una de las fuentes proporcionadas. Puede pasar el nombre de archivo (la extensión `.js` es opcional) de los archivos JavaScript que existen en su directorio `app/assets/javascripts` para incluirlos en la página actual o puede pasar la ruta completa relativa a la raíz del documento.

```ruby
javascript_include_tag "common" # => <script src="/assets/common.js"></script>
```

#### javascript_path

Calcula la ruta a un activo de JavaScript en el directorio `app/assets/javascripts`. Si el nombre del archivo fuente no tiene extensión, se agregará `.js`. Se pasarán las rutas completas desde la raíz del documento. Utilizado internamente por `javascript_include_tag` para construir la ruta del script.

```ruby
javascript_path "common" # => /assets/common.js
```

#### javascript_url

Calcula la URL de un activo de JavaScript en el directorio `app/assets/javascripts`. Esto llamará a `javascript_path` internamente y se fusionará con su host actual o su host activo.

```ruby
javascript_url "common" # => http://www.example.com/assets/common.js
```

#### stylesheet_link_tag

```ruby
stylesheet_link_tag "application" # => <link href="/assets/application.css" media="screen" rel="stylesheet" />
```

#### stylesheet_path

Calcula la ruta a un activo de hoja de estilo en el directorio `app/assets/stylesheets`. Si el nombre del archivo de origen no tiene extensión, se agregará `.css`. Se pasarán las rutas completas desde la raíz del documento. Utilizado internamente por `stylesheet_link_tag` para construir la ruta de la hoja de estilo.

```ruby
stylesheet_path "application" # => /assets/application.css
```

#### stylesheet_url

Calcula la URL de un activo de hoja de estilo en el directorio `app/assets/stylesheets`. Esto llamará a `stylesheet_path` internamente y se fusionará con su host actual o su host activo.

```ruby
stylesheet_url "application" # => http://www.example.com/assets/application.css
```

### AtomFeedHelper

#### atom_feed

Este ayudante facilita la creación de una alimentación Atom. Aquí hay un ejemplo de uso completo:

**config/routes.rb**

```ruby
resources :articles
```

**app/controllers/articles_controller.rb**

```ruby
def index
  @articles = Article.all

  respond_to do |format|
    format.html
    format.atom
  end
end
```

**app/views/articles/index.atom.builder**

```ruby
atom_feed do |feed|
  feed.title("Articles Index")
  feed.updated(@articles.first.created_at)

  @articles.each do |article|
    feed.entry(article) do |entry|
      entry.title(article.title)
      entry.content(article.body, type: 'html')

      entry.author do |author|
        author.name(article.author_name)
      end
    end
  end
end
```

### BenchmarkHelper

#### benchmark

Le permite medir el tiempo de ejecución de un bloque en una plantilla y registra el resultado en el registro. Envuelva este bloque alrededor de operaciones costosas o posibles cuellos de botella para obtener una lectura de tiempo para la operación.

```html+erb
<% benchmark "Process data files" do %>
  <%= expensive_files_operation %>
<% end %>
```

Esto agregaría algo como "Procesar archivos de datos (0.34523)" al registro, que luego puede usar para comparar los tiempos al optimizar su código.

### CacheHelper

#### cache

Un método para almacenar en caché fragmentos de una vista en lugar de una acción o página completa. Esta técnica es útil para almacenar en caché piezas como menús, listas de temas de noticias, fragmentos de HTML estático, etc. Este método toma un bloque que contiene el contenido que desea almacenar en caché. Consulte `AbstractController::Caching::Fragments` para obtener más información.

```erb
<% cache do %>
  <%= render "shared/footer" %>
<% end %>
```

### CaptureHelper

#### capture

El método `capture` le permite extraer parte de una plantilla en una variable. Luego, puede usar esta variable en cualquier lugar de sus plantillas o diseño.

```html+erb
<% @greeting = capture do %>
  <p>Welcome! The date and time is <%= Time.now %></p>
<% end %>
```

La variable capturada se puede usar en cualquier otro lugar.

```html+erb
<html>
  <head>
    <title>Welcome!</title>
  </head>
  <body>
    <%= @greeting %>
  </body>
</html>
```

#### content_for

Calling `content_for` stores a block of markup in an identifier for later use. You can make subsequent calls to the stored content in other templates or the layout by passing the identifier as an argument to `yield`.

For example, let's say we have a standard application layout, but also a special page that requires certain JavaScript that the rest of the site doesn't need. We can use `content_for` to include this JavaScript on our special page without fattening up the rest of the site.

**app/views/layouts/application.html.erb**

```html+erb
<html>
  <head>
    <title>Welcome!</title>
    <%= yield :special_script %>
  </head>
  <body>
    <p>Welcome! The date and time is <%= Time.now %></p>
  </body>
</html>
```

**app/views/articles/special.html.erb**

```html+erb
<p>This is a special page.</p>

<% content_for :special_script do %>
  <script>alert('Hello!')</script>
<% end %>
```

### DateHelper

#### date_select

Devuelve un conjunto de etiquetas de selección (una para año, mes y día) preseleccionadas para acceder a un atributo específico basado en la fecha.

```ruby
date_select("article", "published_on")
```

#### datetime_select

Devuelve un conjunto de etiquetas de selección (una para año, mes, día, hora y minuto) preseleccionadas para acceder a un atributo específico basado en fecha y hora.

```ruby
datetime_select("article", "published_on")
```

#### distance_of_time_in_words

Informa la distancia aproximada en el tiempo entre dos objetos de Hora o Fecha o enteros como segundos. Establezca `include_seconds` en true si desea aproximaciones más detalladas.

```ruby
distance_of_time_in_words(Time.now, Time.now + 15.seconds)        # => less than a minute
distance_of_time_in_words(Time.now, Time.now + 15.seconds, include_seconds: true)  # => less than 20 seconds
```

#### select_date

Devuelve un conjunto de etiquetas de selección HTML (una para año, mes y día) preseleccionadas con la `date` proporcionada.

```ruby
# Generates a date select that defaults to the date provided (six days after today)
select_date(Time.today + 6.days)

# Generates a date select that defaults to today (no specified date)
select_date()
```

#### select_datetime

Devuelve un conjunto de etiquetas de selección HTML (una para año, mes, día, hora y minuto) preseleccionadas con la `datetime` proporcionada.

```ruby
# Generates a datetime select that defaults to the datetime provided (four days after today)
select_datetime(Time.now + 4.days)

# Generates a datetime select that defaults to today (no specified datetime)
select_datetime()
```

#### select_day

Devuelve una etiqueta de selección con opciones para cada uno de los días del 1 al 31 con el día actual seleccionado.

```ruby
# Generates a select field for days that defaults to the day for the date provided
select_day(Time.today + 2.days)

# Generates a select field for days that defaults to the number given
select_day(5)
```

#### select_hour

Devuelve una etiqueta de selección con opciones para cada una de las horas 0 a 23 con la hora actual seleccionada.

```ruby
# Generates a select field for hours that defaults to the hours for the time provided
select_hour(Time.now + 6.hours)
```

#### select_minute

Devuelve una etiqueta de selección con opciones para cada uno de los minutos del 0 al 59 con el minuto actual seleccionado.

```ruby
# Generates a select field for minutes that defaults to the minutes for the time provided.
select_minute(Time.now + 10.minutes)
```

#### select_month

Devuelve una etiqueta selecta con opciones para cada uno de los meses de enero a diciembre con el mes actual seleccionado.

```ruby
# Generates a select field for months that defaults to the current month
select_month(Date.today)
```

#### select_second

Devuelve una etiqueta de selección con opciones para cada uno de los segundos 0 a 59 con el segundo actual seleccionado.

```ruby
# Generates a select field for seconds that defaults to the seconds for the time provided
select_second(Time.now + 16.seconds)
```

#### select_time

Devuelve un conjunto de etiquetas de selección HTML (una por hora y minuto).

```ruby
# Generates a time select that defaults to the time provided
select_time(Time.now)
```

#### select_year

Devuelve una etiqueta de selección con opciones para cada uno de los cinco años en cada lado de la actual, que se selecciona. El radio de cinco años se puede cambiar usando las teclas `:start_year` y `:end_year` en las `options`.

```ruby
# Generates a select field for five years on either side of Date.today that defaults to the current year
select_year(Date.today)

# Generates a select field from 1900 to 2009 that defaults to the current year
select_year(Date.today, start_year: 1900, end_year: 2009)
```

#### time_ago_in_words

Like `distance_of_time_in_words`, but where `to_time` is fixed to `Time.now`.

```ruby
time_ago_in_words(3.minutes.from_now)  # => 3 minutes
```

#### time_select

Devuelve un conjunto de etiquetas de selección (una para hora, minuto y opcionalmente la segunda) preseleccionadas para acceder a un atributo específico basado en el tiempo. Las selecciones están preparadas para la asignación de múltiples parámetros a un objeto de registro activo.

```ruby
# Creates a time select tag that, when POSTed, will be stored in the order variable in the submitted attribute
time_select("order", "submitted")
```

### DebugHelper

Devuelve una etiqueta `pre` que tiene un objeto volcado por YAML. Esto crea una forma muy legible de inspeccionar un objeto.

```ruby
my_hash = { 'first' => 1, 'second' => 'two', 'third' => [1,2,3] }
debug(my_hash)
```

```html
<pre class='debug_dump'>---
first: 1
second: two
third:
- 1
- 2
- 3
</pre>
```

### FormHelper

Los ayudantes de formulario están diseñados para facilitar el trabajo con modelos en comparación con el uso de elementos HTML estándar, proporcionando un conjunto de métodos para crear formularios basados ​​en sus modelos. Este asistente genera el HTML para los formularios, proporcionando un método para cada tipo de entrada (por ejemplo, texto, contraseña, selección, etc.). Cuando se envía el formulario (es decir, cuando el usuario presiona el botón enviar o se llama a form.submit a través de JavaScript), las entradas del formulario se agruparán en el objeto params y se devolverán al controlador.

Hay dos tipos de ayudantes de formulario: los que trabajan específicamente con atributos de modelo y los que no. Este asistente trata con aquellos que trabajan con atributos de modelo; Para ver un ejemplo de ayudantes de formulario que no funcionan con atributos de modelo, consulte la documentación `ActionView :: Helpers :: FormTagHelper`.

El método central de este asistente, `form_with`, le brinda la capacidad de crear un formulario para una instancia de modelo; por ejemplo, digamos que tiene una Persona modelo y desea crear una nueva instancia de ella:

```html+erb
<!-- Note: a @person variable will have been created in the controller (e.g. @person = Person.new) -->
<%= form_with model: @person do |form| %>
  <%= form.text_field :first_name %>
  <%= form.text_field :last_name %>
  <%= submit_tag 'Create' %>
<% end %>
```

El HTML generado para esto sería:


```html
<form class="new_person" id="new_person" action="/people" accept-charset="UTF-8" method="post">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input type="hidden" name="authenticity_token" value="lTuvBzs7ANygT0NFinXj98tfw3Emfm65wwYLbUvoWsK2pngccIQSUorM2C035M9dZswXgWTvKwFS8W5TVblpYw==" />
  <input type="text" name="person[first_name]" id="person_first_name" />
  <input type="text" name="person[last_name]" id="person_last_name" />
  <input type="submit" name="commit" value="Create" data-disable-with="Create" />
</form>
```

El objeto params creado cuando se envía este formulario se vería así:

```ruby
{"utf8" => "✓", "authenticity_token" => "lTuvBzs7ANygT0NFinXj98tfw3Emfm65wwYLbUvoWsK2pngccIQSUorM2C035M9dZswXgWTvKwFS8W5TVblpYw==", "person" => {"first_name" => "William", "last_name" => "Smith"}, "commit" => "Create", "controller" => "people", "action" => "create"}
```

El hash de params tiene un valor de persona anidada, por lo que se puede acceder con `params[:person]` en el controlador.

#### check_box

Devuelve una etiqueta de casilla de verificación diseñada para acceder a un atributo específico.

```ruby
# Let's say that @article.validated? is 1:
check_box("article", "validated")
# => <input type="checkbox" id="article_validated" name="article[validated]" value="1" />
#    <input name="article[validated]" type="hidden" value="0" />
```

#### fields_for

Crea un ámbito alrededor de un objeto de modelo específico. Esto hace que `fields_for` sea adecuado para especificar objetos de modelo adicionales en la misma forma:

```html+erb
<%= form_with model: @person do |person_form| %>
  First name: <%= person_form.text_field :first_name %>
  Last name : <%= person_form.text_field :last_name %>

  <%= fields_for @person.permission do |permission_fields| %>
    Admin?  : <%= permission_fields.check_box :admin %>
  <% end %>
<% end %>
```

#### file_field

Devuelve una etiqueta de entrada de carga de archivo adaptada para acceder a un atributo especificado.

```ruby
file_field(:user, :avatar)
# => <input type="file" id="user_avatar" name="user[avatar]" />
```

#### form_with

Crea un generador de formularios para trabajar. Si se especifica un argumento `model`, los campos de formulario se definirán en ese modelo y los valores de campo de formulario se rellenarán previamente con los atributos de modelo correspondientes.

```html+erb
<%= form_with model: @article do |form| %>#### hidden_field
  <%= form.label :title, 'Title' %>:
  <%= form.text_field :title %><br>
  <%= form.label :body, 'Body' %>:
  <%= form.text_area :body %><br>
<% end %>
```

#### hidden_field

Devuelve una etiqueta de entrada oculta diseñada para acceder a un atributo específico.

```ruby
hidden_field(:user, :token)
# => <input type="hidden" id="user_token" name="user[token]" value="#{@user.token}" />
```

#### label

Devuelve una etiqueta de etiqueta adaptada para etiquetar un campo de entrada para un atributo especificado

```ruby
label(:article, :title)
# => <label for="article_title">Title</label>
```

#### password_field

Devuelve una etiqueta de entrada del tipo "contraseña" adaptada para acceder a un atributo especificado.

```ruby
password_field(:login, :pass)
# => <input type="text" id="login_pass" name="login[pass]" value="#{@login.pass}" />
```

#### radio_button

io etiqueta de botón para acceder a un atributo especificado.

```ruby
# Let's say that @article.category returns "rails":
radio_button("article", "category", "rails")
radio_button("article", "category", "java")
# => <input type="radio" id="article_category_rails" name="article[category]" value="rails" checked="checked" />
#    <input type="radio" id="article_category_java" name="article[category]" value="java" />
```

#### text_area

Devuelve un conjunto de etiquetas de apertura y cierre de área de texto adaptadas para acceder a un atributo especificado.

```ruby
text_area(:comment, :text, size: "20x30")
# => <textarea cols="20" rows="30" id="comment_text" name="comment[text]">
#      #{@comment.text}
#    </textarea>
```

#### text_field

Devuelve una etiqueta de entrada del tipo "texto" adaptada para acceder a un atributo especificado.

```ruby
text_field(:article, :title)
# => <input type="text" id="article_title" name="article[title]" value="#{@article.title}" />
```

#### email_field

Devuelve una etiqueta de entrada del tipo "correo electrónico" diseñada para acceder a un atributo especificado.

```ruby
email_field(:user, :email)
# => <input type="email" id="user_email" name="user[email]" value="#{@user.email}" />
```

#### url_field

Devuelve una etiqueta de entrada del tipo "url" diseñada para acceder a un atributo especificado.

```ruby
url_field(:user, :url)
# => <input type="url" id="user_url" name="user[url]" value="#{@user.url}" />
```

### FormOptionsHelper

Proporciona varios métodos para convertir diferentes tipos de contenedores en un conjunto de etiquetas de opción.

#### collection_select

Devuelve las etiquetas `select` y `option` para la colección de valores de retorno existentes de `method` para la clase del` object`.

Ejemplo de estructura de objeto para usar con este método:

```ruby
class Article < ApplicationRecord
  belongs_to :author
end

class Author < ApplicationRecord
  has_many :articles
  def name_with_initial
    "#{first_name.first}. #{last_name}"
  end
end
```

Ejemplo de uso (seleccionando el Autor asociado para una instancia de Artículo, `@article`):

```ruby
collection_select(:article, :author_id, Author.all, :id, :name_with_initial, { prompt: true })
```

Si `@article.author_id` es 1, esto devolvería

```html
<select name="article[author_id]">
  <option value="">Please select</option>
  <option value="1" selected="selected">D. Heinemeier Hansson</option>
  <option value="2">D. Thomas</option>
  <option value="3">M. Clark</option>
</select>
```

#### collection_radio_buttons

Devuelve etiquetas `radio_button` para la colección de valores de retorno existentes de `method` para la clase de `object`.

Ejemplo de estructura de objeto para usar con este método:

```ruby
class Article < ApplicationRecord
  belongs_to :author
end

class Author < ApplicationRecord
  has_many :articles
  def name_with_initial
    "#{first_name.first}. #{last_name}"
  end
end
```

Ejemplo de uso (seleccionando el Autor asociado para una instancia de Artículo, `@article`):

```ruby
collection_radio_buttons(:article, :author_id, Author.all, :id, :name_with_initial)
```

Si `@article.author_id` es 1, esto devolvería:

```html
<input id="article_author_id_1" name="article[author_id]" type="radio" value="1" checked="checked" />
<label for="article_author_id_1">D. Heinemeier Hansson</label>
<input id="article_author_id_2" name="article[author_id]" type="radio" value="2" />
<label for="article_author_id_2">D. Thomas</label>
<input id="article_author_id_3" name="article[author_id]" type="radio" value="3" />
<label for="article_author_id_3">M. Clark</label>
```

Recuperando alguna opción pasada (por ejemplo, verificando programáticamente un objeto de la colección):

```ruby
collection_radio_buttons(:article, :author_id, Author.all, :id, :name_with_initial, {checked: Author.last})
```

En este caso, se comprobará el último objeto de la colección:

```html
<input id="article_author_id_1" name="article[author_id]" type="radio" value="1" />
<label for="article_author_id_1">D. Heinemeier Hansson</label>
<input id="article_author_id_2" name="article[author_id]" type="radio" value="2" />
<label for="article_author_id_2">D. Thomas</label>
<input id="article_author_id_3" name="article[author_id]" type="radio" value="3" checked="checked" />
<label for="article_author_id_3">M. Clark</label>
```

Para acceder a las opciones aprobadas mediante programación (por ejemplo, agregar una clase personalizada si está marcada):

**Sample html.erb**

```html+erb
<%= collection_radio_buttons(:article, :author_id, Author.all, :id, :name_with_initial, {checked: Author.last, required: true} do |rb| %>
      <%= rb.label(class: "#{'my-custom-class' if rb.value == Author.last.id}") { rb.radio_button + rb.text } %>
<% end %>
```


#### collection_check_boxes

Devuelve las etiquetas `check_box` para la colección de valores de retorno existentes de` method` para la clase `object`.

Ejemplo de estructura de objeto para usar con este método:

```ruby
class Article < ApplicationRecord
  has_and_belongs_to_many :authors
end

class Author < ApplicationRecord
  has_and_belongs_to_many :articles
  def name_with_initial
    "#{first_name.first}. #{last_name}"
  end
end
```

Ejemplo de uso (seleccionando los Autores asociados para una instancia de Artículo, `@article`):

```ruby
collection_check_boxes(:article, :author_ids, Author.all, :id, :name_with_initial)
```

Si `@ article.author_ids` es [1], esto devolvería:

```html
<input id="article_author_ids_1" name="article[author_ids][]" type="checkbox" value="1" checked="checked" />
<label for="article_author_ids_1">D. Heinemeier Hansson</label>
<input id="article_author_ids_2" name="article[author_ids][]" type="checkbox" value="2" />
<label for="article_author_ids_2">D. Thomas</label>
<input id="article_author_ids_3" name="article[author_ids][]" type="checkbox" value="3" />
<label for="article_author_ids_3">M. Clark</label>
<input name="article[author_ids][]" type="hidden" value="" />
```

#### option_groups_from_collection_for_select

Devuelve una cadena de etiquetas `option`, como `options_from_collection_for_select`, pero las agrupa por etiquetas `optgroup` en función de las relaciones de objeto de los argumentos.

Ejemplo de estructura de objeto para usar con este método:

```ruby
class Continent < ApplicationRecord
  has_many :countries
  # attribs: id, name
end

class Country < ApplicationRecord
  belongs_to :continent
  # attribs: id, name, continent_id
end
```

Uso de la muestra:

```ruby
option_groups_from_collection_for_select(@continents, :countries, :name, :id, :name, 3)
```

Salida posible:

```html
<optgroup label="Africa">
  <option value="1">Egypt</option>
  <option value="4">Rwanda</option>
  ...
</optgroup>
<optgroup label="Asia">
  <option value="3" selected="selected">China</option>
  <option value="12">India</option>
  <option value="5">Japan</option>
  ...
</optgroup>
```

NOTE: Solo se devuelven las etiquetas `optgroup` y `option`, por lo que aún debe ajustar la salida en una etiqueta `select` adecuada.

#### options_for_select

Acepta un contenedor (hash, matriz, enumerable, su tipo) y devuelve una cadena de etiquetas de opciones.

```ruby
options_for_select([ "VISA", "MasterCard" ])
# => <option>VISA</option> <option>MasterCard</option>
```

NOTE: Solo se devuelven las etiquetas `option`, debe envolver esta llamada en una etiqueta HTML regular `select`.

#### options_from_collection_for_select

Devuelve una cadena de etiquetas de opciones que se han compilado iterando sobre la `collection` y asignando el resultado de una llamada al `value_method` como el valor de la opción y el `text_method` como el texto de la opción.

```ruby
# options_from_collection_for_select(collection, value_method, text_method, selected = nil)
```

Por ejemplo, imagine un bucle iterando sobre cada persona en `@project.people` para generar una etiqueta de entrada:

```ruby
options_from_collection_for_select(@project.people, "id", "name")
# => <option value="#{person.id}">#{person.name}</option>
```

#### select

Cree una etiqueta de selección y una serie de etiquetas de opciones contenidas para el objeto y método proporcionados.

Ejemplo:

```ruby
select("article", "person_id", Person.all.collect { |p| [ p.name, p.id ] }, { include_blank: true })
```


If `@article.person_id` is 1, this would become:

```html
<select name="article[person_id]">
  <option value=""></option>
  <option value="1" selected="selected">David</option>
  <option value="2">Eileen</option>
  <option value="3">Rafael</option>
</select>
```

#### time_zone_options_for_select

Devuelve una cadena de etiquetas de opciones para prácticamente cualquier zona horaria del mundo.

#### time_zone_select

Devuelve las etiquetas de selección y opción para el objeto y método dados, usando `time_zone_options_for_select` para generar la lista de etiquetas de opción.

```ruby
time_zone_select("user", "time_zone")
```

#### date_field

Devuelve una etiqueta de entrada del tipo "fecha" adaptada para acceder a un atributo especificado.


```ruby
date_field("user", "dob")
```

### FormTagHelper

Proporciona varios métodos para crear etiquetas de formulario que no tienen como ámbito los objetos del modelo. En su lugar, proporciona los nombres y valores manualmente.

#### check_box_tag

Crea una etiqueta de entrada de formulario de casilla de verificación.

```ruby
check_box_tag 'accept'
# => <input id="accept" name="accept" type="checkbox" value="1" />
```

#### field_set_tag

Crea un conjunto de campos para agrupar elementos de formulario HTML.

```html+erb
<%= field_set_tag do %>
  <p><%= text_field_tag 'name' %></p>
<% end %>
# => <fieldset><p><input id="name" name="name" type="text" /></p></fieldset>
```

#### file_field_tag

Crea un campo de carga de archivos.

```html+erb
<%= form_with url: new_account_avatar_path(@account), multipart: true do %>
  <label for="file">Avatar:</label> <%= file_field_tag 'avatar' %>
  <%= submit_tag %>
<% end %>
```

Salida de ejemplo:

```ruby
file_field_tag 'attachment'
# => <input id="attachment" name="attachment" type="file" />
```

#### hidden_field_tag

Crea un campo de entrada de formulario oculto que se utiliza para transmitir datos que se perderían debido a la falta de estado de HTTP o datos que deberían ocultarse al usuario.

```ruby
hidden_field_tag 'token', 'VUBJKB23UIVI1UU1VOBVI@'
# => <input id="token" name="token" type="hidden" value="VUBJKB23UIVI1UU1VOBVI@" />
```

#### image_submit_tag

Muestra una imagen que, al hacer clic, enviará el formulario.

```ruby
image_submit_tag("login.png")
# => <input src="/images/login.png" type="image" />
```

#### label_tag

Crea un campo de etiqueta.

```ruby
label_tag 'name'
# => <label for="name">Name</label>
```

#### password_field_tag

Crea un campo de contraseña, un campo de texto enmascarado que ocultará la entrada de los usuarios detrás de un carácter de máscara.

```ruby
password_field_tag 'pass'
# => <input id="pass" name="pass" type="password" />
```

#### radio_button_tag

Crea un botón de radio; utilice grupos de botones de opción con el mismo nombre para permitir a los usuarios seleccionar entre un grupo de opciones.

```ruby
radio_button_tag 'favorite_color', 'maroon'
# => <input id="favorite_color_maroon" name="favorite_color" type="radio" value="maroon" />
```

#### select_tag

Crea un cuadro de selección desplegable.

```ruby
select_tag "people", "<option>David</option>"
# => <select id="people" name="people"><option>David</option></select>
```

#### submit_tag

Crea un botón de envío con el texto proporcionado como título.

```ruby
submit_tag "Publish this article"
# => <input name="commit" type="submit" value="Publish this article" />
```

#### text_area_tag

Crea un área de entrada de texto; use un área de texto para entradas de texto más largas, como publicaciones de blog o descripciones.

```ruby
text_area_tag 'article'
# => <textarea id="article" name="article"></textarea>
```

#### text_field_tag

Crea un campo de texto estándar; use estos campos de texto para ingresar fragmentos de texto más pequeños, como un nombre de usuario o una consulta de búsqueda.

```ruby
text_field_tag 'name'
# => <input id="name" name="name" type="text" />
```

#### email_field_tag

Crea un campo de entrada estándar de tipo de correo electrónico.

```ruby
email_field_tag 'email'
# => <input id="email" name="email" type="email" />
```

#### url_field_tag

Crea un campo de entrada estándar de tipo URL.

```ruby
url_field_tag 'url'
# => <input id="url" name="url" type="url" />
```

#### date_field_tag

Crea un campo de entrada estándar de tipo de fecha.

```ruby
date_field_tag "dob"
# => <input id="dob" name="dob" type="date" />
```

### JavaScriptHelper

Proporciona funcionalidad para trabajar con JavaScript en sus vistas.

#### escape_javascript

Escape de devoluciones de operadores y comillas simples y dobles para segmentos de JavaScript.

#### javascript_tag

Devuelve una etiqueta de JavaScript que envuelve el c proporcionado

```ruby
javascript_tag "alert('All is good')"
```

```html
<script>
//<![CDATA[
alert('All is good')
//]]>
</script>
```

### NumberHelper

Proporciona métodos para convertir números en cadenas formateadas. Se proporcionan métodos para números de teléfono, moneda, porcentaje, precisión, notación posicional y tamaño de archivo.

#### number_to_currency

Formatea un número en una cadena de moneda (e.g., $13.65).

```ruby
number_to_currency(1234567890.50) # => $1,234,567,890.50
```

#### number_to_human_size

Formatea los bytes de tamaño en una representación más comprensible; útil para informar los tamaños de archivo a los usuarios.

```ruby
number_to_human_size(1234)          # => 1.2 KB
number_to_human_size(1234567)       # => 1.2 MB
```

#### number_to_percentage

Formatea un número como una cadena de porcentaje.

```ruby
number_to_percentage(100, precision: 0)        # => 100%
```

#### number_to_phone

Formatea un número en un número de teléfono (EE. UU. Por defecto).

```ruby
number_to_phone(1235551234) # => 123-555-1234
```

#### number_with_delimiter

Formatea un número con miles agrupados usando un delimitador.

```ruby
number_with_delimiter(12345678) # => 12,345,678
```

#### number_with_precision

Formatea un número con el nivel especificado de `precision`, que por defecto es 3.

```ruby
number_with_precision(111.2345)     # => 111.235
number_with_precision(111.2345, precision: 2)  # => 111.23
```

### SanitizeHelper

El módulo SanitizeHelper proporciona un conjunto de métodos para eliminar texto de elementos HTML no deseados.

#### sanitize

Este asistente de desinfección codificará HTML todas las etiquetas y eliminará todos los atributos que no están específicamente permitidos.

```ruby
sanitize @article.body
```

Si se pasan las opciones `:sttributes` o `:tags`, solo se permiten los atributos y etiquetas mencionados y nada más.

```ruby
sanitize @article.body, tags: %w(table tr td), attributes: %w(id class style)
```

Para cambiar los valores predeterminados para múltiples usos, por ejemplo, agregar etiquetas de tabla a los valores predeterminados:

```ruby
class Application < Rails::Application
  config.action_view.sanitized_allowed_tags = 'table', 'tr', 'td'
end
```

#### sanitize_css(style)

Desinfecta un bloque de código CSS.

#### strip_links(html)
Elimina todas las etiquetas de enlace del texto dejando solo el texto del enlace.

```ruby
strip_links('<a href="https://rubyonrails.org">Ruby on Rails</a>')
# => Ruby on Rails
```

```ruby
strip_links('emails to <a href="mailto:me@email.com">me@email.com</a>.')
# => emails to me@email.com.
```

```ruby
strip_links('Blog: <a href="http://myblog.com/">Visit</a>.')
# => Blog: Visit.
```

#### strip_tags(html)

Elimina todas las etiquetas HTML del html, incluidos los comentarios.
Esta funcionalidad está impulsada por la gema rails-html-sanitizer.

```ruby
strip_tags("Strip <i>these</i> tags!")
# => Strip these tags!
```

```ruby
strip_tags("<b>Bold</b> no more!  <a href='more.html'>See more</a>")
# => Bold no more!  See more
```

NB: La salida aún puede contener caracteres '<', '>', '&' sin escape y confundir a los navegadores.

### UrlHelper

Proporciona métodos para crear enlaces y obtener URL que dependen del subsistema de enrutamiento.

#### url_for

Devuelve la URL del conjunto de `options` proporcionadas.

##### Examples

```ruby
url_for @profile
# => /profiles/1

url_for [ @hotel, @booking, page: 2, line: 3 ]
# => /hotels/1/bookings/1?line=3&page=2
```

#### link_to

LinVínculos a una URL derivada de `url_for` bajo el capó. Principalmente acostumbrado a
   crear enlaces de recursos RESTful, que para este ejemplo, se reduce a
   al pasar modelos a `link_to`.

**Examples**

```ruby
link_to "Profile", @profile
# => <a href="/profiles/1">Profile</a>
```

También puede usar un bloque si su objetivo de enlace no puede caber en el parámetro de nombre. Ejemplo de ERB:

```html+erb
<%= link_to @profile do %>
  <strong><%= @profile.name %></strong> -- <span>Check it out!</span>
<% end %>
```

saldría:

```html
<a href="/profiles/1">
  <strong>David</strong> -- <span>Check it out!</span>
</a>
```

SVeer [the API Doc for more info](https://api.rubyonrails.org/v6.0/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to)

#### button_to

Genera un formulario que se envía a la URL pasada. El formulario tiene un botón de enviar
con el valor del `name`.

##### Examples

```html+erb
<%= button_to "Sign in", sign_in_path %>
```

aproximadamente generaría algo como:

```html
<form method="post" action="/sessions" class="button_to">
  <input type="submit" value="Sign in" />
</form>
```

Veer [the API Doc for more info](https://api.rubyonrails.org/v6.0/classes/ActionView/Helpers/UrlHelper.html#method-i-button_to)

### CsrfHelper

Devuelve las metaetiquetas "csrf-param" y "csrf-token" con el nombre del sitio cruzado
solicitar el parámetro de protección de falsificación y el token, respectivamente.

```html
<%= csrf_meta_tags %>
```

NOTE: Los formularios normales generan campos ocultos, por lo que no utilizan estas etiquetas. Más
los detalles se pueden encontrar en el [Rails Security Guide](security.html#cross-site-request-forgery-csrf).

Localized Views
---------------

Action View tiene la capacidad de representar diferentes plantillas dependiendo de la configuración regional actual.

Por ejemplo, suponga que tiene un `ArticlesController` con una acción show. Por defecto, al llamar a esta acción se mostrará `app/views/articles/show.html.erb`. Pero si configura `I18n.locale = :de`, entonces se renderizará `app/views/articles/show.de.html.erb`. Si la plantilla localizada no está presente, se utilizará la versión no decorada. Esto significa que no es necesario que proporcione vistas localizadas para todos los casos, pero se preferirán y se usarán si están disponibles.

Puede usar la misma técnica para localizar los archivos de rescate en su directorio público. Por ejemplo, configurar `I18n.locale = :de` y crear `public/500.de.html` y `public/404.de.html` le permitiría tener páginas de rescate localizadas.

Como Rails no restringe los símbolos que usa para configurar I18n.locale, puede aprovechar este sistema para mostrar contenido diferente dependiendo de lo que desee. Por ejemplo, suponga que tiene algunos usuarios "expertos" que deberían ver páginas diferentes de los usuarios "normales". Puede agregar lo siguiente a `app/controllers/application.rb`:

```ruby
before_action :set_expert_locale

def set_expert_locale
  I18n.locale = :expert if current_user.expert?
end
```

Luego, podría crear vistas especiales como `app/views/articles/show.expert.html.erb` que solo se mostrarían a usuarios expertos.

Puede leer más sobre la API de internacionalización de Rails (I18n) [here](i18n.html).


































































































































































































































































































































































































































































