**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Active Storage Overview
=======================

Este guía cubre cómo adjuntar archivos a sus modelos de Active Record.

Después de leer este guía, sabrá:

* Cómo adjuntar uno o varios archivos a un registro.
* Cómo eliminar un archivo adjunto.
* Cómo vincular a un archivo adjunto.
* Cómo utilizar variantes para transformar imágenes.
* Cómo generar una representación de imagen de un archivo que no es de imagen, como un PDF o un video.
* Cómo enviar cargas de archivos directamente desde navegadores a un servicio de almacenamiento,
  pasando por alto sus servidores de aplicaciones.
* Cómo limpiar archivos almacenados durante la prueba.
* Cómo implementar soporte para servicios de almacenamiento adicionales.

--------------------------------------------------------------------------------

What is Active Storage?
-----------------------

Active Storage facilita la carga de archivos a un servicio de almacenamiento en la nube como
Amazon S3, Google Cloud Storage o Microsoft Azure Storage y adjuntar los
archivos a objetos Active Record. Viene con un servicio basado en disco local para
desarrollo y pruebas y admite archivos de duplicación para subordinar servicios para
copias de seguridad y migraciones.

Con Active Storage, una aplicación puede transformar la carga de imágenes con
[ImageMagick](https://www.imagemagick.org), genera representaciones de imágenes de
archivos.

## Setup

Active Storage utiliza dos tablas en la base de datos de su aplicación llamadas
`active_storage_blobs` y `active_storage_attachments`. Después de crear un nuevo
aplicación (o actualizando su aplicación a Rails 5.2), ejecute
`bin/rails active_storage:install` para generar una migración que cree estos
mesas. Utilice `bin/rails db:migrate` para ejecutar la migración.

WARNING: `active_storage_attachments` es una tabla de combinación polimórfica que almacena el nombre de la clase de su modelo. Si el nombre de la clase de su modelo cambia, deberá ejecutar una migración en esta tabla para actualizar el `record_type` subyacente al nuevo nombre de la clase de su modelo.

WARNING: Si está utilizando UUID en lugar de números enteros como clave principal en sus modelos, deberá cambiar el tipo de columna de `record_id` para la tabla `active_storage_attachments` en la migración generada en consecuencia.

Declare los servicios de almacenamiento activo en `config/storage.yml`. Para cada servicio su
usos de la aplicación, proporcione un nombre y la configuración requerida. El ejemplo
a continuación se declaran tres servicios denominados `local`, `test` y `amazon`:


```yaml
local:
  service: Disk
  root: <%= Rails.root.join("storage") %>

test:
  service: Disk
  root: <%= Rails.root.join("tmp/storage") %>

amazon:
  service: S3
  access_key_id: ""
  secret_access_key: ""
  bucket: ""
  region: "" # e.g. 'us-east-1'
```

Dile a Active Storage cual servicio debe de usar configurando
`Rails.application.config.active_storage.service`. Porque cada entorno
probablemente utilice un servicio diferente, se recomienda hacerlo en un
por medio ambiente. Para utilizar el servicio de disco del ejemplo anterior en el
entorno de desarrollo, agregaría lo siguiente a
`config/environment/development.rb`:

```ruby
# Store files locally.
config.active_storage.service = :local
```

Para utilizar el servicio S3 en producción, agregue lo siguiente a
`config/environments/production.rb`:

```ruby
# Store files on Amazon S3.
config.active_storage.service = :amazon
```
Para utilizar el servicio de prueba durante la prueba, agregue lo siguiente a
`config/environments/test.rb`:

```ruby
# Store uploaded files on the local file system in a temporary directory.
config.active_storage.service = :test
```

Continúe leyendo para obtener más información sobre los adaptadores de servicio integrados (p. Ej.
`Disk` y` S3`) y la configuración que requieren.

### Disk Service

Declare un servicio de disco en `config/storage.yml`:

```yaml
local:
  service: Disk
  root: <%= Rails.root.join("storage") %>
```

### S3 Service (Amazon S3 and S3-compatible APIs)

Para conectarse a Amazon S3, declare un servicio S3 en `config/storage.yml`:

```yaml
amazon:
  service: S3
  access_key_id: ""
  secret_access_key: ""
  region: ""
  bucket: ""
```

Opcionalmente, proporcione un hash de opciones de carga:

```yaml
amazon:
  service: S3
  access_key_id: ""
  secret_access_key: ""
  region: ""
  bucket: ""
  upload: 
    server_side_encryption: "" # 'aws:kms' or 'AES256'
```

Añade la gema [`aws-sdk-s3`](https://github.com/aws/aws-sdk-ruby) a su `Gemfile`:

```ruby
gem "aws-sdk-s3", require: false
```

NOTE: Las características principales de Active Storage requieren los siguientes permisos: `s3:ListBucket`,` s3:PutObject`, `s3:GetObject` y `s3:DeleteObject`. Si tiene opciones de carga adicionales configuradas, como establecer ACL, es posible que se requieran permisos adicionales.

NOTE: Si desea utilizar variables de entorno, archivos de configuración estándar del SDK, perfiles,
Perfiles de instancia de IAM o roles de tareas, puede omitir el `access_key_id`, `secret_access_key`,
y las claves `region` en el ejemplo anterior. El servicio S3 admite todos las
opciones de autenticación descritas en el [AWS SDK documentation]
(https://docs.aws.amazon.com/sdk-for-ruby/v3/developer-guide/setup-config.html).

Para conectarse a una API de almacenamiento de objetos compatible con S3, como Digital Ocean Spaces, proporcione el `endpoint`:

```yaml
digitalocean:
  service: S3
  endpoint: https://nyc3.digitaloceanspaces.com
  access_key_id: ...
  secret_access_key: ...
  # ...and other options
```

### Microsoft Azure Storage Service

Declare un servicio de Azure Storage en `config/storage.yml`:

```yaml
azure:
  service: AzureStorage
  storage_account_name: ""
  storage_access_key: ""
  container: ""
```

Añade la gema [`azure-storage-blob`](https://github.com/Azure/azure-storage-ruby) a su `Gemfile`:

```ruby
gem "azure-storage-blob", require: false
```

### Google Cloud Storage Service

Declare un servicio de Google Cloud Storage en `config/storage.yml`:

```yaml
google:
  service: GCS
  credentials: <%= Rails.root.join("path/to/keyfile.json") %>
  project: ""
  bucket: ""
```

Opcionalmente, proporcione un hash de credenciales en lugar de una ruta de archivo de claves:

```yaml
google:
  service: GCS
  credentials:
    type: "service_account"
    project_id: ""
    private_key_id: <%= Rails.application.credentials.dig(:gcs, :private_key_id) %>
    private_key: <%= Rails.application.credentials.dig(:gcs, :private_key).dump %>
    client_email: ""
    client_id: ""
    auth_uri: "https://accounts.google.com/o/oauth2/auth"
    token_uri: "https://accounts.google.com/o/oauth2/token"
    auth_provider_x509_cert_url: "https://www.googleapis.com/oauth2/v1/certs"
    client_x509_cert_url: ""
  project: ""
  bucket: ""
```

Añade la gema [`google-cloud-storage`](https://github.com/GoogleCloudPlatform/google-cloud-ruby/tree/master/google-cloud-storage) a su `Gemfile`:

```ruby
gem "google-cloud-storage", "~> 1.11", require: false
```

### Mirror Service

Puede mantener varios servicios sincronizados definiendo un servicio espejo. Un espejo
servicio replica cargas y elimina en dos o más servicios subordinados.

Un servicio espejo está diseñado para usarse temporalmente durante una migración entre
servicios en producción. Puede comenzar a duplicar en un nuevo servicio, copie
archivos preexistentes del servicio antiguo al nuevo, luego haga todo lo posible por el nuevo
Servicio.

NOTE: La duplicación no es atómica. Es posible que una carga se realice correctamente en el
servicio primario y falla en cualquiera de los servicios subordinados. Antes de ir
todo incluido en un nuevo servicio, verifique que se hayan copiado todos los archivos.

Defina cada uno de los servicios que le gustaría reflejar como se describe arriba. Referencia
ellos por su nombre al definir un servicio espejo:

```yaml
s3_west_coast:
  service: S3
  access_key_id: ""
  secret_access_key: ""
  region: ""
  bucket: ""

s3_east_coast:
  service: S3
  access_key_id: ""
  secret_access_key: ""
  region: ""
  bucket: ""

production:
  service: Mirror
  primary: s3_east_coast
  mirrors:
    - s3_west_coast
```

Aunque todos los servicios secundarios reciben cargas, las descargas siempre se manejan
por el servicio primario.

Los servicios espejo son compatibles con cargas directas. Los archivos nuevos son directamente
subido al servicio principal. Cuando un archivo cargado directamente se adjunta a un
registro, se pone en cola un trabajo en segundo plano para copiarlo en los servicios secundarios.

### Public access

De forma predeterminada, Active Storage asume el acceso privado a los servicios. Esto significa generar URL firmadas y de un solo uso para blobs. Si prefiere que los blobs sean accesibles públicamente, especifique `public: true` en el archivo `config/storage.yml` de su aplicación:

```yaml
gcs: &gcs
  service: GCS
  project: ""

private_gcs:
  <<: *gcs
  credentials: <%= Rails.root.join("path/to/private_keyfile.json") %>
  bucket: ""

public_gcs:
  <<: *gcs
  credentials: <%= Rails.root.join("path/to/public_keyfile.json") %>
  bucket: ""
  public: true
```

Asegúrese de que sus depósitos estén configurados correctamente para el acceso público. Consulte los documentos sobre cómo habilitar permisos de lectura públicos para [Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/block-public-access-bucket.html), [Google Cloud Storage ](https://cloud.google.com/storage/docs/access-control/making-data-public#buckets) y [Microsoft Azure](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-manage-access-to-resources#set-container-public-access-level-in-the-azure-portal) servicios de almacenamiento.

Attaching Files to Records
--------------------------

### `has_one_attached`

La macro `has_one_attached` configura un mapeo uno a uno entre registros y
archivos. Cada registro puede tener un archivo adjunto.

Por ejemplo, suponga que su aplicación tiene un modelo `User`. Si desea que cada usuario
tener un avatar, defina el modelo `User` así:

```ruby
class User < ApplicationRecord
  has_one_attached :avatar
end
```

Puedes crear una usuario con un avatar:


```erb
<%= form.file_field :avatar %>
```

```ruby
class SignupController < ApplicationController
  def create
    user = User.create!(user_params)
    session[:user_id] = user.id
    redirect_to root_path
  end

  private
    def user_params
      params.require(:user).permit(:email_address, :password, :avatar)
    end
end
```

Llame a `avatar.attach` para adjuntar un avatar a un usuario existente:

```ruby
user.avatar.attach(params[:avatar])
```

Llame a `avatar.attached?` para determinar si un usuario en particular tiene un avatar:

```ruby
user.avatar.attached?
```

En algunos casos, es posible que desee anular un servicio predeterminado para un archivo adjunto específico.
Puede configurar servicios específicos por archivo adjunto usando la opción `service`:

```ruby
class User < ApplicationRecord
  has_one_attached :avatar, service: :s3
end
```

### `has_many_attached`

La macro `has_many_attached` establece una relación de uno a varios entre registros
y archivos. Cada registro puede tener muchos archivos adjuntos.

Por ejemplo, suponga que su aplicación tiene un modelo de `Message`. Si quieres cada uno
mensaje para tener muchas imágenes, defina el modelo `Message` así:

```ruby
class Message < ApplicationRecord
  has_many_attached :images
end
```

Puedes crear un mensaje con imágenes:

```ruby
class MessagesController < ApplicationController
  def create
    message = Message.create!(message_params)
    redirect_to message
  end

  private
    def message_params
      params.require(:message).permit(:title, :content, images: [])
    end
end
```

Llame a `images.attach` para agregar nuevas imágenes a un mensaje existente:

```ruby
@message.images.attach(params[:images])
```

Llame a `images.attached?` Para determinar si un mensaje en particular tiene imágenes:

```ruby
@message.images.attached?
```

La anulación del servicio predeterminado se hace de la misma manera que `has_one_attached`, usando la opción `service`:

```ruby
class Message < ApplicationRecord
  has_many_attached :images, service: :s3
end
```

### Attaching File/IO Objects

A veces es necesario adjuntar un archivo que no llega a través de una solicitud HTTP.
Por ejemplo, es posible que desee adjuntar un archivo que generó en el disco o descargó
desde una URL enviada por el usuario. Es posible que también desee adjuntar un archivo de accesorio en un
prueba modelo. Para hacer eso, proporcione un Hash que contenga al menos un objeto IO abierto
y un nombre de archivo:


```ruby
@message.image.attach(io: File.open('/path/to/file'), filename: 'file.pdf')
```

Cuando sea posible, proporcione también un tipo de contenido. Active Storage intenta
determinar el tipo de contenido de un archivo a partir de sus datos. Vuelve al contenido
tipo que proporcione si no puede hacer eso.

```ruby
@message.image.attach(io: File.open('/path/to/file'), filename: 'file.pdf', content_type: 'application/pdf')
```

Puede omitir la inferencia del tipo de contenido a partir de los datos pasando
`identificar: false` junto con el `content_type`.

```ruby
@message.image.attach(
  io: File.open('/path/to/file'),
  filename: 'file.pdf',
  content_type: 'application/pdf',
  identify: false
)
```

Si no proporciona un tipo de contenido y Active Storage no puede determinar el
el tipo de contenido del archivo automáticamente, el valor predeterminado es application/octet-stream.


Removing Files
--------------

Para eliminar un archivo adjunto de un modelo, llame a `purge` en el archivo adjunto. Eliminación
se puede hacer en segundo plano si su aplicación está configurada para usar Active Job.
La purga elimina el blob y el archivo del servicio de almacenamiento.

```ruby
# Synchronously destroy the avatar and actual resource files.
user.avatar.purge

# Destroy the associated models and actual resource files async, via Active Job.
user.avatar.purge_later
```

Linking to Files
----------------

Genere una URL permanente para el blob que apunta a la aplicación. Sobre
acceso, se devuelve una redirección al punto final del servicio real. Esta indirección
desacopla la URL del servicio de la real y permite, por ejemplo, duplicar
anexos en diferentes servicios para alta disponibilidad. La redirección tiene un
Caducidad HTTP de 5 min.

```ruby
url_for(user.avatar)
```


Para crear un enlace de descarga, use el ayudante `rails_blob_{path|url}`. Usando esto
helper le permite establecer la disposición.

```ruby
rails_blob_path(user.avatar, disposition: "attachment")
```

WARNING: Para evitar ataques XSS, ActiveStorage fuerza el encabezado Content-Disposition
a "archivo adjunto" para algunos tipos de archivos. Para cambiar este comportamiento, consulte la
opciones de configuración disponibles en  [Configuring Rails Applications](configuring.html#configuring-active-storage).

Si necesita crear un enlace desde fuera del contexto del controlador/vista (Background
jobs, Cronjobs, etc.), puede acceder a rails_blob_path así

```ruby
Rails.application.routes.url_helpers.rails_blob_path(user.avatar, only_path: true)
```

Downloading Files
-----------------

A veces, es necesario procesar un blob después de que se carga, por ejemplo, para convertir
a un formato diferente. Use `ActiveStorage::Blob#download` para leer un blob
datos binarios en la memoria:


```ruby
binary = user.avatar.download
```

Es posible que desee descargar un blob en un archivo en el disco para que un programa externo (p. Ej.
un escáner de virus o un transcodificador de medios) pueden operar en él. Utilizar
`ActiveStorage::Blob open` para descargar un blob a un archivo temporal en el disco:

```ruby
message.video.open do |file|
  system '/path/to/virus/scanner', file.path
  # ...
end
```

Es importante saber que el archivo aún no está disponible en la devolución de llamada ``after_create`, sino solo en el `after_create_commit.

Analyzing Files
---------------

Active Storage [analyzes](https://api.rubyonrails.org/classes/ActiveStorage/Blob/Analyzable.html#method-i-analyze) archivos una vez que se hayan cargado poniendo en cola un trabajo en Trabajo activo. Los archivos analizados almacenarán información adicional en el hash de metadatos, incluido `analyzed: true`. Puede comprobar si un blob ha sido analizado llamando a `analyzed?` En él.

El análisis de imágenes proporciona atributos de `width` y height`. El análisis de video proporciona estos datos, así como la `duration`, `angle`, y `display_aspect_ratio`.

El análisis requiere la gema `mini_magick`. El análisis de video también requiere la biblioteca [FFmpeg](https://www.ffmpeg.org/), que debe incluir por separado.

Transforming Images
-------------------

Para habilitar variantes, agregue la gema `image_processing` a su` Gemfile`:

```ruby
gem 'image_processing'
```

Para crear una variación de una imagen, llame a `variant` en el `Blob`. Puede pasar cualquier transformación al método admitido por el procesador. El procesador predeterminado para Active Storage es MiniMagick, pero también puede usar [Vips] (https://www.rubydoc.info/gems/ruby-vips/Vips/Image).

Cuando el navegador accede a la URL variante, Active Storage transformará la
blob original en el formato especificado y redirigir a su nuevo servicio
ubicación.

```erb
<%= image_tag user.avatar.variant(resize_to_limit: [100, 100]) %>
```

Para cambiar al procesador Vips, debe agregar lo siguiente a
`config/application.rb`:

```ruby
# Use Vips for processing variants.
config.active_storage.variant_processor = :vips
```

Previewing Files
----------------

Se pueden obtener una vista previa de algunos archivos que no son de imagen: es decir, se pueden presentar como imágenes.
Por ejemplo, se puede obtener una vista previa de un archivo de video extrayendo su primer fotograma. Fuera de
En la caja, Active Storage admite la vista previa de videos y documentos PDF.

```erb
<ul>
  <% @message.files.each do |file| %>
    <li>
      <%= image_tag file.preview(resize_to_limit: [100, 100]) %>
    </li>
  <% end %>
</ul>
```

WARNING: La extracción de vistas previas requiere aplicaciones de terceros, FFmpeg para
video y muPDF para PDF, y en macOS también XQuartz y Poppler.
Rails no proporciona estas bibliotecas. Debe instalarlos usted mismo para
utilice los previsualizadores integrados. Antes de instalar y utilizar software de terceros,
asegúrese de comprender las implicaciones de licencias de hacerlo.

Direct Uploads
--------------

Active Storage, con su biblioteca JavaScript incluida, admite la carga
directamente del cliente a la nube

### Usage

1. Incluya `activestorage.js` en el paquete de JavaScript de su aplicación.

    Usando la canalización de activos::

    ```js
    //= require activestorage

    ```

    Usando el paquete npm:

    ```js
    require("@rails/activestorage").start()
    ```

2. Anote las entradas del archivo con la URL de carga directa

    ```erb
    <%= form.file_field :attachments, multiple: true, direct_upload: true %>
    ```

3. Configure CORS en servicios de almacenamiento de terceros para permitir solicitudes de carga directa.

4. ¡Eso es! Las cargas comienzan con el envío del formulario.

### Cross-Origin Resource Sharing (CORS) configuration

Para que las cargas directas a un servicio de terceros funcionen, deberá configurar el servicio para permitir solicitudes de origen cruzado desde su aplicación. Consulte la documentación de CORS para su servicio:

* [S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html#how-do-i-enable-cors)
* [Google Cloud Storage](https://cloud.google.com/storage/docs/configuring-cors)
* [Azure Storage](https://docs.microsoft.com/en-us/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services)

Tenga cuidado de permitir:

* Todos los orígenes desde los que se accede a su aplicación
* El método de solicitud `PUT`
* Los siguientes encabezados:
  * `Origin`
  * `Content-Type`
  * `Content-MD5`
  * `Content-Disposition` (excepto para Azure Storage)
  * `x-ms-blob-content-disposition` (solamente para Azure Storage)
  * `x-ms-blob-type` (solamente para Azure Storage )

No se requiere configuración de CORS para el servicio de disco, ya que comparte el origen de su aplicación.

#### Example: S3 CORS configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>https://www.example.com</AllowedOrigin>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedHeader>Origin</AllowedHeader>
    <AllowedHeader>Content-Type</AllowedHeader>
    <AllowedHeader>Content-MD5</AllowedHeader>
    <AllowedHeader>Content-Disposition</AllowedHeader>
    <MaxAgeSeconds>3600</MaxAgeSeconds>
</CORSRule>
</CORSConfiguration>
```

#### Example: Google Cloud Storage CORS configuration

```json
[
  {
    "origin": ["https://www.example.com"],
    "method": ["PUT"],
    "responseHeader": ["Origin", "Content-Type", "Content-MD5", "Content-Disposition"],
    "maxAgeSeconds": 3600
  }
]
```

#### Example: Azure Storage CORS configuration

```xml
<Cors>
  <CorsRule>
    <AllowedOrigins>https://www.example.com</AllowedOrigins>
    <AllowedMethods>PUT</AllowedMethods>
    <AllowedHeaders>Origin, Content-Type, Content-MD5, x-ms-blob-content-disposition, x-ms-blob-type</AllowedHeaders>
    <MaxAgeInSeconds>3600</MaxAgeInSeconds>
  </CorsRule>
<Cors>
```

### Direct upload JavaScript events

| Event name | Event target | Event data (`event.detail`) | Description |
| --- | --- | --- | --- |
| `direct-uploads:start` | `<form>` | Ninguna | Se envió un formulario que contiene archivos para campos de carga directa. |
| `direct-upload:initialize` | `<input>` | `{id, file}` | Se envía para cada archivo después del envío del formulario. |
| `direct-upload:start` | `<input>` | `{id, file}` | Está comenzando una carga directa. |
| `direct-upload:before-blob-request` | `<input>` | `{id, file, xhr}` | Antes de realizar una solicitud a su aplicación para la carga directa de metadatos. |
| `direct-upload:before-storage-request` | `<input>` | `{id, file, xhr}` | Antes de realizar una solicitud para almacenar un archivo. |
| `direct-upload:progress` | `<input>` | `{id, file, progress}` | A medida que avanzan las solicitudes para almacenar archivos. |
| `direct-upload:error` | `<input>` | `{id, file, error}` | Ocurrió un error. Se mostrará una `alert` a menos que se cancele este evento. |
| `direct-upload:end` | `<input>` | `{id, file}` | Ha finalizado una carga directa. |
| `direct-uploads:end` | `<form>` | Ninguna | Todas las cargas directas han finalizado. |

### Example

Puede utilizar estos eventos para mostrar el progreso de una carga.

![direct-uploads](https://user-images.githubusercontent.com/5355/28694528-16e69d0c-72f8-11e7-91a7-c0b8cfc90391.gif)

Para mostrar los archivos cargados en un formulario:


```js
// direct_uploads.js

addEventListener("direct-upload:initialize", event => {
  const { target, detail } = event
  const { id, file } = detail
  target.insertAdjacentHTML("beforebegin", `
    <div id="direct-upload-${id}" class="direct-upload direct-upload--pending">
      <div id="direct-upload-progress-${id}" class="direct-upload__progress" style="width: 0%"></div>
      <span class="direct-upload__filename"></span>
    </div>
  `)
  target.previousElementSibling.querySelector(`.direct-upload__filename`).textContent = file.name
})

addEventListener("direct-upload:start", event => {
  const { id } = event.detail
  const element = document.getElementById(`direct-upload-${id}`)
  element.classList.remove("direct-upload--pending")
})

addEventListener("direct-upload:progress", event => {
  const { id, progress } = event.detail
  const progressElement = document.getElementById(`direct-upload-progress-${id}`)
  progressElement.style.width = `${progress}%`
})

addEventListener("direct-upload:error", event => {
  event.preventDefault()
  const { id, error } = event.detail
  const element = document.getElementById(`direct-upload-${id}`)
  element.classList.add("direct-upload--error")
  element.setAttribute("title", error)
})

addEventListener("direct-upload:end", event => {
  const { id } = event.detail
  const element = document.getElementById(`direct-upload-${id}`)
  element.classList.add("direct-upload--complete")
})
```

Agregar estilos:

```css
/* direct_uploads.css */

.direct-upload {
  display: inline-block;
  position: relative;
  padding: 2px 4px;
  margin: 0 3px 3px 0;
  border: 1px solid rgba(0, 0, 0, 0.3);
  border-radius: 3px;
  font-size: 11px;
  line-height: 13px;
}

.direct-upload--pending {
  opacity: 0.6;
}

.direct-upload__progress {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  opacity: 0.2;
  background: #0076ff;
  transition: width 120ms ease-out, opacity 60ms 60ms ease-in;
  transform: translate3d(0, 0, 0);
}

.direct-upload--complete .direct-upload__progress {
  opacity: 0.4;
}

.direct-upload--error {
  border-color: red;
}

input[type=file][data-direct-upload-url][disabled] {
  display: none;
}
```

### Integrating with Libraries or Frameworks

Si desea utilizar la función Carga directa desde un marco de JavaScript, o
desea integrar soluciones personalizadas de arrastrar y soltar, puede utilizar el
Clase `DirectUpload` para este propósito. Al recibir un archivo de su biblioteca
de su elección, cree una instancia de DirectUpload y llame a su método de creación. Crear tomas
una devolución de llamada para invocar cuando se complete la carga.

```js
import { DirectUpload } from "@rails/activestorage"

const input = document.querySelector('input[type=file]')

// Bind to file drop - use the ondrop on a parent element or use a
//  library like Dropzone
const onDrop = (event) => {
  event.preventDefault()
  const files = event.dataTransfer.files;
  Array.from(files).forEach(file => uploadFile(file))
}

// Bind to normal file selection
input.addEventListener('change', (event) => {
  Array.from(input.files).forEach(file => uploadFile(file))
  // you might clear the selected files from the input
  input.value = null
})

const uploadFile = (file) => {
  // your form needs the file_field direct_upload: true, which
  //  provides data-direct-upload-url
  const url = input.dataset.directUploadUrl
  const upload = new DirectUpload(file, url)

  upload.create((error, blob) => {
    if (error) {
      // Handle the error
    } else {
      // Add an appropriately-named hidden input to the form with a
      //  value of blob.signed_id so that the blob ids will be
      //  transmitted in the normal upload flow
      const hiddenField = document.createElement('input')
      hiddenField.setAttribute("type", "hidden");
      hiddenField.setAttribute("value", blob.signed_id);
      hiddenField.name = input.name
      document.querySelector('form').appendChild(hiddenField)
    }
  })
}
```

Si necesita realizar un seguimiento del progreso de la carga del archivo, se puede pasar un tercer
parámetro al constructor `DirectUpload`. Durante la carga, DirectUpload
llamará al método `directUploadWillStoreFileWithXHR` del objeto. Entonces puedes
enlazar su propio controlador de progreso en el XHR.

```js
import { DirectUpload } from "@rails/activestorage"

class Uploader {
  constructor(file, url) {
    this.upload = new DirectUpload(this.file, this.url, this)
  }

  upload(file) {
    this.upload.create((error, blob) => {
      if (error) {
        // Handle the error
      } else {
        // Add an appropriately-named hidden input to the form
        // with a value of blob.signed_id
      }
    })
  }

  directUploadWillStoreFileWithXHR(request) {
    request.upload.addEventListener("progress",
      event => this.directUploadDidProgress(event))
  }

  directUploadDidProgress(event) {
    // Use event.loaded and event.total to update the progress bar
  }
}
```

Discarding Files Stored During System Tests
-------------------------------------------

Las pruebas del sistema limpian los datos de prueba al revertir una transacción. Porque destroy
nunca se llama en un objeto, los archivos adjuntos nunca se limpian. Si tu
deseas borrar los archivos, puede hacerlo en una devolución de llamada `after_teardown`. Haciéndolo
aquí asegura que todas las conexiones creadas durante la prueba estén completas y
no recibirá un error de Active Storage diciendo que no puede encontrar un archivo.

```ruby
class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :chrome, screen_size: [1400, 1400]

  def remove_uploaded_files
    FileUtils.rm_rf("#{Rails.root}/storage_test")
  end

  def after_teardown
    super
    remove_uploaded_files
  end
end
```

Si las pruebas de su sistema verifican la eliminación de un modelo con archivos adjuntos y está
usando Active Job, configure su entorno de prueba para usar el adaptador de cola en línea para
el trabajo de purga se ejecuta inmediatamente, más bien en un momento futuro desconocido.

También es posible que desee utilizar una definición de servicio separada para el entorno de prueba.
para que sus pruebas no eliminen los archivos que se crean durante el desarrollo.

```ruby
# Use inline job processing to make things happen immediately
config.active_job.queue_adapter = :inline

# Separate file storage in the test environment
config.active_storage.service = :local_test
```

Discarding Files Stored During Integration Tests
-------------------------------------------

De manera similar a las pruebas del sistema, los archivos cargados durante las pruebas de integración no se
limpian automáticamente. Si desea borrar los archivos, puede hacerlo en una
devolución de llamada `after_teardown`. Hacerlo aquí asegura que todas las conexiones creadas
durante la prueba están completos y no recibirá un error de Active Storage
diciendo que no puede encontrar un archivo.

```ruby
module RemoveUploadedFiles
  def after_teardown
    super
    remove_uploaded_files
  end

  private

  def remove_uploaded_files
    FileUtils.rm_rf(Rails.root.join('tmp', 'storage'))
  end
end

module ActionDispatch
  class IntegrationTest
    prepend RemoveUploadedFiles
  end
end
```

Implementing Support for Other Cloud Services
---------------------------------------------

Si necesita admitir un servicio en la nube que no sean estos, deberá
implementar el Servicio. Cada servicio se extiende
[`ActiveStorage::Service`](https://github.com/rails/rails/blob/master/activestorage/lib/active_storage/service.rb)
implementando los métodos necesarios para cargar y descargar archivos a la nube.
