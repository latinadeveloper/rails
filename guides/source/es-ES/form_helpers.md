**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Ayudantes de Formulario de Action
=================================

Los formularios en las aplicaciones web son una interfaz esencial para la entrada del usuario. Sin embargo, escribir y mantener el marcado de formularios puede volverse tedioso rápidamente debido a la necesidad de manejar la denominación del control de formularios y sus numerosos atributos. Rails elimina esta complejidad al proporcionar ayudantes de vista para generar marcas de formularios. Sin embargo, dado que estos ayudantes tienen diferentes casos de uso, los desarrolladores deben conocer las diferencias entre los métodos auxiliares antes de ponerlos en práctica.

Después de leer esta guía, sabrá:

* Cómo crear formularios de búsqueda y formularios genéricos similares que no representen ningún modelo específico en su aplicación.
* Cómo hacer formularios centrados en modelos para crear y editar registros de bases de datos específicos.
* Cómo generar cuadros de selección a partir de múltiples tipos de datos.
* Qué fecha y hora proporciona Helpers Rails.
* Qué hace que un formulario de carga de archivos sea diferente.
* Cómo publicar formularios en recursos externos y especificar la configuración de un `autenticity_token`.
* Cómo construir formas complejas.

--------------------------------------------------------------------------------

NOTE: Este guía no pretende ser la documentación completa de los ayudantes de formulario disponibles y sus argumentos. Visite [the Rails API documentation](https://api.rubyonrails.org/) para obtener una referencia completa.

Dealing with Basic Forms
------------------------

The main form helper is `form_with`.

```erb
<%= form_with do %>
  Form contents
<% end %>
```

Cuando se llama sin argumentos como este, crea una etiqueta de formulario que, cuando se envía, el POST en la página actual. Por ejemplo, asumiendo que la página actual es una página de inicio, el HTML generado se verá así:

```html
<form accept-charset="UTF-8" action="/" data-remote="true" method="post">
  <input name="authenticity_token" type="hidden" value="J7CBxfHalt49OSHp27hblqK20c9PgwJ108nDHX/8Cts=" />
  Form contents
</form>
```

Notarás que el HTML contiene un elemento `input` con el tipo `hidden`. Esta "entrada" es importante, porque los formularios que no son GET no se pueden enviar correctamente sin ella.
El elemento de entrada oculto con el nombre `autenticity_token` es una característica de seguridad de Rails llamada **protección contra falsificación de solicitudes entre sitios**, y los ayudantes de formulario lo generan para cada formulario que no sea GET (siempre que esta característica de seguridad esté habilitada). Puede leer más sobre esto en la guía [Securing Rails Applications](security.html#cross-site-request-forgery-csrf).

### A Generic Search Form

Uno de los formularios más básicos que ve en la web es un formulario de búsqueda. Este formulario contiene:

* un elemento de formulario con el método "GET",
* una etiqueta para la entrada,
* un elemento de entrada de texto, y
* un elemento de envío.

To create this form you will use `form_with`, `label_tag`, `text_field_tag`, and `submit_tag`, respectively. Like this:

```erb
<%= form_with url: "/search", method: :get do |form| %>
  <%= form.label :q, "Search for:" %>
  <%= form.text_field :q %>
  <%= form.submit "Search" %>
<% end %>
```

Esto generará el siguiente HTML:

```html
<form accept-charset="UTF-8" action="/search" data-remote="true" method="get">
  <label for="q">Search for:</label>
  <input id="q" name="q" type="text" />
  <input name="commit" type="submit" value="Search" data-disable-with="Search" />
</form>
```

TIP: Pasar `url: my_specified_path` a `form_with` le indica al formulario dónde realizar la solicitud. Sin embargo, como se explica a continuación, también puede pasar objetos ActiveRecord al formulario.

TIP: Para cada entrada de formulario, se genera un atributo de ID a partir de su nombre (`"q"` en el ejemplo anterior). Estos ID pueden ser muy útiles para el estilo CSS o la manipulación de controles de formulario con JavaScript.

IMPORTANT: Utilice "GET" como método para los formularios de búsqueda. Esto permite a los usuarios marcar una búsqueda específica y volver a ella. De forma más general, Rails le anima a utilizar el verbo HTTP correcto para una acción.

### Helpers for Generating Form Elements

Rails proporciona una serie de ayudantes para generar elementos de formulario como
casillas de verificación, campos de texto y botones de opción. Estos ayudantes básicos, con nombres
terminando en `_tag` (como `text_field_tag` y `check_box_tag`), genere solo una
elemento único `<input>`. El primer parámetro de estos es siempre el nombre del
entrada. Cuando se envía el formulario, el nombre se pasará junto con el formulario
datos, y se dirigirá a los `params` en el controlador con el
valor ingresado por el usuario para ese campo. Por ejemplo, si el formulario contiene
`<%= text_field_tag(:query) %>`, entonces podrá obtener el valor de este
campo en el controlador con `params[:query]`.

Al nombrar entradas, Rails utiliza ciertas convenciones que hacen posible enviar parámetros con valores no escalares como matrices o hashes, que también serán accesibles en `params`. Puede leer más sobre ellos en el capítulo [Comprensión de las convenciones de nomenclatura de parámetros] (# comprensión-de-convenciones de nomenclatura de parámetros) de esta guía. Para obtener detalles sobre el uso preciso de estos ayudantes, consulte la [API documentation](https://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html).

#### Checkboxes

Las casillas de verificación son controles de formulario que brindan al usuario un conjunto de opciones que pueden habilitar o deshabilitar:

```erb
<%= check_box_tag(:pet_dog) %>
<%= label_tag(:pet_dog, "I own a dog") %>
<%= check_box_tag(:pet_cat) %>
<%= label_tag(:pet_cat, "I own a cat") %>
```

Output:

```html
<input id="age_child" name="age" type="radio" value="child" />
<label for="age_child">I am younger than 21</label>
<input id="age_adult" name="age" type="radio" value="adult" />
<label for="age_adult">I am over 21</label>
```

Al igual que con `check_box_tag`, el segundo parámetro de`radio_button_tag` es el valor de la entrada. Debido a que estos dos botones de radio comparten el mismo nombre (`age`), el usuario solo podrá seleccionar uno de ellos, y `params[:age] `contendrá `"niño"` o `"adulto"`.

NOTE: Utilice siempre etiquetas para las casillas de verificación y los botones de opción. Asocian texto con una opción específica y,
expandiendo la región en la que se puede hacer clic,
les facilita a los usuarios hacer clic en las entradas.


#### Radio Buttons

Los botones de opción, aunque similares a las casillas de verificación, son controles que especifican un conjunto de opciones en las que se excluyen mutuamente (es decir, el usuario solo puede elegir una):

```erb
<%= radio_button_tag(:age, "child") %>
<%= label_tag(:age_child, "I am younger than 21") %>
<%= radio_button_tag(:age, "adult") %>
<%= label_tag(:age_adult, "I am over 21") %>
```

Salida:

```html
<input id="age_child" name="age" type="radio" value="child" />
<label for="age_child">I am younger than 21</label>
<input id="age_adult" name="age" type="radio" value="adult" />
<label for="age_adult">I am over 21</label>
```

Al igual que con `check_box_tag`, el segundo parámetro de `radio_button_tag` es el valor de la entrada. Debido a que estos dos botones de radio comparten el mismo nombre (`age`), el usuario solo podrá seleccionar uno de ellos, y `params [:age] `contendrá `"child"` o `"adulto"`.

NOTE: Utilice siempre etiquetas para las casillas de verificación y los botones de opción. Asocian texto con una opción específica y,
expandiendo la región en la que se puede hacer clic,
facilitar a los usuarios hacer clic en las entradas.

### Other Helpers of Interest

Otros controles de formulario que vale la pena mencionar son áreas de texto, campos de contraseña,
campos ocultos, campos de búsqueda, campos de teléfono, campos de fecha, campos de hora,
campos de color, campos locales de fecha y hora, campos de mes, campos de semana,
Campos de URL, campos de correo electrónico, campos de números y campos de rango:

```erb
<%= text_area_tag(:message, "Hi, nice site", size: "24x6") %>
<%= password_field_tag(:password) %>
<%= hidden_field_tag(:parent_id, "5") %>
<%= search_field(:user, :name) %>
<%= telephone_field(:user, :phone) %>
<%= date_field(:user, :born_on) %>
<%= datetime_local_field(:user, :graduation_day) %>
<%= month_field(:user, :birthday_month) %>
<%= week_field(:user, :birthday_week) %>
<%= url_field(:user, :homepage) %>
<%= email_field(:user, :address) %>
<%= color_field(:user, :favorite_color) %>
<%= time_field(:task, :started_at) %>
<%= number_field(:product, :price, in: 1.0..20.0, step: 0.5) %>
<%= range_field(:product, :discount, in: 1..100) %>
```

Salida

```html
<textarea id="message" name="message" cols="24" rows="6">Hi, nice site</textarea>
<input id="password" name="password" type="password" />
<input id="parent_id" name="parent_id" type="hidden" value="5" />
<input id="user_name" name="user[name]" type="search" />
<input id="user_phone" name="user[phone]" type="tel" />
<input id="user_born_on" name="user[born_on]" type="date" />
<input id="user_graduation_day" name="user[graduation_day]" type="datetime-local" />
<input id="user_birthday_month" name="user[birthday_month]" type="month" />
<input id="user_birthday_week" name="user[birthday_week]" type="week" />
<input id="user_homepage" name="user[homepage]" type="url" />
<input id="user_address" name="user[address]" type="email" />
<input id="user_favorite_color" name="user[favorite_color]" type="color" value="#000000" />
<input id="task_started_at" name="task[started_at]" type="time" />
<input id="product_price" max="20.0" min="1.0" name="product[price]" step="0.5" type="number" />
<input id="product_discount" max="100" min="1" name="product[discount]" type="range" />
```

Las entradas ocultas no se muestran al usuario, sino que contienen datos como cualquier entrada textual. Los valores dentro de ellos se pueden cambiar con JavaScript.

IMPORTANT: La búsqueda, teléfono, fecha, hora, color, fecha y hora, fecha y hora-local,
Las entradas de mes, semana, URL, correo electrónico, número y rango son controles HTML5.
Si necesita que su aplicación tenga una experiencia constante en navegadores más antiguos,
necesitará un polyfill HTML5 (proporcionado por CSS y / o JavaScript).
Definitivamente hay [no shortage of solutions for this](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills), aunque una herramienta popular en este momento es
[Modernizr](https://modernizr.com/), que proporciona una forma sencilla de agregar funcionalidad basada en la presencia de
características HTML5 detectadas.

TIP: Si está utilizando campos de entrada de contraseña (para cualquier propósito), es posible que desee configurar su aplicación para evitar que se registren esos parámetros. Puede obtener más información sobre esto en la guía [Protección de aplicaciones de Rails] (security.html # logging).

Dealing with Model Objects
--------------------------

### Model Object Helpers

Una tarea particularmente común para un formulario es editar o crear un objeto modelo. Si los ayudantes `* _tag` ciertamente se pueden usar para esta tarea, son algo detallados, ya que cada etiqueta debe asegurarse de que se use el nombre de parámetro correcto y establecer el valor predeterminado de la entrada de manera adecuada. Rails proporciona ayudantes adaptados a esta tarea. Estos ayudantes carecen del sufijo `_tag`, por ejemplo, `text_field`, `text_area`.

```erb
<%= text_field(:person, :name) %>
```

producirá una salida similar a

```erb
<input id="person_name" name="person[name]" type="text" value="Henry" />
```

Al enviar el formulario, el valor ingresado por el usuario se almacenará en `params[:person][:name]`.

WARNING: Debe pasar el nombre de una variable de instancia, es decir, `:person` o` "person"`, no una instancia real de su objeto modelo.

Rails proporciona ayudantes para mostrar los errores de validación asociados con un objeto de modelo. Estos se tratan en detalle en la guía [Active Record Validations](active_record_validations.html#displaying-validation-errors-in-views).

### Binding a Form to an Object

Si bien esto es un aumento en la comodidad, está lejos de ser perfecto. Si `Person` tiene muchos atributos para editar, estaríamos repitiendo el nombre del objeto editado muchas veces. Lo que queremos hacer es vincular de alguna manera un formulario a un objeto modelo, que es exactamente lo que hace `form_with` con `:model`.

Supongamos que tenemos un controlador para tratar los artículos `app/controllers/articles_controller.rb`:

```ruby
def new
  @article = Article.new
end
```

La vista correspondiente  `app/views/articles/new.html.erb` que usa `form_with` se ve así:

```erb
<%= form_with model: @article, class: "nifty_form" do |form| %>
  <%= form.text_field :title %>
  <%= form.text_area :body, size: "60x12" %>
  <%= form.submit "Create" %>
<% end %>
```

Hay algunas cosas a tener en cuenta aquí:

* `@article` es el objeto real que se está editando.
* Hay un solo hash de opciones. Las opciones HTML (excepto `id` y `class`) se pasan en el hash `:html`. También puede proporcionar una opción `:namespace` para su formulario para garantizar la unicidad de los atributos de identificación en los elementos del formulario. El atributo de alcance tendrá el prefijo de subrayado en la identificación HTML generada.
* El método `form_with` produce un objeto ** constructor de formularios ** (la variable` form`).
* Si desea dirigir su solicitud de formulario a una URL en particular, debería usar `form_with url: my_nifty_url_path` en su lugar. Para ver opciones más detalladas sobre lo que acepta `form_with`, asegúrese de  [check out the API documentation](https://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_with).
* Los métodos para crear controles de formulario se llaman ** en ** el objeto del generador de formularios `form`.

El HTML resultante es:

```html
<form class="nifty_form" action="/articles" accept-charset="UTF-8" data-remote="true" method="post">
  <input type="hidden" name="authenticity_token" value="NRkFyRWxdYNfUg7vYxLOp2SLf93lvnl+QwDWorR42Dp6yZXPhHEb6arhDOIWcqGit8jfnrPwL781/xlrzj63TA==" />
  <input type="text" name="article[title]" id="article_title" />
  <textarea name="article[body]" id="article_body" cols="60" rows="12"></textarea>
  <input type="submit" name="commit" value="Create" data-disable-with="Create" />
</form>
```

El objeto pasado como `:model` en `form_with` controla la clave utilizada en `params` para acceder a los valores del formulario. Aquí el nombre es `article`, por lo que todas las entradas tienen el formato `article[attribute_name]`. En consecuencia, en la acción `create` `params[:article]` habrá un hash con las claves` :title` y `:body`. Puede leer más sobre la importancia de los nombres de entrada en el capítulo [Understanding Parameter Naming Conventions](#understanding-parameter-naming-conventions) de este guía.

TIP: convencionalmente, sus entradas reflejarán los atributos del modelo. Sin embargo, ¡no tienen que hacerlo! Si hay otra información que necesita, puede incluirla en su formulario al igual que con los atributos y acceder a ella a través de `params[:article][:my_nifty_non_attribute_input]`.

Los métodos auxiliares llamados en el constructor de formularios son idénticos a los ayudantes de objetos de modelo, excepto que no es necesario especificar qué objeto se está editando, ya que este ya está gestionado por el constructor de formularios.

Puede crear un enlace similar sin crear etiquetas `<form>` con el ayudante `fields_for`. Esto es útil para editar objetos de modelo adicionales con el mismo formulario. Por ejemplo, si tuvieras un modelo `Person` con un modelo `ContactDetail` asociado, podrías crear un formulario para crear ambos así:

```erb
<%= form_with model: @person do |person_form| %>
  <%= person_form.text_field :name %>
  <%= fields_for :contact_detail, @person.contact_detail do |contact_detail_form| %>
    <%= contact_detail_form.text_field :phone_number %>
  <% end %>
<% end %>
```

que produce la siguiente salida:


```html
<form action="/people" accept-charset="UTF-8" data-remote="true" method="post">
  <input type="hidden" name="authenticity_token" value="bL13x72pldyDD8bgtkjKQakJCpd4A8JdXGbfksxBDHdf1uC0kCMqe2tvVdUYfidJt0fj3ihC4NxiVHv8GVYxJA==" />
  <input type="text" name="person[name]" id="person_name" />
  <input type="text" name="contact_detail[phone_number]" id="contact_detail_phone_number" />
</form>
```

El objeto generado por `fields_for` es un constructor de formularios como el generado por `form_with`.

### Relying on Record Identification

El modelo de artículo está disponible directamente para los usuarios de la aplicación, por lo que, siguiendo las mejores prácticas para desarrollar con Rails, debe declararlo **a resource**:

```ruby
resources :articles
```

SUGERENCIA: Declarar un recurso tiene varios efectos secundarios. Ver [Rails Routing from the Outside In](routing.html#resource-routing-the-rails-default) guía para obtener más información sobre la configuración y el uso de recursos.

Cuando se trata de recursos RESTful, las llamadas a `form_with` pueden volverse significativamente más fáciles si confía en la **identificación de registro**. En resumen, puede pasar la instancia del modelo y hacer que Rails descubra el nombre del modelo y el resto:

```ruby
## Creating a new article
# long-style:
form_with(model: @article, url: articles_path)
# short-style:
form_with(model: @article)

## Editing an existing article
# long-style:
form_with(model: @article, url: article_path(@article), method: "patch")
# short-style:
form_with(model: @article)
```

Observe cómo la invocación de estilo corto `form_with` es convenientemente la misma, independientemente de que el registro sea nuevo o existente. La identificación del registro es lo suficientemente inteligente como para averiguar si el registro es nuevo preguntando `record.persisted?`. También selecciona la ruta correcta a la que enviar y el nombre según la clase del objeto.

WARNING: Cuando usa STI (herencia de tabla única) con sus modelos, no puede confiar en la identificación de registros en una subclase si solo su clase principal se declara un recurso. Tendrá que especificar `:url` y `:scope` (el nombre del modelo) explícitamente.

#### Dealing with Namespaces

Si ha creado rutas con espacios de nombres, `form_with` también tiene una ingeniosa abreviatura para eso. Si su aplicación tiene un espacio de nombres de administrador, entonces

```ruby
form_with model: [:admin, @article]
```

creará un formulario que se envía al `ArticlesController` dentro del espacio de nombres de administrador (enviándolo a `admin_article_path(@article) `en el caso de una actualización). Si tiene varios niveles de espacio de nombres, la sintaxis es similar:

```ruby
form_with model: [:admin, :management, @article]
```

Para obtener más información sobre el sistema de enrutamiento de Rails y las convenciones asociadas, consulte el 
guía [Rails Routing from the Outside In](routing.html)


### How do forms with PATCH, PUT, or DELETE methods work?

El marco de Rails fomenta el diseño RESTful de sus aplicaciones, lo que significa que estará realizando muchas solicitudes de "PATCH", "PUT" y "DELETE" (además de "GET" y "POST"). Sin embargo, la mayoría de los navegadores _no admiten_ métodos distintos de "GET" y "POST" cuando se trata de enviar formularios.

Rails soluciona este problema emulando otros métodos sobre POST con una entrada oculta llamada `" _method "`, que está configurada para reflejar el método deseado:


```ruby
form_with(url: search_path, method: "patch")
```

Output:

```html
<form accept-charset="UTF-8" action="/search" data-remote="true" method="post">
  <input name="_method" type="hidden" value="patch" />
  <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c432144748a6d4b7a7ddf2b71" />
  ...
</form>
```

Al analizar los datos POSTED, Rails tendrá en cuenta el parámetro especial `_method` y actuará como si el método HTTP fuera el especificado dentro de él (" PATCH "en este ejemplo).

IMPORTANT: Todos los formularios que usan `form_with` implementan `remote: true` de forma predeterminada. Estos formularios enviarán datos mediante una solicitud XHR (Ajax). Para deshabilitar esto, incluya `local: true`. Para profundizar más, consulte la guía [Working with JavaScript in Rails](working_with_javascript_in_rails.html#remote-elements)

Making Select Boxes with Ease
-----------------------------

Los cuadros de selección en HTML requieren una cantidad significativa de marcado (un elemento `OPTION` para cada opción para elegir), por lo tanto, tiene más sentido que se generen dinámicamente.

Así es como podría verse el marcado:

```html
<select name="city_id" id="city_id">
  <option value="1">Lisbon</option>
  <option value="2">Madrid</option>
  <option value="3">Berlin</option>
</select>
```

Aquí tienes una lista de ciudades cuyos nombres se presentan al usuario. Internamente, la aplicación solo quiere manejar sus ID, por lo que se utilizan como atributo de valor de las opciones. Veamos cómo puede ayudar Rails aquí.

### The Select and Option Tags

El ayudante más genérico es `select_tag`, que, como su nombre lo indica, simplemente genera la etiqueta `SELECT` que encapsula una cadena de opciones:

```erb
<%= select_tag(:city_id, raw('<option value="1">Lisbon</option><option value="2">Madrid</option><option value="3">Berlin</option>')) %>
```

Este es un comienzo, pero no crea dinámicamente las etiquetas de opción. Puede generar etiquetas de opciones con el ayudante `options_for_select`:

```html+erb
<%= options_for_select([['Lisbon', 1], ['Madrid', 2], ['Berlin', 3]]) %>
```

Salida:

```html
<option value="1">Lisbon</option>
<option value="2">Madrid</option>
<option value="3">Berlin</option>
```

El primer argumento de `options_for_select` es una matriz anidada donde cada elemento tiene dos elementos: texto de la opción (nombre de la ciudad) y valor de la opción (id de la ciudad). El valor de la opción es lo que se enviará a su controlador. A menudo, esta será la identificación de un objeto de base de datos correspondiente, pero no tiene por qué ser así.

Sabiendo esto, puede combinar `select_tag` y `options_for_select` para lograr el marcado completo deseado:


```erb
<%= select_tag(:city_id, options_for_select(...)) %>
```

`options_for_select` te permite preseleccionar una opción pasando su valor.

```html+erb
<%= options_for_select([['Lisbon', 1], ['Madrid', 2], ['Berlin', 3]], 2) %>
```

Salida:

```html
<option value="1">Lisbon</option>
<option value="2" selected="selected">Madrid</option>
<option value="3">Berlin</option>
```

Siempre que Rails vea que el valor interno de una opción que se está generando coincide con este valor, agregará el atributo `selected` a esa opción.

Puede agregar atributos arbitrarios a las opciones usando hashes:

```html+erb
<%= options_for_select(
  [
    ['Lisbon', 1, { 'data-size' => '2.8 million' }],
    ['Madrid', 2, { 'data-size' => '3.2 million' }],
    ['Berlin', 3, { 'data-size' => '3.4 million' }]
  ], 2
) %>
```

Salida:

```html
<option value="1" data-size="2.8 million">Lisbon</option>
<option value="2" selected="selected" data-size="3.2 million">Madrid</option>
<option value="3" data-size="3.4 million">Berlin</option>
```

### Select Boxes for Dealing with Model Objects

En la mayoría de los casos, los controles de formulario estarán vinculados a un modelo específico y, como es de esperar, Rails proporciona ayudantes diseñados para ese propósito. De acuerdo con otros ayudantes de formulario, cuando se trata de un objeto modelo, elimine el sufijo `_tag` de `select_tag`:

Si su controlador ha definido `@person` y el city_id de esa persona es 2:

```ruby
@person = Person.new(city_id: 2)
```

```erb
<%= select(:person, :city_id, [['Lisbon', 1], ['Madrid', 2], ['Berlin', 3]]) %>
```

producirá una salida similar a

```html
<select name="person[city_id]" id="person_city_id">
  <option value="1">Lisbon</option>
  <option value="2" selected="selected">Madrid</option>
  <option value="3">Berlin</option>
</select>
```

Observe que el tercer parámetro, la matriz de opciones, es el mismo tipo de argumento que pasa a `options_for_select`. Una ventaja aquí es que no tienes que preocuparte por preseleccionar la ciudad correcta si el usuario ya tiene una; Rails hará esto por ti leyendo el atributo `@person.city_id`.

Al igual que con otros ayudantes, si tuviera que usar el ayudante `select` en un generador de formularios en el ámbito del objeto `@person`, la sintaxis sería:

```erb
<%= form_with model: @person do |person_form| %>
  <%= person_form.select(:city_id, [['Lisbon', 1], ['Madrid', 2], ['Berlin', 3]]) %>
<% end %>
```
También puede pasar un bloque al ayudante `select`:

```erb
<%= form_with model: @person do |person_form| %>
  <%= person_form.select(:city_id) do %>
    <% [['Lisbon', 1], ['Madrid', 2], ['Berlin', 3]].each do |c| %>
      <%= content_tag(:option, c.first, value: c.last) %>
    <% end %>
  <% end %>
<% end %>
```

WARNING: Si está utilizando `select` o ayudantes similares para establecer una asociación `belongs_to`, debe pasar el nombre de la clave externa (en el ejemplo anterior, `city_id`), no el nombre de la asociación en sí.

WARNING: Cuando `:include_blank` o `:prompt` no están presentes, `:include_blank` se fuerza como verdadero si el atributo de selección `required` es verdadero, el `size` de visualización es uno y `multiple` no es verdadero.

### Option Tags from a Collection of Arbitrary Objects

Generar etiquetas de opciones con `options_for_select` requiere que crees una matriz que contenga el texto y el valor de cada opción. Pero, ¿qué pasaría si tuvieras un modelo `City` (quizás uno de Active Record) y quisieras generar etiquetas de opción a partir de una colección de esos objetos? Una solución sería hacer una matriz anidada iterando sobre ellos:

```erb
<% cities_array = City.all.map { |city| [city.name, city.id] } %>
<%= options_for_select(cities_array) %>
```

Esta es una solución perfectamente válida, pero Rails ofrece una alternativa menos detallada: `options_from_collection_for_select`. Este ayudante espera una colección de objetos arbitrarios y dos argumentos adicionales: los nombres de los métodos para leer la opción **valor** y **texto**, respectivamente:

```erb
<%= options_from_collection_for_select(City.all, :id, :name) %>
```

Como su nombre lo indica, esto solo genera etiquetas de opción. Para generar un cuadro de selección de trabajo, necesitaría usar `collection_select`:

```erb
<%= collection_select(:person, :city_id, City.all, :id, :name) %>
```

Al igual que con otros ayudantes, si tuviera que usar el ayudante `collection_select` en un generador de formularios en el ámbito del objeto `@person`, la sintaxis sería:

```erb
<%= form_with model: @person do |person_form| %>
  <%= person_form.collection_select(:city_id, City.all, :id, :name) %>
<% end %>
```

NOTE: Los pares pasados ​​a `options_for_select` deben tener el texto primero y el valor en segundo lugar, sin embargo, con `options_from_collection_for_select` debe tener el método value primero y el método text en segundo lugar.

### Time Zone and Country Select


Para aprovechar el soporte de zona horaria en Rails, debe preguntar a sus usuarios en qué zona horaria se encuentran. Hacerlo requeriría generar opciones de selección de una lista de [`ActiveSupport::TimeZone`](https://api.rubyonrails.org/classes/ActiveSupport/TimeZone.html) usando `collection_select`, pero simplemente puedes usar el ayudante `time_zone_select` que ya envuelve esto:

```erb
<%= time_zone_select(:person, :time_zone) %>
```

También hay un ayudante `time_zone_options_for_select` para una forma más manual (por lo tanto, más personalizable) de hacer esto. Lea la  [API documentation](https://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-time_zone_options_for_select) para conocer los posibles argumentos de estos dos métodos.

Rails _usado_ para tener un ayudante `country_select` para elegir países, pero esto se ha extraído al  [country_select plugin](https://github.com/stefanpenner/country_select).

Using Date and Time Form Helpers
--------------------------------

Puede optar por no utilizar los ayudantes de formulario que generan campos de entrada de fecha y hora HTML5 y utilizar los ayudantes de fecha y hora alternativos. Estos ayudantes de fecha y hora difieren de todos los demás ayudantes de formulario en dos aspectos importantes:

* Las fechas y horas no son representables por un solo elemento de entrada. En su lugar, tiene varios, uno para cada componente (año, mes, día, etc.) y, por lo tanto, no hay un valor único en su hash `params` con su fecha u hora.
* Otros ayudantes usan el sufijo `_tag` para indicar si un ayudante es un ayudante básico o uno que opera en objetos de modelo. Con fechas y horas, `select_date`, `select_time` y `select_datetime` son los ayudantes básicos,` date_select`, `time_select` y` datetime_select` son los ayudantes de objetos de modelo equivalentes.

Ambas familias de ayudantes crearán una serie de cuadros de selección para los diferentes componentes (año, mes, día, etc.).

### Barebones Helpers

La familia de ayudantes `select_ *` toma como primer argumento una instancia de `Date`,` Time` o `DateTime` que se utiliza como el valor seleccionado actualmente. Puede omitir este parámetro, en cuyo caso se utiliza la fecha actual. Por ejemplo:

```erb
<%= select_date Date.today, prefix: :start_date %>
```

salidas (con valores de opción reales omitidos para brevedad)

```html
<select id="start_date_year" name="start_date[year]">
</select>
<select id="start_date_month" name="start_date[month]">
</select>
<select id="start_date_day" name="start_date[day]">
</select>
```

Las entradas anteriores darían como resultado que `params[:start_date]` sea un hash con claves `:yaer`,`:month`, `:day`. Para obtener un objeto `Date`,` Time` o `DateTime` real, tendría que extraer estos valores y pasarlos al constructor apropiado, por ejemplo:

```ruby
Date.civil(params[:start_date][:year].to_i, params[:start_date][:month].to_i, params[:start_date][:day].to_i)
```

La opción `:prefix` es la clave utilizada para recuperar el hash de los componentes de la fecha del hash de `params`. Aquí se estableció en `start_date`, si se omite, se establecerá de forma predeterminada en` date`.

### Model Object Helpers

`select_date` no funciona bien con formularios que actualizan o crean objetos de Active Record ya que Active Record espera que cada elemento del hash de `params` corresponda a un atributo.
Los ayudantes de objetos de modelo para fechas y horas envían parámetros con nombres especiales; cuando Active Record ve parámetros con tales nombres, sabe que deben combinarse con los otros parámetros y entregarse a un constructor apropiado para el tipo de columna. Por ejemplo:

```erb
<%= date_select :person, :birth_date %>
```

salidas (con valores de opción reales omitidos para brevedad)

```html
<select id="person_birth_date_1i" name="person[birth_date(1i)]">
</select>
<select id="person_birth_date_2i" name="person[birth_date(2i)]">
</select>
<select id="person_birth_date_3i" name="person[birth_date(3i)]">
</select>
```

que da como resultado un hash `params` como

```ruby
{'person' => {'birth_date(1i)' => '2008', 'birth_date(2i)' => '11', 'birth_date(3i)' => '22'}}
```

Cuando esto se pasa a `Person.new` (o `update`), Active Record detecta que todos estos parámetros deben usarse para construir el atributo `birth_date` y usa la información con sufijos para determinar en qué orden se deben pasar estos parámetros funciones como `Date.civil`.


### Common Options

Ambas familias de ayudantes utilizan el mismo conjunto básico de funciones para generar las etiquetas de selección individuales y, por lo tanto, ambos aceptan en gran medida las mismas opciones. En particular, de forma predeterminada, Rails generará opciones de año 5 años a cada lado del año actual. Si este no es un rango apropiado, las opciones `: start_year` y`: end_year` lo anulan. Para obtener una lista exhaustiva de las opciones disponibles, consulte la [documentación de la API] (https://api.rubyonrails.org/classes/ActionView/Helpers/DateHelper.html).

Como regla general, debe usar `date_select` cuando trabaje con objetos de modelo y `select_date` en otros casos, como un formulario de búsqueda que filtra los resultados por fecha.


### Individual Components

Ocasionalmente, es necesario mostrar solo un componente de fecha, como un año o un mes. Rails proporciona una serie de ayudantes para esto, uno para cada componente `select_year`, `select_month`, `select_day`, `select_hour`, `select_minute`, `select_second`. Estos ayudantes son bastante sencillos. De forma predeterminada, generarán un campo de entrada con el nombre del componente de tiempo (por ejemplo, "año" para `select_year`," mes "para` select_month`, etc.) aunque esto puede anularse con la opción `: field_name`. La opción `: prefix` funciona de la misma manera que para` select_date` y `select_time` y tiene el mismo valor predeterminado.

El primer parámetro especifica qué valor debe seleccionarse y puede ser una instancia de una `Date`, `Time` o `DateTime`, en cuyo caso se extraerá el componente relevante o un valor numérico. Por ejemplo:

```erb
<%= select_year(2009) %>
<%= select_year(Time.new(2009)) %>
```

producirá la misma salida y el valor elegido por el usuario puede ser recuperado por `params[:date][:year]`.

Creating Checkboxes for Relations
------------------------------------------

Generar opciones de casillas de verificación que actúen adecuadamente con los modelos de Active Record
y los parámetros sólidos pueden ser una empresa confusa. Afortunadamente hay un ayudante
función para reducir significativamente el trabajo requerido.

Así es como puede verse el marcado:

```html
<input type="checkbox" value="1" checked="checked" name="person[city_ids][]" id="person_cities_1" />
<label for="person_cities_1">Pittsburgh</label>
<input type="checkbox" value="2" checked="checked" name="person[city_ids][]" id="person_cities_2"/>
<label for="person_cities_2">Madison</label>
<input type="checkbox" value="3" checked="checked" name="person[city_ids][]" id="person_cities_3"/>
<label for="person_cities_3">Santa Rosa</label>
```

Tiene una lista de ciudades cuyos nombres se muestran al usuario y asociados
casillas de verificación que contienen la identificación de cada entrada de la base de datos de ciudades.

### The Collection Check Boxes tag


The `collection_check_boxes` exists to help create these checkboxes with the
name for each input that maps to params that can be passed directly to model
relationships.

In this example we show the currently associated records in a collection:

```erb
<%= form_with model: @person do |person_form| %>
  <%= person_form.collection_check_boxes :city_ids, person.object.cities, :id, :name %>
<% end %>
```

Esto vuelve


```html
<input type="checkbox" value="1" checked="checked" name="person[city_ids][]" id="person_cities_1" checked="checked" />
<label for="person_cities_1">Pittsburgh</label>
<input type="checkbox" value="2" checked="checked" name="person[city_ids][]" id="person_cities_2" checked="checked" />
<label for="person_cities_2">Madison</label>
<input type="checkbox" value="3" checked="checked" name="person[city_ids][]" id="person_cities_3" checked="checked" />
<label for="person_cities_3">Santa Rosa</label>
```

También puede pasar un bloque para formatear el html como desee:

```erb
<%= form_with model: @person do |person_form| %>
  <%= person_form.collection_check_boxes :city_ids, City.all, :id, :name do |city| %>
    <div>
      <%= city.check_box %><%= city.label %>
    </div>
  <% end %>
<% end %>
```

Esto vuelve

```html
<div>
  <input type="checkbox" value="1" checked="checked" name="person[city_ids][]" id="person_cities_1" checked="checked" />
  <label for="person_cities_1">Pittsburgh</label>
</div>
<div>
  <input type="checkbox" value="2" checked="checked" name="person[city_ids][]" id="person_cities_2" checked="checked" />
  <label for="person_cities_2">Madison</label>
</div>
<div>
  <input type="checkbox" value="3" checked="checked" name="person[city_ids][]" id="person_cities_3" checked="checked" />
  <label for="person_cities_3">Santa Rosa</label>
</div>
```

La firma del método es

```erb
collection_check_boxes(object, method, collection, value_method, text_method, options = {}, html_options = {}, &block)
```

* `object` - el objeto de formulario, esto es innecesario si se usa el
  llamada al método `form.collection_check_boxes`
* `method` - el método de asociación para la relación del modelo
* `collection`: una matriz de objetos para mostrar en las casillas de verificación
* `value_method`: el método a llamar al completar el atributo value en
  la casilla de verificación
* `text` - el método para llamar al completar el texto de la etiqueta

Uploading Files
---------------

Una tarea común es cargar algún tipo de archivo, ya sea una imagen de una persona o un archivo CSV que contiene datos para procesar. Lo más importante para recordar con la carga de archivos es que el atributo enctype del formulario representado **debe** establecerse en "multipart / form-data". Si usa `form_with` con `:model`, esto se hace automáticamente. Si usa `form_with` sin `:model`, debe configurarlo usted mismo, según el siguiente ejemplo.

Los dos formularios siguientes cargan un archivo.

```erb
<%= form_with model: @person do |form| %>
  <%= form.file_field :picture %>
<% end %>

<%= form_with url: "/uploads", multipart: true do |form| %>
  <%= form.file_field 'picture' %>
<% end %>
```

Rails proporciona el par habitual de ayudantes: los barebones `file_field_tag` y el modelo orientado a `file_field`. Como era de esperar en el primer caso, el archivo cargado está en `params[:person][:picture]` y en el segundo caso en `params[:picture]`.

### What Gets Uploaded

El objeto en el hash `params` es una instancia de [`ActionDispatch::Http::UploadedFile`](https://api.rubyonrails.org/classes/ActionDispatch/Http/UploadedFile.html). El siguiente fragmento guarda el archivo cargado en `#{Rails.root}/public/uploads` con el mismo nombre que el archivo original.

```ruby
def upload
  uploaded_file = params[:picture]
  File.open(Rails.root.join('public', 'uploads', uploaded_file.original_filename), 'wb') do |file|
    file.write(uploaded_file.read)
  end
end
```

Una vez que se ha cargado un archivo, hay una multitud de tareas potenciales, que van desde dónde almacenar los archivos (en disco, Amazon S3, etc.), asociarlos con modelos, cambiar el tamaño de los archivos de imagen y generar miniaturas, etc. [Active Storage](active_storage_overview.html) está diseñado para ayudar con estas tareas.

Customizing Form Builders
-------------------------

El objeto generado por `form_with` y `fields_for` es una instancia de  [`ActionView::Helpers::FormBuilder`](https://api.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html). Los creadores de formularios encapsulan la noción de mostrar elementos de formulario para un solo objeto. Si bien puede escribir ayudantes para sus formularios de la forma habitual, también puede crear la subclase `ActionView::Helpers::FormBuilder` y agregar los ayudantes allí. Por ejemplo:

```erb
<%= form_with model: @person do |form| %>
  <%= text_field_with_label form, :first_name %>
<% end %>
```

puede ser reemplazado con

```erb
<%= form_with model: @person, builder: LabellingFormBuilder do |form| %>
  <%= form.text_field :first_name %>
<% end %>
```

definiendo una clase `LabellingFormBuilder` similar a la siguiente:

```ruby
class LabellingFormBuilder < ActionView::Helpers::FormBuilder
  def text_field(attribute, options={})
    label(attribute) + super
  end
end
```

If you reuse this frequently you could define a `labeled_form_with` helper that automatically applies the `builder: LabellingFormBuilder` option:

```ruby
def labeled_form_with(model: nil, scope: nil, url: nil, format: nil, **options, &block)
  options.merge! builder: LabellingFormBuilder
  form_with model: model, scope: scope, url: url, format: format, **options, &block
end
```

El constructor de formularios utilizado también determina qué sucede cuando lo hace

```erb
<%= render partial: f %>
```

Si `f` es una instancia de` ActionView::Helpers::FormBuilder`, esto hará que el `formulario` sea parcial, estableciendo el objeto del parcial en el generador de formularios. Si el constructor de formularios es de la clase `LabellingFormBuilder`, en su lugar se representará el parcial de` labelling_form`.

Understanding Parameter Naming Conventions
------------------------------------------

Los valores de los formularios pueden estar en el nivel superior del hash `params` o anidados en otro hash. Por ejemplo, en una acción estándar de `create` para un modelo de Persona, `params[:person]` normalmente sería un hash de todos los atributos que la persona debe crear. El hash `params` también puede contener matrices, matrices de hashes, etc.

Fundamentalmente, los formularios HTML no conocen ningún tipo de datos estructurados, todo lo que generan son pares nombre-valor, donde los pares son simplemente cadenas simples. Las matrices y hashes que ves en tu aplicación son el resultado de algunas convenciones de nomenclatura de parámetros que usa Rails.

### Basic Structures

Las dos estructuras básicas son matrices y hashes. Los hash reflejan la sintaxis utilizada para acceder al valor en `params`. Por ejemplo, si un formulario contiene:

```html
<input id="person_name" name="person[name]" type="text" value="Henry"/>
```

el hash `params` contendrá

```ruby
{'person' => {'name' => 'Henry'}}
```

y `params[:person][:name]` recuperará el valor enviado en el controlador.

Los hash se pueden anidar tantos niveles como sea necesario, por ejemplo

```html
<input id="person_address_city" name="person[address][city]" type="text" value="New York"/>
```

dará como resultado que el hash `params` sea

```ruby
{'person' => {'address' => {'city' => 'New York'}}}
```

Normalmente, Rails ignora los nombres de parámetros duplicados. Si el nombre del parámetro contiene un conjunto vacío de corchetes `[]`, se acumularán en una matriz. Si desea que los usuarios puedan ingresar varios números de teléfono, puede colocar esto en el formulario:

```html
<input name="person[phone_number][]" type="text"/>
<input name="person[phone_number][]" type="text"/>
<input name="person[phone_number][]" type="text"/>
```

Esto daría como resultado que params `params[:person][:phone_number]` sea una matriz que contiene los números de teléfono ingresados.

### Combining Them

We can mix and match these two concepts. One element of a hash might be an array as in the previous example, or you can have an array of hashes. For example, a form might let you create any number of addresses by repeating the following form fragment

```html
<input name="person[addresses][][line1]" type="text"/>
<input name="person[addresses][][line2]" type="text"/>
<input name="person[addresses][][city]" type="text"/>
<input name="person[addresses][][line1]" type="text"/>
<input name="person[addresses][][line2]" type="text"/>
<input name="person[addresses][][city]" type="text"/>
```

Esto daría como resultado que `params[:person][:addresses]`  sea una matriz de hashes con las claves `línea1`, `línea2` y `city`.

Sin embargo, existe una restricción, mientras que los hash se pueden anidar arbitrariamente, solo se permite un nivel de "matriz". Las matrices generalmente se pueden reemplazar por hashes; por ejemplo, en lugar de tener una matriz de objetos modelo, se puede tener un hash de objetos modelo codificados por su ID, un índice de matriz o algún otro parámetro.

WARNING: Los parámetros de la matriz no funcionan bien con el ayudante `check_box`. De acuerdo con la especificación HTML, las casillas de verificación no marcadas no envían ningún valor. Sin embargo, a menudo es conveniente que una casilla de verificación envíe siempre un valor. El ayudante `check_box` falsifica esto creando una entrada auxiliar oculta con el mismo nombre. Si la casilla de verificación no está marcada, solo se envía la entrada oculta y, si está marcada, ambas se envían, pero el valor enviado por la casilla de verificación tiene prioridad.

### Using Form Helpers

Las secciones anteriores no usaron los ayudantes de formulario de Rails en absoluto. Si bien puede crear los nombres de entrada usted mismo y pasarlos directamente a ayudantes como `text_field_tag`, Rails también proporciona soporte de mayor nivel. Las dos herramientas a su disposición aquí son el parámetro de nombre para `form_with` y `fields_for` y la opción `:index` que toman los ayudantes.

Es posible que desee representar un formulario con un conjunto de campos de edición para cada una de las direcciones de una persona. Por ejemplo:

```erb
<%= form_with model: @person do |person_form| %>
  <%= person_form.text_field :name %>
  <% @person.addresses.each do |address| %>
    <%= person_form.fields_for address, index: address.id do |address_form| %>
      <%= address_form.text_field :city %>
    <% end %>
  <% end %>
<% end %>
```

Suponiendo que la persona tuviera dos direcciones, con los identificadores 23 y 45, esto generaría una salida similar a esta:


```html
<form accept-charset="UTF-8" action="/people/1" data-remote="true" method="post">
  <input name="_method" type="hidden" value="patch" />
  <input id="person_name" name="person[name]" type="text" />
  <input id="person_address_23_city" name="person[address][23][city]" type="text" />
  <input id="person_address_45_city" name="person[address][45][city]" type="text" />
</form>
```

Esto resultará en un hash `params` que se parece a

```ruby
{'person' => {'name' => 'Bob', 'address' => {'23' => {'city' => 'Paris'}, '45' => {'city' => 'London'}}}}
```

Rails sabe que todas estas entradas deben ser parte del hash de la persona porque usted
llamado `fields_for` en el primer generador de formularios. Especificando una opción `:index`
le estás diciendo a Rails que en lugar de nombrar las entradas `person[address][city]`
debe insertar ese índice rodeado por [] entre la dirección y la ciudad.
Esto suele ser útil, ya que es fácil localizar qué registro de dirección
debe modificarse. Puede pasar números con algún otro significado,
strings o incluso `nil` (lo que dará como resultado la creación de un parámetro de matriz).

Para crear anidamientos más intrincados, puede especificar la primera parte de la entrada
nombre (`person[address]` en el ejemplo anterior) explícitamente:

```erb
<%= fields_for 'person[address][primary]', address, index: address.id do |address_form| %>
  <%= address_form.text_field :city %>
<% end %>
```

creará entradas como

```html
<input id = "person_address_primary_1_city" name = "person [address] [primary] [1] [city]" type = "text" value = "Bologna" />
```

Como regla general, el nombre de entrada final es la concatenación del nombre dado a `fields_for`/`form_with`, el valor del índice y el nombre del atributo. También puede pasar una opción `:index` directamente a los ayudantes como `text_field`, pero normalmente es menos repetitivo especificar esto en el nivel del constructor de formularios que en los controles de entrada individuales.

Como atajo, puede agregar [] al nombre y omitir la opción `:index`. Esto es lo mismo que especificar `index: address.id` así

```erb
<%= fields_for 'person[address][primary][]', address do |address_form| %>
  <%= address_form.text_field :city %>
<% end %>
```

produce exactamente la misma salida que el ejemplo anterior.

Forms to External Resources
--------------------------

Los ayudantes de formulario de Rails también se pueden utilizar para crear un formulario para publicar datos en un recurso externo. Sin embargo, a veces puede ser necesario establecer un `authenticity_token` para el recurso; esto se puede hacer pasando un parámetro `authentity_token: 'your_external_token'` a las opciones de `form_with`:

```erb
<%= form_with url: 'http://farfar.away/form', authenticity_token: 'external_token' do %>
  Form contents
<% end %>
```

A veces, cuando se envían datos a un recurso externo, como una pasarela de pago, los campos que se pueden usar en el formulario están limitados por una API externa y puede que no sea deseable generar un `autenticity_token`. Para no enviar un token, simplemente pase `false` a la opción`:authentity_token`:

```erb
<%= form_with url: 'http://farfar.away/form', authenticity_token: 'external_token' do %>
  Form contents
<% end %>
```

A veces, cuando se envían datos a un recurso externo, como una pasarela de pago, los campos que se pueden usar en el formulario están limitados por una API externa y puede que no sea deseable generar un `autenticity_token`. Para no enviar un token, simplemente pase `false` a la opción `:authentity_token`:

```erb
<%= form_with url: 'http://farfar.away/form', authenticity_token: false do %>
  Form contents
<% end %>
```

Building Complex Forms
----------------------

Muchas aplicaciones crecen más allá de formularios simples que editan un solo objeto. Por ejemplo, al crear una `Person`, es posible que desee permitir que el usuario (en el mismo formulario) cree varios registros de direcciones (casa, trabajo, etc.). Al editar posteriormente a esa persona, el usuario debe poder agregar, eliminar o modificar direcciones según sea necesario.

### Configuring the Model

Active Record proporciona soporte a nivel de modelo a través del método `accept_nested_attributes_for`:

```ruby
class Person < ApplicationRecord
  has_many :addresses, inverse_of: :person
  accepts_nested_attributes_for :addresses
end

class Address < ApplicationRecord
  belongs_to :person
end
```

Esto crea un método `address_attributes =` en `Person` que le permite crear, actualizar y (opcionalmente) destruir direcciones.

### Nested Forms

El siguiente formulario permite a un usuario crear una `Person` y sus direcciones asociadas.

```html+erb
<%= form_with model: @person do |form| %>
  Addresses:
  <ul>
    <%= form.fields_for :addresses do |addresses_form| %>
      <li>
        <%= addresses_form.label :kind %>
        <%= addresses_form.text_field :kind %>

        <%= addresses_form.label :street %>
        <%= addresses_form.text_field :street %>
        ...
      </li>
    <% end %>
  </ul>
<% end %>
```

Cuando una asociación acepta atributos anidados, `fields_for` representa su bloque una vez por cada elemento de la asociación. En particular, si una persona no tiene direcciones, no rinde nada. Un patrón común es que el controlador construya uno o más elementos secundarios vacíos para que se muestre al usuario al menos un conjunto de campos. El siguiente ejemplo daría como resultado 2 conjuntos de campos de dirección que se representan en el formulario de nueva persona.

```ruby
def new
  @person = Person.new
  2.times { @person.addresses.build }
end
```

El `fields_for` produce un constructor de formularios. El nombre de los parámetros será el
`accept_nested_attributes_for` espera. Por ejemplo, al crear un usuario con
2 direcciones, los parámetros enviados se verían así:

```ruby
{
  'person' => {
    'name' => 'John Doe',
    'addresses_attributes' => {
      '0' => {
        'kind' => 'Home',
        'street' => '221b Baker Street'
      },
      '1' => {
        'kind' => 'Office',
        'street' => '31 Spooner Street'
      }
    }
  }
}
```

Las claves del hash `:address_attributes` no son importantes, simplemente deben ser diferentes para cada dirección.

Si el objeto asociado ya está guardado, `fields_for` genera automáticamente una entrada oculta con el `id` del registro guardado. Puede deshabilitar esto pasando `include_id: false` a `fields_for`.


### The Controller

As usual you need to
[declare the permitted parameters](action_controller_overview.html#strong-parameters) in
el controlador antes de pasarlos al modelo:

```ruby
def create
  @person = Person.new(person_params)
  # ...
end

private
  def person_params
    params.require(:person).permit(:name, addresses_attributes: [:id, :kind, :street])
  end
```

### Removing Objects

Puede permitir que los usuarios eliminen objetos asociados pasando `allow_destroy: true` a` accept_nested_attributes_for`

```ruby
class Person < ApplicationRecord
  has_many :addresses
  accepts_nested_attributes_for :addresses, allow_destroy: true
end
```

Si el hash de los atributos de un objeto contiene la clave `_destroy` con un valor que
se evalúa como `true` (por ejemplo, 1, "1", verdadero o "verdadero"), el objeto se destruirá.
Este formulario permite a los usuarios eliminar direcciones:

```erb
<%= form_with model: @person do |form| %>
  Addresses:
  <ul>
    <%= form.fields_for :addresses do |addresses_form| %>
      <li>
        <%= addresses_form.check_box :_destroy %>
        <%= addresses_form.label :kind %>
        <%= addresses_form.text_field :kind %>
        ...
      </li>
    <% end %>
  </ul>
<% end %>
```

Don't forget to update the permitted params in your controller to also include
the `_destroy` field

```ruby
def person_params
  params.require(:person).
    permit(:name, addresses_attributes: [:id, :kind, :street, :_destroy])
end
```

### Preventing Empty Records

A menudo es útil ignorar conjuntos de campos que el usuario no ha completado. Puede controlar esto pasando un proceso `:reject_if` a `accept_nested_attributes_for`. Este proceso se llamará con cada hash de atributos enviado por el formulario. Si el proceso devuelve `false`, Active Record no creará un objeto asociado para ese hash. El siguiente ejemplo solo intenta construir una dirección si se establece el atributo `kind`.

```ruby
class Person < ApplicationRecord
  has_many :addresses
  accepts_nested_attributes_for :addresses, reject_if: lambda {|attributes| attributes['kind'].blank?}
end
```

Para su comodidad, en su lugar, puede pasar el símbolo `:all_blank` que creará un proceso que rechazará los registros en los que todos los atributos estén en blanco excluyendo cualquier valor para` _destroy`.

### Adding Fields on the Fly

En lugar de representar varios conjuntos de campos con anticipación, es posible que desee agregarlos solo cuando un usuario haga clic en el botón "Agregar nueva dirección". Rails no proporciona ningún soporte integrado para esto. Al generar nuevos conjuntos de campos, debe asegurarse de que la clave de la matriz asociada sea única: la fecha de JavaScript actual (milisegundos desde la [epoch](https://en.wikipedia.org/wiki/Unix_time)) es una opción común.

Using form_for and form_tag
---------------------------

Antes de que se introdujera `form_with` en Rails 5.1, su funcionalidad solía dividirse entre `form_tag` y `form_for`. Ambos están ahora obsoletos. La documentación sobre su uso se puede encontrar en [older versions of this guide](https://guides.rubyonrails.org/v5.2/form_helpers.html).

