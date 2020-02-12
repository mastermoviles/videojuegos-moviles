# Sprites 

En esta sesión vamos a ver un componente básico de los videojuegos: los _sprites_. Vamos a ver las herramientas y técnicas para tratar estos elementos de forma apropiada, cómo animarlos, moverlos por la pantalla y detectar colisiones entre ellos.

Los _sprites_ hemos dicho que son todos aquellos objetos de la escena que se mueven y/o podemos interactuar con ellos de alguna forma. 

Básicamente, un _sprite_ se compone de:

* Su **posición** en la escena.
* Una imagen o animación (por ejemplo el _ciclo de andar_ de un personaje).

La forma más básica de crear el _sprite_ es a partir de una imagen estática, pero habitualmente utilizaremos un conjunto de fotogramas con los que componer diferentes animaciones del _sprite_.


## Fotogramas

Para animar un _sprite_ deberemos definir los distintos fotogramas (o _frames_) de la animación. Podemos definir varias animaciones para cada _sprite_, según las acciones que pueda hacer. Por ejemplo, nuestro personaje puede tener una animación para andar  y otra para correr.

El _sprite_ tendrá un determinado tamaño (ancho y alto), y cada fotograma será una imagen de este tamaño. Cambiando el fotograma que se muestra del _sprite_ en cada momento podremos animarlo. Para ello deberemos tener imágenes para los distintos fotogramas del _sprite_.
  
Sin embargo, la memoria de vídeo es un recurso crítico, y debemos aprovechar al máximo el espacio de las texturas que se almacenan en ella. El tamaño de las texturas en memoria de video debe ser potencia de 2, lo cual en ocasiones nos obliga a dejar _zonas en blanco_ en cada imagen. 

Además, conviene evitar empaquetar con la aplicación un gran número de imágenes, ya que esto hará que el espacio que ocupen sea mayor, y que la carga de las mismas resulte más costosa.

Para almacenar los fotogramas de los _sprites_ de forma óptima, utilizamos lo que se conoce como _sprite sheets_. Se trata de imágenes en las que incluyen de forma conjunta todos los fotogramas
de los _sprites_, dispuestos en forma de mosaico.
  
![Mosaico con los frames de un sprite](imagenes/juegos/sprite.gif)


Podemos crear estos _sprite sheets_ de forma manual, aunque encontramos herramientas que nos facilitarán enórmemente este trabajo, como:

* **TexturePacker** (http://www.texturepacker.com). Se trata de una herramienta muy completa, que cuenta con una versión básica gratuita,y opciones adicionales de pago. Además de organizar los _sprites_ de forma óptima en el espacio de una textura OpenGL, nos permite optimizar el formato de la textura. Esta herramienta permite generar los _sprite sheets_ en varios formatos reconocidos por los diferentes
motores de videojuegos, como por ejemplo Unity, SpriteKit, Cocos2d-x,y libgdx.

    ![Herramienta TexturePacker](imagenes/juegos/texturas_packer.jpg)

* **FreeTexturePacker** (http://free-tex-packer.com). Se trata de una buena alternativa _open source_ a la aplicación anterior. 

* **Shoebox** (https://renderhjs.net/shoebox/). Conjunto de herramientas gratuito desarrollado en Adobe Air, de ayuda para el desarrollo de videojuegos, entre las que se encuentra la generación de _sprite sheets_.

Con estas herramientas simplemente tendremos que arrastrar sobre ellas el conjunto de imágenes con los distintos fotogramas de nuestros _sprites_, y nos generarán una textura optimizada para OpenGL con todos ellos dispuestos en forma de mosaico. Cuando almacenemos esta textura generada, normalmente se guardará un fichero
`.png` con la textura, y un fichero de metadatos que contendrá información sobre los distintos fotogramas que contiene la textura, y la región que ocupa cada uno de ellos.

Para poder utilizar los fotogramas añadidos a la textura deberemos contar con algún mecanismo que nos permita mostrar en pantalla de forma independiente cada región de la textura anterior (cada fotograma). En prácticamente todos los motores para videojuegos encontraremos mecanismos para hacer esto. El motor será capaz de leer el fichero de metadatos del _spritesheet_, y con esto nos permitirá utilizar cada fotograma (_frame_) de forma independiente, pero almacenándolos internamente en una única textura.



## Animaciones

Podremos definir determinadas secuencias de _frames_ para crear animaciones. Cada animación se compondrá de:

* Secuencia de fotogramas (_frames_)
* Periodicidad (tiempo que se tarde en cambiar de fotograma)

Determinados motores nos darán facilidades para crear estas animaciones y aplicarlas sobre los _sprites_, de forma que se reproduzcan de forma automática.



## _Sprite batch_

En OpenGL los _sprites_ se dibujan realmente en un contexto 3D. Es decir, son texturas que se mapean sobre polígonos 3D (concretamente con una geometría rectángular). Muchas veces encontramos en pantalla varios _sprites_ que utilizan la misma textura (o distintas regiones de la misma textura, como hemos visto en el caso de los _sprite sheets_). Podemos optimizar el dibujado de estos _sprites_ 
generando la geometría de todos ellos de forma conjunta en una única operación con la GPU. Esto será posible sólo cuando el conjunto de _sprites_ a dibujar estén contenidos en una misma textura. 

Esto es lo que se conoce como un _sprite batch_ (dibujamos un conjunto de _sprites_ mediante una única operación). Algunos motores 2D nos permitirán utilizar esta característica para optimizar el dibujado.


## Colisiones

Otro aspecto importante de los _sprites_ es la interacción entre ellos. Nos interesará saber cuándo somos tocados por un enemigo o una bala para disminuir la vida, o cuándo alcanzamos nosotros a nuestro enemigo. Para ello deberemos detectar las colisiones entre _sprites_. 

La colisión con _sprites_ de formas complejas puede resultar costosa de calcular. Por ello se suele realizar el cálculo de colisiones con una forma aproximada  de los _sprites_ con la que esta operación resulte más sencilla. Para ello solemos utilizar el _bounding box_, es decir, un rectángulo que englobe el _sprite_. La intersección de  rectángulos es una operación muy sencilla. 
    
> En algunos casos nos puede interesar modificar el tamaño del _bounding box_ respecto al área que ocupa el _sprite_ en pantalla. Por ejemplo, puede ser una buena práctica hacerlo más pequeño para evitar que cuando un enemigo "roce" una esquina del área del _sprite_ nos mate, lo cual puede ser muy frustrante para el jugador. En algunos motores encontraremos de forma separada las propiedades `size` y `boundingBox` de los _sprites_.