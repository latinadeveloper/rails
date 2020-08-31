**NO LEA ESTE ARCHIVO EN GITHUB, LAS GUÍAS SE PUBLICAN EN https://guides.rubyonrails.org.**

Testing Rails Applications
==========================

Esta guía cubre los mecanismos integrados en Rails para probar su aplicación.

Después de leer esta guía, sabrá:

* Terminología de prueba de rieles.
* Cómo escribir pruebas unitarias, funcionales, de integración y de sistema para su aplicación.
* Otros enfoques de prueba y complementos populares.


--------------------------------------------------------------------------------

Why Write Tests for your Rails Applications?
--------------------------------------------

Rails hace que sea muy fácil escribir sus pruebas. Comienza produciendo un código de prueba esqueleto mientras crea sus modelos y controladores.

Al ejecutar sus pruebas de Rails, puede asegurarse de que su código se adhiera a la funcionalidad deseada incluso después de una refactorización importante del código.

Las pruebas de Rails también pueden simular las solicitudes del navegador y, por lo tanto, puede probar la respuesta de su aplicación sin tener que probarla a través de su navegador.

Introduction to Testing
-----------------------

El soporte de prueba se tejió en el tejido Rails desde el principio. No fue una epifanía de "¡oh! Vamos a dar soporte a la ejecución de pruebas porque son nuevas y geniales".

### Rails Sets up for Testing from the Word Go

Rails crea un directorio `test` para usted cuando se crea un proyecto Rails usando `rails new` _application_name_. Si enumera el contenido de este directorio, verá:

```bash
$ ls -F test
application_system_test_case.rb  controllers/                     helpers/                         mailers/                         system/
channels/                        fixtures/                        integration/                     models/                          test_helper.rb
```

Los directorios `helpers`, `mailers` y `models` están hechos para contener pruebas para los asistentes de vistas, los mailers y los modelos, respectivamente. El directorio de `channels` está destinado a contener pruebas para la conexión y los canales de Action Cable. El directorio `controllers` está destinado a contener pruebas para controladores, rutas y vistas. El directorio de `integración` está destinado a contener pruebas de interacciones entre controladores.

El directorio de prueba del sistema contiene pruebas del sistema, que se utilizan para la
prueba de su aplicación en el navegador completo. Las pruebas del sistema le permiten probar su aplicación en
la forma en que sus usuarios lo experimentan y también lo ayudan a probar su JavaScript.
Las pruebas del sistema heredan de Capybara y funcionan en las pruebas del navegador para su
solicitud.

Los accesorios son una forma de organizar los datos de prueba; residen en el directorio `fixtures`.

También se creará un directorio de trabajos cuando se genere por primera vez una prueba asociada.

El archivo `test_helper.rb` contiene la configuración predeterminada para sus pruebas.

El `application_system_test_case.rb` contiene la configuración predeterminada para su sistema
pruebas.

### The Test Environment

De forma predeterminada, cada aplicación Rails tiene tres entornos: desarrollo, prueba y producción.

La configuración de cada entorno se puede modificar de manera similar. En este caso, podemos modificar nuestro entorno de prueba cambiando las opciones que se encuentran en `config/environment/test.rb`.

NOTE: Sus pruebas se ejecutan bajo `RAILS_ENV=test`.

### Rails meets Minitest

Si recuerda, usamos el comando `bin/rails generate model` en el guía
[Getting Started with Rails](getting_started.html). Creamos nuestro primer
model, y entre otras cosas creó stubs de prueba en el directorio `test`:

```bash
$ bin/rails generate model article title:string body:text
...
create  app/models/article.rb
create  test/models/article_test.rb
create  test/fixtures/articles.yml
...
```

El código auxiliar de prueba predeterminado en `test  models/article_test.rb` se ve así:

```ruby
require "test_helper"

class ArticleTest < ActiveSupport::TestCase
  # test "the truth" do
  #   assert true
  # end
end
```

Un examen línea por línea de este archivo le ayudará a orientarse al código y la terminología de prueba de Rails.

```ruby
require "test_helper"
```

Al requerir este archivo, `test_helper.rb` se carga la configuración predeterminada para ejecutar nuestras pruebas. Incluiremos esto con todas las pruebas que escribamos, por lo que cualquier método agregado a este archivo estará disponible para todas nuestras pruebas.

```ruby
class ArticleTest < ActiveSupport::TestCase
```

La clase `ArticleTest` define un _caso de prueba_ porque hereda de `ActiveSupport::TestCase`. Por tanto, `ArticleTest` tiene todos los métodos disponibles en `ActiveSupport::TestCase`. Más adelante en esta guía, veremos algunos de los métodos que nos brinda.

Cualquier método definido dentro de una clase heredada de `Minitest::Test`
(que es la superclase de `ActiveSupport::TestCase`) que comienza con `test` simplemente se llama prueba. Por lo tanto, los métodos definidos como `test_password` y `test_valid_password` son nombres de prueba legales y se ejecutan automáticamente cuando se ejecuta el caso de prueba.

Rails también agrega un método de "prueba" que toma un nombre de prueba y un bloque. Genera una prueba `Minitest::Unit` normal con nombres de métodos con el prefijo `test`. Entonces no tiene que preocuparse por nombrar los métodos, y puede escribir algo como:

```ruby
test "the truth" do
  assert true
end
```

Que es aproximadamente lo mismo que escribir esto:

```ruby
def test_the_truth
  assert true
end
```

Aunque aún puede usar definiciones de métodos regulares, el uso de la macro `test` permite un nombre de prueba más legible.

NOTE: El nombre del método se genera reemplazando espacios con guiones bajos. Sin embargo, el resultado no necesita ser un identificador Ruby válido, el nombre puede contener caracteres de puntuación, etc. Eso es porque en Ruby técnicamente cualquier cadena puede ser un nombre de método. Esto puede requerir el uso de llamadas `define_method` y `send` para funcionar correctamente, pero formalmente hay poca restricción en el nombre.

A continuación, veamos nuestra primera afirmación:

```ruby
assert true
```

Una aserción es una línea de código que evalúa un objeto (o expresión) para obtener los resultados esperados. Por ejemplo, una afirmación puede verificar:

* ¿Este valor = ese valor?
* ¿Este objeto es nulo?
* ¿Esta línea de código genera una excepción?
* ¿La contraseña del usuario tiene más de 5 caracteres?

Cada prueba puede contener una o más aserciones, sin restricción en cuanto a cuántas aserciones se permiten. Solo cuando todas las afirmaciones sean exitosas, la prueba pasará.

#### Your first failing test

Para ver cómo se informa una prueba fallida, puede agregar una prueba fallida al caso de prueba `article_test.rb`.

```ruby
test "should not save article without title" do
  article = Article.new
  assert_not article.save
end
```

Ejecutemos esta prueba recién agregada (donde `6` es el número de línea donde se define la prueba).

```bash
$ bin/rails test test/models/article_test.rb:6
Run options: --seed 44656

# Running:

F

Failure:
ArticleTest#test_should_not_save_article_without_title [/path/to/blog/test/models/article_test.rb:6]:
Expected true to be nil or false


rails test test/models/article_test.rb:6



Finished in 0.023918s, 41.8090 runs/s, 41.8090 assertions/s.

1 runs, 1 assertions, 1 failures, 0 errors, 0 skips

```

En la salida, `F` denota una falla. Puede ver el seguimiento correspondiente que se muestra en `Failure` junto con el nombre de la prueba que falló. Las siguientes líneas contienen el seguimiento de la pila seguido de un mensaje que menciona el valor real y el valor esperado por la aserción. Los mensajes de afirmación predeterminados proporcionan la información suficiente para ayudar a identificar el error. Para que el mensaje de error de la aserción sea más legible, cada aserción proporciona un parámetro de mensaje opcional, como se muestra aquí:

```ruby
test "should not save article without title" do
  article = Article.new
  assert_not article.save, "Saved the article without a title"
end
```

La ejecución de esta prueba muestra el mensaje de afirmación más amigable:

```bash
Failure:
ArticleTest#test_should_not_save_article_without_title [/path/to/blog/test/models/article_test.rb:6]:
Saved the article without a title
```

Ahora, para que esta prueba pase, podemos agregar una validación de nivel de modelo para el campo _title_.

```ruby
class Article < ApplicationRecord
  validates :title, presence: true
end
```

Ahora la prueba debería pasar. Verifiquemos ejecutando la prueba nuevamente:

```bash
$ bin/rails test test/models/article_test.rb:6
Run options: --seed 31252

# Running:

.

Finished in 0.027476s, 36.3952 runs/s, 36.3952 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

Ahora, si se dio cuenta, primero escribimos una prueba que falla para una
funcionalidad, luego escribimos un código que agrega la funcionalidad y finalmente
nos aseguramos de que nuestra prueba pasara. Este enfoque para el desarrollo de software es
denominado
[_Test-Driven Development_ (TDD)](http://c2.com/cgi/wiki?TestDrivenDevelopment).

#### What an Error Looks Like

Para ver cómo se informa un error, aquí hay una prueba que contiene un error:

```ruby
test "should report error" do
  # some_undefined_variable is not defined elsewhere in the test case
  some_undefined_variable
  assert true
end
```

Ahora puede ver aún más resultados en la consola al ejecutar las pruebas:

```bash
$ bin/rails test test/models/article_test.rb
Run options: --seed 1808

# Running:

.E

Error:
ArticleTest#test_should_report_error:
NameError: undefined local variable or method 'some_undefined_variable' for #<ArticleTest:0x007fee3aa71798>
    test/models/article_test.rb:11:in 'block in <class:ArticleTest>'


rails test test/models/article_test.rb:9



Finished in 0.040609s, 49.2500 runs/s, 24.6250 assertions/s.

2 runs, 1 assertions, 0 failures, 1 errors, 0 skips
```

Observe la 'E' en la salida. Denota una prueba con error.

NOTA: La ejecución de cada método de prueba se detiene tan pronto como se produce un error o
se encuentra una falla de aserción y la suite de pruebas continúa con el siguiente
método. Todos los métodos de prueba se ejecutan en orden aleatorio. Se puede utilizar 
[`config.active_support.test_order` option](configuring.html#configuring-active-support)
para configurar el orden de prueba.

Cuando una prueba falla, se le presenta el rastreo correspondiente. Por defecto
Filtros de rieles que retroceden y solo imprimirán líneas relevantes para su
solicitud. Esto elimina el ruido del marco y ayuda a concentrarse en su
código. Sin embargo, hay situaciones en las que desea ver el
retroceso. Establezca el argumento `-b` (o `--backtrace`) para habilitar este comportamiento:

```bash
$ bin/rails test -b test/models/article_test.rb
```

Si queremos que esta prueba pase, podemos modificarla para usar `assert_raises` así:

```ruby
test "should report error" do
  # some_undefined_variable is not defined elsewhere in the test case
  assert_raises(NameError) do
    some_undefined_variable
  end
end
```

Esta prueba debería pasar ahora.

### Available Assertions

A estas alturas ya ha vislumbrado algunas de las afirmaciones que están disponibles. Las afirmaciones son las abejas obreras de la prueba. Ellos son los que realmente realizan las comprobaciones para asegurarse de que las cosas vayan según lo planeado.

Aquí hay un extracto de las afirmaciones que puede usar con
 la biblioteca de pruebas predeterminada
[`Minitest`](https://github.com/seattlerb/minitest),
utilizado por Rails. El parámetro `[msg]` es un mensaje de cadena opcional que puede
especifique para que sus mensajes de error de prueba sean más claros.


| Assertion                                                        | Purpose |
| ---------------------------------------------------------------- | ------- |
| `assert( test, [msg] )`                                          | Asegura que`test` es verdad|
| `assert_not( test, [msg] )`                                      | Asegura que `test` es falso.|
| `assert_equal( expected, actual, [msg] )`                        | Asegura que `expected == actual` es verdad.|
| `assert_not_equal( expected, actual, [msg] )`                    | Asegura que `expected != actual` es verdad.|
| `assert_same( expected, actual, [msg] )`                         | Asegura que `expected.equal?(actual)` es verdad.|
| `assert_not_same( expected, actual, [msg] )`                     | Asegura que `expected.equal?(actual)` es falso.|
| `assert_nil( obj, [msg] )`                                       | Asegura que `obj.nil?` es verdad.|
| `assert_not_nil( obj, [msg] )`                                   | Asegura que `obj.nil?` es falso.|
| `assert_empty( obj, [msg] )`                                     | Asegura que `obj` es `empty?`.|
| `assert_not_empty( obj, [msg] )`                                 | Asegura que `obj` no es `empty?`.|
| `assert_match( regexp, string, [msg] )`                          | Asegura que una cadena coincide con la expresión regular.|
| `assert_no_match( regexp, string, [msg] )`                       | Asegura que una cadena no coincide con la expresión regular.|
| `assert_includes( collection, obj, [msg] )`                      | Asegura que `obj` esta en `collection`.|
| `assert_not_includes( collection, obj, [msg] )`                  | Asegura que `obj` no esta en `collection`.|
| `assert_in_delta( expected, actual, [delta], [msg] )`            | Asegura que los numeros `expected` y `actual` esten entre `delta` el uno del otro.|
| `assert_not_in_delta( expected, actual, [delta], [msg] )`        | Asegura que los numeros `expected` y `actual` no esten entre `delta` el uno del otro.|
| `assert_in_epsilon ( expected, actual, [epsilon], [msg] )`       | Asegura que los numeros `expected` y `actual` tener un error relativo menor que `epsilon`.|
| `assert_not_in_epsilon ( expected, actual, [epsilon], [msg] )`   | Asegura que los numeros `expected` y `actual` no tiene un error relativo menor quen `epsilon`.|
| `assert_throws( symbol, [msg] ) { block }`                       | Asegura que el bloque dado arroja el símbolo.|
| `assert_raises( exception1, exception2, ... ) { block }`         | Asegura que el bloque dado genera una de las excepciones dadas.|
| `assert_instance_of( class, obj, [msg] )`                        | Asegura que `obj` es una instancia de `class`.|
| `assert_not_instance_of( class, obj, [msg] )`                    | Asegura que `obj` no es una instancia de`class`.|
| `assert_kind_of( class, obj, [msg] )`                            | Asegura que `obj` es una instancia de`class` o desciende de él.|
| `assert_not_kind_of( class, obj, [msg] )`                        | Asegura que `obj` no es una instancia def `class` y no o desciende de él.|
| `assert_respond_to( obj, symbol, [msg] )`                        | Asegura que `obj` responde a`symbol`.|
| `assert_not_respond_to( obj, symbol, [msg] )`                    | Asegura que `obj` no responde a `symbol`.|
| `assert_operator( obj1, operator, [obj2], [msg] )`               | Asegura que `obj1.operator(obj2)` es verdad.|
| `assert_not_operator( obj1, operator, [obj2], [msg] )`           | Asegura que `obj1.operator(obj2)` es falso.|
| `assert_predicate ( obj, predicate, [msg] )`                     | Asegura que `obj.predicate` es cierto, por ejemplo `assert_predicate str, :empty?`|
| `assert_not_predicate ( obj, predicate, [msg] )`                 | Asegura que `obj.predicate` es cierto, por ejemplo`assert_not_predicate str, :empty?`|
| `flunk( [msg] )`                                                 | Asegura el fracaso. Esto es útil para marcar explícitamente una prueba que aún no ha terminado.|

Los anteriores son un subconjunto de afirmaciones que admite minitest. Para una exhaustiva y
lista más actualizada, consulte
[Minitest API documentation](http://docs.seattlerb.org/minitest/), específicamente
[`Minitest::Assertions`](http://docs.seattlerb.org/minitest/Minitest/Assertions.html).

Debido a la naturaleza modular del marco de prueba, es posible crear sus propias afirmaciones. De hecho, eso es exactamente lo que hace Rails. Incluye algunas afirmaciones especializadas para facilitarle la vida.

NOTE: Crear sus propias afirmaciones es un tema avanzado que no cubriremos en este tutorial.

### Rails Specific Assertions

Rails agrega algunas afirmaciones personalizadas propias al marco `minitest`:


| Assertion                                                                         | Purpose |
| --------------------------------------------------------------------------------- | ------- |
| [`assert_difference(expressions, difference = 1, message = nil) {...}`](https://api.rubyonrails.org/classes/ActiveSupport/Testing/Assertions.html#method-i-assert_difference) | Pruebe la diferencia numérica entre el valor de retorno de una expresión como resultado de lo que se evalúa en el bloque generado.
| [`assert_no_difference(expressions, message = nil, &block)`](https://api.rubyonrails.org/classes/ActiveSupport/Testing/Assertions.html#method-i-assert_no_difference) | Afirma que el resultado numérico de evaluar una expresión no se modifica antes y después de invocar el bloque pasado.
| [`assert_changes(expressions, message = nil, from:, to:, &block)`](https://api.rubyonrails.org/classes/ActiveSupport/Testing/Assertions.html#method-i-assert_changes) | Probar que el resultado de evaluar una expresión se cambia después de invocar el bloque pasado. Verifique que el resultado de evaluar una expresión cambie después de invocar el bloque pasado.
| [`assert_no_changes(expressions, message = nil, &block)`](https://api.rubyonrails.org/classes/ActiveSupport/Testing/Assertions.html#method-i-assert_no_changes) | Pruebe que el resultado de evaluar una expresión no se cambia después de invocar el bloque pasado.
| [`assert_nothing_raised { block }`](https://api.rubyonrails.org/classes/ActiveSupport/Testing/Assertions.html#method-i-assert_nothing_raised) | Asegura que el bloque dado no genera ninguna excepción.
| [`assert_recognizes(expected_options, path, extras={}, message=nil)`](https://api.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_recognizes) | Afirma que el enrutamiento de la ruta dada se manejó correctamente y que las opciones analizadas (dadas en el hash de opciones_esperadas) coinciden con la ruta. Básicamente, afirma que Rails reconoce la ruta dada por opciones_esperadas.
| [`assert_generates(expected_path, options, defaults={}, extras = {}, message=nil)`](https://api.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html#method-i-assert_generates) | Afirma que las opciones proporcionadas se pueden utilizar para generar la ruta proporcionada. Esta es la inversa de assert_recognizes. El parámetro extras se usa para decirle a la solicitud los nombres y valores de los parámetros de solicitud adicionales que estarían en una cadena de consulta. El parámetro de mensaje le permite especificar un mensaje de error personalizado para las fallas de afirmación.|
| [`assert_response(type, message = nil)`](https://api.rubyonrails.org/classes/ActionDispatch/Assertions/ResponseAssertions.html#method-i-assert_response) |Afirma que la respuesta viene con un código de estado específico. Puede especificar `:success` para indicar 200-299, `:redirect` indica 300-399, `:missing` indica 404, o `:error` para que coincida con el rango 500-599. También puede pasar un número de estado explícito o su equivalente simbólico. Para más información, ver [full list of status codes](http://rubydoc.info/github/rack/rack/master/Rack/Utils#HTTP_STATUS_CODES-constant) y como su [mapping](https://rubydoc.info/github/rack/rack/master/Rack/Utils#SYMBOL_TO_STATUS_CODE-constant) trabaja.|
| [`assert_redirected_to(options = {}, message=nil)`](https://api.rubyonrails.org/classes/ActionDispatch/Assertions/ResponseAssertions.html#method-i-assert_redirected_to) | Afirma que las opciones de redireccionamiento pasadas coinciden con las del redireccionamiento llamado en la última acción. Esta coincidencia puede ser parcial, de modo que `assert_redirected_to(controller: "weblog")` también coincidirá con la redirección de `redirect_to(controller: "weblog", action: "show")` y así. También puede pasar rutas con nombre como `assert_redirected_to root_path` y como objetos de Active Record `assert_redirected_to @article`.|

Verá el uso de algunas de estas afirmaciones en el próximo capítulo.

### A Brief Note About Test Cases

Todas las aserciones básicas como `assert_equal` definidas en `Minitest::Assertions` también están disponibles en las clases que usamos en nuestros propios casos de prueba. De hecho, Rails proporciona las siguientes clases para que las herede:

* [`ActiveSupport::TestCase`](https://api.rubyonrails.org/classes/ActiveSupport/TestCase.html)
* [`ActionMailer::TestCase`](https://api.rubyonrails.org/classes/ActionMailer/TestCase.html)
* [`ActionView::TestCase`](https://api.rubyonrails.org/classes/ActionView/TestCase.html)
* [`ActiveJob::TestCase`](https://api.rubyonrails.org/classes/ActiveJob/TestCase.html)
* [`ActionDispatch::IntegrationTest`](https://api.rubyonrails.org/classes/ActionDispatch/IntegrationTest.html)
* [`ActionDispatch::SystemTestCase`](https://api.rubyonrails.org/classes/ActionDispatch/SystemTestCase.html)
* [`Rails::Generators::TestCase`](https://api.rubyonrails.org/classes/Rails/Generators/TestCase.html)

Cada una de estas clases incluye `Minitest::Assertions`, lo que nos permite utilizar todas las aserciones básicas en nuestras pruebas.

NOTA: Para obtener más información sobre `Minitest`, consulte [its
documentation](http://docs.seattlerb.org/minitest).

### The Rails Test Runner

Podemos ejecutar todas nuestras pruebas a la vez usando el comando `bin/rails test`.

O podemos ejecutar un solo archivo de prueba pasando el comando `bin/rails test` el nombre del archivo que contiene los casos de prueba.


```bash
$ bin/rails test test/models/article_test.rb
Run options: --seed 1559

# Running:

..

Finished in 0.027034s, 73.9810 runs/s, 110.9715 assertions/s.

2 runs, 3 assertions, 0 failures, 0 errors, 0 skips
```

This will run all test methods from the test case.

You can also run a particular test method from the test case by providing the
`-n` or `--name` flag and the test's method name.

```bash
$ bin/rails test test/models/article_test.rb -n test_the_truth
Run options: -n test_the_truth --seed 43583

# Running:

.

Finished tests in 0.009064s, 110.3266 tests/s, 110.3266 assertions/s.

1 tests, 1 assertions, 0 failures, 0 errors, 0 skips
```

También puede ejecutar una prueba en una línea específica proporcionando el número de línea.

```bash
$ bin/rails test test/models/article_test.rb:6 # run specific test and line
```

También puede ejecutar un directorio completo de pruebas proporcionando la ruta al directorio.

```bash
$ bin/rails test test/controllers # run all tests from specific directory
```

El corredor de pruebas también proporciona muchas otras características, como fallar rápidamente y aplazar la salida de la prueba.
al final de la ejecución de prueba y así sucesivamente. Consulte la documentación del corredor de pruebas de la siguiente manera:

```bash
$ bin/rails test -h
Usage: rails test [options] [files or directories]

You can run a single test by appending a line number to a filename:

    bin/rails test test/models/user_test.rb:27

You can run multiple files and directories at the same time:

    bin/rails test test/controllers test/integration/login_test.rb

By default test failures and errors are reported inline during a run.

minitest options:
    -h, --help                       Display this help.
        --no-plugins                 Bypass minitest plugin auto-loading (or set $MT_NO_PLUGINS).
    -s, --seed SEED                  Sets random seed. Also via env. Eg: SEED=n rake
    -v, --verbose                    Verbose. Show progress processing files.
    -n, --name PATTERN               Filter run on /regexp/ or string.
        --exclude PATTERN            Exclude /regexp/ or string from run.

Known extensions: rails, pride
    -w, --warnings                   Run with Ruby warnings enabled
    -e, --environment ENV            Run tests in the ENV environment
    -b, --backtrace                  Show the complete backtrace
    -d, --defer-output               Output test failures and errors after the test run
    -f, --fail-fast                  Abort test run on first failure or error
    -c, --[no-]color                 Enable color in the output
    -p, --pride                      Pride. Show your testing pride!
```

Parallel Testing
----------------

Las pruebas en paralelo le permiten paralelizar su suite de pruebas. Mientras que los procesos de bifurcación es el
método predeterminado, también se admiten subprocesos. La ejecución de pruebas en paralelo reduce el tiempo
necesita todo su conjunto de pruebas para ejecutarse.

### Parallel Testing with Processes

El método de paralelización predeterminado es bifurcar procesos utilizando el sistema DRb de Ruby. Los procesos
se bifurcan en función del número de trabajadores proporcionados. El número predeterminado es el recuento de núcleos real
en la máquina en la que se encuentra, pero se puede cambiar mediante el número pasado al método de paralelización.

Para habilitar la paralelización agregue lo siguiente a su `test_helper.rb`:

```ruby
class ActiveSupport::TestCase
  parallelize(workers: 2)
end
```

El número de trabajadores aprobados es el número de veces que se bifurcará el proceso. Es posible que desee
paralelice su conjunto de pruebas local de manera diferente a su CI, por lo que se proporciona una variable de entorno
para poder cambiar fácilmente la cantidad de trabajadores, una ejecución de prueba debe usar:


```bash
PARALLEL_WORKERS=15 bin/rails test
```

Al paralelizar las pruebas, Active Record maneja automáticamente la creación de una base de datos y carga el esquema en la base de datos para cada
proceso. Las bases de datos llevarán el sufijo del número correspondiente al trabajador. Por ejemplo, si tu
si tiene 2 trabajadores, las pruebas crearán `test-database-0` y `test-database-1` respectivamente.

Si el número de trabajadores aprobados es 1 o menos, los procesos no se bifurcarán y las pruebas no
se paralelizará y las pruebas utilizarán la base de datos original `test-database`.

Se proporcionan dos ganchos, uno se ejecuta cuando el proceso se bifurca y el otro se ejecuta antes de que se cierre el proceso bifurcado.
Estos pueden ser útiles si su aplicación utiliza varias bases de datos o realiza otras tareas que dependen de la cantidad de
trabajadores.

El método `paralelize_setup` se llama justo después de que se bifurcan los procesos. El método `paralelize_teardown`
se llama justo antes de que se cierren los procesos.

```ruby
class ActiveSupport::TestCase
  parallelize_setup do |worker|
    # setup databases
  end

  parallelize_teardown do |worker|
    # cleanup databases
  end

  parallelize(workers: :number_of_processors)
end
```

Estos métodos no son necesarios ni están disponibles cuando se utilizan pruebas paralelas con subprocesos.


### Parallel Testing with Threads

Si prefiere usar subprocesos o está usando JRuby, se proporciona una opción de paralelización de subprocesos. El roscado
paralelizador está respaldado por `Parallel::Executor` de Minitest.

Para cambiar el método de paralelización para usar subprocesos sobre bifurcaciones, coloque lo siguiente en su `test_helper.rb`

```ruby
class ActiveSupport::TestCase
  parallelize(workers: :number_of_processors, with: :threads)
end
```

Las aplicaciones Rails generadas a partir de JRuby o TruffleRuby incluirán automáticamente la opción `with: :threads`.

El número de trabajadores pasados ​​para "paralelizar" determina el número de subprocesos que utilizarán las pruebas. Puedes
desea paralelizar su conjunto de pruebas local de manera diferente a su CI, por lo que se proporciona una variable de entorno
para poder cambiar fácilmente la cantidad de trabajadores, una ejecución de prueba debe usar:

```bash
PARALLEL_WORKERS=15 bin/rails test
```

### Testing Parallel Transactions

Rails envuelve automáticamente cualquier caso de prueba en una transacción de base de datos que se rueda
de vuelta después de que se complete la prueba. Esto hace que los casos de prueba sean independientes entre sí.
y los cambios en la base de datos solo son visibles dentro de una única prueba.

Cuando desee probar el código que ejecuta transacciones paralelas en subprocesos,
las transacciones pueden bloquearse entre sí porque ya están anidadas bajo la prueba
transacción.

Puede deshabilitar transacciones en una clase de caso de prueba estableciendo
`self.use_transactional_tests = false`:

```ruby
class WorkerTest < ActiveSupport::TestCase
  self.use_transactional_tests = false

  test "parallel transactions" do
    # start some threads that create transactions
  end
end
```

NOTE: Con las pruebas transaccionales deshabilitadas, debe limpiar las pruebas de datos
crear como cambios no se revierten automáticamente una vez finalizada la prueba.

The Test Database
-----------------

Casi todas las aplicaciones de Rails interactúan en gran medida con una base de datos y, como resultado, sus pruebas también necesitarán una base de datos para interactuar. Para escribir pruebas eficientes, deberá comprender cómo configurar esta base de datos y completarla con datos de muestra.

De forma predeterminada, cada aplicación Rails tiene tres entornos: desarrollo, prueba y producción. La base de datos de cada uno de ellos se configura en `config/database.yml`.

Una base de datos de prueba dedicada le permite configurar e interactuar con los datos de prueba de forma aislada. De esta manera, sus pruebas pueden manipular los datos de prueba con confianza, sin preocuparse por los datos en las bases de datos de desarrollo o producción.

### Maintaining the test database schema

Para ejecutar sus pruebas, su base de datos de prueba deberá tener la
estructura. El asistente de prueba comprueba si su base de datos de prueba tiene alguna pendiente
migraciones. Intentará cargar su `db/schema.rb` o `db/structure.sql`
en la base de datos de prueba. Si las migraciones aún están pendientes, aparecerá un error.
elevado. Por lo general, esto indica que su esquema no se ha migrado por completo. Corriendo
las migraciones contra la base de datos de desarrollo (`bin/rails db:migrate`)
actualizar el esquema.

NOTE: Si hubo modificaciones en las migraciones existentes, la base de datos de prueba debe
ser reconstruido. Esto se puede hacer ejecutando `bin/rails db:test:prepare`.


### The Low-Down on Fixtures

Para obtener buenas pruebas, deberá pensar un poco en la configuración de datos de prueba.
En Rails, puede manejar esto definiendo y personalizando accesorios.
Puede encontrar documentación completa en el [Fixtures API documentation](https://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html).


#### What are Fixtures?

_Fixtures_ es una palabra elegante para datos de muestra. Los accesorios le permiten completar su base de datos de pruebas con datos predefinidos antes de que se ejecuten sus pruebas. Los accesorios son independientes de la base de datos y están escritos en YAML. Hay un archivo por modelo.

NOTA: Los accesorios no están diseñados para crear todos los objetos que necesitan sus pruebas, y se administran mejor cuando solo se usan para datos predeterminados que se pueden aplicar al caso común.

Encontrará accesorios en su directorio `test/fixtures`. Cuando ejecuta `bin/rails generate model` para crear un nuevo modelo, Rails crea automáticamente stubs de accesorios en este directorio.

#### YAML

Los dispositivos con formato YAML son una forma sencilla de describir sus datos de muestra. Estos tipos de dispositivos tienen la extensión de archivo **.yml** (como en `users.yml`).

Aquí hay un archivo de accesorio YAML de muestra:

```yaml
# lo & behold! I am a YAML comment!
david:
  name: David Heinemeier Hansson
  birthday: 1979-10-15
  profession: Systems development

steve:
  name: Steve Ross Kellock
  birthday: 1974-09-27
  profession: guy with keyboard
```

A cada dispositivo se le asigna un nombre seguido de una lista con sangría de pares clave / valor separados por dos puntos. Los registros suelen estar separados por una línea en blanco. Puede colocar comentarios en un archivo de accesorios usando el carácter # en la primera columna.

Si trabaja con [associations](/association_basics.html), puede
definir un nodo de referencia entre dos dispositivos diferentes. Aquí hay un ejemplo con
una asociación `belongs_to`/`has_many`:

```yaml
# In fixtures/categories.yml
about:
  name: About

# In fixtures/articles.yml
first:
  title: Welcome to Rails!
  body: Hello world!
  category: about
```

Observe que la clave `category` del` first` artículo que se encuentra en `fixtures/articles.yml` tiene un valor de `about`. Esto le dice a Rails que cargue la categoría `about` que se encuentra en `fixtures/categories.yml`.

NOTA: Para que las asociaciones se hagan referencia entre sí por su nombre, puede usar el nombre del dispositivo en lugar de especificar el atributo `id:` en los dispositivos asociados. Los rieles asignarán automáticamente una clave principal para que sea coherente entre las ejecuciones. Para obtener más información sobre el comportamiento de esta asociación, lea la [Fixtures API documentation](https://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html).


#### ERB'in It Up

ERB le permite incrustar código Ruby en plantillas. El formato de dispositivo YAML se procesa previamente con ERB cuando Rails carga dispositivos. Esto le permite usar Ruby para ayudarlo a generar algunos datos de muestra. Por ejemplo, el siguiente código genera mil usuarios:

```erb
<% 1000.times do |n| %>
user_<%= n %>:
  username: <%= "user#{n}" %>
  email: <%= "user#{n}@example.com" %>
<% end %>
```

#### Fixtures in Action

Rails carga automáticamente todos los dispositivos del directorio `test/fixtures` por
defecto. La carga implica tres pasos:

1. Elimina cualquier dato existente de la tabla correspondiente al aparato.
2. Cargue los datos del aparato en la tabla
3. Vierta los datos del dispositivo en un método en caso de que desee acceder a él directamente

TIP: para eliminar datos existentes de la base de datos, Rails intenta deshabilitar los activadores de integridad referencial (como claves externas y restricciones de verificación). Si obtiene molestos errores de permisos al ejecutar pruebas, asegúrese de que el usuario de la base de datos tenga privilegios para desactivar estos activadores en el entorno de prueba. (En PostgreSQL, solo los superusuarios pueden deshabilitar todos los activadores. Lea más sobre los permisos de PostgreSQL [here](http://blog.endpoint.com/2012/10/postgres-system-triggers-error.html)).

#### Fixtures are Active Record objects

Los aparatos son instancias de Active Record. Como se mencionó en el punto # 3 anterior, puede acceder al objeto directamente porque está disponible automáticamente como un método cuyo alcance es local del caso de prueba. Por ejemplo:


```ruby
# this will return the User object for the fixture named david
users(:david)

# this will return the property for david called id
users(:david).id

# one can also access methods available on the User class
david = users(:david)
david.call(david.partner)
```

Para obtener varios dispositivos a la vez, puede pasar una lista de nombres de dispositivos. Por ejemplo:

```ruby
# this will return an array containing the fixtures david and steve
users(:david, :steve)
```


Model Testing
-------------

Las pruebas de modelo se utilizan para probar los distintos modelos de su aplicación.

Las pruebas del modelo Rails se almacenan en el directorio `test/models`. Rails proporciona
un generador para crear un modelo de esqueleto de prueba para usted.

```bash
$ bin/rails generate test_unit:model article title:string body:text
create  test/models/article_test.rb
create  test/fixtures/articles.yml
```

Las pruebas de modelo no tienen su propia superclase como `ActionMailer::TestCase` sino que heredan de [ ActiveSupport::TestCase`](https://api.rubyonrails.org/classes/ActiveSupport/TestCase.html).

System Testing
--------------

Las pruebas del sistema le permiten probar las interacciones del usuario con su aplicación, ejecutando pruebas
en un navegador real o sin cabeza. Las pruebas del sistema utilizan Capybara debajo del capó.

Para crear pruebas del sistema Rails, use el directorio `test/system` en su
solicitud. Rails proporciona un generador para crear un esqueleto de prueba del sistema para usted.

```bash
$ bin/rails generate system_test users
      invoke test_unit
      create test/system/users_test.rb
```

Así es como se ve una prueba de sistema recién generada:

```ruby
require "application_system_test_case"

class UsersTest < ApplicationSystemTestCase
  # test "visiting the index" do
  #   visit users_url
  #
  #   assert_selector "h1", text: "Users"
  # end
end
```

De forma predeterminada, las pruebas del sistema se ejecutan con el controlador Selenium, utilizando Chrome
navegador y un tamaño de pantalla de 1400x1400. La siguiente sección explica cómo
cambiar la configuración predeterminada.

### Changing the default settings

Rails hace que cambiar la configuración predeterminada para las pruebas del sistema sea muy simple. Todas
la configuración se abstrae para que pueda concentrarse en escribir sus pruebas.

Cuando generas una nueva aplicación o andamio, un archivo `application_system_test_case.rb`
se crea en el directorio de prueba. Aquí es donde toda la configuración de su
las pruebas del sistema deberían estar activas.

Si desea cambiar la configuración predeterminada, puede cambiar lo que el sistema
las pruebas son "impulsadas por". Digamos que desea cambiar el controlador de Selenium a
Duende. Primero agregue la gema `poltergeist` a su `Gemfile`. Entonces en tu
El archivo `application_system_test_case.rb` haga lo siguiente:

```ruby
require "test_helper"
require "capybara/poltergeist"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :poltergeist
end
```

El nombre del controlador es un argumento obligatorio para `drive_by`. Los argumentos opcionales
que se pueden pasar a `drive_by` son `:using` para el navegador (esto solo
ser utilizado por Selenium), `:screen_size` para cambiar el tamaño de la pantalla para
capturas de pantalla y `:options` que se pueden utilizar para configurar las opciones admitidas por
conductor.

```ruby
require "test_helper"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :firefox
end
```

Si desea utilizar un navegador sin cabeza, puede usar Chrome sin cabeza o Firefox sin cabeza agregando
`headless_chrome` o `headless_firefox` en el argumento `:using`.

```ruby
require "test_helper"

class ApplicationSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :headless_chrome
end
```

Si su configuración de Capybara requiere más configuración que la proporcionada por Rails, esto
Se podría agregar configuración adicional en el `application_system_test_case.rb`
expediente.

Consulte la [Capybara's documentation](https://github.com/teamcapybara/capybara#setup)
para configuraciones adicionales.

### Screenshot Helper

El `ScreenshotHelper` es un ayudante diseñado para capturar capturas de pantalla de sus pruebas.
Esto puede ser útil para ver el navegador en el punto en el que falló una prueba, o
para ver capturas de pantalla más tarde para depurar.

Se proporcionan dos métodos: `take_screenshot` y `take_failed_screenshot`.
`take_failed_screenshot` se incluye automáticamente en `before_teardown` dentro
Rieles.

El método auxiliar `take_screenshot` se puede incluir en cualquier lugar de sus pruebas para
tomar una captura de pantalla del navegador.

### Implementing a System Test

Ahora vamos a agregar una prueba del sistema a nuestra aplicación de blog. Demostraremos
escribir una prueba del sistema visitando la página de índice y creando un nuevo artículo de blog.

Si usó el generador de andamios, un esqueleto de prueba del sistema se
creado para ti. Si no usó el generador de andamios, comience creando un
esqueleto de prueba del sistema.

```bash
$ bin/rails generate system_test articles
```

Debería habernos creado un marcador de posición de archivo de prueba. Con la salida del
comando anterior debes

```bash
      invoke  test_unit
      create    test/system/articles_test.rb
```

Ahora abramos ese archivo y escribamos nuestra primera afirmación:

```ruby
require "application_system_test_case"

class ArticlesTest < ApplicationSystemTestCase
  test "viewing the index" do
    visit articles_path
    assert_selector "h1", text: "Articles"
  end
end
```

La prueba debería ver que hay un `h1` en la página de índice de artículos y pasar.

Ejecute las pruebas del sistema.

```bash
bin/rails test:system
```

NOTE: Por defecto, ejecutar `bin/rails test` no ejecutará las pruebas de su sistema.
Asegúrese de ejecutar `bin/rails test:system` para ejecutarlos.
También puede ejecutar `bin/rails test:all` para ejecutar todas las pruebas, incluidas las del sistema.

#### Creating Articles System Test

Ahora probemos el flujo para crear un nuevo artículo en nuestro blog.

```ruby
test "creating an article" do
  visit articles_path

  click_on "New Article"

  fill_in "Title", with: "Creating an Article"
  fill_in "Body", with: "Created this article successfully!"

  click_on "Create Article"

  assert_text "Creating an Article"
end
```

El primer paso es llamar a `visit articles_path`. Esto llevará la prueba al
página de índice de artículos.

Luego, el `click_on New Article"` encontrará el botón "Nuevo artículo" en el
Página de inicio. Esto redirigirá el navegador a `/articles/new`.

Luego, la prueba completará el título y el cuerpo del artículo con la especificación
texto. Una vez completados los campos, se hace clic en "Crear artículo", que
envíe una solicitud POST para crear el nuevo artículo en la base de datos.

Seremos redirigidos de nuevo a la página de índice de artículos y allí afirmamos
que el texto del título del nuevo artículo está en la página de índice de artículos.

#### Testing for multiple screen sizes
Si desea probar los tamaños de dispositivos móviles además de las pruebas para computadoras de escritorio,
puede crear otra clase que herede de SystemTestCase y usar en su
Banco de pruebas. En este ejemplo se crea un archivo llamado `mobile_system_test_case.rb`
en el directorio `/test` con la siguiente configuración.

```ruby
require "test_helper"

class MobileSystemTestCase < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :chrome, screen_size: [375, 667]
end
```
Para usar esta configuración, cree una prueba dentro de `test/system` que herede de `MobileSystemTestCase`.
Ahora puede probar su aplicación utilizando varias configuraciones diferentes.

```ruby
require "mobile_system_test_case"

class PostsTest < MobileSystemTestCase

  test "visiting the index" do
    visit posts_url
    assert_selector "h1", text: "Posts"
  end
end
```

#### Taking it further

La belleza de las pruebas del sistema es que es similar a las pruebas de integración en
que prueba la interacción del usuario con su controlador, modelo y vista, pero
La prueba del sistema es mucho más robusta y realmente prueba su aplicación como si
un usuario real lo estaba usando. En el futuro, puede probar cualquier cosa que el usuario
ellos mismos harían en su aplicación, como comentar, eliminar artículos,
publicar borradores de artículos, etc.

Integration Testing
-------------------

Las pruebas de integración se utilizan para probar cómo interactúan varias partes de su aplicación. Generalmente se utilizan para probar flujos de trabajo importantes dentro de nuestra aplicación.

Para crear pruebas de integración de Rails, usamos el directorio `test/integration` para nuestra aplicación. Rails proporciona un generador para crear un esqueleto de prueba de integración para nosotros.

```bash
$ bin/rails generate integration_test user_flows
      exists  test/integration/
      create  test/integration/user_flows_test.rb
```

Así es como se ve una prueba de integración recién generada:

```ruby
require "test_helper"

class UserFlowsTest < ActionDispatch::IntegrationTest
  # test "the truth" do
  #   assert true
  # end
end
```

Aquí la prueba hereda de `ActionDispatch::IntegrationTest`. Esto hace que algunos ayudantes adicionales estén disponibles para que los usemos en nuestras pruebas de integración.

### Helpers Available for Integration Tests

Además de los ayudantes de prueba estándar, heredando de `ActionDispatch::IntegrationTest` viene con algunos ayudantes adicionales disponibles al escribir pruebas de integración. Vamos a presentarnos brevemente a las tres categorías de ayudantes entre las que podemos elegir.

Para tratar con el ejecutor de pruebas de integración, consulte [`ActionDispatch::Integration::Runner`](https://api.rubyonrails.org/classes/ActionDispatch/Integration/Runner.html).

Al realizar solicitudes, haremos [`ActionDispatch::Integration::RequestHelpers`](https://api.rubyonrails.org/classes/ActionDispatch/Integration/RequestHelpers.html) disponible para nuestro uso.

Si necesitamos modificar la sesión o el estado de nuestra prueba de integración, eche un vistazo a [`ActionDispatch::Integration::Session`](https://api.rubyonrails.org/classes/ActionDispatch/Integration/Session.html) para ayuda.

### Implementing an integration test

Agreguemos una prueba de integración a nuestra aplicación de blog. Comenzaremos con un flujo de trabajo básico de creación de un nuevo artículo de blog, para verificar que todo funcione correctamente.

Comenzaremos generando nuestro esqueleto de prueba de integración:

```bash
$ bin/rails generate integration_test blog_flow
```

Debería habernos creado un marcador de posición de archivo de prueba. Con la salida del
comando anterior deberíamos ver:

```bash
      invoke  test_unit
      create    test/integration/blog_flow_test.rb
```

Ahora abramos ese archivo y escribamos nuestra primera afirmación:

```ruby
require "test_helper"

class BlogFlowTest < ActionDispatch::IntegrationTest
  test "can see the welcome page" do
    get "/"
    assert_select "h1", "Welcome#index"
  end
end
```

Echaremos un vistazo a `assert_select` para consultar el HTML resultante de una solicitud en la sección" Vistas de prueba "a continuación. Se utiliza para probar la respuesta de nuestra solicitud al afirmar la presencia de elementos HTML clave y su contenido.

Cuando visitamos nuestra ruta raíz, deberíamos ver `welcome/index.html.erb` renderizado para la vista. Entonces esta afirmación debería pasar.

#### Creating articles integration

¿Qué tal probar nuestra capacidad para crear un artículo nuevo en nuestro blog y ver el artículo resultante?

```ruby
test "can create an article" do
  get "/articles/new"
  assert_response :success

  post "/articles",
    params: { article: { title: "can create", body: "article successfully." } }
  assert_response :redirect
  follow_redirect!
  assert_response :success
  assert_select "p", "Title:\n  can create"
end
```

Analicemos esta prueba para que podamos entenderla.

Comenzamos llamando a la acción `:new` en nuestro controlador de Artículos. Esta respuesta debería tener éxito.

Después de esto, hacemos una solicitud de publicación a la acción `:create` de nuestro controlador de artículos:

```ruby
post "/articles",
  params: { article: { title: "can create", body: "article successfully." } }
assert_response :redirect
follow_redirect!
```

Las dos líneas que siguen a la solicitud son para manejar la redirección que configuramos al crear un nuevo artículo.

NOTE: No olvide llamar a `follow_redirect!` Si planea realizar solicitudes posteriores después de realizar una redirección.

Finalmente podemos afirmar que nuestra respuesta fue exitosa y nuestro nuevo artículo se puede leer en la página.

#### Taking it further

Pudimos probar con éxito un flujo de trabajo muy pequeño para visitar nuestro blog y crear un nuevo artículo. Si quisiéramos llevar esto más lejos, podríamos agregar pruebas para comentar, eliminar artículos o editar comentarios. Las pruebas de integración son un gran lugar para experimentar con todo tipo de casos de uso para nuestras aplicaciones.


Functional Tests for Your Controllers
-------------------------------------

En Rails, probar las diversas acciones de un controlador es una forma de escribir pruebas funcionales. Recuerde que sus controladores manejan las solicitudes web entrantes a su aplicación y eventualmente responden con una vista renderizada. Al escribir pruebas funcionales, está probando cómo sus acciones manejan las solicitudes y el resultado o respuesta esperados, en algunos casos una vista HTML.

### What to include in your Functional Tests

Debes probar cosas como:

* ¿La solicitud web se realizó correctamente?
* ¿Se redirigió al usuario a la página correcta?
* ¿Se autenticó correctamente al usuario?
* ¿Se mostró el mensaje apropiado al usuario en la vista?
* ¿Se mostró la información correcta en la respuesta?

La forma más fácil de ver las pruebas funcionales en acción es generar un controlador usando el generador de andamios:

```bash
$ bin/rails generate scaffold_controller article title:string body:text
...
create  app/controllers/articles_controller.rb
...
invoke  test_unit
create    test/controllers/articles_controller_test.rb
...
```

Esto generará el código del controlador y las pruebas para un recurso `Article`.
Puede echar un vistazo al archivo `articles_controller_test.rb` en el directorio `test/controllers`.

Si ya tiene un controlador y solo desea generar el código de andamio de prueba para
cada una de las siete acciones predeterminadas, puede utilizar el siguiente comando:

```bash
$ bin/rails generate test_unit:scaffold article
...
invoke  test_unit
create    test/controllers/articles_controller_test.rb
...
```

Echemos un vistazo a una de esas pruebas, `test_should_get_index` del archivo `articles_controller_test.rb`.

```ruby
# articles_controller_test.rb
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get articles_url
    assert_response :success
  end
end
```

En la prueba `test_should_get_index`, Rails simula una solicitud en la acción llamada `index`, asegurándose de que la solicitud fue exitosa
y también garantizar que se haya generado el organismo de respuesta adecuado.

El método `get` inicia la solicitud web y completa los resultados en la `@response`. Puede aceptar hasta 6 argumentos:

* El URI de la acción del controlador que está solicitando.
  Esto puede tener la forma de una cadena o un asistente de ruta (por ejemplo, `articles_url`).
* `params`: opción con un hash de parámetros de solicitud para pasar a la acción
  (por ejemplo, parámetros de cadena de consulta o variables de artículo).
* `headers`: para configurar los encabezados que se pasarán con la solicitud.
* `env`: para personalizar el entorno de solicitud según sea necesario.
* `xhr`: si la solicitud es una solicitud Ajax o no. Se puede establecer en verdadero para marcar la solicitud como Ajax.
* `as`: para codificar la solicitud con un tipo de contenido diferente.

Todos estos argumentos de palabras clave son opcionales.

Ejemplo: llamar a la acción `:show` para el primer` Article`, pasando un encabezado `HTTP_REFERER`:

```ruby
get article_url(Article.first), headers: { "HTTP_REFERER" => "http://example.com/home" }
```

Otro ejemplo: llamar a la acción `:update` para el último `article`, pasando un nuevo texto para el `title` en `params`, como una solicitud Ajax:


```ruby
patch article_url(Article.last), params: { article: { title: "updated" } }, xhr: true
```

Un ejemplo más: llamar a la acción `:create` para crear un nuevo artículo, pasando
texto para el `título` en `params`, como solicitud JSON:

```ruby
post articles_path, params: { article: { title: "Ahoy!" } }, as: :json
```

NOTE: Si intenta ejecutar la prueba `test_should_create_article` desde `articles_controller_test.rb`, fallará debido a la validación de nivel de modelo recién agregada y con razón.

Modifiquemos la prueba `test_should_create_article` en `articles_controller_test.rb` para que todas nuestras pruebas pasen:

```ruby
test "should create article" do
  assert_difference("Article.count") do
    post articles_url, params: { article: { body: "Rails is awesome!", title: "Hello Rails" } }
  end

  assert_redirected_to article_path(Article.last)
end
```

Ahora puede intentar ejecutar todas las pruebas y deberían pasar.

NOTE: Si siguió los pasos de la sección [Basic Authentication](getting_started.html#basic-authentication). Deberá agregar autorización a cada encabezado de solicitud para que todas las pruebas pasen:

```ruby
post articles_url, params: { article: { body: "Rails is awesome!", title: "Hello Rails" } }, headers: { Authorization: ActionController::HttpAuthentication::Basic.encode_credentials("dhh", "secret") }
```

### Available Request Types for Functional Tests

Si está familiarizado con el protocolo HTTP, sabrá que `get` es un tipo de solicitud. Hay 6 tipos de solicitud admitidos en las pruebas funcionales de Rails:

* `get`
* `post`
* `patch`
* `put`
* `head`
* `delete`

Todos los tipos de solicitud tienen métodos equivalentes que puede utilizar. En un típico C.R.U.D. aplicación que usará `get`, `post`, `put` y `delete` más a menudo.

NOTE: Las pruebas funcionales no verifican si la acción acepta el tipo de solicitud especificado, estamos más preocupados por el resultado. Las pruebas de solicitud existen para este caso de uso para que sus pruebas sean más útiles.

### Testing XHR (AJAX) requests

Para probar las solicitudes AJAX, puede especificar la opción `xhr: true` en `get`, `post`,
Métodos `patch`, `put` y `delete`. Por ejemplo

```ruby
test "ajax request" do
  article = articles(:one)
  get article_url(article), xhr: true

  assert_equal "hello world", @response.body
  assert_equal "text/javascript", @response.media_type
end
```


### The Three Hashes of the Apocalypse

Después de que se haya realizado y procesado una solicitud, tendrá 3 objetos Hash listos para usar:

* `cookies` - cualquier cookie que se establezca
* `flash` - cualquier objeto que viva en el flash
* `session` - cualquier objeto que viva en variables de sesión

Como es el caso de los objetos Hash normales, puede acceder a los valores haciendo referencia a las claves por cadena. También puede hacer referencia a ellos por el nombre del símbolo. Por ejemplo:

```ruby
flash["gordon"]               flash[:gordon]
session["shmession"]          session[:shmession]
cookies["are_good_for_u"]     cookies[:are_good_for_u]
```

### Instance Variables Available

También tiene acceso a tres variables de instancia en sus pruebas funcionales, después de que se realiza una solicitud:

* `@controller` - El controlador que procesa la solicitud
* `@request` - El objeto de solicitud
* `@response` - El objeto de respuesta

```ruby
class ArticlesControllerTest < ActionDispatch::IntegrationTest
  test "should get index" do
    get articles_url

    assert_equal "index", @controller.action_name
    assert_equal "application/x-www-form-urlencoded", @request.media_type
    assert_match "Articles", @response.body
  end
end
```

### Setting Headers and CGI variables

[HTTP headers](https://tools.ietf.org/search/rfc2616#section-5.3)
y
[CGI variables](https://tools.ietf.org/search/rfc3875#section-4.1)
se pueden pasar como encabezados:

```ruby
# setting an HTTP Header
get articles_url, headers: { "Content-Type": "text/plain" } # simulate the request with custom header

# setting a CGI variable
get articles_url, headers: { "HTTP_REFERER": "http://example.com/home" } # simulate the request with custom env variable
```

### Testing `flash` notices

Si recuerdas de antes, uno de los Tres Hashes del Apocalipsis fue "flash".

Queremos agregar un mensaje `flash` a nuestra aplicación de blog cada vez que alguien
crea correctamente un nuevo artículo.

Comencemos agregando esta afirmación a nuestra prueba `test_should_create_article`:

```ruby
test "should create article" do
  assert_difference("Article.count") do
    post articles_url, params: { article: { title: "Some title" } }
  end

  assert_redirected_to article_path(Article.last)
  assert_equal "Article was successfully created.", flash[:notice]
end
```

Si ejecutamos nuestra prueba ahora, deberíamos ver una falla:

```bash
$ bin/rails test test/controllers/articles_controller_test.rb -n test_should_create_article
Run options: -n test_should_create_article --seed 32266

# Running:

F

Finished in 0.114870s, 8.7055 runs/s, 34.8220 assertions/s.

  1) Failure:
ArticlesControllerTest#test_should_create_article [/test/controllers/articles_controller_test.rb:16]:
--- expected
+++ actual
@@ -1 +1 @@
-"Article was successfully created."
+nil

1 runs, 4 assertions, 1 failures, 0 errors, 0 skips
```

Implementemos el mensaje flash ahora en nuestro controlador. Nuestra acción `:create` ahora debería verse así:

```ruby
def create
  @article = Article.new(article_params)

  if @article.save
    flash[:notice] = "Article was successfully created."
    redirect_to @article
  else
    render "new"
  end
end
```

Ahora, si ejecutamos nuestras pruebas, deberíamos verlo pasar:

```bash
$ bin/rails test test/controllers/articles_controller_test.rb -n test_should_create_article
Run options: -n test_should_create_article --seed 18981

# Running:

.

Finished in 0.081972s, 12.1993 runs/s, 48.7972 assertions/s.

1 runs, 4 assertions, 0 failures, 0 errors, 0 skips
```

### Putting it together

En este punto, nuestro controlador de artículos prueba las acciones `:index` y `:new` y `:create`. ¿Qué hay de tratar con datos existentes?

Escribamos una prueba para la acción `:show`:

```ruby
test "should show article" do
  article = articles(:one)
  get article_url(article)
  assert_response :success
end
```

Recuerde de nuestra discusión anterior sobre accesorios, el método `articles()` nos dará acceso a nuestros accesorios de Artículos.

¿Qué tal eliminar un artículo existente?

```ruby
test "should destroy article" do
  article = articles(:one)
  assert_difference("Article.count", -1) do
    delete article_url(article)
  end

  assert_redirected_to articles_path
end
```

También podemos agregar una prueba para actualizar un artículo existente.

```ruby
test "should update article" do
  article = articles(:one)

  patch article_url(article), params: { article: { title: "updated" } }

  assert_redirected_to article_path(article)
  # Reload association to fetch updated data and assert that title is updated.
  article.reload
  assert_equal "updated", article.title
end
```

Observe que estamos comenzando a ver algunas duplicaciones en estas tres pruebas, ambas acceden a los mismos datos de fijación del artículo. Podemos D.R.Y. esto con los métodos `setup` y` teardown` proporcionados por `ActiveSupport :: Callbacks`.

Nuestra prueba ahora debería tener el siguiente aspecto. Ignore las otras pruebas por ahora, las dejamos fuera por brevedad.


```ruby
require "test_helper"

class ArticlesControllerTest < ActionDispatch::IntegrationTest
  # called before every single test
  setup do
    @article = articles(:one)
  end

  # called after every single test
  teardown do
    # when controller is using cache it may be a good idea to reset it afterwards
    Rails.cache.clear
  end

  test "should show article" do
    # Reuse the @article instance variable from setup
    get article_url(@article)
    assert_response :success
  end

  test "should destroy article" do
    assert_difference("Article.count", -1) do
      delete article_url(@article)
    end

    assert_redirected_to articles_path
  end

  test "should update article" do
    patch article_url(@article), params: { article: { title: "updated" } }

    assert_redirected_to article_path(@article)
    # Reload association to fetch updated data and assert that title is updated.
    @article.reload
    assert_equal "updated", @article.title
  end
end
```

De manera similar a otras devoluciones de llamada en Rails, los métodos `setup` y `teardown` también se pueden usar pasando un bloque, lambda o nombre de método como símbolo para llamar.

### Test helpers

Para evitar la duplicación de código, puede agregar sus propios ayudantes de prueba.
El asistente de inicio de sesión puede ser un buen ejemplo:

```ruby
# test/test_helper.rb

module SignInHelper
  def sign_in_as(user)
    post sign_in_url(email: user.email, password: user.password)
  end
end

class ActionDispatch::IntegrationTest
  include SignInHelper
end
```

```ruby
require "test_helper"

class ProfileControllerTest < ActionDispatch::IntegrationTest

  test "should show profile" do
    # helper is now reusable from any controller test case
    sign_in_as users(:david)

    get profile_url
    assert_response :success
  end
end
```

#### Using Separate Files

Si encuentra que sus ayudantes están saturando `test_helper.rb`, puede extraerlos en archivos separados.
Un buen lugar para almacenarlos es `test/lib` o `test/test_helpers`.

```ruby
# test/test_helpers/multiple_assertions.rb
module MultipleAssertions
  def assert_multiple_of_forty_two(number)
    assert (number % 42 == 0), 'expected #{number} to be a multiple of 42'
  end
end
```

Estos ayudantes pueden requerirse explícitamente según sea necesario e incluirse según sea necesario

```ruby
require "test_helper"
require "test_helpers/multiple_assertions"

class NumberTest < ActiveSupport::TestCase
  include MultipleAssertions

  test "420 is a multiple of forty two" do
    assert_multiple_of_forty_two 420
  end
end
```

o pueden seguir incluyéndose directamente en las clases principales relevantes

```ruby
# test/test_helper.rb
require "test_helpers/sign_in_helper"

class ActionDispatch::IntegrationTest
  include SignInHelper
end
```

#### Eagerly Requiring Helpers

Puede que le resulte conveniente solicitar ayudantes en `test_helper.rb` para que sus archivos de prueba tengan acceso implícito a ellos. Esto se puede lograr usando globbing, de la siguiente manera

```ruby
# test/test_helper.rb
Dir[Rails.root.join("test", "test_helpers", "**", "*.rb")].each { |file| require file }
```

Esto tiene la desventaja de aumentar el tiempo de inicio, en lugar de requerir manualmente solo los archivos necesarios en sus pruebas individuales.

Testing Routes
--------------

Como todo lo demás en su aplicación Rails, puede probar sus rutas. Las pruebas de ruta residen en `test/controllers/` o son parte de las pruebas del controlador.

NOTA: Si su aplicación tiene rutas complejas, Rails proporciona una serie de ayudantes útiles para probarlas.

Para obtener más información sobre las aserciones de enrutamiento disponibles en Rails, consulte la documentación de la API para [`ActionDispatch::Assertions::RoutingAssertions`](https://api.rubyonrails.org/classes/ActionDispatch/Assertions/RoutingAssertions.html).

Testing Views
-------------

Probar la respuesta a su solicitud afirmando la presencia de elementos HTML clave y su contenido es una forma común de probar las vistas de su aplicación. Al igual que las pruebas de ruta, las pruebas de vista residen en `test / controllers /` o son parte de las pruebas del controlador. El método `assert_select` le permite consultar elementos HTML de la respuesta usando una sintaxis simple pero poderosa.

Hay dos formas de `assert_select`:

`assert_select(selector, [equality], [message])` asegura que se cumpla la condición de igualdad en los elementos seleccionados a través del selector. El selector puede ser una expresión de selector de CSS (cadena) o una expresión con valores de sustitución.

`assert_select(element, selector, [equality], [message])` asegura que se cumpla la condición de igualdad en todos los elementos seleccionados a través del selector comenzando desde el _elemento_ (instancia de `Nokogiri::XML::Node` or `Nokogiri::XML::NodeSet`) y sus descendientes.

Por ejemplo, puede verificar el contenido del elemento del título en su respuesta con:

```ruby
assert_select "title", "Welcome to Rails Testing Guide"
```

También puede usar bloques anidados `assert_select` para una investigación más profunda.

En el siguiente ejemplo, se ejecuta el `assert_select` interno para `li.menu_item`
dentro de la colección de elementos seleccionados por el bloque exterior:

```ruby
assert_select "ul.navigation" do
  assert_select "li.menu_item"
end
```

Se puede iterar una colección de elementos seleccionados de modo que se pueda llamar a `assert_select` por separado para cada elemento.

Por ejemplo, si la respuesta contiene dos listas ordenadas, cada una con cuatro elementos de lista anidados, las siguientes pruebas pasarán.

```ruby
assert_select "ol" do |elements|
  elements.each do |element|
    assert_select element, "li", 4
  end
end

assert_select "ol" do
  assert_select "li", 8
end
```

Esta afirmación es bastante poderosa. Para un uso más avanzado, consulte su [documentation](https://github.com/rails/rails-dom-testing/blob/master/lib/rails/dom/testing/assertions/selector_assertions.rb).

#### Additional View-Based Assertions

Hay más afirmaciones que se utilizan principalmente en las vistas de prueba:

| Assertion                                                 | Purpose |
| --------------------------------------------------------- | ------- |
| `assert_select_email`                                     | Le permite hacer afirmaciones en el cuerpo de un correo electrónico |
| `assert_select_encoded`                                   | Le permite hacer afirmaciones en HTML codificado. Para ello, descifra el contenido de cada elemento y luego llama al bloque con todos los elementos no codificados.|
| `css_select(selector)` or `css_select(element, selector)` | Devuelve una matriz de todos los elementos seleccionados por el _selector_. En la segunda variante, primero coincide con el _element_ base e intenta hacer coincidir la expresión _selector_ en cualquiera de sus hijos. Si no hay coincidencias, ambas variantes devuelven una matriz vacía.|

A continuación, se muestra un ejemplo del uso de `assert_select_email`:

```ruby
assert_select_email do
  assert_select "small", "Please click the 'Unsubscribe' link if you want to opt-out."
end
```

Testing Helpers
---------------

Un ayudante es solo un módulo simple donde puede definir métodos que son
disponible en sus vistas.

Para probar los ayudantes, todo lo que necesita hacer es verificar que la salida del
El método auxiliar coincide con lo esperado. Las pruebas relacionadas con los ayudantes son
ubicado en el directorio `test/helpers`.

Dado que tenemos el siguiente ayudante:

```ruby
module UsersHelper
  def link_to_user(user)
    link_to "#{user.first_name} #{user.last_name}", user
  end
end
```

Podemos probar la salida de este método de esta manera:

```ruby
class UsersHelperTest < ActionView::TestCase
  test "should return the user's full name" do
    user = users(:david)

    assert_dom_equal %{<a href="/user/#{user.id}">David Heinemeier Hansson</a>}, link_to_user(user)
  end
end
```

Además, dado que la clase de prueba se extiende desde `ActionView::TestCase`, tienes
acceso a los métodos auxiliares de Rails como `link_to` o `pluralize`.


Testing Your Mailers
--------------------

Testing mailer classes requires some specific tools to do a thorough job.


### Keeping the Postman in Check

Sus clases de correo, como cualquier otra parte de su aplicación Rails, deben probarse para asegurarse de que funcionan como se espera.

Los objetivos de probar las clases de correo electrónico son garantizar que:

* los correos electrónicos están siendo procesados ​​(creados y enviados)
* el contenido del correo electrónico es correcto (asunto, remitente, cuerpo, etc.)
* los correos electrónicos correctos se envían en el momento adecuado

#### From All Sides

Hay dos aspectos de probar su mailer, las pruebas unitarias y las pruebas funcionales. En las pruebas unitarias, ejecuta el mailer de forma aislada con entradas estrictamente controladas y compara la salida con un valor conocido (un accesorio). En las pruebas funcionales, no prueba tanto los detalles minuciosos producidos por el mailer; en su lugar, probamos que nuestros controladores y modelos estén usando el correo de la manera correcta. Prueba para demostrar que se envió el correo electrónico correcto en el momento correcto.


### Unit Testing

Para probar que su envío de correo funciona como se esperaba, puede usar pruebas unitarias para comparar los resultados reales del envío de correo con ejemplos preescritos de lo que debería producirse.

#### Revenge of the Fixtures

A los efectos de la prueba unitaria de un mailer, los accesorios se utilizan para proporcionar un ejemplo de cómo debería verse la salida. Debido a que estos son correos electrónicos de ejemplo y no datos de Active Record como los otros dispositivos, se guardan en su propio subdirectorio aparte de los otros dispositivos. El nombre del directorio dentro de `test/fixtures` corresponde directamente al nombre del correo. Entonces, para un mailer llamado `UserMailer`, los dispositivos deben residir en el directorio `test/fixtures/user_mailer`.

Si generó su mailer, el generador no crea accesorios stub para las acciones de mailers. Tendrá que crear esos archivos usted mismo como se describe anteriormente.

#### The Basic Test Case

Aquí hay una prueba unitaria para probar un correo llamado `UserMailer` cuya acción `invite` se usa para enviar una invitación a un amigo. Es una versión adaptada de la prueba base creada por el generador para una acción de `invite`.

```ruby
require "test_helper"

class UserMailerTest < ActionMailer::TestCase
  test "invite" do
    # Create the email and store it for further assertions
    email = UserMailer.create_invite("me@example.com",
                                     "friend@example.com", Time.now)

    # Send the email, then test that it got queued
    assert_emails 1 do
      email.deliver_now
    end

    # Test the body of the sent email contains what we expect it to
    assert_equal ["me@example.com"], email.from
    assert_equal ["friend@example.com"], email.to
    assert_equal "You have been invited by me@example.com", email.subject
    assert_equal read_fixture("invite").join, email.body.to_s
  end
end
```

En la prueba creamos el correo electrónico y almacenamos el objeto devuelto en el `email`
variable. Luego nos aseguramos de que se envió (la primera afirmación), luego, en el
segundo lote de afirmaciones, nos aseguramos de que el correo electrónico realmente contenga lo que
esperar. El ayudante `read_fixture` se usa para leer el contenido de este archivo.

NOTA: `email.body.to_s` está presente cuando solo hay una parte (HTML o texto) presente.
Si el correo proporciona ambos, puede probar su accesorio contra partes específicas
con `email.text_part.body.to_s` o `email.html_part.body.to_s`.

Aquí está el contenido del accesorio de "invitación":

```
Hi friend@example.com,

You have been invited.

Cheers!
```

Este es el momento adecuado para comprender un poco más sobre la redacción de pruebas para su
mailers. La línea `ActionMailer::Base.delivery_method = :test` en
`config/environment/test.rb` establece el método de entrega en modo de prueba para que
el correo electrónico no se entregará realmente (útil para evitar enviar spam a sus usuarios mientras
testing) pero en su lugar se agregará a una matriz
(`ActionMailer::Base.deliveries`).

NOTA: La matriz `ActionMailer::Base.deliveries` solo se restablece automáticamente en
Pruebas `ActionMailer::TestCase` y `ActionDispatch::IntegrationTest`.
Si desea tener una pizarra limpia fuera de estos casos de prueba, puede restablecerla
manualmente con: `ActionMailer::Base.deliveries.clear`

### Functional and System Testing

Las pruebas unitarias nos permiten probar los atributos del correo electrónico, mientras que las pruebas funcionales y del sistema nos permiten probar si las interacciones de los usuarios activan adecuadamente el envío del correo electrónico. Por ejemplo, puede verificar que la operación de invitación a un amigo envíe un correo electrónico de manera adecuada:


```ruby
# Integration Test
require "test_helper"

class UsersControllerTest < ActionDispatch::IntegrationTest
  test "invite friend" do
    # Asserts the difference in the ActionMailer::Base.deliveries
    assert_emails 1 do
      post invite_friend_url, params: { email: "friend@example.com" }
    end
  end
end
```

```ruby
# System Test
require "test_helper"

class UsersTest < ActionDispatch::SystemTestCase
  driven_by :selenium, using: :headless_chrome

  test "inviting a friend" do
    visit invite_users_url
    fill_in "Email", with: "friend@example.com"
    assert_emails 1 do
      click_on "Invite"
    end
  end
end
```

NOTE: El método `assert_emails` no está vinculado a un método de entrega en particular y funcionará con los correos electrónicos entregados con el método `deliver_now` o `deliver_later`. Si queremos afirmar explícitamente que el correo electrónico se ha puesto en cola, podemos usar el método `assert_enqueued_emails`. Puede encontrar más información en la [documentaion here](https://api.rubyonrails.org/classes/ActionMailer/TestHelper.html).

Testing Jobs
------------

Dado que sus trabajos personalizados se pueden poner en cola en diferentes niveles dentro de su aplicación,
deberá probar ambos trabajos por sí mismos (su comportamiento cuando se ponen en cola)
y que otras entidades los pongan correctamente en cola.

### A Basic Test Case

De forma predeterminada, cuando genere un trabajo, también se generará una prueba asociada
en el directorio `test/jobs`. Aquí hay una prueba de ejemplo con un trabajo de facturación:


```ruby
require "test_helper"

class BillingJobTest < ActiveJob::TestCase
  test "that account is charged" do
    BillingJob.perform_now(account, product)
    assert account.reload.charged_for?(product)
  end
end
```

Esta prueba es bastante simple y solo afirma que el trabajo hizo el trabajo
como se esperaba.

De forma predeterminada, `ActiveJob::TestCase` establecerá el adaptador de cola en `:test` para que
sus trabajos se realizan en línea. También se asegurará de que todo lo realizado previamente
y los trabajos en cola se borran antes de cualquier ejecución de prueba, por lo que puede asumir con seguridad que
ya no se han ejecutado trabajos en el alcance de cada prueba.

### Custom Assertions and Testing Jobs inside Other Components

Active Job se envía con un montón de afirmaciones personalizadas que se pueden usar para disminuir la verbosidad de las pruebas. Para obtener una lista completa de las afirmaciones disponibles, consulte la documentación de la API para [`ActiveJob :: TestHelper`] (https://api.rubyonrails.org/classes/ActiveJob/TestHelper.html).

Es una buena práctica asegurarse de que sus trabajos se pongan en cola o se realicen correctamente
donde sea que los invoque (por ejemplo, dentro de sus controladores). Aquí es precisamente donde
las afirmaciones personalizadas proporcionadas por Active Job son bastante útiles. Por ejemplo,
dentro de un modelo:

```ruby
require "test_helper"

class ProductTest < ActiveSupport::TestCase
  include ActiveJob::TestHelper

  test "billing job scheduling" do
    assert_enqueued_with(job: BillingJob) do
      product.charge(account)
    end
  end
end
```

### Asserting Time Arguments in Jobs

Al serializar argumentos de trabajo, `Time`, `DateTime` y `ActiveSupport::TimeWithZone` pierden precisión de microsegundos. Esto significa que comparar el tiempo deserializado con el tiempo real no siempre funciona. Para compensar la pérdida de precisión, `assert_enqueued_with` y `assert_performed_with` eliminarán microsegundos de los objetos de tiempo en las afirmaciones de los argumentos.

```ruby
require "test_helper"

class ProductTest < ActiveSupport::TestCase
  include ActiveJob::TestHelper

  test "that product is reserved at a given time" do
    now = Time.now
    assert_performed_with(job: ReservationJob, args: [product, now]) do
      product.reserve(now)
    end
  end
end
```

Testing Action Cable
--------------------

Dado que Action Cable se utiliza en diferentes niveles dentro de su aplicación,
Deberá probar tanto los canales, las clases de conexión en sí mismas y ese otro
las entidades difunden mensajes correctos.

### Connection Test Case

De forma predeterminada, cuando genera una nueva aplicación Rails con Action Cable, también se genera una prueba para la clase de conexión base (`ApplicationCable :: Connection`) en el directorio` test / channels / application_cable`.

Las pruebas de conexión tienen como objetivo comprobar si los identificadores de una conexión se asignan correctamente
o que cualquier solicitud de conexión incorrecta sea rechazada. Aquí hay un ejemplo:

```ruby
class ApplicationCable::ConnectionTest < ActionCable::Connection::TestCase
  test "connects with params" do
    # Simulate a connection opening by calling the `connect` method
    connect params: { user_id: 42 }

    # You can access the Connection object via `connection` in tests
    assert_equal connection.user_id, "42"
  end

  test "rejects connection without params" do
    # Use `assert_reject_connection` matcher to verify that
    # connection is rejected
    assert_reject_connection { connect }
  end
end
```

También puede especificar las cookies de solicitud de la misma manera que lo hace en las pruebas de integración:

```ruby
test "connects with cookies" do
  cookies.signed[:user_id] = "42"

  connect

  assert_equal connection.user_id, "42"
end
```

Consulte la documentación de la API para [`ActionCable::Connection::TestCase`](https://api.rubyonrails.org/classes/ActionCable/Connection/TestCase.html) [`ActionCable::Connection::TestCase`](https://api.rubyonrails.org/classes/ActionCable/Connection/TestCase.html) más información.

### Channel Test Case

De forma predeterminada, cuando genere un canal, también se generará una prueba asociada
en el directorio `test/channels`. Aquí hay una prueba de ejemplo con un canal de chat:

```ruby
require "test_helper"

class ChatChannelTest < ActionCable::Channel::TestCase
  test "subscribes and stream for room" do
    # Simulate a subscription creation by calling `subscribe`
    subscribe room: "15"

    # You can access the Channel object via `subscription` in tests
    assert subscription.confirmed?
    assert_has_stream "chat_15"
  end
end
```

Esta prueba es bastante simple y solo afirma que el canal suscribe la conexión a una transmisión en particular.

También puede especificar los identificadores de conexión subyacentes. Aquí hay una prueba de ejemplo con un canal de notificaciones web:

```ruby
require "test_helper"

class WebNotificationsChannelTest < ActionCable::Channel::TestCase
  test "subscribes and stream for user" do
    stub_connection current_user: users(:john)

    subscribe

    assert_has_stream_for users(:john)
  end
end
```

Consulte la documentación de la API para [`ActionCable::Connection::TestCase`](https://api.rubyonrails.org/classes/ActionCable/Connection/TestCase.html) [`ActionCable::Connection::TestCase`](https://api.rubyonrails.org/classes/ActionCable/Connection/TestCase.html) más información.

### Custom Assertions And Testing Broadcasts Inside Other Components

Action Cable se envía con un montón de afirmaciones personalizadas que se pueden usar para disminuir la verbosidad de las pruebas. Para obtener una lista completa de las afirmaciones disponibles, consulte la documentación de la API para [`ActionCable :: TestHelper`] (https://api.rubyonrails.org/classes/ActionCable/TestHelper.html).

Es una buena práctica asegurarse de que se haya transmitido el mensaje correcto dentro de otros componentes (por ejemplo, dentro de sus controladores). Aquí es precisamente donde
las afirmaciones personalizadas proporcionadas por Action Cable son bastante útiles. Por ejemplo,
dentro de un modelo:

```ruby
require "test_helper"

class ProductTest < ActionCable::TestCase
  test "broadcast status after charge" do
    assert_broadcast_on("products:#{product.id}", type: "charged") do
      product.charge(account)
    end
  end
end
```

Si desea probar la transmisión realizada con `Channel.broadcast_to`, debe usar
`Channel.broadcasting_for` para generar un nombre de transmisión subyacente:

```ruby
# app/jobs/chat_relay_job.rb
class ChatRelayJob < ApplicationJob
  def perform_later(room, message)
    ChatChannel.broadcast_to room, text: message
  end
end

# test/jobs/chat_relay_job_test.rb
require "test_helper"

class ChatRelayJobTest < ActiveJob::TestCase
  include ActionCable::TestHelper

  test "broadcast message to room" do
    room = rooms(:all)

    assert_broadcast_on(ChatChannel.broadcasting_for(room), text: "Hi!") do
      ChatRelayJob.perform_now(room, "Hi!")
    end
  end
end
```

Additional Testing Resources
----------------------------

### Testing Time-Dependent Code

Rails proporciona métodos auxiliares integrados que le permiten afirmar que su código sensible al tiempo funciona como se esperaba.

Aquí hay un ejemplo usando el helper [`travel_to`](https://api.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html#method-i-travel_to):

```ruby
# Lets say that a user is eligible for gifting a month after they register.
user = User.create(name: "Gaurish", activation_date: Date.new(2004, 10, 24))
assert_not user.applicable_for_gifting?
travel_to Date.new(2004, 11, 24) do
  assert_equal Date.new(2004, 10, 24), user.activation_date # inside the `travel_to` block `Date.current` is mocked
  assert user.applicable_for_gifting?
end
assert_equal Date.new(2004, 10, 24), user.activation_date # The change was visible only inside the `travel_to` block.
```

Veer [`ActiveSupport::Testing::TimeHelpers` API Documentation](https://api.rubyonrails.org/classes/ActiveSupport/Testing/TimeHelpers.html)
for in-depth information about the available time helpers.
