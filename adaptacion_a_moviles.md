# Adaptación a móviles



## Gestión multi-resolución

Una de las principales problemáticas en el desarrollo de dispositivos móviles es la
gran diferencia de tamaños de pantalla existentes, con distintas relaciones de aspecto. 
Esto plantea diferentes problemas:


* **Tamaño de los recursos**: Un enfoque sencillo podría ser proporcionar 
las texturar a resolución máxima, para así aprovechar las pantallas de mayor resolución. 
El problema es que los dispositivos con menor resolución disponen también de una menor
memoria de vídeo, por lo que es probable que no puedan albergar las texturas en resolución
máxima. Por este motivo es necesario proporcionar diferentes versiones de los recursos
para diferentes resoluciones de pantalla.
* **Sistema de coordenadas**: Debemos evitar utilizar un sistema de coordenadas
en pixels, ya que el tamaño cambiará en cada dispositivo. Lo que se hará es utilizar 
siempre un sistema de coordenadas del mismo tamaño independientemente de la resolución
del dispositivo. Esto se conoce como un sistema de coordenadas en puntos. El tamaño de
cada punto dependerá de la resolución real de la pantalla.
* **Relación de aspecto**: A pesar de trabajar en puntos para que las 
dimensiones del sistema de coordenadas utilizado sean siempre las mismas, tenemos el problema
de que la relación de aspecto puede ser distinta. Para resolver esto podemos añadir un borde
cuando la relación de aspecto del dispositivo no coincide con la que se ha utilizado en el diseño,
o bien recortar la pantalla en alguna de sus dimensiones, o estirarla a pesar de deformar
la imagen. Si decidimos recortar, deberíamos diseñar el juego de forma que sobre suficiente espacio 
como para que no pase nada si algún dispositivo recorta.


Vamos a ver cómo implementar todo lo anterior en Cocos2d-x. En primer lugar, para
soportar distintas versiones de recursos lo que haremos es guardarlos en diferentes directorios.
Por ejemplo, podemos crear un directorio `sd` para la versión normal y otro
directorio `hd` para la versión para dispositivos de alta resolución. Ambos
directorios tendrán los mismos ficheros de texturas, pero con distintas resoluciones. Lo
que deberemos hacer es indicar al motor dónde buscar los recursos en función de la resolución:

```cpp
CCSize screenSize = CCEGLView::sharedOpenGLView()->getFrameSize();

std::vector<std::string> searchPaths;

if (screenSize.height > 320) { // iPhone retina
    searchPaths.push_back("hd");
    searchPaths.push_back("sd");
}
else { // iPhone
    searchPaths.push_back("sd");
}
CCFileUtils::sharedFileUtils()->setSearchPaths(searchPaths);
```

En el ejemplo anterior, en el caso del iPhone retina buscará primero los recursos en el
directorio `hd`, y si no los encuentra ahí buscará en `sd`. En caso
caso de tener menor resolución buscará todo directamente en `sd`.

Para resolver el problema de los distintos tamaños de pantalla lo que haremos será definir
tres resoluciones distintas:


* **Resolución de recursos**: Resolución para la que están preparados los 
recursos utilizados.
* **Resolución de diseño**: Resolución para la que hemos diseñado el juego. 
Será esta resolución la que utilizaremos en el código del juego (resolución en puntos).
* **Resolución de pantalla**: Resolución real de la pantalla del dispositivo.


En el objeto `AppDelegate` se inicializa el juego. Este es un buen punto
para configurar las resoluciones anteriores. Por ejemplo, podemos definir esta
configuración de la siguiente forma:

```cpp
CCSize screenSize = CCEGLView::sharedOpenGLView()->getFrameSize();
CCSize designSize = CCSizeMake(480, 320);
CCSize resourceSize;
std::vector<std::string> searchPaths;

if (screenSize.height >= 768) { // iPad
    searchPaths.push_back("hd");
    searchPaths.push_back("sd");
    resourceSize = CCSizeMake(1024, 768);
    designSize = CCSizeMake(1024, 768);
}
else if (screenSize.height > 320) { // iPhone retina
    searchPaths.push_back("hd");
    searchPaths.push_back("sd");
    resourceSize = CCSizeMake(960, 640);          
}
else { // iPhone
    searchPaths.push_back("sd");
    resourceSize = CCSizeMake(480, 320);
}
CCFileUtils::sharedFileUtils()->setSearchPaths(searchPaths);
pDirector->setContentScaleFactor(resourceSize.width / designSize.width);

cocos2d::Director::getInstance()->getOpenGLView()->setDesignResolutionSize(320, 480, ResolutionPolicy::FIXED_WIDTH);
```

En este caso, además de configurar los directorios de recursos fijamos dos ajustes más:


* `setContentScaleFactor`: Establece el factor de escala a aplicar a los
recursos. Por ejemplo, si la resolución de diseño con la que trabajamos es 480x320, pero 
el móvil tiene una resolución de 960x640, los recursos deberán tener el doble de resolución
en pixels que en puntos, para aprovechar la definición de la pantalla, pero deberán ocupar
el mismo espacio en pantalla que en otros dispositivos con distinta definición.
* `setDesignResolutionSize`: Establecemos la resolución de diseño a utilizar
en el juego. Además el tercer parámetro permite definir la forma de adaptarse a
diferentes relaciones de aspecto (recortar, añadir borde, o estirar).

### Estrategias de adaptación

Hemos visto que el tercer parámetro de `setDesignResolutionSize` nos permite indicar la forma de adaptar la resolución de diseño a la resolución de pantalla cuando la relación de aspecto de ambas resoluciones no coincida. Encontramos las siguientes estrategias:

* `kResolutionShowAll`: Hace que todo el contenido de la resolución de diseño quede dentro de la pantalla, dejando franjas negras en los laterales si la relación de aspecto no es la misma. Estas franjas negras hacen que desperdiciemos espacio de pantalla y causan un efecto bastante negativo, por lo que a pesar de la sencillez de esta estrategia, **no será recomendable** si buscamos un producto con un buen acabado.
* `kResolutionExactFit`: Hace que el contenido dentro de la resolución de diseño se estire para adaptarse a la resolución de pantalla, deformando el contenido si la relación de aspecto no es la misma. Aunque en este caso se llene la pantalla, la deformación de la imagen también causará muy mal efecto y por lo tanto debemos **evitar utilizar esta técnica**.
* `kResolutionNoBorder`: Ajusta el contenido de la resolución de diseño a la resolución de pantalla, sin dejar borde y sin deformar el contenido, pero dejando parte de éste fuera de la pantalla si la relación de aspecto no coincide. En este caso no habrá problema si implementamos el juego de forma correcta, ayudándonos de los métodos `Director::getInstance()->getVisibleSize()` y `Director::getInstance()->getVisibleOrigin()` que nos darán el tamaño y el origen, respectivamente, de la zona visible de nuestra resolución de diseño. De esta forma deberemos asegurarnos de dibujar todos los componentes del HUD dentro de esta zona, y a la hora de implementar _scroll_ lo alinearemos de forma correcta con el origen de la zona visible. 
* `kResolutionFixedHeight`, `kResolutionFixedWidth` modifican la resolución de diseño para que tenga la misma relación de aspecto que la resolución de pantalla, manteniendo fija la altura o la anchura de diseño respectivamente. Podremos consultar la resolución de diseño con `Director::getInstance()->getWinSize()`. En estos casos toda la resolución de diseño es visible en pantalla, pero ésta puede variar en altura o en anchura, según la estrategia indicada. 

¿Qué estrategia debemos utilizar? Dependerá de lo que busquemos en nuestro juego, pero normalmente nos quedaremos con `kResolutionNoBorder`, `kResolutionFixedHeight` o `kResolutionFixedWidth`. Por ejemplo, si tenemos un plataformas de avance lateral, normalmente querremos que la altura sea fija, por lo que `kResolutionFixedHeight` podría ser la opción más adecuada. Si por el contrario es un juego de naves que avanza hacia arriba, será más adecuado `kResolutionFixedWidth`. En un juego de rol con vista cenital con _scroll_ en cualquier dirección y el personaje centrado en pantalla podría venir bien `kResolutionNoBorder`, nos da igual la parte que quede cortada siempre que en el caso de haber HUD nos aseguremos de dibujarlo dentro de la zona visible. 

![](imagenes/adaptacion/1942.png)

![](imagenes/adaptacion/ff3.jpg)

![](imagenes/adaptacion/jetpack.jpg)


### Depuración del cambio de densidad de pantalla

Para comprobar que nuestra aplicación se adapta de forma correcta podemos utilizar diferentes tamaños de ventana durante el desarrollo. Sin embargo, también será necesario comprobar lo que ocurre al tener diferentes densidades de pantalla, teniendo algunos dispositivos resoluciones superiores a la de nuestra máquina de desarrollo. 


Para resolver este problema podemos utilizar la función `GLView::setFrameZoomFactor`. Con esta función podemos aplicar un factor de _zoom_ al contenido de la ventana. De esta forma podemos tener altas resoluciones, como los 2048x1536 pixeles de un iPad retina, dentro del espacio de nuestra pantalla.

Esta función deberá invocarse únicamente en el código específico de la plataforma de desarrollo (Windows, Linux o Mac). Por ejemplo, en el caso de Mac añadiremos las siguientes líneas al fichero `AppDelegate.cpp`:

```cpp
bool AppDelegate::applicationDidFinishLaunching() {
    // initialize director
    auto director = Director::getInstance();
    auto glview = director->getOpenGLView();
    if(!glview) {
        glview = GLViewImpl::create("Mi Juego");
        director->setOpenGLView(glview);
    }

    // Depuracion multi-resolucion
    GLView* eglView = Director::getInstance()->getOpenGLView();
    eglView->setFrameSize(1536, 2048);
    eglView->setFrameZoomFactor(0.4f);
    
    // Soporte multi-resolucion
    cocos2d::Director::getInstance()->getOpenGLView()->setDesignResolutionSize(768, 1024, ResolutionPolicy::FIXED_WIDTH);
    
    // turn on display FPS
    director->setDisplayStats(true);

    // set FPS. the default value is 1.0/60 if you don't call this
    director->setAnimationInterval(1.0 / 60);

    // create a scene. it's an autorelease object
    auto scene = TitleScene::createScene();

    // run
    director->runWithScene(scene);

    return true;
}

```

