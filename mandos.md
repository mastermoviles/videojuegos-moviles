# Mandos

La principal forma de control del móvil es la pantalla táctil, por lo que los videojuegos diseñados específicamente para móviles se adaptan a esta forma de entrada. Sin embargo, cuando se quiere trasladar a móvil un juego diseñado originalmente para otro sistema deberemos adaptar su forma de manejo. 

Los juegos diseñados para videoconsolas o máquinas recreativas se manejan normalmente mediante _joystick_ o _pad_. Al portar uno de estos juegos a móvil podemos optar por:

* Adaptar el control de videojuego a pantalla táctil. Esto implica grandes cambios en el diseño del juego y en el _gameplay_ y no siempre es posible hacerlo.
* Añadir un _pad_ virtual en pantalla. Permite mantener el mismo mecanismo de control que el juego original, pero resulta más complicado de manejar que con un mando real.
* Añadir soporte para mandos físicos. Nos permitirá trasladar la misma experiencia de juego que la versión de videoconsola/recreativa pero necesita que el usuario cuente con este dispositivo. Se pierde una de las ventajas de los juegos móviles, que es el llevarlos siempre con nosotros.  

Vamos en esta sesión a centrarnos en este tipo de juegos y en la forma de diseñar un control adecuado para ellos. Veremos tanto la forma de incorporar un _pad_ virtual como la forma de añadir soporte para diferentes tipos de mandos físicos. Dentro de estos mandos encontramos tanto mandos soportados por las APIs oficiales de iOS y Android, como mandos con APIs de terceros, como por ejemplo iCade.

## Buenas prácticas para juegos basados en _control pad_

Si queremos implementar un juego cuyo manejo esté basado en _control pad_, será recomendable seguir las siguientes prácticas:

* Permitir el manejo del juego mediante _pad_ virtual en pantalla si no se dispone de mando real.
* Añadir compatibilidad con mandos reales. Se recomienda añadir soporte para las APIs oficiales y para aquellos mandos más utilizados, como iCade.
* En caso de tener conectado un mando real, ocultar el _pad_ virtual para que no moleste en pantalla.
* Respetaremos la función estándar de cada botón. El botón de _pausa_ del mando debe permitir pausar el juego en cualquier momento. Determinados botones se suelen utilizar para realizar las mismas acciones en todos los juegos (saltos, ataque, acción, etc). Deberemos intentar seguir estas convenciones.
* La pantalla del móvil no debe apagarse mientras utilizamos el juego con el mando externo.

## Soporte de mandos físicos

### Controladores iCade para iOS

http://www.raywenderlich.com/8618/adding-icade-support-to-your-game

### Controladores oficiales iOS

https://developer.apple.com/library/ios/documentation/ServicesDiscovery/Conceptual/GameControllerPG/Introduction/Introduction.html#//apple_ref/doc/uid/TP40013276-CH1-SW1

### Controladores oficiales Android

http://developer.android.com/training/game-controllers/index.html


