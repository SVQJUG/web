---
layout: post
title: "DeveloperVago 3: It's all about the Plugin"
author: "Agustín"
---
TLDR; Resumen de la Sesión DeveloperVago 3: It's all about the Plugin, en la que hablamos de plugins imprescindibles en Eclipse para [JuanRa](https://www.twitter.com/jr__rios)

Siguiendo con la filosofía de DeveloperVago, la sesión fue corta y MUY productiva, focalizada en facilitarnos la vida y aprovechar al máximo las herramientas que tenemos.
Esta vez también variamos un poco la filosofía y la idea fue que tras la presentación, los asistentes presentaran los plugins que usan en su día a día.
Yendo al grano, estos son los plugins que nos recomendó Juanra:

* DevStyle: Anteriormente conocido como DarkestDark, ha evolucionado de simple tema de Eclipse a "algo más", ya que nos permite configurar iconos, cambiar la splash screen y hasta escoger un tema distinto.
* QuickSearch: Este viene por defecto con STS y permite buscar una expresión regular en todos los archivos
* SonarLint: Enfocado hacia la calidad, nos dá distintos avisos sobre posibles errores en el código. Lo guay es que lo va haciendo conforme programamos, como si fueran fallos de compilación. Otra cosa muy interesante es que por defecto trae una serie de reglas, pero si tenemos un servidor de SonarQube en nuestra red, podemos sincronizarlo para que aplique las reglas que esten definidas en él.
* EclEmma: Este permite verificar la cobertura de código en nuestras pruebas. Por algún motivo, trata de calcular también las cobertura de código en los mismos tests (WTF??), así que hay que configurarlo para que ignore la carpeta de tests. Otra cosa aconsejable es indicarle que solo indique la cobertura al comienzo de las líneas, ya que por defecto asigna un color a todas las líneas y termina pareciendo un carnaval.
* MoreUnit: Como la automatización de Eclipse en cuanto a pruebas unitarias deja bastante que desear, este plugin cubre esa funcionalidad. Nos permite crear tests automáticamente con ctrl+u y con ctrl+j podemos alternar entre nuestra clase y su clase de pruebas. Además el plugin automáticamente renombra las clases de tests si renombramos nuestras clases.
* StartExplorer: La utilidad de este plugin es que permite abrir cualquier archivo en el explorador o la consola, pero no solo eso, sino que reconoce las rutas en los archivos y también nos permite abrirlas.

Pero como no todo el monte es orégano, hay una utilidad fundamental (por lo menos para los que vivimos en la edad de bronce tecnológica) que no tiene ningún buen plugin... Subversion.
La recomendación es pasar de plugin y usar [Meld](http://meldmerge.org/).

Con ésto ya la verdad que no quedaba mucho más tiempo para el público, pero aún así, Jorge nos recomendó Infinitest, que básicamente se encarga de estar a la escucha de cuando modificamos código para lanzar de nuevo todos los tests y si fallan, nos marca la clase afectada como si fuera un error de compilación.
Y por fín llegó Edu (que había pinchado la bici) y casi ni le dió tiempo a enseñarnos JBoss Tools, una colección de plugins que nos ayudan en nuestro trabajo sobre todo si utilizamos Java EE.

Muchas gracias a Juanra, y a todos los que vinisteis y comentasteis cosas sobre los plugins y si alguien tiene alguna sugerencia más, ya sabéis, enviadla a la lista de correos :)

Felices fiestas!