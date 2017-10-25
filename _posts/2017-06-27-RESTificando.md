---
layout: post
title: "DDD Sesión 2"
author: "Agustín"
---
TLDR; Resumen de la Sesión RESTificando Cosas

En primer lugar, darle las gracias a Sergio y Víctor de [Arcadia Coworking](http://arcadiacoworking.com/), que no solo 
prestaron un sitio magnífico para la sesión en pleno centro de Sevilla sino que además hicieron posible poder ampliar 
sobre la marcha el número de asistentes de 15 a 30 dado el interés de la comunidad.
En segundo lugar, como no, a Jose que dió una sesión increíble dada la densidad de los temas a abordar. Jose se 
autodefine como arquitecto y programador Java y creo que eso se notó perfectamente ya que se atrevió a dar la sesión en 
formato de live coding.

El primer paso en la sesión fue hacer un breve repaso teórico de los principios de REST, por si alguien tenía alguna 
duda y compartir todos el mismo vocabulario. Por lo que pude ver estamos todos bien puestos, ¿eh?
Tras ésto Jose presentó rápidamente los tres frameworks que usaríamos para implementar la API:
* Spring Boot: Es una capa sobre Spring que nos provee una configuración rápida de las funcionalidades de Spring que 
seleccionemos. El problema de Spring Boot es las posibles incompatibilidades con versiones de Spring ya instaladas en 
el sistema y el tamaño del JAR generado.
* JAX-RS: Es el estándar de Web Services REST en Java (creo que son los JSR 311 y 339). No aporta las facilidades de 
Spring Boot para la autoconfiguración, pero se encuentra presente en todos los servidores de aplicaciones como puede 
ser JBoss o Weblogic, por tanto, su fuerte es la compatibilidad.
* Spark: Es un microframework, quizás el más limitado en funcionalidad de los tres, pero a cambio es el de menor tamaño 
y en el que es más fácil realizar prototipos rápidos o desplegar microservicios realmente pequeños.

Una vez repasado lo básico, se explicó el formato de la presentación. El objetivo era hacer un cliente mediante un HTML 
estático y consumir una API REST que devuelve JSON para mostrar la temperatura en una ciudad usando para ello JQuery. 
Mediante distintos botones en la página invocaríamos un API u otra (y todas estarían corriendo simultáneamente en la 
máquina de desarrollo).
Para la creación del cliente Jose utilizó npm ya que además del HTML estático añadió [liteserver](https://github.com/johnpapa/lite-server), 
un pequeño servidor que se actualiza automáticamente cuando detecta cambios en los fuentes que contiene. De esta forma 
en cuanto se guardan los archivos estos se refrescan en el servidor. Muy útil para desarrollar, la verdad. También 
mediante npm añadió la dependencia con JQuery.
Me pareció muy interesante que una vez creado el HTML y añadido JQuery al mismo, el siguiente paso de Jose fue crear un 
JSON de prueba en local para tenerlo siempre de referente. Esto nos permite dos cosas importantes. Primero poder 
comprobar en cualquier momento si el frontend funciona correctamente. Segundo (y ésto es mi punto de vista), estamos 
definiendo la interfaz de la API claramente en un único punto.
El siguiente paso fue definir los contenedores de los datos: temperatura y ciudad y crear la función JavaScript que 
llama a las distintas URLs, lee la respuesta y modifica los datos de los contenedores con los contenidos en esta 
respuesta. Esta función la probó, con el JSON local, creando un botón que pasaba la URL del servidor local (el 
liteserver que nos había enseñado).

Empezamos con Spring Boot. La forma más cómoda de crear el proyecto (y la que Jose siguió) es seleccionar los módulos 
que se desean en [Spring Initializr](https://start.spring.io/) que como resultado nos dará un zip con un proyecto Maven 
que podremos cargar en nuestro IDE.
La implementación fué bastante directa, una clase con un método que devuelve un POJO de tipo Tiempo y este POJO tiene 
únicamente dos atributos: ciudad y temperatura. Al coincidir los atributos con los especificados en el JSON, Spring 
automágicamente (y usando Jackson) convierte el POJO a JSON con los campos que se espera que contenga.
Para servir las peticiones se añade al método la anotación @RequestMapping y como nuestro cliente va a ser una página 
web, hay que definir la política de CORS, usando @CrossOrigin (y permitiendo todos los accesos en nuestro caso). 
Siguiendo estos pasos ya se puede arrancar la aplicación usando java -jar o directamente ejecutando la clase con el main 
que hay en el proyecto desde el IDE, que fue la opción que tomó Jose.
Para probar si efectivamente funcionaba, añadió un botón asociado a la URL de Spring Boot y andando… ¡a la primera!

JAX-RS. El problema de JAX-RS es que la configuración es completamente manual, así que creo que invertimos más tiempo en 
toda la creación inicial del proyecto que en la implementación de la API como tal.
El primer paso fue crear un proyecto Maven indicando la versión de Java 8 y que el empaquetado sería war ya que el 
objetivo es desplegarlo en un servidor de aplicaciones. Como servidor de aplicaciones y para desarrollar rápidamente, Jose 
usa [Payara Micro](https://www.payara.fish/payara_micro), un derivado de Glassfish que además de ser ligero, se puede 
empotrar. Basta con añadir la dependencia con Payara Micro al POM.
Para lanzar Payara se crea una clase con un main que lo instancia y le indica el war que tiene que desplegar y en que 
contexto. Así de de fácil. Me pareció genial tener un servidor de aplicaciones completo tan sencillo de utilizar para 
desarrollo. Jose además le indicó un puerto distinto, el 8081, para poder tener las dos APIs accesibles simultáneamente.
Siguiendo un camino similar al de Spring se crea una clase con un método que atenderá las peticiones. La recomendación 
de Jose fue anotar la clase con @Path y una ruta base para garantizar el funcionamiento, ya que si solo se anota el 
método en ocasiones funciona y en otras no.
JAX-RS no tiene la serialización a JSON usando Jackson de tapadillo como Spring Boot, así que el método que atiende las 
peticiones devuelve un objeto de tipo Response que a su vez contiene los datos de respuesta como el estado HTTP (ok), 
la cabecera de control de CORS y el POJO de Tiempo que devolvemos.
El método se marca con la anotación @GET para especificar que atiende ese tipo de peticiones y @Path (de nuevo) para 
especificar el recurso concreto.

![RESTificando1](https://drive.google.com/uc?export=view&id=0BymoUnvpt9LxNjJqSUpRUjJreVE)

Hay un último paso que dar, y es crear una clase que extiende Application y en la que se le indican las clases que 
publican nuestra API. Esta clase a su vez hay que anotarla con @ApplicationPath para indicar el contexto inicial de la 
aplicación (que se dejó vacío ya que para que estuviera en la raíz).
El último paso es generar el war (para que Payara tenga algo que desplegar) y lanzar el main.
Para probar, Jose añadió y otro botón al cliente HTML… y de nuevo a la primera. Este tío es un máquina.

Último apartado, Spark. Hasta aquí el ritmo había sido intenso y eran ya las 20:15, así que pensé que se lo iba a 
saltar o que iba a enseñar un proyecto ya terminado. Pues no. Con sangre fría y pulso de acero Jose se lanzó a crear 
un nuevo proyecto Maven, indicando Java 8 como versión a utilizar y añadiendo la dependencia de Spark.
Una clase nueva con un main en el que indica el puerto de Jetty (Spark si trae un servidor empotrado) y mediante el 
método before le indicó de nuevo la cabecera de CORS.
La devolución del String con el JSON se hizo mediante un método get sin más. Entiendo que al ser un framework tan 
diminuto, Spark no provee ningún serializador por defecto y habría que añadir a mano Jackson o Gson (cosa que me parece 
bien ya que el enfoque del framework esta en el minimalismo).
En total, un pom.xml al uso y 20 líneas de código. Espectacular, creo que como herramienta de prototipado rápido merece 
mucho la pena tener en consideración Spark.
Jose añadió rápidamente el cuarto y último botón al cliente et voilá!
Ahí teníamos la misma API implementada con tres frameworks distintos y con el JSON local para comprobación de posibles 
errores.

![RESTificando3](https://drive.google.com/uc?export=view&id=0BymoUnvpt9LxdHc1ZklrbEJrSzQ)

Llegamos al turno de preguntas y aquí me vais a perdonar, pero con 26 asistentes, no puedo recordar los nombres de todos :)
El primero en preguntar fue Joaquín, por un tema fundamental, la documentación. Por las respuestas parece que el 
estándar es Swagger (personalmente yo estoy MUY contento con Swagger), aunque otro compañero comentó que mediante Maven 
se puede generar la documentación (si lees ésto, ¿puedes pasar referencias? Creo que es interesante tener más opciones).
Yo pregunté por el tema de la securización. Si bien hay un “estándar” como es JWT, por lo que comenta Jose es 
relativamente encontrar otros esquemas como gestión de tokens con cookies, digest, OAuth…
Otro compañero nos dijo que para generar servicios web de prueba se puede usar SOAPUI que por lo visto tiene la 
funcionalidad de levantar un servidor local y devolver el JSON que le indiquemos.
Enfocándonos al tema de los microservicios se preguntó que, en opinión de Jose, cual era la mejor manera de compartir 
código entre distintos proyectos. El piensa que un repositorio de Maven propio, jar compartido y a volar.
Otro compañero (Sergio?) ahondó un poco más en la diferencia entre Spring Boot y JAX-RS, que al final pienso que podemos 
resumir en que Spring Boot es lo más cómodo (y rápido para empezar), mientras que JAX-RS lo tendrás que usar… cuando no 
haya más necesidad, jeje.
Y por último se volvió a repasar el tema CORS, sobre todo haciendo hincapié en que al final, no te puedes fiar de las 
cabeceras, ya que son fácilmente falseadas con distintas herramientas.
Con ésto ya nos habíamos comido las dos horas y pasamos a las cervezas y el networking :)

El código del ejemplo y la presentación lo tenéis en https://github.com/programaconsaba/RESTificando-cosas
