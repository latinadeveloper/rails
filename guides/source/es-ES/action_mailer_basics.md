**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Conceptos Básicos de Action Mailer
==================================

Esta guía le proporciona todo lo que necesita para comenzar a enviar
correos electrónicos desde y hacia su aplicación, y muchos aspectos internos de Action
Remitente. También cubre cómo probar sus anuncios publicitarios.

Después de leer esta guía, sabrá:

* Cómo enviar correo electrónico dentro de una aplicación Rails.
* Cómo generar y editar una clase de Action Mailer y una vista de mailer.
* Cómo configurar Action Mailer para su entorno.
* Cómo probar tus clases de Action Mailer.

--------------------------------------------------------------------------------

What is Action Mailer?
----------------------

Action Mailer le permite enviar correos electrónicos desde su aplicación utilizando clases de correo
y vistas.

#### Mailers are similar to controllers

Heredan de `ActionMailer::Base` y viven en `app/mailers`. Los mailers también funcionan
de manera muy similar a los controladores. A continuación se enumeran algunos ejemplos de similitudes.
Los correos tienen:

* Acciones, y también, vistas asociadas que aparecen en `app/views`.
* Variables de instancia a las que se puede acceder en vistas.
* La capacidad de utilizar diseños y parciales.
* La capacidad de acceder a un hash de parámetros.

Sending Emails
--------------

Esta sección proporcionará una guía paso a paso para crear un envío de correo y su
puntos de vista.

### Walkthrough to Generating a Mailer

#### Create the Mailer

```bash
$ bin/rails generate mailer UserMailer
create  app/mailers/user_mailer.rb
create  app/mailers/application_mailer.rb
invoke  erb
create    app/views/user_mailer
create    app/views/layouts/mailer.text.erb
create    app/views/layouts/mailer.html.erb
invoke  test_unit
create    test/mailers/user_mailer_test.rb
create    test/mailers/previews/user_mailer_preview.rb
```


```ruby
# app/mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout 'mailer'
end

# app/mailers/user_mailer.rb
class UserMailer < ApplicationMailer
end
```

Como puede ver, puede generar anuncios publicitarios del mismo modo que usa otros generadores con
Rieles.

Si no desea utilizar un generador, puede crear su propio archivo dentro de
`app/mailers`, solo asegúrate de que herede de `ActionMailer::Base`:

```ruby
class MyMailer < ActionMailer::Base
end
```

#### Edit the Mailer

Los remitentes de correo tienen métodos llamados "acciones" y utilizan vistas para estructurar su contenido.
Donde un controlador genera contenido como HTML para enviar de vuelta al cliente, un Mailer
crea un mensaje que se enviará por correo electrónico.

`app/mailers/user_mailer.rb` contiene un mailer vacío:

```ruby
class UserMailer < ApplicationMailer
end
```

Agreguemos un método llamado `welcome_email`, que enviará un correo electrónico al usuario
Dirección de correo electrónico registrado:

```ruby
class UserMailer < ApplicationMailer
  default from: 'notifications@example.com'

  def welcome_email
    @user = params[:user]
    @url  = 'http://example.com/login'
    mail(to: @user.email, subject: 'Welcome to My Awesome Site')
  end
end
```

A continuación se ofrece una explicación rápida de los elementos presentados en el método anterior. En
una lista completa de todas las opciones disponibles, eche un vistazo más abajo en la
lista completa de la sección de atributos configurables por el usuario de Action Mailer.

* `default Hast`: este es un hash de valores predeterminados para cualquier correo electrónico que envíe desde
este anuncio publicitario. En este caso, estamos configurando el encabezado `:from` a un valor para todos
mensajes en esta clase. Esto se puede anular por correo electrónico.
* `mail` - El mensaje de correo electrónico real, estamos pasando el `:to` y `:subject`
encabezados en.

#### Create a Mailer View

Crea un archivo llamado `welcome_email.html.erb` en `app/views/user_mailer/`. Este
será la plantilla utilizada para el correo electrónico, formateada en HTML:

```html+erb
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <h1>Welcome to example.com, <%= @user.name %></h1>
    <p>
      You have successfully signed up to example.com,
      your username is: <%= @user.login %>.<br>
    </p>
    <p>
      To login to the site, just follow this link: <%= @url %>.
    </p>
    <p>Thanks for joining and have a great day!</p>
  </body>
</html>
```

También hagamos una parte de texto para este correo electrónico. No todos los clientes prefieren los correos electrónicos HTML,
por lo que enviar ambos es una buena práctica. Para hacer esto, cree un archivo llamado
`welcome_email.text.erb` en `app/views/user_mailer/`:

```erb
Welcome to example.com, <%= @user.name %>
===============================================

You have successfully signed up to example.com,
your username is: <%= @user.login %>.

To login to the site, just follow this link: <%= @url %>.

Thanks for joining and have a great day!
```

Cuando llame al método `mail` ahora, Action Mailer detectará las dos plantillas
(texto y HTML) y generara automáticamente un correo electrónico `multipart/alternative`.

#### Calling the Mailer

Los anuncios de correo son realmente otra forma de representar una vista. En lugar de representar una
vista y enviarlo a través del protocolo HTTP, simplemente lo están enviando a través de
los protocolos de correo electrónico. Debido a esto, tiene sentido que su
controlador le diga al Mailer que envíe un correo electrónico cuando se cree un usuario con éxito.

Configurar esto es simple.

Primero, creemos un andamio simple de `User`:

```bash
$ bin/rails generate scaffold user name email login
$ bin/rails db:migrate
```

Ahora que tenemos un modelo de usuario para jugar, simplemente editaremos el
`app/controllers/users_controller.rb` hace que indique al `UserMailer` que entregue
un correo electrónico al usuario recién creado editando la acción de creación e insertando un
llamar a `UserMailer.with(user:@user).welcome_email` justo después de que el usuario se haya guardado correctamente.

Action Mailer está muy bien integrado con Active Job para que se pueda enviar correos electrónicos al exterior
del ciclo de solicitud-respuesta, por lo que el usuario no tiene que esperarlo:

```ruby
class UsersController < ApplicationController
  # POST /users
  # POST /users.json
  def create
    @user = User.new(params[:user])

    respond_to do |format|
      if @user.save
        # Tell the UserMailer to send a welcome email after save
        UserMailer.with(user: @user).welcome_email.deliver_later

        format.html { redirect_to(@user, notice: 'User was successfully created.') }
        format.json { render json: @user, status: :created, location: @user }
      else
        format.html { render action: 'new' }
        format.json { render json: @user.errors, status: :unprocessable_entity }
      end
    end
  end
end
```

NOTE: El comportamiento predeterminado de Active Job es ejecutar trabajos a través del adaptador `:async`. Entonces, puedes usar
`deliver_later` ahora para enviar correos electrónicos de forma asincrónica.
El adaptador predeterminado de Active Job ejecuta trabajos con un grupo de subprocesos en proceso.
Es adecuado para los entornos de desarrollo/prueba, ya que no requiere
cualquier infraestructura externa, pero no se ajusta bien a la producción, ya que cae
trabajos pendientes al reiniciar.
Si necesita un backend persistente, deberá usar un adaptador de Active Job
que tiene un backend persistente (Sidekiq, Resque, etc.).

NOTE: Al llamar a `deliver_later`, el trabajo se colocará en la cola de `mailers`. Asegúrese de que el adaptador de trabajo activo lo admita, de lo contrario, el trabajo puede ignorarse silenciosamente y evitar la entrega de correo electrónico. Puede cambiar eso especificando la opción `config.action_mailer.deliver_later_queue_name`.

Si desea enviar correos electrónicos de inmediato (desde un cronjob, por ejemplo) simplemente llame
`deliver_now`:

```ruby
class SendWeeklySummary
  def run
    User.find_each do |user|
      UserMailer.with(user: user).weekly_summary.deliver_now
    end
  end
end
```

Cualquier par de clave-valor que se pase a `with` se convierte en el `params` del envío de correo
acción. Entonces, `with(user:@user,account:@user.account)` hace `params[:user]` y
`params[:account]` disponible en la acción de correo. Al igual que los controladores
params.

El método `welcome_email` devuelve un objeto `ActionMailer::MessageDelivery` que
entonces simplemente se le puede decir a `deliver_now` o `deliver_later` que se envíe a sí mismo.
El objeto `ActionMailer::MessageDelivery` es solo un envoltorio alrededor de un `Mail::Message`. Si
desea inspeccionar, alterar o hacer cualquier otra cosa con el objeto `Mail::Message` que puede
acceda a él con el método `message` en el objeto `ActionMailer::MessageDelivery`.

### Auto encoding header values

Action Mailer maneja la codificación automática de caracteres multibyte dentro de
encabezados y cuerpos.

Para ejemplos más complejos, como definir conjuntos de caracteres alternativos o
texto de autocodificación primero, consulte la biblioteca
[Mail](https://github.com/mikel/mail).

### Complete List of Action Mailer Methods

Solo hay tres métodos que necesita para enviar prácticamente cualquier correo electrónico
mensaje:

* `headers`: especifica cualquier encabezado del correo electrónico que desee. Puedes pasar un hash de
  nombres de campo de encabezado y pares de valores, o puede llamar `headers[:field_name] =
  'value'`.
* `attachments`: le permite agregar archivos adjuntos a su correo electrónico. Por ejemplo,
  `attachments['file-name.jpg'] = File.read('file-name.jpg')`.
* `mail`: envía el correo electrónico real. Puede pasar encabezados como hash a
  el método de correo como parámetro, el correo creará un correo electrónico, ya sea sin formato
  texto, o multiparte, según las plantillas de correo electrónico que haya definido.

#### Adding Attachments

Action Mailer hace que sea muy fácil agregar archivos adjuntos.

* Pase el nombre y el contenido del archivo y Action Mailer y la
  [Mail gem](https://github.com/mikel/mail) adivinará automáticamente el
  mime_type, establezca la codificación y cree el archivo adjunto.
  
    ```ruby
    attachments['filename.jpg'] = File.read('/path/to/filename.jpg')
    ```
  
    Cuando se active el método `mail`, enviará un correo electrónico de varias partes con
    un archivo adjunto, correctamente anidado con el nivel superior siendo `multipart/mixed` y
    la primera parte es una `multipart/alternative` que contiene el texto plano y
    Mensajes de correo electrónico HTML.

NOTE: Mail codificará automáticamente un archivo adjunto en Base64. Si quieres algo
diferente, codifique su contenido y pase el contenido codificado y la codificación en un
`Hash` al método ` attachments`.

* Pase el nombre del archivo y especifique los encabezados y el contenido y Action Mailer y Mail
  utilizará la configuración que ingrese.
  
      ```ruby
      encoded_content = SpecialEncode(File.read('/path/to/filename.jpg'))
      attachments['filename.jpg'] = {
        mime_type: 'application/gzip',
        encoding: 'SpecialEncoding',
        content: encoded_content
      }
      ```
NOTE: Si especifica una codificación, Mail asumirá que su contenido ya está
codificado y no intentara codificarlo en Base64.

#### Making Inline Attachments

Action Mailer 3.0 crea archivos adjuntos en línea, que implicaron una gran cantidad de piratería en las versiones anteriores a 3.0, mucho más simples y triviales como deberían ser.

* Primero, para decirle a Mail que convierta un adjunto en un adjunto en línea, simplemente llame a `#inline` en el método de adjuntos dentro de su Mailer:
  
    ```ruby
    def welcome
      attachments.inline['image.jpg'] = File.read('/path/to/image.jpg')
    end
    ```
  
  * Luego, en su opinión, puede hacer referencia a `attachments` como un hash y especificar
    qué archivo adjunto desea mostrar, llamando a `url` y luego pasando el
    resultado en el método `image_tag`:
    
    ```html+erb
    <p>Hello there, this is our image</p>

    <%= image_tag attachments['image.jpg'].url %>
    ```

  * Como esta es una llamada estándar a `image_tag`, puedes pasar un hash de opciones
    después de la URL del archivo adjunto como podría hacerlo con cualquier otra imagen:

    ```html+erb
    <p>Hello there, this is our image</p>

    <%= image_tag attachments['image.jpg'].url, alt: 'My Photo', class: 'photos' %>
    ```

#### Sending Email To Multiple Recipients

Es posible enviar un correo electrónico a uno o a más destinatarios en un correo electrónico (por ejemplo,
informando a todos los administradores de un nuevo registro) configurando la lista de correos electrónicos en el `:to`
llave. La lista de correos electrónicos puede ser una serie de direcciones de correo electrónico o una sola cadena
con las direcciones separadas por comas.

```ruby
class AdminMailer < ApplicationMailer
  default to: -> { Admin.pluck(:email) },
          from: 'notification@example.com'

  def new_registration(user)
    @user = user
    mail(subject: "New User Signup: #{@user.email}")
  end
end
```

Se puede utilizar el mismo formato para configurar copia carbón (Cc :) y copia carbón oculta
(Cco :), utilizando las teclas `:cc` y `:bcc` respectivamente.

#### Sending Email With Name

A veces desea mostrar el nombre de la persona en lugar de solo su correo electrónico
dirección cuando reciben el correo electrónico. Puede utilizar `email_address_with_name` para
ese:

```ruby
def welcome_email
  @user = params[:user]
  mail(
    to: email_address_with_name(@user.email, @user.name),
    subject: 'Welcome to My Awesome Site'
  )
end
```

### Mailer Views

Las vistas de correo se encuentran en el directorio `app/views/name_of_mailer_class`.
Las vista de correo específica es conocida por la clase porque su nombre es el mismo que el
método de correo. En nuestro ejemplo anterior, nuestra vista de correo para el
El método `welcome_email` estará en `app/views/user_mailer/welcome_email.html.erb`
para la versión HTML y `welcome_email.text.erb` para la versión de texto sin formato.

Para cambiar la vista de correo predeterminada para su acción, haga algo como:

```ruby
class UserMailer < ApplicationMailer
  default from: 'notifications@example.com'

  def welcome_email
    @user = params[:user]
    @url  = 'http://example.com/login'
    mail(to: @user.email,
         subject: 'Welcome to My Awesome Site',
         template_path: 'notifications',
         template_name: 'another')
  end
end
```

En este caso, buscará plantillas en `app/views/Notifications` con nombre
`another`. También se puede especificar una matriz de rutas para `template_path`, y
se buscará en orden.

Si desea más flexibilidad, también puede pasar un bloque y renderizar específicos
plantillas o incluso renderizar en línea o texto sin usar un archivo de plantilla:

```ruby
class UserMailer < ApplicationMailer
  default from: 'notifications@example.com'

  def welcome_email
    @user = params[:user]
    @url  = 'http://example.com/login'
    mail(to: @user.email,
         subject: 'Welcome to My Awesome Site') do |format|
      format.html { render 'another_template' }
      format.text { render plain: 'Render text' }
    end
  end
end
```

Esto representará la plantilla 'another_template.html.erb' para la parte HTML y
utilice el texto renderizado para la parte de texto. El comando de renderizado es el mismo que se usa
dentro de Action Controller, por lo que puede usar las mismas opciones, como
`:text`,`:inline` etc.

Si deseas renderizar una plantilla ubicada fuera del directorio predeterminado `app/views/mailer_name/`, puedes aplicar el `prepend_view_path`, así:

```ruby
class UserMailer < ApplicationMailer
  prepend_view_path "custom/path/to/mailer/view"

  # This will try to load "custom/path/to/mailer/view/welcome_email" template
  def welcome_email
    # ...
  end
end
```

También puede considerar usar el método [append_view_path](https://guides.rubyonrails.org/action_view_overview.html#view-paths).
                                 
#### Caching mailer view

Puede realizar el almacenamiento en caché de fragmentos en vistas de correo como en vistas de aplicaciones utilizando el método `cache`.

```html+erb
<% cache do %>
  <%= @company.name %>
<% end %>
```

Y para utilizar esta función, debe configurar su aplicación con esto:

```ruby
config.action_mailer.perform_caching = true
```

El almacenamiento en caché de fragmentos también se admite en correos electrónicos de varias partes.
Obtenga más información sobre el almacenamiento en caché en [Rails caching guide](caching_with_rails.html).
                                                            
### Action Mailer Layouts

Al igual que las vistas de controlador, también puede tener diseños de correo. El nombre del diseño
debe ser el mismo que su correo, como `user_mailer.html.erb` y
`user_mailer.text.erb` para que su gestor de correo lo reconozca automáticamente como
diseño.

Para usar un archivo diferente, llame a `layout` en su mailer:

```ruby
class UserMailer < ApplicationMailer
  layout 'awesome' # use awesome.(html|text).erb as the layout
end
```

Al igual que con las vistas del controlador, use `yield` para representar la vista dentro del
diseño.

También puede pasar una opción `layout: 'layout_name'` a la llamada de renderización dentro
el bloque de formato para especificar diferentes diseños para diferentes formatos:

```ruby
class UserMailer < ApplicationMailer
  def welcome_email
    mail(to: params[:user].email) do |format|
      format.html { render layout: 'my_layout' }
      format.text
    end
  end
end
```

Representará la parte HTML usando el archivo `my_layout.html.erb` y la parte de texto
con el archivo `user_mailer.text.erb` habitual si existe.

### Previewing Emails

Las vistas previas de Action Mailer proporcionan una manera de ver cómo se ven los correos electrónicos visitando un
URL especial que los muestra. En el ejemplo anterior, la clase de vista previa para
`UserMailer` debe llamarse `UserMailerPreview` y estar ubicado en
`test/mailers/previews/user_mailer_preview.rb`. Para ver la vista previa de
`welcome_email`, implemente un método que tenga el mismo nombre y llame
`UserMailer.welcome_email`:

```ruby
class UserMailerPreview < ActionMailer::Preview
  def welcome_email
    UserMailer.with(user: User.first).welcome_email
  end
end
```

Entonces la vista previa estará disponible en <http://localhost:3000/rails/mailers/user_mailer/welcome_email>.

Si cambia algo en `app/views/user_mailer/welcome_email.html.erb`
o el propio mailer, se recargará y renderizará automáticamente para que puedas
vea visualmente el nuevo estilo al instante. También hay disponible una lista de vistas previas
en <http://localhost:3000/rails/mailers>.

De forma predeterminada, estas clases de vista previa se encuentran en `test/mailers/previews`.
Esto se puede configurar usando la opción `preview_path`. Por ejemplo, si tu
desea cambiarlo a `lib/mailer_previews`, puede configurarlo en
`config/application.rb`:

```ruby
config.action_mailer.preview_path = "#{Rails.root}/lib/mailer_previews"
```

### Generating URLs in Action Mailer Views

A diferencia de los controladores, la instancia de correo no tiene ningún contexto sobre
la solicitud entrante, por lo que deberá proporcionar el parámetro `:host` usted mismo.

Como el `:host` suele ser coherente en toda la aplicación, puede configurarlo
globalmente en `config/application.rb`:

```ruby
config.action_mailer.default_url_options = { host: 'example.com' }
```

Debido a este comportamiento, no puede utilizar ninguno de los ayudantes `* _path` dentro de
un correo electrónico. En su lugar, necesitará usar el ayudante asociado `* _url`. Por ejemplo
En lugar de usar

```html+erb
<%= link_to 'welcome', welcome_path %>
```

Necesitarás usar:

```html+erb
<%= link_to 'welcome', welcome_url %>
```

Al usar la URL completa, sus enlaces ahora funcionarán en sus correos electrónicos.

#### Generating URLs with `url_for`

`url_for` genera una URL completa por defecto en las plantillas.

Si no configuró la opción `:host` globalmente, asegúrese de pasarla a
`url_for`.

```erb
<%= url_for(host: 'example.com',
            controller: 'welcome',
            action: 'greeting') %>
```

#### Generating URLs with Named Routes

Los clientes de correo electrónico no tienen contexto web, por lo que las rutas no tienen una base URL para completar
la direcciones web. Por lo tanto, siempre debe usar la variante "_url" de la ruta nombrada ayudantes.

Si no configuró la opción `:host` globalmente, asegúrese de pasarla al
asistente de URL.

```erb
<%= user_url(@user, host: 'example.com') %>
```

NOTE: los enlaces que no sean `GET` requieren [rails-ujs](https://github.com/rails/rails/blob/master/actionview/app/assets/javascripts) o
[jQuery UJS](https://github.com/rails/jquery-ujs) y no funcionará en plantillas de correo.
Ellas darán lugar a solicitudes normales `GET`.

### Adding images in Action Mailer Views

A diferencia de los controladores, la instancia de correo no tiene ningún contexto sobre el
solicitud entrante, por lo que deberá proporcionar el parámetro `:asset_host` usted mismo.

Como el `:asset_host` suele ser coherente en toda la aplicación, puede
configúrelo globalmente en `config/application.rb`:

```ruby
config.action_mailer.asset_host = 'http://example.com'
```

Ahora puede mostrar una imagen dentro de su correo electrónico.


```ruby
<%= image_tag 'image.jpg' %>
```

### Sending Multipart Emails

Action Mailer enviará automáticamente correos electrónicos de varias partes si tiene diferentes
plantillas para la misma acción. Entonces, para nuestro ejemplo de UserMailer, si tiene
`welcome_email.text.erb` y `welcome_email.html.erb` en
`app/views/user_mailer`, Action Mailer enviará automáticamente un correo electrónico de varias partes
con las versiones HTML y de texto configuradas como partes diferentes.

El orden de las piezas que se insertan está determinado por el `:parts_order`
dentro del método `ActionMailer::Base.default`.

### Sending Emails with Dynamic Delivery Options

Si desea anular las opciones de entrega predeterminadas (por ejemplo, credenciales SMTP)
mientras envía correos electrónicos, puede hacerlo usando `delivery_method_options` en el
acción de correo.

```ruby
class UserMailer < ApplicationMailer
  def welcome_email
    @user = params[:user]
    @url  = user_url(@user)
    delivery_options = { user_name: params[:company].smtp_user,
                         password: params[:company].smtp_password,
                         address: params[:company].smtp_host }
    mail(to: @user.email,
         subject: "Please see the Terms and Conditions attached",
         delivery_method_options: delivery_options)
  end
end
```

### Sending Emails without Template Rendering

Si desea anular las opciones de entrega predeterminadas (por ejemplo, credenciales SMTP)
mientras envía correos electrónicos, puede hacerlo usando `delivery_method_options` en la
acción de correo.

```ruby
class UserMailer < ApplicationMailer
  def welcome_email
    @user = params[:user]
    @url  = user_url(@user)
    delivery_options = { user_name: params[:company].smtp_user,
                         password: params[:company].smtp_password,
                         address: params[:company].smtp_host }
    mail(to: @user.email,
         subject: "Please see the Terms and Conditions attached",
         delivery_method_options: delivery_options)
  end
end
```

### Sending Emails without Template Rendering

Puede haber casos en los que desee omitir el paso de representación de la plantilla y
proporcione el cuerpo del correo electrónico como una cadena. Puedes lograr esto usando el `:body`
opción. En tales casos, no olvide agregar la opción `:content_type`. Rieles
de lo contrario, estará predeterminado en `text/plain`.

```ruby
class UserMailer < ApplicationMailer
  def welcome_email
    mail(to: params[:user].email,
         body: params[:email_body],
         content_type: "text/html",
         subject: "Already rendered!")
  end
end
```

Action Mailer Callbacks
-----------------------

Action Mailer le permite especificar un `before_action`, `after_action` y
`around_action`.

* Los filtros se pueden especificar con un bloque o un símbolo a un método en la clase del
 correo similar a los controladores.

* Puede usar un `before_action` para completar el objeto de correo con valores predeterminados,
  delivery_method_options o inserte encabezados y adjuntos predeterminados.

```ruby
class InvitationsMailer < ApplicationMailer
  before_action { @inviter, @invitee = params[:inviter], params[:invitee] }
  before_action { @account = params[:inviter].account }

  default to:       -> { @invitee.email_address },
          from:     -> { common_address(@inviter) },
          reply_to: -> { @inviter.email_address_with_name }

  def account_invitation
    mail subject: "#{@inviter.name} invited you to their Basecamp (#{@account.name})"
  end

  def project_invitation
    @project    = params[:project]
    @summarizer = ProjectInvitationSummarizer.new(@project.bucket)

    mail subject: "#{@inviter.name.familiar} added you to a project in Basecamp (#{@account.name})"
  end
end
```

* Podrías usar un `after_action` para hacer una configuración similar a un `before_action` pero
  se usan variables de instancia establecidas en su acción de correo.

```ruby
class UserMailer < ApplicationMailer
  before_action { @business, @user = params[:business], params[:user] }

  after_action :set_delivery_options,
               :prevent_delivery_to_guests,
               :set_business_headers

  def feedback_message
  end

  def campaign_message
  end

  private

    def set_delivery_options
      # You have access to the mail instance,
      # @business and @user instance variables here
      if @business && @business.has_smtp_settings?
        mail.delivery_method.settings.merge!(@business.smtp_settings)
      end
    end

    def prevent_delivery_to_guests
      if @user && @user.guest?
        mail.perform_deliveries = false
      end
    end

    def set_business_headers
      if @business
        headers["X-SMTPAPI-CATEGORY"] = @business.code
      end
    end
end
```

* Los filtros de correo anulan el procesamiento posterior si el cuerpo se establece en un valor distinto de nulo.

Using Action Mailer Helpers
---------------------------

Action Mailer hereda de `AbstractController`, por lo que tiene acceso a la mayoría
de los mismos ayudantes que hay en  Action Controller.

También hay algunos métodos de ayuda específicos de Action Mailer disponibles en
`ActionMailer::MailHelper`. Por ejemplo, estos permiten acceder al correo
instancia desde su vista con `mailer`, y accediendo al mensaje como `message`:

```erb
<%= stylesheet_link_tag mailer.name.underscore %>
<h1><%= message.subject %></h1>
```

Action Mailer Configuration
---------------------------

Las siguientes opciones de configuración se realizan mejor en uno de los entornos
archivos (environment.rb, production.rb, etc ...)

| configuración | Descripción |
|---------------|-------------|
|`logger`|Genera información sobre el envío de correos, si está disponible. Se puede establecer en `nil` para no registrar. Compatible con los registradores `Logger` y `Log4r` de Ruby.|
|`smtp_settings`|Permite una configuración detallada para método de entrega: `:smtp` :<ul><li>`:address` - Le permite utilizar un servidor de correo remoto. Solo cámbialo de su valor predeterminado `"localhost"`.</li><li>`:port` - En caso de que su servidor de correo no se ejecute en el puerto 25, puede cambiarlo.</li><li>`:domain` - Si necesita especificar un dominio HELO, puede hacerlo aquí.</li><li>`:user_name` - Si su servidor de correo requiere autenticación, configure el nombre de usuario en esta configuración.</li><li>`:password` - Si su servidor de correo requiere autenticación, configure la contraseña en esta configuración.</li><li>`:authentication` - Si su servidor de correo requiere autenticación, debe especificar el tipo de autenticación aquí. Este es un símbolo y uno de `:plain` (enviará la contraseña en claro), `:login` (enviará la contraseña codificada en Base64) o `:cram_md5` (combina un mecanismo de desafío / respuesta para intercambiar información y un algoritmo criptográfico Message Digest 5 para codificar información importante)</li><li>`:enable_starttls_auto` - Detecta si STARTTLS está habilitado en su servidor SMTP y comienza a usarlo. Predeterminado a `true`.</li><li>`:openssl_verify_mode` - Al usar TLS, puede configurar cómo OpenSSL verifica el certificado. Esto es realmente útil si necesita validar un certificado autofirmado y / o comodín. Puede usar el nombre de una constante de verificación OpenSSL ('none' o 'peer') o directamente la constante (`OpenSSL::SSL::VERIFY_NONE` o `OpenSSL::SSL::VERIFY_PEER`).</li><li>`:ssl/:tls` - Permite que la conexión SMTP utilice SMTP / TLS (SMTPS: SMTP sobre conexión TLS directa)</li></ul>|
|`sendmail_settings`|Le permite anular opciones para el método de entrega:`:sendmail`.<ul><li>`:location` - La ubicación del ejecutable de sendmail. Predeterminado a `/usr/sbin/sendmail`.</li><li>`:arguments` - Los argumentos de la línea de comando que se pasarán a sendmail. Predeterminado a `-i`.</li></ul>|
|`raise_delivery_errors`|Si se deben generar errores o no si el correo electrónico no se entrega. Esto solo funciona si el servidor de correo electrónico externo está configurado para entrega inmediata.|
|`delivery_method`|Define un método de entrega. Los valores posibles son:<ul><li>`:smtp` (predeterminado), se puede configurar usando `config.action_mailer.smtp_settings`.</li><li>`:sendmail`, se puede configurar usando `config.action_mailer.sendmail_settings`.</li><li>`:file`: guardar correos electrónicos en archivos; se puede configurar usando `config.action_mailer.file_settings`.</li><li>`:test`: guardar correos electrónicos para `ActionMailer::Base.deliveries` formación.</li></ul>Veer [API docs](https://api.rubyonrails.org/classes/ActionMailer/Base.html) para más información.|
|`perform_deliveries`|Determina si las entregas se llevan a cabo realmente cuando se invoca el método `deliver` en el mensaje de correo. De forma predeterminada, lo son, pero esto se puede desactivar para ayudar a las pruebas funcionales. Si este valor es `false`, la matriz de `deliveries` no se completará incluso si `delivery_method` es `:test`.|
|`deliveries`|Mantiene una matriz de todos los correos electrónicos enviados a través de Action Mailer con delivery_method: test. Más útil para pruebas funcionales y unitarias.|
|`default_options`|Le permite establecer valores predeterminados para las opciones del método `mail` (`:from`, `:reply_to`, etc.).|

Para obtener una descripción completa de las posibles configuraciones, consulte el
[Configuring Action Mailer](configuring.html#configuring-action-mailer) en
nuestra guía Configuración de aplicaciones de rieles.

### Example Action Mailer Configuration

Un ejemplo sería agregar lo siguiente a su expediente
`config/environments/$RAILS_ENV.rb`:

```ruby
config.action_mailer.delivery_method = :sendmail
# Defaults to:
# config.action_mailer.sendmail_settings = {
#   location: '/usr/sbin/sendmail',
#   arguments: '-i'
# }
config.action_mailer.perform_deliveries = true
config.action_mailer.raise_delivery_errors = true
config.action_mailer.default_options = {from: 'no-reply@example.com'}
```

### Action Mailer Configuration for Gmail

Como Action Mailer ahora usa la [Mail gem](https://github.com/mikel/mail), esto
se vuelve tan simple como agregar a su archivo `config/environment/$RAILS_ENV.rb`:

```ruby
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address:              'smtp.gmail.com',
  port:                 587,
  domain:               'example.com',
  user_name:            '<username>',
  password:             '<password>',
  authentication:       'plain',
  enable_starttls_auto: true }
```

NOTE: A partir del 15 de julio de 2014, Google aumentó [its security measures](https://support.google.com/accounts/answer/6010255) y ahora bloquea los intentos de las aplicaciones que considera menos seguras.
Puedes cambiar tu configuración de Gmail [here](https://www.google.com/settings/security/lesssecureapps) para permitir los intentos. Si su cuenta de Gmail tiene habilitada la autenticación de 2 factores,
entonces deberás establecer un [app password](https://myaccount.google.com/apppasswords) y utilícelo en lugar de su contraseña habitual. Alternativamente, puede
utilice otro ESP para enviar un correo electrónico reemplazando "smtp.gmail.com" arriba con la dirección de su proveedor.

Mailer Testing
--------------

Puede encontrar instrucciones detalladas sobre cómo probar sus anuncios en el
[testing guide](testing.html#testing-your-mailers).

Intercepting and Observing Emails
-------------------

Action Mailer proporciona enlaces a los métodos de observador e interceptor de correo. Estos le permiten registrar clases que se llaman durante el ciclo de vida de la entrega de correo de cada correo electrónico enviado.

### Intercepting Emails

Los interceptores le permiten realizar modificaciones en los correos electrónicos antes de que se entreguen a los agentes de entrega. Una clase de interceptor debe implementar el método `:deliver_email(message)` que será llamado antes de que se envíe el correo electrónico.

```ruby
class SandboxEmailInterceptor
  def self.delivering_email(message)
    message.to = ['sandbox@example.com']
  end
end
```

Antes de que el interceptor pueda hacer su trabajo, debe registrarlo con Action
Marco de correo. Puede hacer esto en un archivo inicializador
`config/initializers/sandbox_email_interceptor.rb`

```ruby
if Rails.env.staging?
  ActionMailer::Base.register_interceptor(SandboxEmailInterceptor)
end
```

NOTE: El ejemplo anterior utiliza un entorno personalizado llamado "ensayo" para un
producción como servidor pero con fines de prueba. Puedes leer
[Creating Rails environments](configuring.html#creating-rails-environments)
para obtener más información sobre los entornos personalizados de Rails.

### Observing Emails

Los observadores le dan acceso al mensaje de correo electrónico después de que se haya enviado. Una clase de observador debe implementar el método `:deliver_email(message)`, que se llamará después de que se envíe el correo electrónico.

```ruby
class EmailDeliveryObserver
  def self.delivered_email(message)
    EmailDelivery.log(message)
  end
end
```

Al igual que los interceptores, debe registrar observadores con el marco de Action Mailer. Puede hacer esto en un archivo inicializador
`config/initializers/email_delivery_observer.rb`

```ruby
ActionMailer::Base.register_observer(EmailDeliveryObserver)
```
