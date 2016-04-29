# El motor Unity

**Unity** es un motor genérico para la creación de videojuegos 2D y 3D enfocado hacia el desarrollo casual. La curva de aprendizaje del motor es bastante suave, especialmente si lo comparamos con motores más complejos como Unreal Engine 4, y nos permitirá realizar un desarrollo rápido de videojuegos. Esta característica hace este motor muy apropiado también para crear prototipos rápidos de nuestros juegos.

A partir de la versión Unity 5, existen dos ediciones: _Personal_ y _Profesional_. La primera es gratuita e incluye todas las funcionalidades del motor. La segunda incluye funcionalidades adicionales de soporte (construcción en la nube, herramientas de trabajo en equipo, etc), y es de pago (suscripción de $75 o pago único de $1.500). La versión _Personal_ podrá ser utilizada por cualquier individuo o empresa cuyas ganancias anuales no superen los $100.000.

Uno de los puntos fuertes de Unity es la posibilidad de exportar a gran cantidad de plataformas. Soporta las **plataformas móviles iOS, Android, Windows Phone y Blackberry**, y además también permite exportar a web (WebGL), a videoconsolas (PS4, PS3, PS Vita, Xbox One, Xbox 360, Wii U, etc) y a ordenadores (Mac, Windows y Linux). 


## Introducción a Unity

### El editor de Unity

Unity incorpora su propia herramienta integrada para la creación de videojuegos, que nos permite incluso crear algunos videojuegos de forma visual sin la necesidad de programar. 

Dentro del entorno del editor de Unity encontramos diferentes paneles, de los cuales destacamos los siguientes:

* **Project**: Encontramos aquí todos los recursos (_assets_) que componen nuestro proyecto. Estos recursos pueden ser por ejemplo texturas, _clips_ de audio, _scripts_, o escenas. Destacamos aquí el _asset_ de tipo **escena**, que es el componente que nos permite definir cada estado (pantalla) del juego. Al hacer doble _click_ sobre una escena se abrirá para trabajar con ella desde el editor.
* **Hierarchy**: La escena está formada por una serie de nodos (_game objects_) organizados de forma jerárquica. En este panel vemos el árbol de objetos que contiene la escena abierta actualmente. Podemos seleccionar en ella cualquier objeto pulsando sobre su nombre.
* **Scene**: En este panel vemos de forma visual los elementos de la escena actual. Podremos movernos libremente por el espacio 3D de la escena para ubicar de forma correcta cada _game object_ y tener una previsualización del escenario del juego.
* **Inspector**: Muestra las propiedades del _game object_ o el _asset_ seleccionado actualmente en el entorno. 
 
 
IMAGEN EDITOR
 
### Arquitectura Orientada a Componentes

Como hemos comentado, **todos** los elementos de la escena son objetos de tipo `GameObject` organizados de forma jerárquica. Todos los objetos son del mismo tipo, independientemente de la función que desempeñen en el juego. Lo que diferencia a unos de otros son los _componentes_ que incorporen. Cada objeto podrá contener varios componentes, y estos componentes determinarán las funciones del objeto.

Por ejemplo, un objeto que incorpore un componente `Camera` será capaz de renderizar en pantalla lo que se vea en la escena desde su punto de vista. Si además incorpora un componente `Light`, emitirá luz que se proyectará sobre otros elementos de la escena, y si tiene un componente `Renderer`, tendrá un contenido gráfico que se renderizará dentro de la escena. 

Esto es lo que se conoce como **Arquitectura Basada en Componentes**, que nos proporciona la ventaja de que las funcionalidades de los componentes se podrán reutilizar en diferentes tipos de entidades del juego. Es especialmente útil cuando tener un gran número de diferentes entidades en el juego, pero que comparten módulos de funcionalidad.

En Unity esta arquitectura se implementa mediante agregación. Si bien en todos los objetos de la escena son objetos que heredan de `GameObject`, éstos podrán contener un conjunto de componentes de distintos tipos (`Light`, `Camera`, `Renderer`, etc) que determinarán el comportamiento del objeto.

En el inspector podremos ver la lista de componentes que incorpora el objeto seleccionado actualmente, y modificar sus propiedades:

IMAGEN COMPONENTES

## La escena 3D

El el editor de Unity veremos la escena con la que estemos trabajando actualmente, tanto de forma visual (_Scene_) como de forma jerárquica (_Hierarchy_). Nos podremos mover por ella y podremos añadir diferentes tipos de objetos. 

### Posicionamiento de los objetos en la escena

Todos los _game objects_ incorporan al menos un componente `Transform` que nos permite situarlo en la escena, indicando su traslación, orientación y escala. Podremos introducir esta información en el editor, para así poder ajustar la posición del objeto de forma precisa. 

También podemos mover un objeto de forma visual desde la vista _Scene_. Al seleccionar un objeto, bien en _Scene_ o en _Hierarchy_, veremos sobre él en _Scene_ una serie de ejes que nos indicarán que podemos moverlo. El tipo de ejes que se mostrarán dependerá del tipo de transformación que tengamos activa en la barra superior:

IMAGEN MODOS

Las posibles transformaciones son:

* **Traslación**: Los ejes aparecerán como flechas y nos permitirán cambiar la posición del objeto.
* **Rotación**: Veremos tres círculos alrededor del objeto que nos pemtirán rotarlo alrededor de sus ejes _x_, _y_, _z_. 
* **Escalado**: Veremos los ejes acabando en cajas, indicando que podemos escalar el objeto en _x_, _y_, _z_.

Si pinchamos sobre uno de los ejes y arrastramos, trasladaremos, rotaremos, o escalaremos el objeto sólo en dicha eje. Si pinchamos sobre el objeto, pero no sobre ninguno de los ejes, podremos trasladarlo, rotarlo y escalarlo en todos los ejes al mismo tiempo.

### Añadir _game objects_ a la escena

Podemos añadir a la escena nuevos _game objects_ seleccionando en el menú la opción _GameObject > Create Empty_, lo cual creará un nuevo objeto vacío con un único componente `Transform`, al que le deberíamos añadir los componentes que necesitásemos, o bien podemos crear objetos ya predefinidos mediante _GameObject > Create Other_. 

Entre los tipos de objetos predefinidos que nos permite crear, encontramos diferentes formas geométricas como _Cube_, _Sphere_, _Capsule_ o _Plane_ entre otras. Estas figuras pueden resultarnos de utilidad como objetos _impostores_ en primeras versiones del juego en las que todavía no contamos con nuestros propios modelos gráficos. Por ejemplo, podríamos utilizar un cubo que de momento haga el papel de nuestro personaje hasta que contemos con su modelo 3D.

IMAGEN MENU CREATE OTHER

Podremos **organizar de forma jerárquica** los objetos de la escena mediante la vista _Hierarchy_. Si arrastramos un _game object_ sobre otro en esta vista, haremos que pase a ser su hijo en el árbol de la escena. Los objetos vacíos con un único componente `Transform` pueden sernos de gran utilidad para agrupar dentro de él varios objetos. De esta forma, moviendo el objeto padre podremos mover de forma conjunta todos los objetos que contiene. De esta forma estaremos creando objetos compuestos.

También resulta de utilidad **dar nombre** a los objetos de la escena, para poder identificarlos fácilmente. Si hacemos _click_ sobre el nombre de un objeto en la vista _Hierarchy_ podremos editarlo y darle un nombre significativo (por ejemplo _Suelo_, _Pared_, _Enemigo_, etc).

IMAGEN NOMBRE

### Navegación en la escena

Además de podemos añadir objetos a la escena y moverlos a diferentes posiciones, deberemos poder movernos por la escena para posicionarnos en los puntos de vista que nos interesen para crear el contenido. Será importante conocer una serie de atajos de teclado para poder movernos con fluidez a través de la escena.

Encontramos tres tipos de movimientos básicos para movernos por la escena en el editor:

* **Traslación lateral**: Hacemos _click_ sobre la escena y arrastramos el ratón.
* **Giro**: Pulsamos _Alt + click_ y arrastramos el ratón para girar nuestro punto de vista.
* **Avance**: Pultamos _Ctrl + click_ y arrastramos el ratón, o bien movemos la rueda del ratón para avanzar hacia delante o hace atrás en la dirección en la que estamos mirando.

Con los comandos anteriores podremos desplazarnos libremente sobre la escena, pero también es importante conocer otras forma más directas de movernos a la posición que nos interese:

* **Ver un objeto**: Si nos interesa ir rápidamente a un punto donde veamos de cerca un objeto concreto de la escena, podemos hacer doble _click_ sobre dicho objeto en la vista _Hierarchy_.
* **Alineación con un objeto**: Alinea la vista de la escena con el objeto seleccionado. Es especialmente útil cuando se utiliza con la cámara, ya que veremos la escena tal como se estaría bien desde la camara.