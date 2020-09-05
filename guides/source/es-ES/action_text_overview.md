**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Descripción General de Action Text
==================================

Este guía le proporciona todo lo que necesita para comenzar a manejar
contenido de texto enriquecido.

Después de leer esta guía, sabrá:

* Cómo configurar Action Text.
* Cómo manejar contenido de texto enriquecido.
* Cómo diseñar contenido de texto enriquecido.

--------------------------------------------------------------------------------

What is Action Text?
--------------------

Action Text trae contenido de texto enriquecido y edición a Rails. Incluye
el [Trix editor](https://trix-editor.org) que maneja todo, desde el formateo
a enlaces a citas a listas a imágenes y galerías incrustadas.
El contenido de texto enriquecido generado por el editor Trix se guarda en su propio
Modelo RichText que está asociado con cualquier modelo de Active Record existente en la aplicación.
Todas las imágenes incrustadas (u otros archivos adjuntos) se almacenan automáticamente utilizando
Active Storage y asociado con el modelo RichText incluido.

## Trix compared to other rich text editors

La mayoría de los editores WYSIWYG son envoltorios de las API `contenteditable` y `execCommand` de HTML,
diseñado por Microsoft para admitir la edición en vivo de páginas web en Internet Explorer 5.5,
y [eventually reverse-engineered](https://blog.whatwg.org/the-road-to-html-5-contenteditable#history)
y copiado por otros navegadores.

Debido a que estas API nunca se especificaron ni documentaron completamente,
y debido a que los editores HTML WYSIWYG tienen un alcance enorme, cada
la implementación del navegador tiene su propio conjunto de errores y peculiaridades,
y los desarrolladores de JavaScript deben resolver las inconsistencias.

Trix elude estas inconsistencias al tratar los
como dispositivo de I/O: cuando la entrada llega al editor, Trix convierte esa entrada
en una operación de edición en su modelo de documento interno, luego vuelve a renderizar
ese documento de vuelta al editor. Esto le da a Trix control total sobre lo que
ocurre después de cada pulsación de tecla y evita la necesidad de utilizar execCommand en absoluto.

## Installation

Ejecute `bin/rails action_text:install` para agregar el paquete Yarn y copie la migración necesaria. Además, debe configurar Active Storage para imágenes incrustadas y otros archivos adjuntos. Consulte la guía [Active Storage Overview](active_storage_overview.html).

Una vez completada la instalación, una aplicación Rails que utilice Webpacker debería tener los siguientes cambios:

1. Tanto `trix` como` @ rails/actiontext` deberían ser necesarios en su paquete de JavaScript.

   ```js
    // application.js
    require("trix")
    require("@rails/actiontext")
    ```

2. La hoja de estilo `trix` debe importarse a `actiontext.scss`.

    ```scss
    @import "trix/dist/trix";
    ```
    
    Además, este archivo `actiontext.scss` debe importarse a su paquete de hojas de estilo.
    
    ```
    // application.scss
    @import "./actiontext.scss";
    ```

## Examples

Agregar un campo de texto enriquecido a un modelo existente:

```ruby
# app/models/message.rb
class Message < ApplicationRecord
  has_rich_text :content
end
```

Tenga en cuenta que no necesita agregar un campo de `content` a su tabla de `messages`.

Luego, consulte este campo en el formulario del modelo:

```erb
<%# app/views/messages/_form.html.erb %>
<%= form_with model: message do |form| %>
  <div class="field">
    <%= form.label :content %>
    <%= form.rich_text_area :content %>
  </div>
<% end %>
```

Y finalmente, muestre el texto enriquecido desinfectado en una página:

```erb
<%= @message.content %>
```

Para aceptar el contenido de texto enriquecido, todo lo que tiene que hacer es permitir el atributo al que se hace referencia:

```ruby
class MessagesController < ApplicationController
  def create
    message = Message.create! params.require(:message).permit(:title, :content)
    redirect_to message
  end
end
```

## Avoid N+1 queries

Si desea precargar la dependiente modelo `ActionText::RichText`, puede usar el alcance con nombre:

```ruby
Message.all.with_rich_text_content # Preload the body without attachments.
Message.all.with_rich_text_content_and_embeds # Preload both body and attachments.
```

## Custom styling

De forma predeterminada, el contenido y el editor de texto de acción tienen el estilo predeterminado de Trix.
Si desea cambiar estos valores predeterminados, querrá eliminar
el enlazador `app/assets/stylesheets/actiontext.scss` y basa tus estilos en
el [contents of that file](https://raw.githubusercontent.com/basecamp/trix/master/dist/trix.css).

También puede diseñar el HTML utilizado para imágenes incrustadas y otros archivos adjuntos (conocidos como blobs).
En la instalación, Action Text se copiará en una parte de
`app/views/active_storage/blobs/_blob.html.erb`, que puede especializarse.

## API / Backend development

1. Una API de backend (por ejemplo, usando JSON) necesita un punto final separado para subir archivos que crea un `ActiveStorage::Blob` y devuelve su `attachable_sgid`:
    ```json
    {
      "attachable_sgid": "BAh7CEkiCG…"
    }
    ```

2. Tome ese `attachable_sgid` y pídale a su interfaz que lo inserte en contenido de texto enriquecido usando una etiqueta `<action-text-attachment>`:

    ```html
    <action-text-attachment sgid="BAh7CEkiCG…"></action-text-attachment>
    ```

Esto se basa en Basecamp, por lo que si aún no puede encontrar lo que está buscando, consulte este [Basecamp Doc](https://github.com/basecamp/bc3-api/blob/master/sections/rich_text.md).
