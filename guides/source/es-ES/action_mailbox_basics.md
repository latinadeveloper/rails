**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Action Mailbox Basics
=====================

Este guía le proporciona todo lo que necesita para comenzar a recibir
correos electrónicos a su aplicación.

Después de leer este guía, sabrá:

* Cómo recibir correo electrónico dentro de una aplicación de Rails.
* Cómo configurar Action Mailbox.
* Cómo generar y enrutar correos electrónicos a un buzón.
* Cómo probar los correos electrónicos entrantes.

--------------------------------------------------------------------------------

What is Action Mailbox?
-----------------------

Action Mailbox enruta los correos electrónicos entrantes a buzones de correo tipo controlador para
procesamiento en Rails. Se envía con entradas para Mailgun, Mandrill, Postmark,
y SendGrid. También puede manejar correos entrantes directamente a través del Exim integrado,
Postfix y Qmail ingresa.

Los correos electrónicos entrantes se convierten en registros "InboundEmail" utilizando Active Record
y cuenta con seguimiento del ciclo de vida, almacenamiento del correo electrónico original en el almacenamiento en la nube
a través de Active Storage y el manejo responsable de datos con
en incineración por defecto.

Estos correos electrónicos entrantes se enrutan de forma asincrónica utilizando Active Job a uno o
varios buzones de correo dedicados, que son capaces de interactuar directamente
con el resto de su modelo de dominio.

## Setup

Instale las migraciones necesarias para `InboundEmail` y asegúrese de que Active Storage esté configurado:

```bash
$ bin/rails action_mailbox:install
$ bin/rails db:migrate
```

## Configuration

### Exim

Dile a Action Mailbox que acepte correos electrónicos de una retransmisión SMTP:

```ruby
# config/environments/production.rb
config.action_mailbox.ingress = :relay
```

Genere una contraseña segura que Action Mailbox pueda usar para autenticar las solicitudes al ingreso de retransmisión.

Use `bin/rails credentials:edit` para agregar la contraseña a las credenciales encriptadas de su aplicación en
`action_mailbox.ingress_password`, donde Action Mailbox la encontrará automáticamente:

```yaml
action_mailbox:
  ingress_password: ...
```

Alternativamente, proporcione la contraseña en el variable de entorno `RAILS_INBOUND_EMAIL_PASSWORD`.

Configure Exim para canalizar correos electrónicos entrantes a `bin/rails action_mailbox:ingress:exim`,
proporcionando la `URL` de la entrada del relé y la` INGRESS_PASSWORD`
generado previamente. Si su aplicación residía en `https://example.com`, el
comando completo se vería así:

```shell
bin/rails action_mailbox:ingress:exim URL=https://example.com/rails/action_mailbox/relay/inbound_emails INGRESS_PASSWORD=...
```

### Mailgun

Dale a Action Mailbox tu
Clave de firma de Mailgun (que puede encontrar en Configuración -> Seguridad y usuarios -> Seguridad de API en Mailgun)
para que pueda autenticar solicitudes al ingreso de Mailgun.

Use `bin / rails credentials: edit` para agregar su clave de firma a la
credenciales cifradas en `action_mailbox.mailgun_signing_key`,
donde Action Mailbox lo encontrará automáticamente:

```yaml
action_mailbox:
  mailgun_signing_key: ...
```

Alternativamente, proporcione su clave de firma en el entorno `MAILGUN_INGRESS_SIGNING_KEY`
variable.

Dile a Action Mailbox que acepte correos electrónicos de Mailgun:

```ruby
# config/environments/production.rb
config.action_mailbox.ingress = :mailgun
```

[Configure Mailgun](https://documentation.mailgun.com/en/latest/user_manual.html#receiving-forwarding-and-storing-messages)
para reenviar correos electrónicos entrantes a `/rails/action_mailbox/mailgun/inbound_emails/mime`.
Si su solicitud vive en `https://example.com`, usted especificaría el
URL totalmente calificado `https://example.com/rails/action_mailbox/mailgun/inbound_emails/mime`.

### Mandrill

Dé a Action Mailbox su clave API de Mandrill para que pueda autenticar solicitudes a
la entrada de Mandrill.

Use `bin/rails credentials:edit` para agregar su clave API a la de su aplicación
credenciales cifradas en `action_mailbox.mandrill_api_key`,
donde Action Mailbox lo encontrará automáticamente:

```yaml
action_mailbox:
  mandrill_api_key: ...
```

Alternativamente, proporcione su clave API en el `MANDRILL_INGRESS_API_KEY`
variable ambiental.

Dile a Action Mailbox que acepte correos electrónicos de Mandrill:

```ruby
# config/environments/production.rb
config.action_mailbox.ingress = :mandrill
```

[Configure Mandrill](https://mandrill.zendesk.com/hc/en-us/articles/205583197-Inbound-Email-Processing-Overview)
para enrutar correos electrónicos entrantes a `/rails/action_mailbox/mandrill/inbound_emails`.
Si su solicitud vive en `https://example.com`, usted especificaría el
URL totalmente calificado `https://example.com/rails/action_mailbox/mandrill/inbound_emails`.

### Postfix

Dile a Action Mailbox que acepte correos electrónicos de una retransmisión SMTP:

```ruby
# config/environments/production.rb
config.action_mailbox.ingress = :relay
```

Genere una contraseña segura que Action Mailbox pueda usar para autenticar las solicitudes al ingreso de retransmisión.

Use `bin/rails credentials:edit` para agregar la contraseña a las credenciales encriptadas de su aplicación en
`action_mailbox.ingress_password`, donde Action Mailbox la encontrará automáticamente:


```yaml
action_mailbox:
  ingress_password: ...
```

Alternativamente, proporcione la contraseña en la variable de entorno `RAILS_INBOUND_EMAIL_PASSWORD`.

[Configure Postfix](https://serverfault.com/questions/258469/how-to-configure-postfix-to-pipe-all-incoming-email-to-a-script)
para canalizar correos electrónicos entrantes a `bin/rails action_mailbox:ingress:postfix`, proporcionando
la `URL` de la entrada de Postfix y la` INGRESS_PASSWORD` que previamente
genero. Si su aplicación vive en `https: // example.com`, el comando completo
se vería así:

```shell
$ bin/rails action_mailbox:ingress:postfix URL=https://example.com/rails/action_mailbox/relay/inbound_emails INGRESS_PASSWORD=...
```

### Postmark

Dile a Action Mailbox que acepte correos electrónicos de matasellos:

```ruby
# config/environments/production.rb
config.action_mailbox.ingress = :postmark
```

Genere una contraseña segura que Action Mailbox pueda usar para autenticarse
solicitudes al ingreso del matasellos.

Use `bin/rails credentials:edit` para agregar la contraseña a la
credenciales cifradas en `action_mailbox.ingress_password`,
donde Action Mailbox lo encontrará automáticamente:

```yaml
action_mailbox:
  ingress_password: ...
```

Alternatively, provide the password in the `RAILS_INBOUND_EMAIL_PASSWORD`
environment variable.

[Configure Postmark inbound webhook](https://postmarkapp.com/manual#configure-your-inbound-webhook-url)
para reenviar correos electrónicos entrantes a `/rails/action_mailbox/postmark/inbound_emails` con el nombre de usuario `actionmailbox`
y la contraseña que generó anteriormente. Si su aplicación se encuentra en `https://example.com`,
configure Matasellos con la siguiente URL completa:

```
https://actionmailbox:PASSWORD@example.com/rails/action_mailbox/postmark/inbound_emails
```

NOTE: Cuando configure su webhook entrante de Matasellos, asegúrese de marcar la casilla etiquetada **"Incluir contenido de correo electrónico sin procesar en la carga útil JSON"**.
Action Mailbox necesita el contenido de correo electrónico sin procesar para funcionar.

### Qmail

Dile a Action Mailbox que acepte correos electrónicos de una retransmisión SMTP:

```ruby
# config/environments/production.rb
config.action_mailbox.ingress = :relay
```

Genere una contraseña segura que Action Mailbox pueda usar para autenticar las solicitudes al ingreso de retransmisión.

Use `bin/rails credentials:edit` para agregar la contraseña a las credenciales encriptadas de su aplicación en
`action_mailbox.ingress_password`, donde Action Mailbox la encontrará automáticamente:

```yaml
action_mailbox:
  ingress_password: ...
```

Alternativamente, proporcione la contraseña en la variable de entorno `RAILS_INBOUND_EMAIL_PASSWORD`.

Configure Qmail para canalizar correos electrónicos entrantes a `bin/rails action_mailbox: ingress: mail`,
proporcionando la `URL` de la entrada del relé y la `INGRESS_PASSWORD`
generado previamente. Si su aplicación residía en `https://example.com`, el
comando completo se vería así:

```shell
bin/rails action_mailbox:ingress:qmail URL=https://example.com/rails/action_mailbox/relay/inbound_emails INGRESS_PASSWORD=...
```

### SendGrid

Dile a Action Mailbox que acepte correos electrónicos de SendGrid:

```ruby
# config/environments/production.rb
config.action_mailbox.ingress = :sendgrid
```

Genere una contraseña segura que Action Mailbox pueda usar para autenticarse
solicitudes a la entrada de SendGrid.

Use `bin/rails credentials:edit` para agregar la contraseña a la
credenciales cifradas en `action_mailbox.ingress_password`,
donde Action Mailbox lo encontrará automáticamente:

```yaml
action_mailbox:
  ingress_password: ...
```

Alternativamente, proporcione la contraseña en `RAILS_INBOUND_EMAIL_PASSWORD`
variable ambiental.

[Configure SendGrid Inbound Parse](https://sendgrid.com/docs/for-developers/parsing-email/setting-up-the-inbound-parse-webhook/)
para reenviar correos electrónicos entrantes a
`/rails/action_mailbox/sendgrid/inbound_emails` con el nombre de usuario` actionmailbox`
y la contraseña que generó anteriormente. Si su aplicación se encuentra en `https://example.com`,
configurarías SendGrid con la siguiente URL:

```
https://actionmailbox:PASSWORD@example.com/rails/action_mailbox/sendgrid/inbound_emails
```

NOTE: Cuando configure su webhook SendGrid Inbound Parse, asegúrese de marcar la casilla con la etiqueta ** "Publicar el mensaje MIME completo y sin procesar". ** Action Mailbox necesita el mensaje MIME sin procesar para funcionar.

## Examples

Configure el enrutamiento básico:

```ruby
# app/mailboxes/application_mailbox.rb
class ApplicationMailbox < ActionMailbox::Base
  routing /^save@/i     => :forwards
  routing /@replies\./i => :replies
end
```

Luego configura un buzón:

```ruby
# Generate new mailbox
$ bin/rails generate mailbox forwards
```

```ruby
# app/mailboxes/forwards_mailbox.rb
class ForwardsMailbox < ApplicationMailbox
  # Callbacks specify prerequisites to processing
  before_processing :require_projects

  def process
    # Record the forward on the one project, or…
    if forwarder.projects.one?
      record_forward
    else
      # …involve a second Action Mailer to ask which project to forward into.
      request_forwarding_project
    end
  end

  private
    def require_projects
      if forwarder.projects.none?
        # Use Action Mailers to bounce incoming emails back to sender – this halts processing
        bounce_with Forwards::BounceMailer.no_projects(inbound_email, forwarder: forwarder)
      end
    end

    def record_forward
      forwarder.forwards.create subject: mail.subject, content: mail.content
    end

    def request_forwarding_project
      Forwards::RoutingMailer.choose_project(inbound_email, forwarder: forwarder).deliver_now
    end

    def forwarder
      @forwarder ||= User.find_by(email_address: mail.from)
    end
end
```

## Incineration of InboundEmails

De forma predeterminada, un InboundEmail que se haya procesado correctamente será
incinerado después de 30 días. Esto asegura que no está reteniendo los datos de las personas.
quiera o no después de que hayan cancelado sus cuentas o eliminado sus
contenido. La intención es que después de haber procesado un correo electrónico, debería haber
extrajo todos los datos que necesitaba y los convirtió en modelos de dominio y contenido
en su lado de la aplicación. El InboundEmail simplemente permanece en el sistema
por el tiempo extra para proporcionar opciones de depuración y forenses.

La incineración real se realiza a través del `IncinerationJob` que está programado
para ejecutarse después del tiempo de `config.action_mailbox.incinerate_after`. Este valor es
por defecto establecido en `30.days`, pero puede cambiarlo en su producción.rb
configuración. (Tenga en cuenta que esta programación de incineración en el futuro lejano se basa en
su cola de trabajos puede retener trabajos durante tanto tiempo).

## Working with Action Mailbox in development

Es útil poder probar los correos electrónicos entrantes en desarrollo sin realmente
enviar y recibir correos electrónicos reales. Para lograr esto, hay un director
controlador montado en `/rails/conductor/action_mailbox/inbound_emails`,
que le da un índice de todos los InboundEmails en el sistema, su
estado de procesamiento y un formulario para crear un nuevo InboundEmail también.

## Testing mailboxes

Ejemplo:

```ruby
class ForwardsMailboxTest < ActionMailbox::TestCase
  test "directly recording a client forward for a forwarder and forwardee corresponding to one project" do
    assert_difference -> { people(:david).buckets.first.recordings.count } do
      receive_inbound_email_from_mail \
        to: 'save@example.com',
        from: people(:david).email_address,
        subject: "Fwd: Status update?",
        body: <<~BODY
          --- Begin forwarded message ---
          From: Frank Holland <frank@microsoft.com>

          What's the status?
        BODY
    end

    recording = people(:david).buckets.first.recordings.last
    assert_equal people(:david), recording.creator
    assert_equal "Status update?", recording.forward.subject
    assert_match "What's the status?", recording.forward.content.to_s
  end
end
```
