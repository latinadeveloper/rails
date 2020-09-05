**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Rails Routing Desde el Exterior Hacia Adentro
=============================================

Esta guía cubre las funciones orientadas al usuario del enrutamiento Rails.

Después de leer esta guía, sabrá:

* Cómo interpretar el código en `config/routes.rb`.
* Cómo construir sus propias rutas, usando el estilo ingenioso preferido o el método `match`.
* Cómo declarar los parámetros de ruta, que se pasan a las acciones del controlador.
* Cómo crear automáticamente rutas y URL usando ayudantes de ruta.
* Técnicas avanzadas como la creación de restricciones y el montaje de extremos de Rack.

--------------------------------------------------------------------------------

The Purpose of the Rails Router
-------------------------------

El enrutador Rails reconoce las URL y las envía a la acción de un controlador o a una aplicación de Rack. También puede generar rutas y URL, evitando la necesidad de codificar cadenas en sus vistas.

### Connecting URLs to Code

Cuando su aplicación Rails recibe una solicitud entrante para:

```
GET /patients/17
```

le pide al enrutador que lo haga coincidir con una acción del controlador. Si la primera ruta coincidente es:

```ruby
get '/patients/:id', to: 'patients#show'
```

la solicitud se envía a la acción `show` del controlador de `patients` con `{id: '17'}` en `params`.

NOTE: Rails usa snake_case para los nombres de los controladores aquí, si tiene un controlador de varias palabras como `MonsterTrucksController`, desea usar `monster_trucks#show`, por ejemplo.

### Generating Paths and URLs from Code

También puede generar rutas y URL. Si la ruta anterior se modifica para ser:

```ruby
get '/patients/:id', to: 'patients#show', as: 'patient'
```

y su aplicación contiene este código en el controlador:

```ruby
@patient = Patient.find(params[:id])
```

y esto en la vista correspondiente:

```erb
<%= link_to 'Patient Record', patient_path(@patient) %>
```

entonces el enrutador generará la ruta `/pacientes/17`. Esto reduce la fragilidad de su vista y hace que su código sea más fácil de entender. Tenga en cuenta que no es necesario especificar la identificación en el asistente de ruta.

### Configuring the Rails Router

Las rutas para su aplicación o motor se encuentran en el archivo `config/routes.rb` y normalmente se ven así:

```ruby
Rails.application.routes.draw do
  resources :brands, only: [:index, :show] do
    resources :products, only: [:index, :show]
  end

  resource :basket, only: [:show, :update, :destroy]

  resolve("Basket") { route_for(:basket) }
end
```

Dado que este es un archivo fuente normal de Ruby, puede usar todas sus características para ayudarlo a definir sus rutas, pero tenga cuidado con los nombres de las variables, ya que pueden entrar en conflicto con los métodos DSL del enrutador.

NOTE: El bloque `Rails.application.routes.draw do ... end` que envuelve las definiciones de ruta es necesario para establecer el alcance del enrutador DSL y no debe eliminarse.

Resource Routing: the Rails Default
-----------------------------------

El enrutamiento de recursos le permite declarar rápidamente todas las rutas comunes para un controlador con recursos dado. En lugar de declarar rutas separadas para sus acciones de `index`,`show`, `new`,` edit`, `create`,` update` y `destroy`, una ruta ingeniosa las declara en una sola línea de código.

### Resources on the Web

Los navegadores solicitan páginas de Rails solicitando una URL mediante un método HTTP específico, como `GET`,` POST`, `PATCH`,` PUT` y `DELETE`. Cada método es una solicitud para realizar una operación en el recurso. Una ruta de recursos mapea una serie de solicitudes relacionadas con acciones en un solo controlador.

Cuando su aplicación Rails recibe una solicitud entrante para:

```
DELETE /photos/17
```

le pide al enrutador que lo asigne a una acción de controlador. Si la primera ruta coincidente es:

```ruby
resources :photos
```

Rails enviaría esa solicitud a la acción `destroy` en el controlador` photos` con `{id: '17'}` en `params`.

### CRUD, Verbs, and Actions

En Rails, una ruta ingeniosa proporciona un mapeo entre verbos HTTP y URL para
acciones del controlador. Por convención, cada acción también se asigna a un CRUD específico
operación en una base de datos. Una sola entrada en el archivo de enrutamiento, como:

```ruby
resources :photos
```

crea siete rutas diferentes en su aplicación, todas asignadas al controlador `Photos`:

| HTTP Verb | Path             | Controller#Action | Used for                                     |
| --------- | ---------------- | ----------------- | -------------------------------------------- |
| GET       | /photos          | photos#index      | display a list of all photos                 |
| GET       | /photos/new      | photos#new        | return an HTML form for creating a new photo |
| POST      | /photos          | photos#create     | create a new photo                           |
| GET       | /photos/:id      | photos#show       | display a specific photo                     |
| GET       | /photos/:id/edit | photos#edit       | return an HTML form for editing a photo      |
| PATCH/PUT | /photos/:id      | photos#update     | update a specific photo                      |
| DELETE    | /photos/:id      | photos#destroy    | delete a specific photo                      |

NOTE: Debido a que el enrutador usa el verbo HTTP y la URL para hacer coincidir las solicitudes entrantes, cuatro URL se asignan a siete acciones diferentes.

NOTE: Las rutas de los rieles coinciden en el orden en que se especifican, por lo que si tiene un `resources: photos` encima de un` get 'photos/poll'`, la ruta de la acción `show` para la línea` resources` coincidirá antes que la línea `get`. Para solucionar este problema, mueva la línea `get` ** encima de ** la línea `resources` para que coincida primero.

### Path and URL Helpers

La creación de una ruta ingeniosa también expondrá una serie de ayudantes a los controladores de su aplicación. En el caso de `resources :photos:

* `photos_path` devuelve `/photos`
* `new_photo_path` devuelve `/photos/new`
* `edit_photo_path(:id)` devuelve `/photos/:id/edit` (por ejemplo, `edit_photo_path(10)` por ejemplo `/photos/10/edit`)
* `photo_path(:id)` devuelve `/photos/:id` (por ejemplo, `photo_path(10)` por ejemplo `/photos/10`)

Cada uno de estos ayudantes tiene un ayudante `_url` correspondiente (como` photos_url`) que devuelve la misma ruta con el prefijo actual de host, puerto y ruta.

### Defining Multiple Resources at the Same Time

Si necesita crear rutas para más de un recurso, puede ahorrar un poco de escritura definiéndolas todas con una sola llamada a `resources`:

```ruby
resources :photos, :books, :videos
```

Esto funciona exactamente igual que:

```ruby
resources :photos
resources :books
resources :videos
```

### Singular Resources

A veces, tiene un recurso que los clientes siempre buscan sin hacer referencia a una identificación. Por ejemplo, le gustaría que `/profile` muestre siempre el perfil del usuario actualmente conectado. En este caso, puede usar un recurso singular para mapear `/profile` (en lugar de`/profile/:id`) a la acción `show`:

```ruby
get 'profile', to: 'users#show'
```

Pasar una `String` a `to:` esperará un formato de `controller#action`. Cuando se usa un `Symbol`, la opción`to:` debe reemplazarse por` acción: `. Cuando se usa un `String` sin un `#`, la opción` to:` debe reemplazarse por `controller:`:

```ruby
get 'profile', action: :show, controller: 'users'
```

Esta ruta ingeniosa:

```ruby
resource :geocoder
resolve('Geocoder') { [:geocoder] }
```

crea seis rutas diferentes en su aplicación, todas asignadas al controlador `Geocoders`:

| HTTP Verb | Path           | Controller#Action | Used for                                      |
| --------- | -------------- | ----------------- | --------------------------------------------- |
| GET       | /geocoder/new  | geocoders#new     | return an HTML form for creating the geocoder |
| POST      | /geocoder      | geocoders#create  | create the new geocoder                       |
| GET       | /geocoder      | geocoders#show    | display the one and only geocoder resource    |
| GET       | /geocoder/edit | geocoders#edit    | return an HTML form for editing the geocoder  |
| PATCH/PUT | /geocoder      | geocoders#update  | update the one and only geocoder resource     |
| DELETE    | /geocoder      | geocoders#destroy | delete the geocoder resource                  |

NOTE: Debido a que es posible que desee utilizar el mismo controlador para una ruta singular (`/ account`) y una ruta plural (` / accounts / 45`), los recursos singulares se asignan a controladores plurales. De modo que, por ejemplo, `resource: photo` y` resources: photos` crea rutas tanto en singular como en plural que se asignan al mismo controlador (`PhotosController`).

Una ruta de recursos singular genera estas ayudantes:

* `new_geocoder_path` regresa `/geocoder/new`
* `edit_geocoder_path` regresa `/geocoder/edit`
* `geocoder_path` regresa `/geocoder`

Al igual que con los recursos plurales, los mismos ayudantes que terminan en `_url` también incluirán el prefijo de host, puerto y ruta.

### Controller Namespaces and Routing

Es posible que desee organizar grupos de controladores en un espacio de nombres. Lo más común es agrupar varios controladores administrativos en un espacio de nombres `Admin::`. Colocaría estos controladores en el directorio `app/controllers/admin`, y puede agruparlos en su enrutador:

```ruby
namespace :admin do
  resources :articles, :comments
end
```

Esto creará una serie de rutas para cada uno de los controladores `artcicles` y `comments`. Para `Admin::ArticlesController`, Rails creará:

| HTTP Verb | Path                     | Controller#Action      | Named Route Helper           |
| --------- | ------------------------ | ---------------------- | ---------------------------- |
| GET       | /admin/articles          | admin/articles#index   | admin_articles_path          |
| GET       | /admin/articles/new      | admin/articles#new     | new_admin_article_path       |
| POST      | /admin/articles          | admin/articles#create  | admin_articles_path          |
| GET       | /admin/articles/:id      | admin/articles#show    | admin_article_path(:id)      |
| GET       | /admin/articles/:id/edit | admin/articles#edit    | edit_admin_article_path(:id) |
| PATCH/PUT | /admin/articles/:id      | admin/articles#update  | admin_article_path(:id)      |
| DELETE    | /admin/articles/:id      | admin/articles#destroy | admin_article_path(:id)      |

Si desea enrutar `/articles` (sin el prefijo `/admin`) a `Admin::ArticlesController`, puede usar:

```ruby
scope module: 'admin' do
  resources :articles, :comments
end
```

o, para un solo caso:

```ruby
resources :articles, module: 'admin'
```

Si desea enrutar `/admin/articles` a `ArticlesController` (sin el prefijo del módulo `Admin::`), puede usar:

```ruby
scope '/admin' do
  resources :articles, :comments
end
```

o, para un solo caso:

```ruby
resources :articles, path: '/admin/articles'
```

En cada uno de estos casos, las rutas nombradas siguen siendo las mismas que si no usara `scope`. En el último caso, las siguientes rutas se asignan a `ArticlesController`:

| HTTP Verb | Path                     | Controller#Action    | Named Route Helper     |
| --------- | ------------------------ | -------------------- | ---------------------- |
| GET       | /admin/articles          | articles#index       | articles_path          |
| GET       | /admin/articles/new      | articles#new         | new_article_path       |
| POST      | /admin/articles          | articles#create      | articles_path          |
| GET       | /admin/articles/:id      | articles#show        | article_path(:id)      |
| GET       | /admin/articles/:id/edit | articles#edit        | edit_article_path(:id) |
| PATCH/PUT | /admin/articles/:id      | articles#update      | article_path(:id)      |
| DELETE    | /admin/articles/:id      | articles#destroy     | article_path(:id)      |


TIP: _Si necesita usar un espacio de nombres de controlador diferente dentro de un bloque `namespace`, puede especificar una ruta de controlador absoluta, por ejemplo: `get '/foo', to: '/foo#index'`._

### Nested Resources

Es común tener recursos que lógicamente son hijos de otros recursos. Por ejemplo, suponga que su aplicación incluye estos modelos:

```ruby
class Magazine < ApplicationRecord
  has_many :ads
end

class Ad < ApplicationRecord
  belongs_to :magazine
end
```

Las rutas anidadas le permiten capturar esta relación en su ruta. En este caso, podría incluir esta declaración de ruta:

```ruby
resources :magazines do
  resources :ads
end
```

Además de las rutas para revistas, esta declaración también enrutará anuncios a un `AdsController`. Las URL de los anuncios requieren una revista:

| HTTP Verb | Path                                 | Controller#Action | Used for                                                                   |
| --------- | ------------------------------------ | ----------------- | -------------------------------------------------------------------------- |
| GET       | /magazines/:magazine_id/ads          | ads#index         | display a list of all ads for a specific magazine                          |
| GET       | /magazines/:magazine_id/ads/new      | ads#new           | return an HTML form for creating a new ad belonging to a specific magazine |
| POST      | /magazines/:magazine_id/ads          | ads#create        | create a new ad belonging to a specific magazine                           |
| GET       | /magazines/:magazine_id/ads/:id      | ads#show          | display a specific ad belonging to a specific magazine                     |
| GET       | /magazines/:magazine_id/ads/:id/edit | ads#edit          | return an HTML form for editing an ad belonging to a specific magazine     |
| PATCH/PUT | /magazines/:magazine_id/ads/:id      | ads#update        | update a specific ad belonging to a specific magazine                      |
| DELETE    | /magazines/:magazine_id/ads/:id      | ads#destroy       | delete a specific ad belonging to a specific magazine                      |


Esto también creará ayudantes de enrutamiento como `magazine_ads_url` y `edit_magazine_ad_path`. Estos ayudantes toman una instancia de Magazine como primer parámetro (`magazine_ads_url(@magazine)`).

#### Limits to Nesting

Puede anidar recursos dentro de otros recursos anidados si lo desea. Por ejemplo:

```ruby
resources :publishers do
  resources :magazines do
    resources :photos
  end
end
```

Los recursos profundamente anidados se vuelven rápidamente engorrosos. En este caso, por ejemplo, la aplicación reconocería rutas como:

```
/publishers/1/magazines/2/photos/3
```

The corresponding route helper would be `publisher_magazine_photo_url`, requiring you to specify objects at all three levels. Indeed, this situation is confusing enough that a popular [article](http://weblog.jamisbuck.org/2007/2/5/nesting-resources) by Jamis Buck proposes a rule of thumb for good Rails design:

TIP: _Los recursos nunca deben anidarse a más de 1 nivel de profundidad.

#### Shallow Nesting

Una forma de evitar la anidación profunda (como se recomendó anteriormente) es generar las acciones de colección dentro del ámbito del padre, para tener una idea de la jerarquía, pero no anidar las acciones de los miembros. En otras palabras, para construir solo rutas con la mínima cantidad de información para identificar de forma única el recurso, así:

```ruby
resources :articles do
  resources :comments, only: [:index, :new, :create]
end
resources :comments, only: [:show, :edit, :update, :destroy]
```

Esta idea logra un equilibrio entre rutas descriptivas y anidación profunda. Existe una sintaxis abreviada para lograr precisamente eso, a través de la opción `:shallow`:

```ruby
resources :articles do
  resources :comments, shallow: true
end
```

Esto generará exactamente las mismas rutas que el primer ejemplo. También puede especificar la opción `:shallow` en el recurso principal, en cuyo caso todos los recursos anidados serán poco profundos:

```ruby
resources :articles, shallow: true do
  resources :comments
  resources :quotes
  resources :drafts
end
```

El método `shallow` del DSL crea un alcance dentro del cual cada anidamiento es superficial. Esto genera las mismas rutas que el ejemplo anterior:

```ruby
shallow do
  resources :articles do
    resources :comments
    resources :quotes
    resources :drafts
  end
end
```

Existen dos opciones para `scope` para personalizar rutas poco profundas. `:shallow_path` antepone las rutas de miembros con el parámetro especificado:

```ruby
scope shallow_path: "sekret" do
  resources :articles do
    resources :comments, shallow: true
  end
end
```

El recurso de comentarios aquí tendrá las siguientes rutas generadas para él:

| HTTP Verb | Path                                         | Controller#Action | Named Route Helper       |
| --------- | -------------------------------------------- | ----------------- | ------------------------ |
| GET       | /articles/:article_id/comments(.:format)     | comments#index    | article_comments_path    |
| POST      | /articles/:article_id/comments(.:format)     | comments#create   | article_comments_path    |
| GET       | /articles/:article_id/comments/new(.:format) | comments#new      | new_article_comment_path |
| GET       | /sekret/comments/:id/edit(.:format)          | comments#edit     | edit_comment_path        |
| GET       | /sekret/comments/:id(.:format)               | comments#show     | comment_path             |
| PATCH/PUT | /sekret/comments/:id(.:format)               | comments#update   | comment_path             |
| DELETE    | /sekret/comments/:id(.:format)               | comments#destroy  | comment_path             |

La opción `:shallow_prefix` agrega el parámetro especificado a los ayudantes de ruta nombrados:

```ruby
scope shallow_prefix: "sekret" do
  resources :articles do
    resources :comments, shallow: true
  end
end
```

El recurso de comentarios aquí tendrá las siguientes rutas generadas para él:

| HTTP Verb | Path                                         | Controller#Action | Named Route Helper          |
| --------- | -------------------------------------------- | ----------------- | --------------------------- |
| GET       | /articles/:article_id/comments(.:format)     | comments#index    | article_comments_path       |
| POST      | /articles/:article_id/comments(.:format)     | comments#create   | article_comments_path       |
| GET       | /articles/:article_id/comments/new(.:format) | comments#new      | new_article_comment_path    |
| GET       | /comments/:id/edit(.:format)                 | comments#edit     | edit_sekret_comment_path    |
| GET       | /comments/:id(.:format)                      | comments#show     | sekret_comment_path         |
| PATCH/PUT | /comments/:id(.:format)                      | comments#update   | sekret_comment_path         |
| DELETE    | /comments/:id(.:format)                      | comments#destroy  | sekret_comment_path         |

### Routing Concerns

Las preocupaciones sobre el enrutamiento le permiten declarar rutas comunes que se pueden reutilizar dentro de otros recursos y rutas. Para definir una preocupación:

```ruby
concern :commentable do
  resources :comments
end

concern :image_attachable do
  resources :images, only: :index
end
```

Estas preocupaciones se pueden utilizar en los recursos para evitar la duplicación de código y compartir el comportamiento entre rutas:

```ruby
resources :messages, concerns: :commentable

resources :articles, concerns: [:commentable, :image_attachable]
```

Lo anterior es equivalente a:

```ruby
resources :messages do
  resources :comments
end

resources :articles do
  resources :comments
  resources :images, only: :index
end
```

También puede usarlos en cualquier lugar que desee dentro de las rutas, por ejemplo, en una llamada a `scope` o `namespace`:

```ruby
namespace :articles do
  concerns :commentable
end
```

### Creating Paths and URLs from Objects

Además de utilizar los ayudantes de enrutamiento, Rails también puede crear rutas y URL a partir de una serie de parámetros. Por ejemplo, suponga que tiene este conjunto de rutas:

```ruby
resources :magazines do
  resources :ads
end
```

Al usar `magazine_ad_path`, puede pasar instancias de `Magazine` y `Ad` en lugar de los ID numéricos:

```erb
<%= link_to 'Ad details', magazine_ad_path(@magazine, @ad) %>
```

También puede usar `url_for` con un conjunto de objetos, y Rails determinará automáticamente qué ruta desea:

```erb
<%= link_to 'Ad details', url_for([@magazine, @ad]) %>
```

En este caso, Rails verá que `@magazine` es una` Magazine` y `@ad` es un `Ad` y, por lo tanto, usará el ayudante `magazine_ad_path`. En ayudantes como `link_to`, puedes especificar solo el objeto en lugar de la llamada completa a `url_for`:

```erb
<%= link_to 'Ad details', [@magazine, @ad] %>
```

Si desea vincular solo a una revista:

```erb
<%= link_to 'Magazine details', @magazine %>
```

Para otras acciones, solo necesita insertar el nombre de la acción como el primer elemento de la matriz:

```erb
<%= link_to 'Edit Ad', [:edit, @magazine, @ad] %>
```

Esto le permite tratar las instancias de sus modelos como URL y es una ventaja clave para usar el estilo ingenioso.

### Adding More RESTful Actions

No está limitado a las siete rutas que el enrutamiento RESTful crea de forma predeterminada. Si lo desea, puede agregar rutas adicionales que se apliquen a la colección o miembros individuales de la colección.

#### Adding Member Routes

Para agregar una ruta miembro, simplemente agregue un bloque `member` en el bloque de recursos:

```ruby
resources :photos do
  member do
    get 'preview'
  end
end
```

Esto reconocerá `/photos/1/preview` con GET, y se dirigirá a la acción `preview` de `PhotosController`, con el valor de id de recurso pasado en `params[:id] `. También creará los ayudantes `preview_photo_url` y `preview_photo_path`.

Dentro del bloque de rutas miembro, cada nombre de ruta especifica el verbo HTTP que
será reconocido. Puede usar `get`,` patch`, `put`,` post` o `delete` aquí
. Si no tiene varias rutas `member`, también puede pasar `:on` a un
ruta, eliminando el bloque:

```ruby
resources :photos do
  get 'preview', on: :member
end
```

Puede omitir la opción `:on`, esto creará la misma ruta de miembro excepto que el valor de id del recurso estará disponible en `params[:photo_id] `en lugar de` params [: id] `. Los asistentes de ruta también cambiarán de nombre de `preview_photo_url` y `preview_photo_path` a `photo_preview_url` y `photo_preview_path`.


#### Adding Collection Routes

Para agregar una ruta a la colección

```ruby
resources :photos do
  collection do
    get 'search'
  end
end
```

Esto permitirá a Rails reconocer rutas como `/photos/search` con GET, y dirigirse a la acción de `search` de `PhotosController`. También creará los ayudantes de ruta `search_photos_url` y `search_photos_path`.

Al igual que con las rutas de miembros, puede pasar `:on` a una ruta:

```ruby
resources :photos do
  get 'search', on: :collection
end
```

NOTE: Si está definiendo rutas de recursos adicionales con un símbolo como primer argumento posicional, tenga en cuenta que no es equivalente a usar una cadena. Los símbolos infieren acciones del controlador mientras que las cadenas infieren rutas.

#### Adding Routes for Additional New Actions

Para agregar una nueva acción alternativa usando el atajo `:on`:

```ruby
resources :comments do
  get 'preview', on: :new
end
```

Esto permitirá a Rails reconocer rutas como `/comments/new/preview` con GET, y dirigirse a la acción de `preview` de `CommentsController`. También creará los ayudantes de ruta `preview_new_comment_url` y `preview_new_comment_path`.

TIP: si se encuentra agregando muchas acciones adicionales a una ruta ingeniosa, es hora de detenerse y preguntarse si está ocultando la presencia de otro recurso.

Non-Resourceful Routes
----------------------

Además del enrutamiento de recursos, Rails tiene un poderoso soporte para enrutar URL arbitrarias a acciones. Aquí, no obtiene grupos de rutas generados automáticamente por enrutamiento ingenioso. En su lugar, configura cada ruta por separado dentro de su aplicación.

Si bien generalmente debe utilizar un enrutamiento ingenioso, todavía hay muchos lugares donde el enrutamiento más simple es más apropiado. No es necesario tratar de calzar hasta la última pieza de su aplicación en un marco ingenioso si no es una buena opción.

En particular, el enrutamiento simple hace que sea muy fácil asignar URL heredadas a nuevas acciones de Rails.

### Bound Parameters

Cuando configuras una ruta regular, proporcionas una serie de símbolos que Rails asigna a partes de una solicitud HTTP entrante. Por ejemplo, considere esta ruta:


```ruby
get 'photos(/:id)', to: 'photos#display'
```

Si una solicitud entrante de `/photos/1` es procesada por esta ruta (porque no ha coincidido con ninguna ruta anterior en el archivo), el resultado será invocar la acción `display` del `PhotosController`, y para que el parámetro final `"1"` esté disponible como `params[:id]`. Esta ruta también enrutará la solicitud entrante de `/photos` a` PhotosController#display`, ya que `:id` es un parámetro opcional, indicado entre paréntesis.

### Dynamic Segments

Puede configurar tantos segmentos dinámicos dentro de una ruta regular como desee. Cualquier segmento estará disponible para la acción como parte de "params". Si configura esta ruta:


```ruby
get 'photos/:id/:user_id', to: 'photos#show'
```

Se enviará una ruta entrante de `/photos/1/2` a la acción `show` del `PhotosController`. `params[:id]` será `"1"`, y `params[:user_id]` será `"2"`.

SUGERENCIA: De forma predeterminada, los segmentos dinámicos no aceptan puntos; esto se debe a que el punto se usa como separador para rutas formateadas. Si necesita usar un punto dentro de un segmento dinámico, agregue una restricción que anule esto, por ejemplo,  `id: /[^\/]+/` permite cualquier cosa excepto una barra.

### Static Segments

Puede especificar segmentos estáticos al crear una ruta sin anteponer dos puntos a un fragmento:

```ruby
get 'photos/:id/with_user/:user_id', to: 'photos#show'
```

Esta ruta respondería a rutas como `/photos/1/with_user/2`. En este caso, `params` sería ` { controller: 'photos', action: 'show', id: '1', user_id: '2' } `.

### The Query String

Los `params` también incluirán cualquier parámetro de la cadena de consulta. Por ejemplo, con esta ruta:

```ruby
get 'photos/:id', to: 'photos#show'
```

Una ruta entrante de `/photos/1? User_id=2` será enviada a la acción `show` del controlador `Photos`. `params` será` { controller: 'photos', action: 'show', id: '1', user_id: '2' }`.

### Defining Defaults

Puede definir valores predeterminados en una ruta proporcionando un hash para la opción `:defaults`. Esto incluso se aplica a los parámetros que no especifica como segmentos dinámicos. Por ejemplo:

```ruby
get 'photos/:id', to: 'photos#show', defaults: { format: 'jpg' }
```

Rails haría coincidir `photos/12` con la acción` show` de `PhotosController`, y establecería `params[:format]` en `"jpg"`.

También puede utilizar `defaults` en un formato de bloque para definir los valores predeterminados para varios elementos:

```ruby
defaults format: :json do
  resources :photos
end
```

NOTE: No puede anular los valores predeterminados mediante los parámetros de consulta, esto es por razones de seguridad. Los únicos valores predeterminados que se pueden anular son los segmentos dinámicos mediante sustitución en la ruta de la URL.

### Naming Routes

Puede especificar un nombre para cualquier ruta usando la opción `:as`:

```ruby
get 'exit', to: 'sessions#destroy', as: :logout
```

Esto creará `logout_path` y `logout_url` como ayudantes de ruta con nombre en su aplicación. Llamar a `logout_path` devolverá `/exit`

También puede usar esto para anular los métodos de enrutamiento definidos por los recursos, como este:

```ruby
get ':username', to: 'users#show', as: :user
```

Esto definirá un método `user_path` que estará disponible en controladores, ayudantes y vistas que irán a una ruta como `/bob`. Dentro de la acción `show` de `UsersController`, `params [:username]` contendrá el nombre de usuario del usuario. Cambie `:username` en la definición de ruta si no desea que el nombre de su parámetro sea `:username`.

### HTTP Verb Constraints

En general, debes usar los métodos `get`,` post`, `put`, `patch` y `delete` para restringir una ruta a un verbo en particular. Puede usar el método `match` con la opción `:via` para hacer coincidir varios verbos a la vez:

```ruby
match 'photos', to: 'photos#show', via: [:get, :post]
```

Puede hacer coincidir todos los verbos con una ruta en particular usando `via: :all`:

```ruby
match 'photos', to: 'photos#show', via: :all
```

NOTE: Enrutar las solicitudes `GET` y `POST` a una sola acción tiene implicaciones de seguridad. En general, debe evitar enrutar todos los verbos a una acción a menos que tenga una buena razón para hacerlo.

NOTE: `GET` en Rails no buscará el token CSRF. Nunca debe escribir en la base de datos desde solicitudes `GET`. Para obtener más información, consulte la [security guide](security.html#csrf-countermeasures) sobre las contramedidas CSRF.

### Segment Constraints

Puede utilizar la opción `:constraints` para imponer un formato para un segmento dinámico:

```ruby
get 'photos/:id', to: 'photos#show', constraints: { id: /[A-Z]\d{5}/ }
```

Esta ruta coincidiría con rutas como `/photos/A12345`, pero no con `/photos/893`. Puede expresar más sucintamente la misma ruta de esta manera:

```ruby
get 'photos/:id', to: 'photos#show', id: /[A-Z]\d{5}/
```

`:constraints` toma expresiones regulares con la restricción de que los anclajes de expresiones regulares no se pueden usar. Por ejemplo, la siguiente ruta no funcionará:

```ruby
get '/:id', to: 'articles#show', constraints: { id: /^\d/ }
```

Sin embargo, tenga en cuenta que no es necesario utilizar anclajes porque todas las rutas están ancladas al principio.

Por ejemplo, las siguientes rutas permitirían `articles` con valores `to_param` como `1-hello-world` que siempre comienzan con un número y `users` con valores `to_param` como `david` que nunca comienzan con un número para compartir el espacio de nombres raíz:

```ruby
get '/:id', to: 'articles#show', constraints: { id: /\d.+/ }
get '/:username', to: 'users#show'
```

### Request-Based Constraints

También puede restringir una ruta en función de cualquier método en el [Request object](action_controller_overview.html#the-request-object) que devuelve una `String`.

Especifica una restricción basada en solicitud de la misma manera que especifica una restricción de segmento:

```ruby
get 'photos', to: 'photos#index', constraints: { subdomain: 'admin' }
```

También puede especificar restricciones en forma de bloque:

```ruby
namespace :admin do
  constraints subdomain: 'admin' do
    resources :photos
  end
end
```

NOTE: Las restricciones de solicitud funcionan llamando a un método en el [Request object](action_controller_overview.html#the-request-object)  con el mismo nombre que la clave hash y luego compare el valor de retorno con el valor hash. Por lo tanto, los valores de restricción deben coincidir con el tipo de retorno del método de objeto Request correspondiente. Por ejemplo: `constraints: {subdomain: 'api'}` coincidirá con un subdominio `api` como se esperaba, sin embargo, usando un símbolo` constraints: {subdominio: :api} `no lo hará, porque `request.subdomain` devuelve `'api'` como una cadena.

NOTE: Hay una excepción para la restricción `format`: si bien es un método en el objeto Request, también es un parámetro opcional implícito en cada ruta. Las restricciones de segmento tienen prioridad y la restricción de "formato" solo se aplica como tal cuando se aplica mediante un hash. Por ejemplo, `get 'foo', las restricciones: {formato: 'json'}` coincidirá con `GET/foo` porque el formato es opcional por defecto. Sin embargo, puede  [use a lambda](#advanced-constraints) como en `get 'foo', constraints: lambda { |req| req.format == :json }` y la ruta solo coincidirá con solicitudes JSON explícitas.

### Advanced Constraints

Si tiene una restricción más avanzada, puede proporcionar un objeto que responda a "¿coincidencias?" Que Rails debería usar. Supongamos que desea enrutar a todos los usuarios de una lista restringida al `RestrictedListController`. Podrías hacerlo:

```ruby
class RestrictedListConstraint
  def initialize
    @ips = RestrictedList.retrieve_ips
  end

  def matches?(request)
    @ips.include?(request.remote_ip)
  end
end

Rails.application.routes.draw do
  get '*path', to: 'restricted_list#index',
    constraints: RestrictedListConstraint.new
end
```

También puede especificar restricciones como lambda:

```ruby
Rails.application.routes.draw do
  get '*path', to: 'restricted_list#index',
    constraints: lambda { |request| RestrictedList.retrieve_ips.include?(request.remote_ip) }
end
```

Tanto el método `matches?` Como el lambda obtienen el objeto `request` como argumento

### Route Globbing and Wildcard Segments

El globbing de ruta es una forma de especificar que un parámetro particular debe coincidir con todas las partes restantes de una ruta. Por ejemplo:

```ruby
get 'photos/*other', to: 'photos#unknown'
```

Esta ruta coincidiría con `photos/12` o `/photos/long/path/to/12`, estableciendo `params[:other]` en `"12"` o `"long/path/to/12"`. Los fragmentos precedidos de una estrella se denominan "segmentos comodín".

Los segmentos comodín pueden aparecer en cualquier lugar de una ruta. Por ejemplo:

```ruby
get 'books/*section/:title', to: 'books#show'
```

coincidiría `books/some/section/last-words-a-memoir` con `params[:section]` es igual a `'some/section'`, y `params[:title]` es igual a `'last-words-a-memoir'`.

Técnicamente, una ruta puede tener incluso más de un segmento comodín. El comparador asigna segmentos a parámetros de forma intuitiva. Por ejemplo:

```ruby
get '*a/foo/*b', to: 'test#index'
```


coincidiría `zoo/woo/foo/bar/baz` con `params[:a]` es igual a  `'zoo/woo'`, y `params[:b]` es igual a  `'bar/baz'`.

NOTE: Al solicitar `'/foo/bar.json'`, sus `params[:pages] `serán iguales a `'foo/bar'` con el formato de solicitud JSON. Si desea recuperar el antiguo comportamiento de 3.0.x, puede proporcionar `format: false` como este:

```ruby
get '*pages', to: 'pages#show', format: false
```

NOTE: Si desea que el segmento de formato sea obligatorio, por lo que no se puede omitir, puede proporcionar `format: true` así:

```ruby
get '*pages', to: 'pages#show', format: true
```

### Redirection

Puede redirigir cualquier ruta a otra ruta usando el ayudante `redirect` en su enrutador:

```ruby
get '/stories', to: redirect('/articles')
```

También puede reutilizar segmentos dinámicos de la coincidencia en la ruta para redirigir a:

```ruby
get '/stories/:name', to: redirect('/articles/%{name}')
```

También puede proporcionar un bloque para redirigir, que recibe los parámetros de ruta simbolizados y el objeto de solicitud:

```ruby
get '/stories/:name', to: redirect { |path_params, req| "/articles/#{path_params[:name].pluralize}" }
get '/stories', to: redirect { |path_params, req| "/articles/#{req.subdomain}" }
```

Tenga en cuenta que la redirección predeterminada es una redirección 301 "Movido permanentemente". Tenga en cuenta que algunos navegadores web o servidores proxy almacenarán en caché este tipo de redireccionamiento, haciendo que la página anterior sea inaccesible. Puede utilizar la opción `:status` para cambiar el estado de la respuesta:

```ruby
get '/stories/:name', to: redirect('/articles/%{name}', status: 302)
```

En todos estos casos, si no proporciona el host principal (`http://www.example.com`), Rails tomará esos detalles de la solicitud actual.

### Routing to Rack Applications

En lugar de una cadena como `'articles#index'`, que corresponde a la acción `index` en el `ArticlesController`, puedes especificar cualquier [Rack application](rails_on_rack.html) como el punto final para un comparador:

```ruby
match '/application.js', to: MyRackApp, via: :all
```

Mientras `MyRackApp` responda a `call` y devuelva un `[status, headers, body]`, el enrutador no reconocerá la diferencia entre la aplicación Rack y una acción. Este es un uso apropiado de `via: :all`, ya que querrá permitir que su aplicación Rack maneje todos los verbos como lo considere apropiado.

NOTE: Para los curiosos, `'articles#index'` en realidad se expande a `ArticlesController.action(:index)`, que devuelve una aplicación Rack válida.

Si especifica una aplicación Rack como el punto final para un comparador, recuerde que
la ruta no se modificará en la aplicación receptora. Con lo siguiente
route su aplicación Rack debe esperar que la ruta sea `/admin`:

```ruby
match '/admin', to: AdminApp, via: :all
```

Si prefiere que su aplicación Rack reciba solicitudes en la raíz
ruta en su lugar, use `mount`:

```ruby
mount AdminApp, at: '/admin'
```

### Using `root`

Puede especificar a qué Rails debe enrutar `'/'` con el método `root`:


```ruby
root to: 'pages#main'
root 'pages#main' # shortcut for the above
```

Debe colocar la ruta `root` en la parte superior del archivo, porque es la ruta más popular y debe coincidir primero.

NOTE: La ruta `root` solo enruta las solicitudes `GET` a la acción.

También puede utilizar la raíz dentro de espacios de nombres y ámbitos. Por ejemplo:

```ruby
namespace :admin do
  root to: "admin#index"
end

root to: "home#index"
```

### Unicode Character Routes

Puede especificar rutas de caracteres Unicode directamente. Por ejemplo:


```ruby
get 'こんにちは', to: 'welcome#index'
```

### Direct Routes

Puede crear ayudantes de URL personalizados directamente. Por ejemplo:

```ruby
direct :homepage do
  "http://www.rubyonrails.org"
end

# >> homepage_url
# => "http://www.rubyonrails.org"
```

El valor de retorno del bloque debe ser un argumento válido para el método `url_for`. Por lo tanto, puede pasar una URL de cadena válida, Hash, Array, una instancia de modelo activo o una clase de modelo activo.

```ruby
direct :commentable do |model|
  [ model, anchor: model.dom_id ]
end

direct :main do
  { controller: 'pages', action: 'index', subdomain: 'www' }
end
```

### Using `resolve`

El método `resolve` permite personalizar el mapeo polimórfico de modelos. Por ejemplo:

``` ruby
resource :basket

resolve("Basket") { [:basket] }
```

``` erb
<%= form_with model: @basket do |form| %>
  <!-- basket form -->
<% end %>
```

Esto generará la URL singular `/basket` en lugar de la habitual `/baskets/:id`.


Customizing Resourceful Routes
------------------------------

Si bien las rutas predeterminadas y los ayudantes generados por `resources :articles` generalmente le servirán bien, es posible que desee personalizarlos de alguna manera. Rails le permite personalizar prácticamente cualquier parte genérica de los ayudantes ingeniosos.

### Specifying a Controller to Use

La opción `:controller` le permite especificar explícitamente un controlador para usar para el recurso. Por ejemplo:


```ruby
resources :photos, controller: 'images'
```

reconocerá las rutas entrantes que comienzan con `/photos` pero se dirigirán al controlador` Images`:

| HTTP Verb | Path             | Controller#Action | Named Route Helper   |
| --------- | ---------------- | ----------------- | -------------------- |
| GET       | /photos          | images#index      | photos_path          |
| GET       | /photos/new      | images#new        | new_photo_path       |
| POST      | /photos          | images#create     | photos_path          |
| GET       | /photos/:id      | images#show       | photo_path(:id)      |
| GET       | /photos/:id/edit | images#edit       | edit_photo_path(:id) |
| PATCH/PUT | /photos/:id      | images#update     | photo_path(:id)      |
| DELETE    | /photos/:id      | images#destroy    | photo_path(:id)      |

NOTE: Utilice `photos_path`, `new_photo_path`, etc. para generar rutas para este recurso.

Para los controladores de espacio de nombres, puede utilizar la notación de directorio. Por ejemplo:

```ruby
resources :user_permissions, controller: 'admin/user_permissions'
```

Esto se enrutará al controlador `Admin::UserPermissions`.

NOTE: Solo se admite la notación de directorio. Especificando el
controlador con notación constante de Ruby (por ejemplo, `controller: 'Admin::UserPermissions'`)
puede conducir a problemas de enrutamiento y resulta en
una advertencia.

### Specifying Constraints

Puede usar la opción `: constraints` para especificar un formato requerido en el` id` implícito. Por ejemplo:

```ruby
resources :photos, constraints: { id: /[A-Z][A-Z][0-9]+/ }
```

Esta declaración restringe el parámetro `:id` para que coincida con la expresión regular proporcionada. Entonces, en este caso, el enrutador ya no coincidirá con `/ photos / 1` con esta ruta. En cambio, `/ photos / RR27` coincidiría.

Puede especificar una única restricción para aplicar a varias rutas utilizando el formulario de bloque:

```ruby
constraints(id: /[A-Z][A-Z][0-9]+/) do
  resources :photos
  resources :accounts
end
```

NOTE: Por supuesto, puede utilizar las restricciones más avanzadas disponibles en rutas sin recursos en este contexto.

TIP: De forma predeterminada, el parámetro `:id` no acepta puntos; esto se debe a que el punto se usa como separador para rutas formateadas. Si necesita usar un punto dentro de un `:id` agregue una restricción que anule esto - por ejemplo, `id: /[^\/]+/` permite cualquier cosa excepto una barra.

### Overriding the Named Route Helpers

La opción `:as` le permite anular el nombre normal de los ayudantes de ruta nombrados. Por ejemplo:


```ruby
resources :photos, as: 'images'
```

reconocerá las rutas entrantes que comienzan con `/photos` y enrutará las solicitudes a `PhotosController`, pero usará el valor de la opción `:as` para nombrar los ayudantes.

| HTTP Verb | Path             | Controller#Action | Named Route Helper   |
| --------- | ---------------- | ----------------- | -------------------- |
| GET       | /photos          | photos#index      | images_path          |
| GET       | /photos/new      | photos#new        | new_image_path       |
| POST      | /photos          | photos#create     | images_path          |
| GET       | /photos/:id      | photos#show       | image_path(:id)      |
| GET       | /photos/:id/edit | photos#edit       | edit_image_path(:id) |
| PATCH/PUT | /photos/:id      | photos#update     | image_path(:id)      |
| DELETE    | /photos/:id      | photos#destroy    | image_path(:id)      |

### Overriding the `new` and `edit` Segments

La opción `:path_names` te permite anular los segmentos `new` y `edit` generados automáticamente en las rutas:

```ruby
resources :photos, path_names: { new: 'make', edit: 'change' }
```

Esto haría que el enrutamiento reconociera rutas como:

```
/photos/make
/photos/1/change
```

NOTE: Esta opción no cambia los nombres reales de las acciones. Las dos rutas mostradas aún se enrutarían a las acciones `nuevo` y `edit`.

TIP: Si desea cambiar esta opción de manera uniforme para todas sus rutas, puede usar un visor.

```ruby
scope path_names: { new: 'make' } do
  # rest of your routes
end
```

### Prefixing the Named Route Helpers

Puede usar la opción `:as` para prefijar los ayudantes de ruta con nombre que Rails genera para una ruta. Utilice esta opción para evitar colisiones de nombres entre rutas utilizando un alcance de ruta. Por ejemplo:

```ruby
scope 'admin' do
  resources :photos, as: 'admin_photos'
end

resources :photos
```

Esto proporcionará ayudantes de ruta como `admin_photos_path`, `new_admin_photo_path`, etc.

Para prefijar un grupo de ayudantes de ruta, use `:as` con `scope`:

```ruby
scope 'admin', as: 'admin' do
  resources :photos, :accounts
end

resources :photos, :accounts
```

Esto generará rutas como `admin_photos_path` y `admin_accounts_path` que se asignan a `/admin/photos` y `/admin/accounts` respectivamente.

NOTE: El alcance del `namespace` agregará automáticamente los prefijos `: as` así como `:module` y `:path`.

También puede prefijar rutas con un parámetro con nombre:

```ruby
scope ':username' do
  resources :articles
end
```

Esto le proporcionará URL como `/bob/articles/1` y le permitirá hacer referencia a la parte del `username` de la ruta como `params[:username]` en controladores, ayudantes y vistas.

### Restricting the Routes Created

De forma predeterminada, Rails crea rutas para las siete acciones predeterminadas (`index`, `show`, `new`, `create`, `edit`, `update`, and `destroy`) para cada ruta RESTful en su aplicación. Puede utilizar las opciones `:only` y `:except` para ajustar este comportamiento. La opción `:only` le dice a Rails que cree solo las rutas especificadas:

```ruby
resources :photos, only: [:index, :show]
```

Ahora, una solicitud `GET` a `/photos` tendría éxito, pero una solicitud `POST` a `/photos` (que normalmente se enrutaría a la acción `create`) fallará.

La opción `: except` especifica una ruta o lista de rutas que Rails _no_ debería crear:

```ruby
resources :photos, except: :destroy
```

En este caso, Rails creará todas las rutas normales excepto la ruta para `destroy` (una solicitud `DELETE` a `/photos/:id`).

TIP: Si su aplicación tiene muchas rutas RESTful, usar `:only` y `:except` para generar solo las rutas que realmente necesita puede reducir el uso de memoria y acelerar el proceso de enrutamiento.

### Translated Paths

Usando `scope`, podemos alterar los nombres de ruta generados por `resources`:

```ruby
scope(path_names: { new: 'neu', edit: 'bearbeiten' }) do
  resources :categories, path: 'kategorien'
end
```

Rails ahora crea rutas al `CategoriesController`.

| HTTP Verb | Path                       | Controller#Action  | Named Route Helper      |
| --------- | -------------------------- | ------------------ | ----------------------- |
| GET       | /kategorien                | categories#index   | categories_path         |
| GET       | /kategorien/neu            | categories#new     | new_category_path       |
| POST      | /kategorien                | categories#create  | categories_path         |
| GET       | /kategorien/:id            | categories#show    | category_path(:id)      |
| GET       | /kategorien/:id/bearbeiten | categories#edit    | edit_category_path(:id) |
| PATCH/PUT | /kategorien/:id            | categories#update  | category_path(:id)      |
| DELETE    | /kategorien/:id            | categories#destroy | category_path(:id)      |

### Overriding the Singular Form

Si desea definir la forma singular de un recurso, debe agregar reglas adicionales al `Inflector`:

```ruby
ActiveSupport::Inflector.inflections do |inflect|
  inflect.irregular 'tooth', 'teeth'
end
```

### Using `:as` in Nested Resources

La opción `:as` anula el nombre generado automáticamente para el recurso en los ayudantes de ruta anidados. Por ejemplo:

```ruby
resources :magazines do
  resources :ads, as: 'periodical_ads'
end
```

Esto creará ayudantes de enrutamiento como `magazine_periodical_ads_url` y `edit_magazine_periodical_ad_path`.

### Overriding Named Route Parameter

La opción `:param` anula el identificador de recurso predeterminado `:id` (nombre de
el  [dynamic segment](routing.html#dynamic-segments) que se utiliza para generar el
rutas). Puede acceder a ese segmento desde su controlador usando
`params[<:param>]`.

```ruby
resources :videos, param: :identifier
```

```
    videos GET  /videos(.:format)                  videos#index
           POST /videos(.:format)                  videos#create
 new_video GET  /videos/new(.:format)              videos#new
edit_video GET  /videos/:identifier/edit(.:format) videos#edit
```

```ruby
Video.find_by(identifier: params[:identifier])
```

Puede anular `ActiveRecord::Base#to_param` de un modelo relacionado para construir
una URL:

```ruby
class Video < ApplicationRecord
  def to_param
    identifier
  end
end

video = Video.find_by(identifier: "Roman-Holiday")
edit_video_path(video) # => "/videos/Roman-Holiday/edit"
```

Breaking up *very* large route file into multiple small ones:
-------------------------------------------------------

Si trabaja en una gran aplicación con miles de rutas,
Un solo archivo `config/routes.rb` puede resultar engorroso y difícil de leer.

Rails ofrece una manera de dividir un archivo gigantesco único `routes.rb` en varios pequeños usando la macro `draw`.


```ruby
# config/routes.rb

Rails.application.routes.draw do
  get 'foo', to: 'foo#bar'

  draw(:admin) # Will load another route file located in `config/routes/admin.rb`
end

# config/routes/admin.rb

namespace :admin do
  resources :comments
end
```

Llamar a `draw(:admin)` dentro del bloque `Rails.application.routes.draw` intentará cargar una ruta
archivo que tiene el mismo nombre que el argumento dado (`admin.rb` en este caso).
El archivo debe estar ubicado dentro del directorio `config/routes` o cualquier subdirectorio (es decir, `config/routes/admin.rb`, `config/routes/external/admin.rb`).

Puede usar el DSL de enrutamiento normal dentro del archivo de enrutamiento `admin.rb`, **sin embargo** no debe rodearlo con el bloque `Rails.application.routes.draw` como lo hizo en el archivo `config/routes.rb`.

### When to use and not use this feature

Dibujar rutas a partir de archivos externos puede resultar muy útil para organizar un gran conjunto de rutas en varias rutas organizadas. Podría tener una ruta `admin.rb` que contenga todas las rutas para el área de administración, otro archivo `api.rb` para enrutar los recursos relacionados con la API, etc.

Sin embargo, no debe abusar de esta función, ya que tener demasiados archivos de ruta dificulta la detección y la comprensión. Dependiendo de la aplicación, puede ser más fácil para los desarrolladores tener un solo archivo de enrutamiento incluso si tiene algunos cientos de rutas. No debe intentar crear un nuevo archivo de enrutamiento para cada categoría (admin, api ...) a toda costa; El DSL de enrutamiento de Rails ya ofrece una forma de romper rutas de manera organizada con `namespaces` y `scopes`.


Inspecting and Testing Routes
-----------------------------

Rails ofrece instalaciones para inspeccionar y probar sus rutas.

### Listing Existing Routes

Para obtener una lista completa de las rutas disponibles en su aplicación, visite `http://localhost:3000/rails/info/routes` in your browser while your server is running in the **development** environment. You can also execute the `bin/rails routes` command in your terminal to produce the same output. 


Ambos métodos enumerarán todas sus rutas, en el mismo orden en que aparecen en `config/routes.rb`. Para cada ruta, verá:

* El nombre de la ruta (si corresponde)
* El verbo HTTP utilizado (si la ruta no responde a todos los verbos)
* El patrón de URL para que coincida
* Los parámetros de enrutamiento para la ruta

Por ejemplo, aquí hay una pequeña sección de la salida de `bin/rails route` para una ruta RESTful:

```
    users GET    /users(.:format)          users#index
          POST   /users(.:format)          users#create
 new_user GET    /users/new(.:format)      users#new
edit_user GET    /users/:id/edit(.:format) users#edit
```

También puede usar la opción `--expanded` para activar el modo de formato de tabla expandido.

```
$ bin/rails routes --expanded

--[ Route 1 ]----------------------------------------------------
Prefix            | users
Verb              | GET
URI               | /users(.:format)
Controller#Action | users#index
--[ Route 2 ]----------------------------------------------------
Prefix            |
Verb              | POST
URI               | /users(.:format)
Controller#Action | users#create
--[ Route 3 ]----------------------------------------------------
Prefix            | new_user
Verb              | GET
URI               | /users/new(.:format)
Controller#Action | users#new
--[ Route 4 ]----------------------------------------------------
Prefix            | edit_user
Verb              | GET
URI               | /users/:id/edit(.:format)
Controller#Action | users#edit
```

Puede buscar en sus rutas con la opción grep: -g. Esto genera cualquier ruta que coincida parcialmente con el nombre del método auxiliar de URL, el verbo HTTP o la ruta URL.

```
$ bin/rails routes -g new_comment
$ bin/rails routes -g POST
$ bin/rails routes -g admin
```

Si solo desea ver las rutas que se asignan a un controlador específico, existe la opción -c.

```
$ bin/rails routes -c users
$ bin/rails routes -c admin/users
$ bin/rails routes -c Comments
$ bin/rails routes -c Articles::CommentsController
```

TIP: Encontrará que la salida de `bin/rails route` es mucho más legible si amplía la ventana de su terminal hasta que las líneas de salida no se ajustan.

### Testing Routes

Las rutas deben incluirse en su estrategia de prueba (al igual que el resto de su aplicación). Rails ofrece tres [built-in assertions](https://api.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html) diseñadas para simplificar las rutas de prueba:

* `assert_generates`
* `assert_recognizes`
* `assert_routing`

#### The `assert_generates` Assertion

`assert_generates` afirma que un conjunto particular de opciones genera una ruta particular y se puede usar con rutas predeterminadas o rutas personalizadas. Por ejemplo:

```ruby
assert_generates '/photos/1', { controller: 'photos', action: 'show', id: '1' }
assert_generates '/about', controller: 'pages', action: 'about'
```

#### The `assert_recognizes` Assertion

`assert_recognizes` es el inverso de `assert_generates`. Afirma que se reconoce una ruta determinada y la enruta a un lugar particular en su aplicación. Por ejemplo:

```ruby
assert_recognizes({ controller: 'photos', action: 'show', id: '1' }, '/photos/1')
```

You can supply a `:method` argument to specify the HTTP verb:

```ruby
assert_recognizes({ controller: 'photos', action: 'create' }, { path: 'photos', method: :post })
```

#### The `assert_routing` Assertion

La aserción `assert_routing` verifica la ruta en ambos sentidos: prueba que la ruta genera las opciones y que las opciones generan la ruta. Por lo tanto, combina las funciones de `assert_generates` y `assert_recognizes`:

```ruby
assert_routing({ path: 'photos', method: :post }, { controller: 'photos', action: 'create' })
```
