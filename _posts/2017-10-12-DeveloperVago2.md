---
layout: post
title: "DeveloperVago 2: Comunicando Microservicios"
author: "Agustín"
---
TLDR; Resumen de la Sesión DeveloperVago 2: Comunicando Microservicios

Ayer David, Team Lead en WilliamHill, nos dió una fantástica charla sobre como optimizar la comunicación entre nuestros 
microservicios.  Personalmente me gustó mucho el enfoque, muy al grano e introductorio para un tema que puede ser 
potencialmente complicadísimo.
Hoy el resumen será breve porque, entre otros motivos, nuestro ponente se curró una presentación muy apañada que tenéis 
[aquí](https://docs.google.com/presentation/d/e/2PACX-1vSq2hp_XhBLtXviT8ZfnQ4ysQAOB4jq7wGKedNOZkwon4aJHoOyf6M_st-R6wIe7b60RWiHl--nsEpd/pub?start=false&loop=false&delayms=3000&slide=id.p)

Por no entrar en más detalles, trataré de añadir alguna cosa que me llamase la atención en este mail.

El punto de partida es que los microservicios molan (otra cosa es que nos dejen usarlos, etc...) pero que la 
comunicación entre ellos (bien xml o json) es bastante ineficiente. Personalmente nunca me lo había planteado, pero si 
es verdad que en la época de SOAP ya tuve algunos problemas de comunicación por el tamaño de los XML y este problema es 
igualmente reproducible en JSON. No solo eso, sino que además perdemos tiempo en el proceso de serialización y 
deserialización, así que en determinadas circunstancias, podría ser deseable tener un protocolo más eficiente.
La solución pasa por emplear un formato binario y David nos introduce Protobuf. Lo interesante de Protobuf es que toma 
un enfoque de primero el contrato, mediante un lenguaje propio definimos una interfaz que después la misma librería 
compila a distintos lenguajes, con lo que no nos tenemos que  preocupar de posibles equivocaciones en la interpretación 
de éste contrato.
El uso en sí consiste en definir un archivo .proto en el que definimos nuestras estructuras de datos y su composición, 
básicamente, tipo de dato del atributo y posición del atributo. La posición es importante desde el punto de vista del 
versionado de la API, ya que podremos añadir nuevos atributos y eliminar los existentes, pero no sobreescribir (me 
parece bastante lógico ya que si no se podría montar un enorme lío). Los tipos de datos, además de los habituales 
(string, int...) incluyen colecciones, creo que este punto es todo un acierto ya que en  mi experiencia suele ser 
bastante habitual que expongas un set en el backend y cuando ves el cliente, lo han mapeado como un list :)
Una vez definida la estructura de datos, basta con compilarlo (proveen con un plugin de maven/gradle) y en el caso de 
Spring, se añade un MessageConverter (para que sepa como serializar/deserializar) y hará falta añadir a los métodos que 
utilicen Protobuf su oportuno produces/consumes con el tipo "application/x-protobuf". Todo muy directo y sencillo de 
implementar.
Como podéis ver en las mediciones que aporta David (hechas con Gatling), los resultados son espectaculares, no solo 
disminuyen los tiempos de respuesta a una quinta parte sino que además la desviación estándar decrece en el mismo orden 
de magnitud, lo que significa que nuestra comunicación se vuelve más eficiente.

Aún así, David le da una vuelta de tuerca más y añade a la ecuación gRPC, una implementación de RPC que usa Protobuf y 
HTTP2. El funcionamiento es sencillo, se definen las operaciones que queremos exponer en el archivo .proto y el plugin 
se encarga de generar el controller y los clientes. De nuevo podemos utilizar casi cualquier lenguaje genérico (Java, 
Python, Go, C#...). Lo importante es que de esta manera nos abstraemos del uso de HTTP (ya tenemos el cliente!) y que 
además tan solo tenemos que distribuir los archivos .proto, es decir, es una solución agnóstica del lenguaje. A mí 
particularmente los .proto me recuerdan al final a los archivos .h (header) de C...
Otro punto muy interesante de gRPC es que incluye soporte para RxJava, con lo que podemos utilizar programación 
asíncrona sobre HTTP2!! 
Para usarlo, pues incluimos la definición de nuestro método en el archivo .proto, se generan los fuentes con el plugin 
y tan solo tenemos que incluir el código en el Controller que ha generado extendiendo una clase propia. Una vez hecho 
ésto, al arrancar el contexto de Spring hay que arrancar también el servidor de gRPC (ahora digamos que tendríamos dos 
contextos). Lo bueno de esto es que nos permite tener conviviendo la versión sin gRPC y la versión con gRPC, para poder 
hacer una migración más gradual o disponer de un fallback.
Con ésto y como se pueden ver en las últimas medidas, la media de tiempos de respuesta pasa a ser de... 1 msg!!! El 
máximo se dispara hasta 558 porque gRPC levanta todo su sistema con la primera petición, por lo que puede ser 
interesante realizar esa primera petición nada más arrancar el microservicio para no penalizar al primer cliente :)

Y esta fue la charla, como ya decía muy focalizada y productiva :)

Me gustaría darle las gracias a David (como no) y a WilliamHill, no solo por patrocinar la charla, sino además por 
liberar el código e implicarse con la comunidad.
