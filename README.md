# Saliendo del fango

Traducción al castellano del artículo "Out of the Tar Pit"

**Traducción en progreso**

Autores:

 - Ben Moseley ben@moseley.name
 - Peter Marks public@indigomail.net

Traducción:
 - Sebastián Ortega
 - Álvaro Castellanos

## Resumen

El mayor obstáculo en el desarrollo de sistemas software de envergadura es la
complejidad. En consonancia con Brooks, distinguimos entre complejidad
accidental y esencial pero no aceptamos su premisa de que la mayoría de la
complejidad que se puede encontrar en los sistemas actuales es esencial.
Identificamos las causas más comunes de la complejidad y discutimos
aproximaciones generales para minimizar la complejidad de naturaleza
accidental. Para hacer más concreta la exposición, se bosqueja una potencial
aproximación a la minimización de complejidad basada en la *programación
funcional* y en el *modelo de datos relacional de Codd*.

## 1. Introducción

La "crisis del software" fue identificada por primera vez en 1968 [NR69, p70] y
en las décadas que han transcurrido desde entonces se ha profundizado en lugar
de aliviarse. El mayor problema en el desarrollo y mantenimiento de grandes
sistemas software es la complejidad: estos sistemas son difíciles de comprender.
Creemos que en muchos casos el mayor contribuyente para esta complejidad es la
gestión del *estado* y el lastre que añade cuando se intenta analizar y
razonar acerca del sistema. Otros contribuyentes estrechamente relacionados son
el *volumen de código* y, de forma especialmente preocupante, el *flujo de
control* a través del sistema.

Las estrategias clásicas para gestionar el estado incluyen a la programación
orientada a objetos, que acopla estrechamente estado y comportamiento, y la
programación funcional, que en su forma más pura evita completamente el estado y
los efectos laterales. Estas estrategias padecen de varios (y diferentes)
problemas cuando se aplican al diseño de grandes sistemas de forma tradicional.

Defendemos que es posible tomar ideas útiles de ambas estrategias y que,
en combinación con algunas ideas del mundo de las bases de datos relacionales,
se puede tomar una aproximación con un significativo potencial para simplificar
la construcción de grandes sistemas software.

Este artículo está dividido en dos mitades. En la primera mitad nos centramos en
la complejidad. En la sección 2, examinamos la complejidad en general y
justificamos nuestra creencia de que ésta es la raíz de la crisis para, a
continuación examinar cómo se intenta comprender un sistema en la actualidad en
la sección 3. En la sección 4, examinamos las causas de la complejidad (i.e.
aquello que dificulta la compresión) antes de discutir los enfoques clásicos
para gestionar estas causas en la sección 5. En la sección 6, definimos qué
entendemos por "accidental" y "esencial" y después en la sección 7 damos
recomendaciones para abordar de forma alternativa las causas de la complejidad
(con énfasis en prevenir los problemas en lugar de afrontarlos).

En la segunda mitad del artículo detallamos un enfoque que se deriva de nuestra
estrategia recomendada. Comenzamos con una revisión del modelo relacional en la
sección 8 y una descripción de una potencial estrategia en la sección 9. En la
sección 10 proveemos un breve ejemplo de cómo éste podría ser usado.

Finalmente contrastamos nuestra aproximación con otras posibles en la sección 11
y concluimos en la sección 12.

## 2. Complejidad

En su clásico artículo, "No Silver Bullet", Brooks [Bro86] identificó cuatro
propiedades de los sistemas software que hacen el desarrollo de software
difícil: Complejidad, Conformidad, Facilidad de cambio e Invisibilidad. De entre
ellas, creemos que la Complejidad es la *única* significativa: las otras pueden
bien *ser clasificadas* como formas de complejidad o bien considerarse
problemáticas únicamente *a causa* de la complejidad en el sistema.

La complejidad es *la* causa última de la inmensa mayoría de los problemas
relacionados con el software hoy en día. Inestabilidad, retrasos, falta de
seguridad, a menudo hasta los problemas de rendimiento, pueden ser vistos como
el producto de una complejidad inmanejable en estos grandes sistemas. Esta
clasificación de la complejidad como *la* causa principal es consecuencia del
hecho de que ser capaces de *comprender* un sistema es un pre-requisito para
evitar todos estos problemas y es la complejidad incapacita esta comprensión.

La relevancia de la complejidad es ampliamente reconocida. Como dijo Dijkstra [Dij97, EWD1243]:

> "...tenemos que mantenerlo ordenado, desenmarañado y simple si queremos
> evitar ser aplastado por las complejidades de nuestra propia obra..."
    
...y The Economist dedicó un artículo completo a la complejidad del software
[Eco04] señalando que algunas estimaciones valoran en 59 miles de millones de
dólares anuales de coste para la economía americana de los problemas
relacionados con el software.

La capacidad de pensar y razonar sobre nuestros sistemas (en particular sobre
los efectos de los cambios que se les realicen) es de una *crucial* importancia.
Los peligros de la complejidad y la importancia de la simplicidad han sido un tema popular en las exposiciones del premio Turing de la ACM. En su exposición de 1990, Corbato dijo [Cor91]:

> "El problema general con los sistemas ambiciosos es la complejidad.",
> "...es importante enfatizar el valor de la simplicidad y la elegancia, dado
> que la complejidad es un camino de dificultades creciente"
    
En 1997, Backus [Bac78] habló sobre las "complejidades y debilidades" de los lenguajes tradiciones y apuntó que:

> "hay una necesidad desesperada de una metodología poderosa que nos ayude a
> razonar acerca de los programas. ... los lenguajes convencionales crean una
> confusión innecesaria en nuestra forma de razonar sobre los programas"

Finalmente en su discurso de aceptación del premio Turing en 1980, Hoare [Hoa81]
hizo esta observación:

> "...hay una cualidad que no puede ser comprada... y es la fiabilidad. El
> precio de la fiabilidad es la búsqueda de la mayor simplicidad"

y

> "Llego a la conclusión de que hay dos formas de construir un diseño
> software: o bien se hace algo tan simple que *obviamente* no tiene defectos
> o bien se hace tan complicado que no hay deficiencias *obvias*. El primer
> método es mucho más difícil."
    
Esta es la desafortunada realidad:

    La simplicidad es *difícil*.
    
...pero el propósito de este artículo es dar motivos para el optimismo.

Nuestro argumento final es que el tipo de complejidad del que estamos
discutiendo en este artículo es la que hace a los grandes sistemas *difíciles de
comprender*. Este es el motivo que nos hace consumir recursos ingentes para
*crear y mantener* dichos sistemas. Este tipo de complejidad no tiene nada que
ver con teoría de la complejidad (la rama de la ciencia de la computación que
estudia los recursos consumidos *por una máquina al ejecutar* un programa).
No tienen relación alguna: es sencillo escribir un pequeño programa de pocas
líneas que sea increíblemente simple (en nuestro sentido) y que pertenezca a la
mayor clase de complejidad (en el sentido de la teoría de la complejidad). De
aquí en adelante sólo nos referiremos a complejidad del primer tipo.

Debemos examinar las que consideramos como principales causes comunes de la
complejidad (aspectos que dificultan la comprensión) después de examinar de qué
formas tratamos de *comprender* los sistemas habitualmente.

## 3. Formas de comprender un sistema

Defendíamos antes que el daño de la complejidad viene dado desde su impacto en
nuestros intentos por *entender* el sistema. Debido a esto, es de útil
considerar los mecanismos que se utilizan comúnmente para tratar entender los
sistemas. Podremos entonces considerar más adelante el impacto que las posibles
causas de la complejidad tienen en estos enfoques. Hay dos enfoques ampliamente
utilizados para la compresión de los sistemas (o componentes de los sistemas):

- **Pruebas**. Estas tratan de entender un sistema desde fuera, como una "caja
negra". Las conclusiones sobre el sistema se dibujan sobre la base de
observaciones de cómo se comportan en ciertas situaciones específicas. Las
pruebas pueden ser desempeñadas por humanos o por máquinas. El primero es más
común para las pruebas de todo el sistema, el último es más común para las
pruebas de componentes individuales.
- **Razonamiento informal**. Estos tratan de comprender el sistema mediante un
examen interno. La expectativa es que usando la información adicional
disponible, se pueda adquirir una compresión más exacta.

De los dos, el razonamiento informal es el más importante con diferencia. Esto
se debe a que, como veremos más adelante, hay límites inherentes a lo que puede
lograrse por medio de pruebas, y por ello el razonamiento informal (en virtud de
ser una parte inherente del proceso de desarrollo) es *siempre* usado. La otra
justificación es que las mejoras en los razonamientos informales darán lugar a
*menos errores cuando son creados* mientras que todas las mejoras en las pruebas
lo que llevan a cabo es a *más errores cuando son detectados*. Como Dijkstra
dijo en su discurso de su premio Turing [Dij72, EWD340]:

> "Los que realmente quieren software fiable descubrirán que tienen que
> encontrar la manera de evitar la mayoría de los errores para empezar."

y como O'Keefe (quien también hizo hincapié en la importancia de "*la compresión
de su problema*" y que "*La elegancia no es opcional*") dijo [O'K90]:

> "Nuestra respuesta a los errores debería ser buscar maneras en que podamos
> hacerlos evitar, no culpar a la naturaleza de las cosas."

El principal problema con las pruebas es que una prueba (de cualquier tipo) que
utiliza un particular conjunto de entradas no dice *nada en absoluto* sobre
el comportamiento del sistema o el componente cuando se le da un conjunto
diferente de entradas. El gran número de las diferentes posibles entradas
generalmente descarta la posibilidad de poner a todas ellas a prueba, de ahí la
preocupación ineludible con las pruebas será siempre: *¿has realizado las
pruebas correctas?*. La única respuesta correcta que se obtendrá a esta
pregunta es una respuesta negativa, cuando el sistema se rompe. De nuevo, como
Dijkstra observó [Dij71, EWD303]:

> "las pruebas son desesperadamente inadecuadas... pueden ser usadas muy
> eficientemente para mostrar la presencia de errores pero nunca para mostrar
> su ausencia."

Estamos de acuerdo con Dijkstra. *Confiar* en pruebas es peligroso.
Esto *no* dice que no hagamos pruebas. La conclusión es que *todas* las formas
de intentar entender un sistema tienen sus limitaciones (y esto incluye ambos:
*razomamiento informal*, el cual está limitado en su alcance, imprecisión y que
por lo tanto está sujeto a errores, como el *razonamiento formal*, el cual es
dependiente de la precisión de una especificación). Debido a estas limitaciones
con frecuencia puede ser prudente emplear ambos, pruebas y razonamiento, juntos.

Es precisamente *debido* a estas limitaciones de todas estos enfoques que la
simplicidad es vital. Cuando se considera junto a las pruebas y el razonamiento,
la simplicidad es más importante que cualquiera de estos dos. Dada una dura
elección entre la inversión en las pruebas y las inversión en la simplicidad,
este último puede con frecuencia ser la mejor opción, ya que facilitará todos
los futuros intentos de entender el sistema, los intentos de cualquier tipo.

## 4. Causas de la complejidad

En cualquier sistema no trivial hay una cierta cantidad de complejidad que es
inherente al problema a resolver. Sin embargo, en los grandes sistemas nos
encontramos con complejidad sospechosa de no ser "inherente al problema". Ahora
consideraremos algunas de las causas de esta complejidad.

### 4.1 Complejidad causada por el estado

Cualquiera que haya llamado al servicio de soporte de algún sistema software y
haya sido invitado a "probar de nuevo", "recargar el documento", "reiniciar el
programa", "reiniciar el ordenador", "reinstalar el programa" o incluso
"reinstalar el sistema operativo y después el programa" tiene experiencia de
primera mano sobre los problemas que el *estado*[^1] produce cuando se
intenta escribir software robusto y comprensible.

[^1]: Por "estado" nos referimos específicamente a *estado mutable*. Es decir,
    excluyendo cosas como por ejemplo las variables inmutables que sólo se
    pueden asignar una vez y que ofrecen los lenguajes de programación lógicos.
    
El motivo por el que estas situaciones son familiares para tanta gente es que
son habitualmente propuestas porque habitualmente resuelven el problema. Esto
sucede porque muchos sistemas gestionan su estado de forma errónea. A su vez,
estos errores son introducidos porque la existencia del propio estado hace los
programas difíciles de *comprender*. Los hace complejos.

En relación al estado, nos mostramos de acuerdo con la opinión de Brooks cuando
afirma [Bro86]:

> "De la complejidad nace la dificultad para enumerar, ni siquiera comprender,
> todos los posibles estados del programa y de ahí nace falta de fiabilidad"

Estamos de acuerdo con esto pero creemos que es la misma existencia de tantos
posibles estados lo que en primer lugar da lugar a la complejidad, y:

> "computadoras... tienen un número muy grande de estados. Esto hace que
> concebirlos, describirlos y probarlos sea difícil. Los sistemas software
> tienen órdenes de magnitud más estados que las propias computadoras."

#### 4.1.1 Impacto del estado en las pruebas

La gravedad del impacto del *estado* en las pruebas que advirtió Brooks es
difícil de exagerar. El estado afecta a todos los tipos de pruebas, desde las
pruebas a nivel de sistema (donde el desarrollador padece los mismos problemas
que el desafortunado usuario que se mencionó) pasando por las de nivel de
componente y hasta las de unidad. El principal problema es que una prueba (de
cualquier tipo) en un sistema o componente que está en un *estado* concreto
aporta *cero información* acerca del comportamiento del sistema o componente en
cualquier otro *estado*.

La estrategia más común para probar un sistema con estado (a nivel de componente
o de sistema) pasa por establecer un estado "limpio" o "inicial" (no obstante
mayormente *oculto*), realizar las pruebas utilizando unas entradas y asumir (de
forma infundada en caso de error) que el sistema se va a comportar siempre de la
misma forma, *independientemente de su estado interno oculto*, cada vez que se
prueben esas entradas.

Simple y llanamente, esta estrategia barre bajo la alfombra el problema del
estado. Los motivos por los que se hace esto son, en primer lugar, que es
posible hacer progresos con esta estrategia y, en segundo lugar, porque no hay
otra alternativa cuando se trata de probar un sistema con un estado interno
oculto y complicado.

Por supuesto, no *siempre* es posible "hacer progresos" con esta estrategia. Si
alguna secuencia de eventos (entradas) puede hacer que el sistema "alcance un
estado *malo*" (específicamente un estado oculto *diferente* del que se utilizó
durante las pruebas) la situación puede salir mal y saldrá mal. Esto es lo que
le sucede al usuario del hipotético servicio de soporte citado al principio de
esta sección. Las soluciones propuestas son todas intentos de forzar que el
sistema vuelva a un "estado interno bueno".

Este problema (que las pruebas en un *estado* no te dicen *nada en absoluto*
acerca del sistema en un *estado* diferente) es completamente análogo a uno de
los problemas discutidos antes: probar con un *conjunto de entradas* no te dice
*nada en absoluto* sobre el comportamiento con otro *conjunto de entradas*. Sin
embargo, el problema que causa el estado suele ser peor, especialmente cuando se
prueban porciones grandes de un sistema, porque aunque el espacio de *entradas*
pueda ser muy grande, el espacio de *estados* tiende a ser incluso mayor.

Estos dos problemas similares, uno intrínseco al hecho de probar y el otro fruto
del *estado*, se combinan *horrendamente*. Cada uno de ellos introduce una
ingente cantidad de incertidumbre y por lo tanto son muy pocas las cosas que
*podemos* dar por seguras si el sistema/componente bajo escrutinio tiene estado.

#### 4.1.2 Impacto del estado en el razonamiento informal

### 4.2 Complejidad causada por el control

### 4.3 Complejidad causada por el volumen de código

### 4.4 Otras causas de la complejidad

## 5. Aproximaciones clásicas a la gestión de la complejidad

## 6. Accidentes y esencia

## 7. Recomendaciones generales

Dado que nuestras recomendaciones principales giran alrededor de intentar
*evitar* tanta complejidad accidental como sea posible, necesitamos preguntarnos
*qué* complejidad deben ser considerada accidental y cuál esencial.

Debemos responder esta pregunta considerando qué complejidad es *imposible*
evitar incluso en un mundo ideal (esto es básicamente nuestra *definición* de
esencial). A continuación veremos cómo de realista es este mundo ideal y
finalmente daremos algunas recomendaciones.

### 7.1 Mundo ideal
#### 7.1.1 Estado en el mundo ideal
#### 7.1.2 Control en el mundo ideal
#### 7.1.3 Resumen

### 7.2 Limitaciones teóricas y prácticas
#### 7.2.1 Lenguajes de especificación formal
#### 7.2.2 Facilidad de expresión
#### 7.2.3 Complejidad accidental requerida

### 7.3 Recomendaciones
#### 7.3.1 Complejidad accidental requerida
#### 7.3.2 Separación y relaciones entre los componentes

### 7.4 Resumen

## 8. Modelo relacional

A pesar de sus orígenes, el modelo relacional [Cod70] *en si mismo* no tiene
nada que ver con bases de datos. No es más que una forma elegante de
*estructurar* datos, una forma de *manipular* los mismos y  un mecanismo para
mantener estado *íntegro* y consistente. Estas características son aplicables
a estado y datos en cualquier contexto.

A parte de estas tres áreas [Cod79, section 2.1], [Dat04, p109], un cuarto
punto fuerte del modelo relacional es el énfasis en tener una marcada
separación entre la capa lógica y física del sistema. Esto se traduce en que
el diseño del modelo lógico (minimizando la complejidad) es atacado de forma
separada del diseño del modelo de almacenamiento físico y de la correspondencia
entre ambos[^15]. Este principio de denomina *independencia de datos* y es una
parte crucial del modelo relacional [Cod70, section 1.1].

[^15]: Desafortunadamente la mayoría de sistemas gestores de bases de datos
están bastante limitados en el grado de flexibilidad permitida por la
correspondencia física/lógica. Desafortunadamente, los asuntos relativos al
rendimiento físico pueden invadir el diseño lógico incluso cuando evitar esto
era uno de los principales objetivos originales de Codd.

Vemos el modelo relacional compuesto de los siguientes cuatro aspectos:

 - **Estructura.** Se utilizan *relaciones* par representar todos los datos.
 - **Manipulación.** Un medio para especificar datos derivados.
 - **Integridad.** Un medio para especificar restricciones de datos de
   obligado cumplimiento.
 - **Independencia de datos.** Hay una separación clara entre los datos
   lógicos y su representación física.

Trataremos brevemente cada uno de estos aspectos. [Dat04] ofrece una visión de
conjunto sobre el modelo relacional más exhaustiva.

Como comentario final, recordar al lector que SQL, en cualquier versión, no
sigue fielmente el modelo relacional [Cod90, p371, Serious flaws in SQL],
[Dat04, p xxiv] por lo que aconsejamos no equiparar ambos.

### 8.1 Estructura

#### 8.1.1 Relaciones

Como se ha mencionado previamente, las relaciones son la única forma de
estructurar datos en el modelo relacional. Una relación se puede ver como un
*conjunto de registros* homogéneo en el que cada registro es conjunto
heterogéneo de *atributos* con nombre (ligeramente distinto del concepto
matemático de relación como conjunto de tuplas cuyos componentes se identifican
por su posición en lugar de por nombre).

Esta definición implica que una relación, por el hecho de ser un conjunto, no
puede contener duplicados y no tiene orden. Ambas restricciones contrastan con
el uso cotidiano de la palabra *tabla* que obviamente puede contener filas
duplicadas (y nombres de columna) y, por el hecho de ser una entidad visual en
una página, inevitablemente sus filas y columnas aparecen en un orden
determinado.

Las relaciones puede ser de estos tipos:

 - **Relaciones base**. Aquellas directamente almacenadas.
 - **Relaciones derivadas** (conocidas como *vistas*). Definidas en términos de
   otras relaciones (a su vez base o derivadas). Ver sección 8.2.

Como propone Date [Dat04], es útil pensar en cada relación como un único *valor*
(aunque compuesto) y considerar el estado mutable no como una "relación mutable"
sino como una *variable* que en cada momento está asociada a una relación
*valor* concreta que cambia con el tiempo. Date llama a estas variables
*variables relación* o *relvars*, lo que lleva a los términos *relvar base* y
*relvar derivada*. Utilizaremos su terminología de aquí en adelante. (Sin
embargo, nuestra definición de relación es ligeramente distinta ya que,
siguiendo las prácticas estándar en tipado estático, no consideramos el tipo
como parte del valor.)

#### 8.1.2 Beneficios estructurales de las relaciones - Independencia de la forma de acceso

Estructurar datos mediante relaciones es una idea atractiva porque no es
necesario tomar decisiones *subjetivas* de antemano acerca de la *forma de
acceso* que será más adelante utilizada para consultar y procesar los datos.

Para comprender que entendemos por *forma o camino de acceso*, consideremos un
ejemplo simple. Supongamos que queremos representar información acerca de
empleados y los departamentos en los que éstos trabajan. Un sistema en el que
elegir la estructura de los datos implique decidir las "rutas" posibles entre
las instancias de los datos (por ejemplo *desde* un empleado dado *hasta* un
departamento) es *dependiente del camino de acceso*.

Las dos principales formas de estructurar datos que precedieron al modelo
relacional (los modelos jerárquico y en red) eran ambos dependientes del camino
en este sentido. Por ejemplo, en el modelo jerárquico se debe tomar de forma
temprana una decisión sobre si el departamento es el nivel raíz (cada
departamento "contiene" sus empleados) o viceversa (los empleados "contienen"
sus departamentos). Esta decisión impactará todos los usos futuros de los datos.
En el primer caso, será fácil enumerar todos los empleados de un determinado
departamento (siguiendo el camino de acceso) pero será más difícil enumerar el
departamento de un empleado dado (habrá que usar alguna otra técnica que
implique buscar en todos los departamentos). En el segundo caso tendremos el
problema inverso.

El modelo en red alivió el problema en cierto modo al permitir múltiples caminos
de acceso entre las instancias de datos (con lo que se podría decidir tener
ambos caminos desde departamento a empleado y viceversa). El problema consiste
en que es imposible predecir de antemano cuales serán los caminos de acceso y
por este motivo siempre habrá una discrepancia entre:

 - **Requisitos de recuperación primarios.** Éstos pueden ser previstos y
   satisfechos simplemente siguiendo los caminos de acceso definidos.
 - **Requisitos de recuperación secundarios.** Éstos puede ser no previstos o
   simplemente no tener un soporte específico y por lo tanto sólo pueden ser
   provistos mediante mecanismos alternativos como búsquedas.
   
Una de las razones principales por las que el modelo relacional prevaleció sobre
los modelos jerárquico y en red es la capacidad de *evitar* completamente el
concepto de camino de acceso.

También es interesante considerar brevemente lo que implica aplicar una
aproximación orientada a objetos (OOP) a nuestro ejemplo. Podemos elegir entre
estas opciones:

 - Dar a los objetos `Employee` una referencia a su `Department`.
 - Dar a los objetos `Department` un conjunto (o array) de referencias a sus
   objetos `Employee`.
 - Todas las anteriores.
 
Si elegimos la tercera opción nos exponemos en el mejor caso al trabajo
adicional de mantener referencias redundantes y en el peor a introducir errores.

Hay inquietantes similitudes entre las aproximaciones de la OOP y el XML a cómo
estructurar datos y los modelos jerárquico y en red.

Finalmente, una ventaja de las relaciones como forma de estructurar datos en contraposición a aproximaciones como el modelo entidad-relación de Chen [Che76] es que no se hace distinción alguna entre *entidad* y *relación*. (Utilizar dicha distinción puede ser problemático porque puede ser muy subjetivo clasificar algo como *entidad* o como *relación*.)

### 8.2 Manipulación

Codd introdujo dos mecanismos diferentes para expresar la manipulación de datos
en el modelo relacional: el cálculo relacional y el álgebra relacional. Ambos
son formalmente equivalentes en el sentido de que expresiones de cada uno pueden
convertirse a otras equivalentes del otro y por lo tanto nos bastará con
considerar solo el álgebra.

El álgebra relacional (en una forma ligeramente diferente de cómo lo presentó
Codd originalmente) consiste en las siguientes ocho operaciones:

 - **Restricción (restrict).** Operación unaria que permite la selección de un
   subconjunto
   de los registros de acuerdo a un criterio dado.
 - **Proyección (project).** Operación unaria que produce una nueva relación en
   la que
   algunos atributos han sido eliminados.
 - **Producto (product).** Operación binaria que se corresponde con el producto
   cartesiano tal y como se define en matemáticas.
 - **Unión (union).** Operación binaria que produce una relación que contiene
   todos los registros que figuran en uno u otro operando.
 - **Intersección (intersection).** Operación binaria que produce una relación
   con todos los registros que figuran en ambos operandos simultáneamente.
 - **Diferencia (difference).** Operación binaria que produce una relación con
   los registros del primer operando que no figuran en el segundo.
 - **Combinar (join).** Operación binaria que produce todos los posibles
   registros que resultan juntar registros de ambos operandos cuyo valor de
   ciertos atributos coincide.
 - **Dividir (divide).** Es una operación ternaria que devuelve todos los
   registros del primer argumento que aparecen en el segundo asociados con *cada
   uno* de los registros del tercero.
   
Uno de los beneficios más significativos de este lenguaje de manipulación
(aparte de su simplicidad) es que tiene la propiedad de *clausura*: todos los operandos y resultados son de la misma clase (relaciones). Por lo tanto, las operaciones se pueden anidar de forma arbitraria (lo que de hecho es inherente a cualquier álgebra monotipo).

### 8.3 Integridad

La integridad en el modelo relacional se mantiene simplemente con especificar,
de forma puramente declarativa, una serie de restricciones que deben cumplirse
en todo momento.

Cualquier infraestructura que implemente el modelo relacional debe garantizar
que estas restricciones se cumplen, específicamente en los intentos de modificar
el estado que puedan contravenir las restricciones tienen que ser rechazadas o
restringidas a operar dentro de los límites.

Las restricciones más comunes son aquellas identificando claves *candidatas* o
*primarias* y claves *extrajeras*. Las restricciones pueden llegar a ser
arbitrariamente complejas, involucrar múltiples relaciones y ser construidas
utilizando tanto el álgebra como el cálculo relacional.

Finalmente, muchos sistemas gestores de bases de datos comerciales proveen
mecanismos *imperativos* como los disparadores (*triggers*) para mantener la
integridad de los datos. Estos mecanismos sufren de los problemas sobre flujo de
control comentados en la sección 4.2 y *no se consideran* parte del modelo
relacional.

### 8.4 Independencia de datos

La *independencia de datos* es el principio por el que se separa el modelo
lógico de los datos de la representación física almacenada, y fue una de las
motivaciones que originaron el modelo relacional.

Es interesante darse cuenta de que el principio de *independencia de datos* es
de hecho análogo a la separación entre lo accidental y lo esencial recomendada
en la sección 7.3.2. Esto mismo es una de las razones que motivan la adopción de
la Programación Relacional Funcional (ver sección 9).

### 8.5 Extensiones

A pesar de ser flexible, el álgebra relacional es un lenguaje restrictivo en
términos computacionales (no es Turing-completo) y de forma habitual es
extendido de diversas formas en sus implementaciones prácticas. Algunas
extensiones comunes son:

 - **Capacidades de computación generales.** Como por ejemplo aritmética simple
   y capacidad de utilizar funciones definidas por el usuario.
 - **Operadores de agregación.** Como `MAX`, `MIN`, `COUNT`, `SUM`, etc.
 - **Capacidades de agrupado y resumen.** Para poder aplicar fácilmente las
   operaciones de agregación.

## 9. Programación relacional funcional

## 10. Ejemplo de un sistema FRP

## 11. Trabajo relacionado

## 12. Conclusiones

Hemos presentado como la *complejidad* es la mayor fuente de problemas para los
grandes sistemas software. También hemos discutido cómo *puede* ser domesticada
pero sólo a través de un esfuerzo consciente de *evitarla* en la cuando es
posible y *separándola* cuando no lo es. Específicamente, hemos determinado que
un sistema puede ser fructíferamente *separado* en tres partes principales: el
*estado esencial*, la *lógica esencial* y el *estado y control accidentales*.

Creemos que la aplicación de estos principios al diseño del alto nivel de un
sistema (utilizando diferentes *lenguajes* especializados para los diferentes
componentes) puede ofrecer más en términos de simplicidad que la adopción no
estructura de un único lenguaje de propósito general cualquiera (bien
imperativo, lógico o funcional). La argumentación de este punto incluye un
repaso de cada uno de los paradigmas de programación más comunes con especial
atención a los puntos débiles de la orientación a objetos como un ejemplo
particular de la aproximación imperativa.

En aquellos casos (como en los grandes sistemas preexistentes) donde esta
*separación* no puede ser aplicada directamente creemos que se debe hacer
énfasis en evitar el estado, el control explícito en la medida de lo posible y
luchar por todos los medios de *librarse del código*.

Finalmente, ¿cómo salir del fango? ¿Cuál es la bala de plata? ...podría no ser
el FRP pero creemos indudablemente que es la *simplicidad*.

## Referencias

- [Bac78] John W. Backus. Can programming be liberated from the von Neumann
  style? a functional style and its algebra of programs. *Commun. ACM*,
  21(8):613–641, 1978.

- [Bak93] Henry G. Baker. Equal rights for functional objects or, the more
  things change, the more they are the same. *Journal of Object-Oriented
  Programming*, 4(4):2–27, October 1993.

- [Boo91] G. Booch. *Object Oriented Design with Applications*.
  Benjamin/Cummings, 1991.

- [Bro86] Frederick P. Brooks, Jr. No silver bullet: Essence and accidents of
  software engineering. *Information Processing 1986, Proceedings of the Tenth
  World Computing Conference*, H.-J. Kugler, ed.: 1069–76. Reprinted in IEEE
  Computer, 20(4):10-19, April 1987, and in Brooks, The Mythical Man-Month:
  Essays on Software Engineering, Anniversary Edition, Chapter 16,
  Addison-Wesley, 1995.

- [Che76] P. P. Chen. The Entity-Relationship Model. *ACM Trans. on Database
  Systems (TODS)*, 1:9–36, 1976.

- [Cod70] E. F. Codd. A relational model of data for large shared data banks.
  *Comm. ACM*, 13(6):377–387, June 1970.

- [Cod79] E. F. Codd. Extending the database relational model to capture more
  meaning. *ACM Trans. on Database Sys.*, 4(4):397, December 1979.

- [Cod90] E. F. Codd. *The Relational Model for Database Management, Version
  2*.  Addison-Wesley, 1990.

- [Cor91] Fernando J. Corbató. On building systems that will fail. *Commun.
  ACM*, 34(9):72–81, 1991.

- [Dat04] C. J. Date. *An Introduction to Database Systems*. Addison Wesley,
  8th edition, 2004.

- [DD00] Hugh Darwen and C. J. Date. *Foundation for Future Database Systems:
  The Third Manifesto*. Addison-Wesley, 2nd edition, 2000.

- [Dij71] Edsger W. Dijkstra. On the reliability of programs. circulated
  privately, 1971.

- [Dij72] Edsger W. Dijkstra. The humble programmer. *Commun. ACM*,
  15(10):859–866, 1972.

- [Dij97] Dijkstra. The tide, not the waves. In Peter J. Denning and Robert M.
  Metcalfe, editors, *Beyond Calculation: The Next Fifty Years of Computing,
  Copernicus, 1997*. 1997.

- [Eco04] Managing complexity. *The Economist*, 373(8403):89–91, 2004.

- [EH97] Conal Elliott and Paul Hudak. Functional reactive animation. In
  *Proceedings of the ACM SIGPLAN International Conference on Functional
  Programming (ICFP-97)*, volume 32,8 of ACM SIGPLAN Notices, pages 263–273,
  New York, June 9–11 1997. ACM Press.

- [HJ89] I. Hayes and C. Jones. Specifications are not (necessarily) executable.
  *IEE Software Engineering Journal*, 4(6):330–338, November 1989.

- [Hoa81] C. A. R. Hoare. The emperor’s old clothes. *Commun. ACM*, 24(2):75–83,
  1981.

- [Kow79] Robert A. Kowalski. Algorithm = logic + control. *Commun. ACM*,
  22(7):424–436, 1979.

- [Mer85] T. H. Merrett. Persistence and Aldat. In *Data Types and Persistence
  (Appin)*, pages 173–188, 1985.

- [NR69] P.Naur and B.Randell. Software engineering report of a conference
  sponsored by the NATO science committee Garmisch Germany 7th-11th October
  1968, January 01 1969.

- [OB88] A. Ohori and P. Buneman. Type inference in a database programming
  language. In *Proceedings of the 1988 ACM Conference on LISP and Functional
  Programming*, Snowbird, UT, pages 174–183, New York, NY, 1988. ACM.

- [O’K90] Richard A. O’Keefe. *The Craft of Prolog*. The MIT Press, Cambridge,
  1990.

- [PJ+03] Simon Peyton Jones et al., editors. *Haskell 98 Language and
  Libraries, the Revised Report*. CUP, April 2003.

- [SS94] Leon Sterling and Ehud Y. Shapiro. *The Art of Prolog - Advanced
  Programming Techniques*, 2nd Ed. MIT Press, 1994.

- [SU96] Randall B. Smith and David Ungar. A simple and unifying approach to
  subjective objects. TAPOS, 2(3):161–178, 1996.

- [vRH04] Peter van Roy and Seif Haridi. *Concepts, Techniques, and Models of
  Computer Programming*. MIT Press, 2004.

- [Wad95] Philip Wadler. Monads for functional programming. *In Advanced
  Functional Programming*, pages 24–52, 1995.

- [Won00] Limsoon Wong. Kleisli, a functional query system. *J. Funct.
  Program*, 10(1):19–56, 2000.
