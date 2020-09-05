**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Descripción General de Action Cable
===================================

En esta guía, aprenderá cómo funciona Action Cable y cómo usar WebSockets para
incorpore funciones en tiempo real en su aplicación Rails.

Después de leer esta guía, sabrá:

* Qué es Action Cable y su integración backend y frontend
* Cómo configurar Action Cable
* Cómo configurar canales
* Configuración de implementación y arquitectura para ejecutar Action Cable

--------------------------------------------------------------------------------

What is Action Cable?
---------------------

Action Cable seamlessly integrates
[WebSockets](https://en.wikipedia.org/wiki/WebSocket) con el resto de tu
Aplicación de Rails. Permite escribir características en tiempo real en Ruby en el
mismo estilo y forma que el resto de su aplicación Rails, sin dejar de ser
eficiente y escalable. Es una oferta de pila completa que proporciona tanto
marco JavaScript del lado del cliente y un marco Ruby del lado del servidor. Tienes
acceso a su modelo de dominio completo escrito con Active Record o su ORM de
elección.

Terminology
-----------

Un solo servidor de Action Cable puede manejar múltiples instancias de conexión. Tiene uno
instancia de conexión por conexión WebSocket. Un solo usuario puede tener múltiples
WebSockets abiertos en su aplicación si utilizan varias pestañas o dispositivos del navegador.
El cliente de una conexión WebSocket se denomina consumidor.

Cada consumidor puede, a su vez, suscribirse a múltiples canales de cable. Cada canal
encapsula una unidad lógica de trabajo, similar a lo que hace un controlador en
una configuración regular de MVC. Por ejemplo, podría tener un `ChatChannel` y
un `AppearancesChannel`, y un consumidor podría suscribirse a
oa ambos canales. Como mínimo, un consumidor debe estar suscrito
a un canal.

When the consumer is subscribed to a channel, they act as a subscriber.
The connection between the subscriber and the channel is, surprise-surprise,
called a subscription. A consumer can act as a subscriber to a given channel
any number of times. For example, a consumer could subscribe to multiple chat rooms
at the same time. (And remember that a physical user may have multiple consumers,
one per tab/device open to your connection).

Cada canal puede volver a transmitir cero o más transmisiones.
Una radiodifusión es un enlace pubsub donde cualquier cosa transmitida por la emisora ​​es
enviado directamente a los suscriptores del canal que están transmitiendo esa transmisión con nombre.

Como puede ver, esta es una pila arquitectónica bastante profunda. Hay muchas novedades
terminología para identificar las nuevas piezas, y además de eso, está tratando
con reflejos del lado del cliente y del servidor de cada unidad.

What is Pub/Sub?
----------------

[Pub/Sub](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern), o
Publicar-Suscribir, se refiere a un paradigma de cola de mensajes mediante el cual los remitentes de
información (editores), envian datos a una clase abstracta de destinatarios
(suscriptores), sin especificar destinatarios individuales. Action Cable utiliza este
enfoque para comunicarse entre el servidor y muchos clientes.

## Server-Side Components

### Connections

*Connections* forman la base de la relación cliente-servidor. Para cada
WebSocket aceptado por el servidor, se crea una instancia de un objeto de conexión. Este
objeto se convierte en el padre de todas las *channel subscriptions* que se crean
de ahí en adelante. La conexión en sí no se ocupa de ninguna aplicación específica
lógicamente más allá de la autenticación y autorización. El cliente de la conexión  de
un WebSocket se llama conexión *consumer*. Un usuario individual creará
un par de conexión de consumidor por pestaña, ventana o dispositivo del navegador que tengan abierto.

Las conexiones son instancias de `ApplicationCable::Connection`. En esta clase, tu
autorizas la conexión entrante y procedes a establecerla si el usuario puede
ser identificado.

#### Connection Setup

```ruby
# app/channels/application_cable/connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private
      def find_verified_user
        if verified_user = User.find_by(id: cookies.encrypted[:user_id])
          verified_user
        else
          reject_unauthorized_connection
        end
      end
  end
end
```

Aquí, `protected_by` es un identificador de conexión que se puede usar para encontrar la
conexión específica más tarde. Tenga en cuenta que todo lo que esté marcado como identificador
cree un delegado con el mismo nombre en cualquiera instancia de canal creada fuera de la conexión.

Este ejemplo se basa en el hecho de que ya habrá manejado la autenticación del usuario
en otro lugar de su aplicación, y que una autenticación exitosa establece un cifrado
cookie con el ID de usuario.

La cookie se envía automáticamente a la instancia de conexión cuando una nueva conexión
se intenta, y lo usa para establecer el `current_user`. Identificando la conexión
por este mismo usuario actual, también se asegura de que luego pueda recuperar todas las
conexiones de un usuario determinado (y potencialmente desconectarlas todas si el usuario es eliminado
o no autorizado).

Si su enfoque de autenticación incluye el uso de una sesión, utiliza el almacén de cookies para
la sesión, su cookie de sesión se llama `_session` y la clave de ID de usuario es` user_id` usted
puede utilizar este enfoque:

```ruby
verified_user = User.find_by(id: cookies.encrypted['_session']['user_id'])
```

#### Exception Handling

De forma predeterminada, las excepciones no controladas se capturan y registran en el registrador de Rails. Si a ti te gustaría
interceptar globalmente estas excepciones e informarlas a un servicio de seguimiento de errores externo, por
ejemplo, puede hacerlo con
[`rescue_from`](https://api.rubyonrails.org/classes/ActiveSupport/Rescuable/ClassMethods.html#method-i-rescue_from).

```ruby
# app/channels/application_cable/connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    rescue_from StandardError, with: :report_error

    private

    def report_error(e)
      SomeExternalBugtrackingService.notify(e)
    end
  end
end
```

### Channels

Un *channel* encapsula una unidad lógica de trabajo, similar a lo que hace un controlador en un
configuración regular de MVC. Por defecto, Rails crea una clase principal `ApplicationCable::Channel`
para encapsular la lógica compartida entre sus canales.

#### Parent Channel Setup

```ruby
# app/channels/application_cable/channel.rb
module ApplicationCable
  class Channel < ActionCable::Channel::Base
  end
end
```

Luego, crearía sus propias clases de canal. Por ejemplo, podría tener un
`ChatChannel` y un` AppearanceChannel`:

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
end

# app/channels/appearance_channel.rb
class AppearanceChannel < ApplicationCable::Channel
end
```

A continuación, un consumidor podría suscribirse a uno de estos canales oa ambos.

#### Subscriptions

Los consumidores se suscriben a los canales, actuando como *subscribers*. Su conexión es
llamada *subscription*. Los mensajes producidos se enrutan luego a estos canales
suscripciones basadas en un identificador enviado por el consumidor de cable.

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
  # Called when the consumer has successfully
  # become a subscriber to this channel.
  def subscribed
  end
end
```

#### Exception Handling

Al igual que con `ActionCable::Connection::Base`, también puedes usar `rescue_from` en un
canal específico para manejar excepciones planteadas:

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
  rescue_from 'MyError', with: :deliver_error_message

  private

  def deliver_error_message(e)
    broadcast_to(...)
  end
end
```

## Client-Side Components

### Connections

Los consumidores necesitan una instancia de la conexión de su lado. Esto puede ser
establecido usando el siguiente JavaScript, que es generado por defecto por Rails:

#### Connect Consumer

```js
// app/javascript/channels/consumer.js
// Action Cable provides the framework to deal with WebSockets in Rails.
// You can generate new channels where WebSocket features live using the `bin/rails generate channel` command.

import { createConsumer } from "@rails/actioncable"

export default createConsumer()
```

Esto preparará a un consumidor que se conectará contra `/cable` en su servidor de forma predeterminada.
La conexión no se establecerá hasta que también haya especificado al menos una suscripción
estás interesado en tener.

El consumidor puede, opcionalmente, tomar un argumento que especifique la URL a la que conectarse. Esta
puede ser una cadena, o una función que devuelve una cadena que se llamará cuando se abre el WebSocket.

```js
// Specify a different URL to connect to
createConsumer('https://ws.example.com/cable')

// Use a function to dynamically generate the URL
createConsumer(getWebSocketURL)

function getWebSocketURL {
  const token = localStorage.get('auth-token')
  return `https://ws.example.com/cable?token=${token}`
}
```

#### Subscriber

Un consumidor se convierte en suscriptor al crear una suscripción a un canal determinado:

```js
// app/javascript/channels/chat_channel.js
import consumer from "./consumer"

consumer.subscriptions.create({ channel: "ChatChannel", room: "Best Room" })

// app/javascript/channels/appearance_channel.js
import consumer from "./consumer"

consumer.subscriptions.create({ channel: "AppearanceChannel" })
```

Si bien esto crea la suscripción, la funcionalidad necesaria para responder a
los datos recibidos se describirán más adelante.

Un consumidor puede actuar como suscriptor de un canal determinado tantas veces como desee.
Por ejemplo, un consumidor podría suscribirse a varias salas de chat al mismo tiempo:

```js
// app/javascript/channels/chat_channel.js
import consumer from "./consumer"

consumer.subscriptions.create({ channel: "ChatChannel", room: "1st Room" })
consumer.subscriptions.create({ channel: "ChatChannel", room: "2nd Room" })
```

## Client-Server Interactions

### Streams

*Streams* proporcionan el mecanismo por el cual los canales enrutan el contenido publicado
(transmisiones) a sus suscriptores. El siguiente ejemplo
suscríbase a la transmisión `chat_Best Room` si el parámetro de la sala
es `Best Room`:

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
  def subscribed
    stream_from "chat_#{params[:room]}"
  end
end
```

Si tiene una transmisión relacionada con un modelo, la transmisión utilizada
se puede generar a partir del canal y el modelo. El siguiente ejemplo
suscríbete a una transmisión como `comments:Z2lkOi8vVGVzdEFwcC9Qb3N0LzE`,
donde `Z2lkOi8vVGVzdEFwcC9Qb3N0LzE` es el GlobalID del modelo de publicación.

```ruby
class CommentsChannel < ApplicationCable::Channel
  def subscribed
    post = Post.find(params[:id])
    stream_for post
  end
end
```

Luego puede transmitir a este canal de esta manera:

```ruby
CommentsChannel.broadcast_to(@post, @comment)
```

### Broadcasting

Un *broadcasting* es un enlace pub/sub donde cualquier cosa transmitida por un editor
se enruta directamente a los suscriptores del canal que están transmitiendo ese nombre
radiodifusión. Cada canal puede transmitir cero o más transmisiones.

Las transmisiones son puramente una cola en línea y dependen del tiempo. Si un consumidor
no  esta transmitiendo (suscrito a un canal determinado), no recibirán la transmisión
si se conectan más tarde.

Las transmisiones se llaman en otra parte de su aplicación Rails:

```ruby
WebNotificationsChannel.broadcast_to(
  current_user,
  title: 'New things!',
  body: 'All the news fit to print'
)
```

La llamada `WebNotificationsChannel.broadcast_to` coloca un mensaje en el
la cola pubsub del adaptador de suscripción con un nombre de transmisión independiente para cada usuario.
La cola de pubsub predeterminada para Action Cable es `redis` en producción y `async` en desarrollo y
entornos de prueba. Para un usuario con un ID de 1, el nombre de la transmisión sería `web_notifications:1`.

El canal ha recibido instrucciones para transmitir todo lo que llega a
`web_notifications:1` directamente al cliente invocando el `recibido`
llamar de vuelta.

### Subscriptions

Cuando un consumidor está suscrito a un canal, actúa como suscriptor. Esta
conexión se llama suscripción. Los mensajes entrantes luego se enrutan a
estas suscripciones de canal que se basan en un identificador enviado por el consumidor de cable.

```js
// app/javascript/channels/chat_channel.js
// Assumes you've already requested the right to send web notifications
import consumer from "./consumer"

consumer.subscriptions.create({ channel: "ChatChannel", room: "Best Room" }, {
  received(data) {
    this.appendLine(data)
  },

  appendLine(data) {
    const html = this.createLine(data)
    const element = document.querySelector("[data-chat-room='Best Room']")
    element.insertAdjacentHTML("beforeend", html)
  },

  createLine(data) {
    return `
      <article class="chat-line">
        <span class="speaker">${data["sent_by"]}</span>
        <span class="body">${data["body"]}</span>
      </article>
    `
  }
})
```

### Passing Parameters to Channels

Puede pasar parámetros del lado del cliente al lado del servidor al crear un
suscripción. Por ejemplo:

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
  def subscribed
    stream_from "chat_#{params[:room]}"
  end
end
```

Un objeto pasado como primer argumento a `subscriptions.create` se convierte en el
hash de params en el canal de cable. Se requiere la palabra clave `canal`:


```js
// app/javascript/channels/chat_channel.js
import consumer from "./consumer"

consumer.subscriptions.create({ channel: "ChatChannel", room: "Best Room" }, {
  received(data) {
    this.appendLine(data)
  },

  appendLine(data) {
    const html = this.createLine(data)
    const element = document.querySelector("[data-chat-room='Best Room']")
    element.insertAdjacentHTML("beforeend", html)
  },

  createLine(data) {
    return `
      <article class="chat-line">
        <span class="speaker">${data["sent_by"]}</span>
        <span class="body">${data["body"]}</span>
      </article>
    `
  }
})
```

```ruby
# Somewhere in your app this is called, perhaps
# from a NewCommentJob.
ActionCable.server.broadcast(
  "chat_#{room}",
  sent_by: 'Paul',
  body: 'This is a cool chat app.'
)
```

### Rebroadcasting a Message

Un caso de uso común es *rebroadcast* un mensaje enviado por un cliente a cualquier
otros clientes conectados.

```ruby
# app/channels/chat_channel.rb
class ChatChannel < ApplicationCable::Channel
  def subscribed
    stream_from "chat_#{params[:room]}"
  end

  def receive(data)
    ActionCable.server.broadcast("chat_#{params[:room]}", data)
  end
end
```

```js
// app/javascript/channels/chat_channel.js
import consumer from "./consumer"

const chatChannel = consumer.subscriptions.create({ channel: "ChatChannel", room: "Best Room" }, {
  received(data) {
    // data => { sent_by: "Paul", body: "This is a cool chat app." }
  }
}

chatChannel.send({ sent_by: "Paul", body: "This is a cool chat app." })
```

La retransmisión será recibida por todos los clientes conectados, _incluyendo_ el
cliente que envió el mensaje. Tenga en cuenta que los parámetros son los mismos que cuando
te suscribiste al canal.

## Full-Stack Examples

Los siguientes pasos de configuración son comunes a ambos ejemplos:

  1. [Setup your connection](#connection-setup).
  2. [Setup your parent channel](#parent-channel-setup).
  3. [Connect your consumer](#connect-consumer).

### Example 1: User Appearances

Aquí hay un ejemplo simple de un canal que rastrea si un usuario está en línea o no
y en qué página están. (Esto es útil para crear funciones de presencia como mostrar
un punto verde junto a un nombre de usuario si está en línea).

Cree el canal de apariencia del lado del servidor:

```ruby
# app/channels/appearance_channel.rb
class AppearanceChannel < ApplicationCable::Channel
  def subscribed
    current_user.appear
  end

  def unsubscribed
    current_user.disappear
  end

  def appear(data)
    current_user.appear(on: data['appearing_on'])
  end

  def away
    current_user.away
  end
end
```

Cuando se inicia una suscripción, la devolución de llamada `subscribed` se activa y uno
aprovecha la oportunidad para decir "el usuario actual ha aparecido". Ese
API de aparecer/desaparecer podría estar respaldado por Redis, una base de datos o cualquier otra cosa.

Cree la suscripción del canal de apariencia del lado del cliente:


```js
// app/javascript/channels/appearance_channel.js
import consumer from "./consumer"

consumer.subscriptions.create("AppearanceChannel", {
  // Called once when the subscription is created.
  initialized() {
    this.update = this.update.bind(this)
  },

  // Called when the subscription is ready for use on the server.
  connected() {
    this.install()
    this.update()
  },

  // Called when the WebSocket connection is closed.
  disconnected() {
    this.uninstall()
  },

  // Called when the subscription is rejected by the server.
  rejected() {
    this.uninstall()
  },

  update() {
    this.documentIsActive ? this.appear() : this.away()
  },

  appear() {
    // Calls `AppearanceChannel#appear(data)` on the server.
    this.perform("appear", { appearing_on: this.appearingOn })
  },

  away() {
    // Calls `AppearanceChannel#away` on the server.
    this.perform("away")
  },

  install() {
    window.addEventListener("focus", this.update)
    window.addEventListener("blur", this.update)
    document.addEventListener("turbolinks:load", this.update)
    document.addEventListener("visibilitychange", this.update)
  },

  uninstall() {
    window.removeEventListener("focus", this.update)
    window.removeEventListener("blur", this.update)
    document.removeEventListener("turbolinks:load", this.update)
    document.removeEventListener("visibilitychange", this.update)
  },

  get documentIsActive() {
    return document.visibilityState == "visible" && document.hasFocus()
  },

  get appearingOn() {
    const element = document.querySelector("[data-appearing-on]")
    return element ? element.getAttribute("data-appearing-on") : null
  }
})
```

##### Client-Server Interaction

1. **Client** se conecta al **Server** vía `App.cable =
ActionCable.createConsumer("ws://cable.example.com")`. (`cable.js`). El
**Server** identifica esta conexión por `current_user`.

2. **Client** se suscribe al canal de aparición a través de
`consumer.subscriptions.create({ channel: "AppearanceChannel" })`. (`appearance_channel.js`)

3. **Server** reconoce que se ha iniciado una nueva suscripción para el
canal de apariencia y ejecuta su callback `subscribed`, llamando al 
método `appear` en `current_user`. (`appearance_channel.rb`)

4. **Client** reconoce que se ha establecido una suscripción y llama
`connected` (`appearance_channel.js`) que a su vez llama `install` y `appear`.
`appear` llama `AppearanceChannel#appear(data)` en el servidor y proporciona un
hash de datos de `{ appearing_on: this.appearingOn }`. Esto es
posible porque la instancia del canal del lado del servidor expone automáticamente todos
métodos públicos declarados en la clase (menos las devoluciones de llamada), para que estos puedan ser
alcanzado como llamadas de procedimiento remoto a través del método `perform` de una suscripción.

5. **Server** recibe la solicitud de `appear` acción sobre la apariencia
canal para la conexión identificada por `current_user`
(`appearance_channel.rb`). **Server** recupera los datos con el
`:appearing_on` clave del hash de datos y lo establece como el valor para la clave `:on`
que se pasa a `current_user.appear`.

### Example 2: Receiving New Web Notifications

El ejemplo de apariencia tenía que ver con exponer la funcionalidad del servidor a
invocación del lado del cliente a través de la conexión WebSocket. Pero la gran cosa
sobre WebSockets es que es una calle de doble sentido. Así que ahora mostremos un ejemplo
donde el servidor invoca una acción en el cliente.

Este es un canal de notificación web que le permite activar el lado del cliente
notificaciones web cuando transmite a las transmisiones correctas:

Cree el canal de notificaciones web del lado del servidor:

```ruby
# app/channels/web_notifications_channel.rb
class WebNotificationsChannel < ApplicationCable::Channel
  def subscribed
    stream_for current_user
  end
end
```

Cree la suscripción al canal de notificaciones web del lado del cliente:

```js
// app/javascript/channels/web_notifications_channel.js
// Client-side which assumes you've already requested
// the right to send web notifications.
import consumer from "./consumer"

consumer.subscriptions.create("WebNotificationsChannel", {
  received(data) {
    new Notification(data["title"], body: data["body"])
  }
})
```

Transmita contenido a una instancia de canal de notificación web desde cualquier lugar de su
solicitud:

```ruby
# Somewhere in your app this is called, perhaps from a NewCommentJob
WebNotificationsChannel.broadcast_to(
  current_user,
  title: 'New things!',
  body: 'All the news fit to print'
)
```

La llamada `WebNotificationsChannel.broadcast_to` coloca un mensaje en el
la cola pubsub del adaptador de suscripción con un nombre de transmisión independiente para cada
usuario. Para un usuario con un ID de 1, el nombre de la transmisión sería
`web_notifications: 1`.

El canal ha recibido instrucciones para transmitir todo lo que llega a
`web_notifications: 1` directamente al cliente invocando el `recibido`
llamar de vuelta. Los datos pasados ​​como argumento son el hash enviado como segundo parámetro
a la llamada de difusión del lado del servidor, codificado JSON para el viaje a través del cable
y desempaquetado para el argumento de datos que llega como "recibido".

### More Complete Examples

Ver el [rails/actioncable-examples](https://github.com/rails/actioncable-examples)
repositorio para ver un ejemplo completo de cómo configurar Action Cable en una aplicación Rails y agregar canales.

## Configuration

Action Cable tiene dos configuraciones requeridas: un adaptador de suscripción y orígenes de solicitud permitidos.

### Subscription Adapter

De forma predeterminada, Action Cable busca un archivo de configuración en `config/cable.yml`.
El archivo debe especificar un adaptador para cada entorno de Rails. Ver el
Sección [Dependencies](#dependencies) para obtener información adicional sobre adaptadores.

```yaml
development:
  adapter: async

test:
  adapter: async

production:
  adapter: redis
  url: redis://10.10.3.153:6381
  channel_prefix: appname_production
```
#### Adapter Configuration

Below is a list of the subscription adapters available for end users.

##### Async Adapter

El adaptador asincrónico está diseñado para desarrollo/prueba y no debe usarse en producción.

##### Redis Adapter

El adaptador de Redis requiere que los usuarios proporcionen una URL que apunte al servidor de Redis.
Además, se puede proporcionar un `prefijo_canal` para evitar colisiones de nombres de canales
cuando se usa el mismo servidor Redis para múltiples aplicaciones. Ver el [Redis PubSub documentation](https://redis.io/topics/pubsub#database-amp-scoping) para más detalles.

 ##### PostgreSQL Adapter

El adaptador PostgreSQL usa el grupo de conexiones de Active Record y, por lo tanto, el
configuración de la base de datos `config / database.yml` de la aplicación, para su conexión.
Esto puede cambiar en el futuro. [#27214](https://github.com/rails/rails/issues/27214)

### Allowed Request Origins

Action Cable solo aceptará solicitudes de orígenes específicos, que son
pasado a la configuración del servidor como una matriz. Los orígenes pueden ser instancias de
cadenas o expresiones regulares, contra las cuales se realizará una comprobación de la coincidencia.

```ruby
config.action_cable.allowed_request_origins = ['https://rubyonrails.com', %r{http://ruby.*}]
```

Para deshabilitar y permitir solicitudes de cualquier origen:

```ruby
config.action_cable.disable_request_forgery_protection = true
```

De forma predeterminada, Action Cable permite todas las solicitudes de localhost: 3000 cuando se ejecuta
en el entorno de desarrollo.

### Consumer Configuration

Para configurar la URL, agregue una llamada a `action_cable_meta_tag` en su diseño HTML
CABEZA. Esto utiliza una URL o ruta establecida normalmente a través de `config.action_cable.url` en el
archivos de configuración del entorno.

### Worker Pool Configuration

El grupo de trabajadores se utiliza para ejecutar devoluciones de llamada de conexión y acciones de canal en
aislamiento del hilo principal del servidor. Action Cable le permite a la aplicación
a configurar el número de subprocesos procesados simultáneamente en el grupo de trabajadores.

```ruby
config.action_cable.worker_pool_size = 4
```

Además, tenga en cuenta que su servidor debe proporcionar al menos el mismo número de bases de datos
conexiones como tienes trabajadores. El tamaño del grupo de trabajadores predeterminado se establece en 4, por lo que
eso significa que se debe hacer disponibles por lo menos 4 conexiones de base de datos.
Puede cambiar eso en `config/database.yml` a través del atributo` pool`.

### Other Configurations

La otra opción común para configurar son las etiquetas de registro aplicadas al
registrador por conexión. Aquí hay un ejemplo en cual se usa
la identificación de la cuenta de usuario si está disponible, de lo contrario "sin cuenta" al etiquetar:

```ruby
config.action_cable.log_tags = [
  -> request { request.env['user_account_id'] || "no-account" },
  :action_cable,
  -> request { request.uuid }
]
```

Para obtener una lista completa de todas las opciones de configuración, consulte la
Clase `ActionCable :: Server :: Configuration`

## Running Standalone Cable Servers

### In App

Action Cable puede funcionar junto con su aplicación Rails. Por ejemplo, para
escuchar las solicitudes de WebSocket en `/ websocket`, especifique esa ruta en
`config.action_cable.mount_path`:

```ruby
# config/application.rb
class Application < Rails::Application
  config.action_cable.mount_path = '/websocket'
end
```

Puede utilizar `ActionCable.createConsumer()` para conectarse al cable
servidor si se invoca `action_cable_meta_tag` en el diseño. De lo contrario, un camino es
especificado como primer argumento para `createConsumer` (por ejemplo, `ActionCable.createConsumer("/websocket")`).

Para cada instancia de su servidor que cree y para cada trabajador su servidor
genera, también tendrá una nueva instancia de Action Cable, pero el uso de Redis
mantiene los mensajes sincronizados entre conexiones.

### Standalone

Los servidores de cable se pueden separar de su servidor de aplicaciones normal. Sus
sigue siendo una aplicación de Rack, pero es su propia aplicación de Rack. El recomendado
La configuración básica es la siguiente:

```ruby
# cable/config.ru
require_relative "../config/environment"
Rails.application.eager_load!

run ActionCable.server
```

Luego inicias el servidor usando un binstub en `bin / cable` ala:

```
#!/bin/bash
bundle exec puma -p 28080 cable/config.ru
```

Lo anterior iniciará un servidor de cable en el puerto 28080.

El servidor WebSocket no tiene acceso a la sesión, pero tiene
acceso a las cookies. Esto se puede utilizar cuando necesite manipular
autenticación. Puede ver una forma de hacerlo con Devise en este [article](https://greg.molnar.io/blog/actioncable-devise-authentication/).

## Dependencies

Action Cable proporciona una interfaz de adaptador de suscripción para procesar su
internos de pubsub. De forma predeterminada adaptadores incluidos son, asincrónica, en línea, PostgreSQL y Redis.
El adaptador predeterminado en las nuevas aplicaciones Rails es el adaptador asincrónico (`async`).

El lado Ruby de las cosas se basa en [websocket-driver](https://github.com/faye/websocket-driver-ruby),
[nio4r](https://github.com/celluloid/nio4r), y [concurrent-ruby](https://github.com/ruby-concurrency/concurrent-ruby).

## Deployment

Action Cable funciona con una combinación de WebSockets e hilos. Ambos
la plomería del marco y el trabajo de canal especificado por el usuario son manejados internamente por
utilizando el soporte de hilos nativos de Ruby. Esto significa que puede usar todos sus
Modelos de rieles sin problemas, siempre que no haya cometido ningún pecado de seguridad de hilos.

El servidor de Action Cable implementa la API de secuestro de sockets de Rack,
permitiendo así el uso de un patrón multiproceso para gestionar conexiones
internamente, independientemente de si el servidor de aplicaciones es multiproceso o no.

En consecuencia, Action Cable funciona con servidores populares como Unicorn, Puma y
Pasajero.

## Testing

Puede encontrar instrucciones detalladas sobre cómo probar la funcionalidad de su Action Cable en el 
guía [testing guide](testing.html#testing-action-cable).
