# Redes sociales para videojuegos

Existen diferentes redes sociales específicas para videojuegos. Estas redes nos permiten ver por ejemplo a qué juegos están jugando nuestros amigos, y los **logros** y **puntuaciones** que han conseguido en ellos. También nos permitirán entrar en partidas multijugador con nuestros amigos o almacenar los datos de nuestra partida en la nube. 

Normalmente estas redes están ligadas a la plataforma que utilizamos para jugar (**Game Center** para las plataformas de Apple: iOS, Mac, Apple TV; **Xbox live** para las plataformas de Microsoft: Xbox y Windows; **PS Network** para las plataformas de Sony: PS3, PS4, PS Vita; **Steam** en PC, Linux y Mac), pero también encontramos redes que podemos utilizar en diferentes plataformas (**Google Play Games**, además de Android, puede utilizarse en iOS o plataformas Web). También existen plataformas sociales para videojuegos ofrecidas por terceros (por ejemplo _Game Sparks_, _Playphone_ o _OpenKit_), siendo la mayoría de ellos servicios de pago.

Vamos a ver cómo diseñar e implementar **logros** y **marcadores** con las principales redes sociales para juegos disponibles en las plataformas móviles.

## Diseño de logros y marcadores

### Logros

Los logros son recompensas que podremos obtener cumpliendo determinados retos dentro del juego. Cada logro tiene asociado un reto sobre algo que podemos realizar dentro del juego (por ejemplo, "Destruye 100 naves enemigas"). Una vez consigamos realizar el objetivo de este reto seremos recompensados con el logro. Normalmente veremos los logros conseguidos como medallas, y podremos ver también los logros obtenidos por otros jugadores y compararlos con los que hemos obtenido nosotros.

![Medalla obtenido con un logro](social_logro.png)

Bien diseñados, los logros podrán hacer que los jugadores tengan más incentivos para jugar a nuestro juego. 


#### Datos de un logro

Para cada logro deberemos proporcionar la siguiente información:

* **Título**: Texto que se mostrará cuando el usuario consiga el logro. Normalmente se indicará el reto asociado o algo relacionado con él. Por ejemplo, podríamos indicar de forma literal a qué corresponde el logro (por ejemplo "Consigue 1.000.000 de puntos"), o algo relacionado (por ejemplo "Millonario"). 
* **Puntos**: Cada logro tiene asociado un número de puntos. Cuanto más complejo sera conseguir un logro, más puntos le deberíamos asignar. El número total de puntos que podrán sumar todos los logros de nuestro juego será como **máximo de 1.000 puntos**, por lo que debemos repartir los puntos con cuidado. Deberemos evitar asignar todos los puntos en la primera versión del juego. A partir del análisis de la forma de jugar de los usuarios seguramente se determine la conveniencia de añadir nuevos logros que se adapten a su dinámica de juego.
* **Oculto**: Podemos marcar los logros como ocultos para que el usuario no pueda verlos hasta que los haya conseguido (por ejemplo para evitar _spoilers_ o crear _"huevos de pascua"_). Si el logro no es oculto, podremos ver si título en la lista de logros del juego, aunque todavía no lo hayamos obtenido, con lo que tendremos una indicación de qué tendríamos que hacer para conseguirlo.



#### Tipos de logros

Antes de ver una serie de consejos para el diseño de logros, vamos a realizar una clasificación de tipos de logros que podemos incluir:

* **Logros de progreso**: Logros que se conceden conforme progresamos en el juegos. Por ejemplo, _"Completa el nivel 1"_. Estos logros se obtendrán siempre que avancemos en el juego. No suponen un reto extra, pero son un buen incentivo para completar el juego.
* **Retos extra**: Suponen retos adicionales al mero avance en el juego. Por ejemplo, _"Completa un nivel sin recibir ningún daño"_. Estos logros aumentan la rejugabilidad y alargan la vida del juego. 
* **Logros ocultos**: Los logros ocultos pueden ser de cualquiera de los tipos anteriores, pero el reto para obtener el logro no es visible hasta que no se haya obtenido. Esto es especialmente últil en el caso de logros de progreso, para evitar _spoilers_.  


#### Consejos para el diseño de logros

El diseño de logros será una tarea que normalmente realizaremos en las fases finales del desarrollo del videojuego. Será importante tener muy bien definidas cuáles son las mecánicas y modos del juego y los contenidos que vamos a ofrecer. A continuación mostramos una serie de consejos para el diseño de logros:

* La lista de logros debe ser una buena **representación del juego**. Es decir, deberíamos tener logros que se consigan con cada modo de juego (por ejemplo, modo "Historia" y modo "Contrarreloj"), y con cada mecánica del juego (por ejemplo, matar enemigos, coger monedas, etc). 
* Deben existir logros para jugadores con **diferentes grados de experiencia**, desde logros que cualquier jugador pueda conseguir, hasta logros dirigidos a los jugadores más experimentados. Algunos juegos ofrecen como incentivo dar "logros fáciles", con lo cual pueden conseguir usuarios que desean ganar puntos de logros y así competir con sus amigos, pero esto no favorece la experiencia de juego.
* **Utilizar logros ocultos para eventos inesperados**, y conseguir así sorprender al jugador. Podemos hacer que estos logros aperezcan cuando se realizar algo que no está contemplado en la historia del juego, o cuando el jugador falla en algo. Por ejemplo, dar un logro cuando hemos muerto _N_ veces, o cuando hemos recorrido un escenario en dirección contraria.
* **No ofrecer todos los logros en la primera versión del juego**. Es conveniente observar el comportamiento de los jugadores una vez el juego ha sido lanzado, y así poder añadir nuevos logros que se adapten a lo que los jugadores buscan en el juego. Además, podremos añadir logros para nuevos contenidos que podamos incorporar.


### Marcadores

Los marcadores anotarán la **puntuación máxima** que hemos conseguido en el juego. Además, no sólo nos permitirán ver la puntuación que hemos obtenido, sino que podremos **compararla** con la de nuestros amigos y con la de otros jugadores de todo el mundo. Esta es una cuestión importante, ya que en un marcador mundial es muy difícil conseguir estar en puestos destacados, lo cual puede desanimar a la mayoría de jugadores. Sin embargo, si tenemos la opción de ver en el marcador sólo a nuestros amigos, es más probable que podamos "pelear" por los primeros puestos, y esto mejorará la **retención** de los usuarios, bien para conseguir llegar a ocupar las primeras posiciones, o para conservarlas.

Podremos tener **varios marcadores** en nuestro juego, con distintos tipos de datos (por ejemplo, _puntuaciones máximas_, _monedas recolectadas_, _mejores tiempos_, etc). Según el tipo de marcador, podremos indicar si la ordenación debe ser **ascendente o descendente**. Por ejemplo, para la _puntuación_ debería ser descendente (la máxima encabezará la lista), pero para _mejores tiempos_ deberíamos utilizar un marcador ascendente (el que menor tiempo haya hecho encabezará la lista).

Al igual que en el caso de los logros, la incorporación de los marcadores se suele hacer en las fases finales del desarrollo. Son algo independiente del juego, durante el transcurso de la partida no tendrán ningún efecto. Normalmente los veremos siempre en una pantalla independiente, ya fuera de la pantalla del juego, donde se mostrará la lista de las mejores puntuaciones. 


## Configuración de logros y marcadores


## Implementación de logros y marcadores



## Referencias

* Artículo sobre diseño de marcadores: 

[Leaderboards - The original social feature](http://www.gamesparks.com/blog/leaderboards/)

* Articulos sobre diseño de logros:

[Achievement Design 101](http://www.gamasutra.com/blogs/GregMcClanahan/20091202/86035/Achievement_Design_101.php)
[The Cake Is Not a Lie: How to Design Effective Achievements](http://www.gamasutra.com/view/feature/6360/the_cake_is_not_a_lie_how_to_.php?print=1)

* Artículo sobre juegos sociales:

[The social network game boom](http://www.gamasutra.com/view/feature/4009/the_social_network_game_boom.php)



