---
layout: post
title: "Sesión Doble: DeveloperVago1: Lombok y MapStruct y Que hay de nuevo en Java 9"
author: "Agustín"
---
TLDR; Resumen de la Sesión Doble DeveloperVago1: Lombok y MapStruct y Que hay de nuevo en Java 9

La primera parte de la sesión, DeveloperVago1 la dió nuestro compañero Jorge, desarrollador senior en intelliment, que 
además aprovechó y se trajo a varios compañeros a ver si los convencía. Jorge, si después de eso no los has convencido, 
yo creo que ya los puedes dar por inmunizados, no hay nada que hacer :)

El objetivo de esta serie de DeveloperVago es introducir herramientas que nos permitan automatizar el trabajo o eliminar 
tareas repetitivas, permitiendo que nos concentremos en tareas más productivas que escribir código repetitivo. Si alguno 
estáis usando alguna herramienta de este tipo, no os cortéis y enseñádnosla!!!

En este sentido la primera librería que introdujo Jorge, [Lombok](https://projectlombok.org/), nos permite eliminar 
getters, setters, toString, constructores y otro tipo de código del que solemos generar automáticamente con el IDE. 
Tan solo añadiendo la dependencia de Lombok al pom.xml e instalando el plugin oportuno para el IDE podremos disfrutar 
de toda esta funcionalidad.
Quizás lo más explicativo sea el primer ejemplo que nos mostró Jorge, donde utilizando la anotación @Data a nivel de 
clase, automáticamente tenemos disponibles getters y setters para todos los atributos de la clase, el hashcode, el 
toString y un constructor con aquellos atributos que sean final.
Entrando en el detalle de las anotaciones disponibles, Jorge nos mostró como podemos configurar el toString generado 
con @toString e indicar los atributos que se desean utilizar con of o exclude. 
Otro caso en el que se hace necesaria la personalización es en la generación de getters y setters, por ejemplo, hay 
gente que en vez de anotar los atributos de las clases, prefieren anotar los getters en el caso de anotaciones como 
@JacksonIgnore o @Column. Pues bien, la anotación se puede indicar en la misma anotación @Getter de Lombok, de una 
forma tal que así: @Getter(onMethod=@__({@Id, @Column})).
A título personal y como disgresión, ¿alguien me podría comparar los pros/contras de anotar atributos frente a métodos 
de acceso? Yo siempre he utilizado atributos por comodidad y mantener toda la configuración junta, pero me gustaría 
que alguien me explicara los pros de usar métodos :)

Siguiendo con anotaciones básicas de Lombok, @EqualsAndHashCode se puede configurar de manera similar a @toString, ya 
que en este caso si resultaría bastante raro incluir todos los atributos, ¿no?
En cuanto a los constructores, podemos indicar que queremos el clásico constructor sin parámetros (@NoArgsConstructor), 
un constructor con aquellos atributos requeridos (que sean final o marcados como @NonNull) con @RequiredArgsConstructor 
o un constructor con tooooodos los atributos con @AllArgsConstructor.

Hasta el momento creo que podemos decir que sí, que Lombok ahorra código, pero bueno, no es nada que nuestro IDE no 
genere. Sin embargo, estas son solo las anotaciones básicas, ahora es cuando Jorge nos empezó a mostrar lo realmente 
interesante.
Por ejemplo, con @Builder se genera una clase interna que implementa el patrón builder. Además esta anotación es 
configurable y podemos por ejemplo marcar un atributo de la clase como @Singular para indicar que es una colección y no 
queremos que se genere un setter sino un add.
Y ahora que esta de moda la inmutabilidad... resulta que Lombok provee una anotación @Value que nos permite convertir 
la clase en inmutable automáticamente y generar un método factoría como un of(...) o inclusive con @Wither indicar que 
se puede hacer un set de ese atributo pero se devolverá una nueva instancia con ese atributo a ese valor (para 
entendernos, como el append de String).
Otra anotación interesante es @Log y sus hermanas @Slf4j, @Log4j2, etc..., que hacen exactamente lo que estáis 
suponiendo, ahorrarnos el tedioso trabajo de declarar el log según el framework que usemos.
Como último detalle y bastante simpático, vimos que Lombok respeta siempre tu código. Si generas un toString a mano y 
tienes la anotación @toString, esta no hace nada. Muy respetuoso, así evitamos follones.

Una vez vista la funcionalidad que nos ofrece Lombok... ¿por qué no lo estamos usando todos?¿acaso no tiene 
inconvenientes? Pues Jorge nos destacó varios (como siempre es una cuestión de pros y contras):
Lombok es un generador de código. Es decir, la podemos liar tanto con Lombok como al escribir un método. Si has 
definido mal el equals, Lombok no te libra de eso, simplemente lo automatiza.
Si bien la refactorización se agiliza (si estamos usando @Getter, al cambiar el nombre de un atributo, automáticamente 
cambia la signatura de su getter), esto sigue produciendo que se rompan las clases que utilicen los métodos (es decir, 
no se produce el efecto del refactorizar del IDE que busca las invocaciones del método y las renombra también).
Lombok funciona en tiempo de compilación, modificando la clase generada y añadiendo automáticamente el código generado 
(por eso es necesario el plugin del IDE), por lo que si tenemos clases que dependan de estos getters y setters, al 
compilarlas, primero hay que ejecutar Lombok.
Creo que en general se pudo ver como Lombok es una herramienta útil pero no tiene "inteligencia" ni es "mágica", así 
que yo personalmente os animo a usarla, pero dentro de sus limitaciones.

Y como bonus, aquí tenéis a Jorge a pleno rendimiento y a su público entregao: https://twitter.com/trisha_gee/status/913076242907320321

El siguiente apartado de Jorge, que tuvimos que cubrir un poco deprisa por falta de tiempo fue MapStruct. La verdad que 
ahora que lo pienso deberíamos de haber hecho una sesión triple más bien, ¿no? :P

MapStruct es un mapeador, es decir, nos permite pasar valores de una clase a otra. Su funcionamiento, se define una 
interfaz (por ejemplo, PersonMapper) y se marca mediante @Mapper, para a continuación definir métodos del estilo 
public PersonLink asLink(Person person). Esto automáticamente generaría una clase de implementacion PersonMapperImpl 
que tendría el código necesario para hacer esta conversión. ¿Cómo sabe que reglas aplicar a la conversión? Pues 
inspecciona la clase destino (PersonLink) y basándose en los nombres de los setters, busca los mismos getters en la 
clase de origen (Person). Si PersonLink tiene un setId(Long id), buscará en Person un Long getId().

Wops, ¿y si mi clase origen no tiene el mismo nombre de atributo que la de destino? Se puede utilizar @Mapping en el 
método para indicar las reglas de mapeo, por ejemplo, podemos decirle que el método setName de PersonLink se corresponde 
con getFullName de Person: @Mapping(source = "fullName", target = "name"). Muy bonito, pero, ¿si el tipo no es el mismo? 
¿Si mi id en PersonLink es un String? Podemos definir un conversor en la misma clase tal que así: String 
numberAsString(Long number) y automáticamente MapStruct lo utilizaría, o bien, utilizar uses en @Mapper para indicarle 
que use otro mapeador que tendría la lógica oportuna. Por ejemplo, si Person tuviera un Address y PersonLink tuviera un 
AddressLink, podríamos definir @Mapper(uses=AddressMapper.class) y MapStruct ya sabría como hacer la conversión.
Otro caso especial es si queremos hacer una conversión específica, por ejemplo, ¿y si en PersonLink queremos cargar el 
Address formateado de una forma especial? Bueno, pues como gracias a Java 8 podemos definir métodos default en las 
interfaces, podríamos escribir algo tal que así:

public default String addressAsString(Address address) {
  return address.getStreet() + " " + address.getNumber();
}

y tendríamos nuestro mapeador específico de Address para PersonLink :)
Por lo demás, podemos definir decoradores (útiles cuando por ejemplo tenemos una jerarquía y tras la conversión del 
hijo queremos ponerle el padre) y métodos pre y post que ejecutar antes y después de los mapeos (aunque Jorge nos 
advirtió de que no son buena práctica).
Por último, una anotación especialmente útil es @TargetType, si definimos public void update (PersonLink personLink, 
@TargetType Person person), nos genera automáticamente el código que carga en person aquellos atributos de personLink 
que hayan cambiado!!

Repasamos un poco la documentación (muy buena en ambos proyectos!) y la configuración del IDE y con ésto se acabó la 
hora de Jorge. Sí, vimos todo ésto en un hora, lo de que fue una sesión intensa no era broma.

Turno de Trisha, Developer Advocate de [Jetbrains](https://www.jetbrains.com/) (los creadores de Intellij IDEA, el IDE 
autor del sorpasso del año a Eclipse). En su presentación nos repasó MUY MUY MUY rápidamente las novedades de Java 9. 
Hay que tener en cuenta que en Java 9 se introduce la modularización y se ha modularizado el JDK, un proyecto que ha 
durado la friolera de 10 años y que en  mi humilde opinión era el Vietnam de Java (pa que nos vamos a engañar, yo no 
daba un duro por el proyecto Jigsaw).
La introducción de Trisha era clara. Hay que actualizar. Aunque no utilicemos las nuevas funcionalidades, Java 9, como 
todas las versiones nuevas trae un montón de ganancias gratuitas, mejora de la gestión de memoria, del rendimiento, 
nuevo recolector de basura por defecto, compilación más rápida, etc, etc, etc... Asi que simplemente merece la pena.

Entrando ya en la modularización, el enfoque de Trisha fue hacer lo que haríamos cualquiera de nosotros, ¿qué pasa si 
cojo una aplicación y la cambio directamente a Java 9? Pues lo primero que nos encontramos es que _ ya no es un 
identificador válido de variables (en serio? alguien lo usaba?).
Por otra parte se han deprecado algunos paquetes como sun.misc que tenían sustitutos en APIs públicas (como Base64), que 
se encuentra ahora en java.util. Para los paquetes que no tienen sustituto se ha creado el concepto de unssuported, que 
indica que un paquete esta deprecado.
Con tan solo estos detalles, la aplicación ya compilaba para Java 9, así que si alguno tenía miedo (como era mi caso), 
ya se lo puede ir quitando :)

Ahondando mas, Trisha nos explicó brevemente cual es el objetivo de los módulos, que no es otro que encapsular la API 
que quiero ofrecer, en cierto sentido, restringiendo el funcionamiento de public. Para definir un módulo hay que crear 
un archivo module-info.java en la raíz del paquete que deseamos definir como módulo. En ese archivo tenemos que indicar 
que otros módulos utilizaremos como puede ser org.reactivestreams para utilizar esta librería. Por defecto hay dos 
módulos que se incluyen siempre, java.lang y java.util (vaya tostón si no). Si tenemos librerías antiguas (como en el 
caso de reactivestreams), se "exporta" un módulo que se llama como el jar de la librería eliminando números de version.
En este sentido nos explicó que el classpath ya no existe y ha sido sustituido por un nuevo artefacto llamado 
modulepath, que se encarga de asegurar que un módulo este una sola vez, no haya conflictos de versiones, etc... Creo 
que esta parte es MUY importante y que nos va arreglar bastante la vida.
Una vez definido en module-info que módulos vamos a utilizar, mediante la palabra clave exports se puede definir que va 
a exportar nuestro módulo. Todo lo demás queda inaccesible, ni siquiera por reflexión. En caso de querer permitir que 
se use reflexión utilizaríamos opens, pero además hay que indicar a qué módulo se abre, y no se puede utilizar ninguna 
notación de asterisco, como podría ser opens * to *, ya que esto contravendría lo que es el nuevo sistema modular.
Como inconvenientes del sistema de módulos nos reseñó los siguientes:
* Al utilizarlo es posible que tengamos que reorganizar la estructura del proyecto.
* El soporte de los IDEs todavía no esta maduro al 100%
* Hay situaciones extrañas. Por ejemplo, para utilizar JavaFX hay que abrir algunos módulos a JavaFX ya que este los 
invoca por ejemplo para rellenar las plantillas fxml. Lo bueno es que hay una herramienta que se llama JDep que te 
clarifica las dependencias de tu proyecto.

Por supuesto, hay cosas buenas:
* Es más limpio.
* La configuración es mucho más fiable (adios a librerías repetidas en el classpath)
* Nuestros paquetes serán más pequeños ya que el linker (jlink) se encargará de incluir tan solo los módulos necesarios.

Y con ésto ya nos habíamos comido otros 25 minutos del tiempo disponible, así que repasamos brevemente otra de las 
novedades, Reactive Streams. En Java 9 se propone un estándar que van a cumplir los distintos proveedores de librerías 
de programación reactiva (¿? ¿Se dice así?) como RxJava, Akka, etc... Creo que este es un cambio "pequeño" pero muy 
importante para los que usan o estan pensando en utilizar esta tecnología.

Ahora... tooooodoooo lo demás. Por ejemplo, a nivel de lenguaje se han añadido métodos factoria para las colecciones, 
ahora podemos hacer Lists.of(1, 2,3) o Set.of("a","b"). Lo que es más importante, podemos hacer 
Map.of("uno", 1, "dos", 2, "tres", 3). Simple y efectivo.
Enlazando con lo que comenté en la sección de MapStruct, ahora también se pueden definir implementaciones para métodos 
privados en interfaces. Parece que las clases abstractas van estando cada vez más muertas, ¿no?
Para el procesamiento de streams tenemos ahora dos métodos nuevos muy útiles: takeWhile que nos permite especificar una 
condición de parada del stream (es decir, takeWhile(number -> number < 10) interrumpe el procesamiento si number vale 
10) y su homólogo dropWhile que nos permite especificar cuando empezará el procesamiento (dropWhile(number -> number 
< 20). Si usamos estas dos, ignoraríamos los números entre 10 y 20. Vaya, creo que me estoy explicando bastante mal, 
pero espero que se entienda :)
Optional también tiene mejoras similares, con métodos nuevos como or, ifPresentOrElse y stream.
Además tenemos una nueva API StackWalker para manejar más eficientemente el stack (y no de una tacada) y una nueva API 
de procesos!! (en una de estas vemos fibers y todo).
A la hora de empaquetar podemos marcar un jar como multirelease, incluyendo código compatible con varias versiones. Por 
ejemplo, podríamos tener un jar con código para Java 8 y código para Java 9. Ojo, que esto no permite cambiar la API, 
sino simplemente las clases internas de implementación, así que nos puede servir para aplicar optimizaciones, por 
ejemplo.
Y por último, a nivel de código, la deprecación también ha mejorado, ahora podremos indicar en que versión se ha 
deprecado y si se va a eliminar en un futuro.

Ah, si, también han adaptado JavaDoc al siglo XXI, ahora usa HTML5 e incluye un buscador (como podéis ver [aquí](http://download.java.net/java/jdk9/docs/api/overview-summary.html)). Ya iba tocando.

Y ahora, mi cambio favorito. El número de versión. Java 9 NO es Java1.9 sino Java9 a secas :D

Creo que los pros y contras más importantes reseñados por Trisha son los siguientes:
* La modularización hay que afrontarla con cuidado, ya que es compleja. sobre todo con bases de código ya existentes.
* Lo bueno es que se fomenta una nueva forma de trabajar, más limpia, más ordenada.
* Y además se incluyen ciertas mejoras a nivel de código que hacen que trabajar sea más cómodo y agradable.

Para el que quiera profundizar más, esta es la [página de recursos de Java 9 de Trisha](https://trishagee.github.io/presentation/real_world_java_9/)

Listo, como habéis visto la sesión fue bastante cargada de información. A mí la verdad que me ha gustado mucho el 
enfoque de DeveloperVago, pero creo que teníamos para dos horas! Y encima de premio, las novedades de Java 9 que me ha 
hecho perderle el miedo (creía que se me iba a romper todo).

Como de costumbre, este es el resumen de un servidor. Cualquier imprecisión o error o directamente que no se me 
entienda, ya sabéis de quien es responsabilidad :) Perdonad por el tostón!
