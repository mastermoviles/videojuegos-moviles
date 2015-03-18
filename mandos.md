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

## Mandos virtuales

Cuando la mecanica de nuestro juego exige que se controles mediante un mando tradicional, y no contamos con ningún mando _hardware_ que podamos utilizar, la única solución será introducir en nuestro juego un mando virtual en pantalla. 

Vamos a ver diferentes tipos de mandos que podemos implementar en pantalla, emulando controles tanto digitales como analógicos.

### Pad virtual

El _pad_ virtual consiste en dibujar la cruceta de control digital sobre la pantalla y mediante los eventos de la pantalla táctil detectar cuándo se pulsa sobre él. Esta es la forma más sencilla de implementar un control virtual, y será suficiente en el caso de juegos que sólo requieran controles digitales.

Crearemos los diferentes botones del _pad_ virtual como _sprites_, los posicionaremos en pantalla, y programaremos los eventos necesarios para detectar cuándo pulsamos sobre ellos. Vamos a ver un ejemplo sencillo con tres botones, un _pad_ direccional con botones para movernos a la izquierda y derecha, y un botón de acción:

```cpp
private: 
    cocos2d::Sprite *m_buttonAction;
    cocos2d::Sprite *m_buttonLeft;
    cocos2d::Sprite *m_buttonRight;
    ...
```

Vamos además a crear una enumeración que nos ayude a identificar cada botón:

```cpp
enum PadButton {
    BUTTON_LEFT, BUTTON_RIGHT, BUTTON_ACTION
};
```

Algo que debemos tener en cuenta al posicionar los controles, es que éstos siempre deben quedar en la parte visible de la pantalla. Por ejemplo, al inicializar nuestro _pad_ virtual podemos posicionar los botones de la siguiente forma:

```cpp
Size visibleSize = Director::getInstance()->getVisibleSize();
Vec2 visibleOrigin = Director::getInstance()->getVisibleOrigin();
        
m_buttonLeft = Sprite::createWithSpriteFrameName("boton-direccion.png");
m_buttonLeft->setAnchorPoint(Vec2(0,0));
m_buttonLeft->setPosition(visibleOrigin.x+kMARGEN_MANDO, visibleOrigin.y+kMARGEN_MANDO);
m_buttonLeft->setOpacity(127);
m_buttonLeft->setTag(PadButton::BUTTON_LEFT);
        
m_buttonRight = Sprite::createWithSpriteFrameName("boton-direccion.png");
m_buttonRight->setAnchorPoint(Vec2(1,0));
m_buttonRight->setScaleX(-1);
m_buttonRight->setOpacity(127);
m_buttonRight->setPosition(visibleOrigin.x+ kMARGEN_MANDO + m_buttonLeft->getContentSize().width + kMARGEN_MANDO, visibleOrigin.y+kMARGEN_MANDO);
m_buttonRight->setTag(PadButton::BUTTON_RIGHT);

m_buttonAction = Sprite::createWithSpriteFrameName("boton-accion.png");
m_buttonAction->setAnchorPoint(Vec2(1,0));
m_buttonAction->setPosition(visibleOrigin.x + visibleSize.width - kMARGEN_MANDO, visibleOrigin.y+kMARGEN_MANDO);
m_buttonAction->setOpacity(127);
m_buttonAction->setTag(PadButton::BUTTON_ACTION);
```

En este ejemplo vemos además que hacemos los botones **semitransparentes**. Esta es una práctica habitual, que hará que los botones virtuales afecten menos al apartado visual de nuestro videojuego.

También podemos observar que hemos aprovechado la propiedad _tag_ de los botones para identificarlos mediante los elementos de la enumeración `PadButton`. 

Una vez hemos creado los _sprites_ de los botones los añadiremos a la pantalla:

```cpp
m_node= Node::create();
m_node->addChild(m_buttonLeft,0);
m_node->addChild(m_buttonRight,0);
m_node->addChild(m_buttonAction,0);
m_node->setLocalZOrder(100);
```

Tras esto, debemos definir un _listener_ de eventos táctiles para detectar cuándo pulsamos sobre ellos: 

```cpp
m_listener = EventListenerTouchOneByOne::create();
m_listener->setSwallowTouches(true);
```

Definiremos además en nuestra clase dos funciones _callback_ a las que avisaremos cuando se pulse sobre un botón o cuando se suelte:

```cpp
public: 
    ...
    
    std::function<void(PadButton)> onButtonPressed;
    std::function<void(PadButton)> onButtonReleased;
```

También nos vendrá bien contar con un _array_ mediante el que controlemos el estado actual de cada botón, para poderlo consultar en cualquier momento sin tener que utilizar los _callbacks_ anteriores:

```cpp
private: 
    ...
    
    bool buttonState[kNUM_BOTONES];
```

Una vez definidos los elementos anteriores, ya podemos programar los eventos del _listener_ de la pantalla táctil. Al comenzar un contacto comprobaremos si se ha pulsado sobre el botón:

```cpp
m_listener->onTouchBegan = [=](Touch* touch, Event* event) {

    auto target = static_cast<Sprite*>(event->getCurrentTarget());
    Point locationInNode = target->convertToNodeSpace(touch->getLocation());
            
    Size s = target->getContentSize();
    Rect rect = Rect(0, 0, s.width, s.height);
            
    if(rect.containsPoint(locationInNode)) {
        buttonState[target->getTag()] = true;
        if(onButtonPressed) {
            onButtonPressed((PadButton)target->getTag());
        }
        target->setOpacity(255);
        return true;
    }
            
    return false;
};
```

En este caso `target` se refiere al botón sobre el que se ha definido el _listener_. Comprobamos si hemos pulsado sobre el área del botón (`target`) y en tal caso anotamos que dicho botón está pulsado y avisamos al _callback_ correspondiente, en caso de que se haya asignado uno.

De forma similar podemos programar el evento de finalización del contacto:

```cpp
m_listener->onTouchEnded = [=](Touch* touch, Event* event) {
    auto target = static_cast<Sprite*>(event->getCurrentTarget());
    target->setOpacity(127);
    buttonState[target->getTag()] = false;
    if(onButtonReleased) {
        onButtonReleased((PadButton)target->getTag());
    }
};
```

Obtenemos el botón (`target`) sobre el que se ha definido el _listener_ y anotamos que el botón ya no está pulsado, además de llamar al _callback_ correspondiente en caso de estar asignado.

Por último, añadiremos el _listener_ sobre cada uno de los botones. Podemos observar que hay una instancia del _listener_ para cada botón, con lo que en cada uno de ellos el `target` será un único botón concreto:

```cpp
m_node->getEventDispatcher()->addEventListenerWithSceneGraphPriority(m_listener, m_buttonLeft);
m_node->getEventDispatcher()->addEventListenerWithSceneGraphPriority(m_listener->clone(), m_buttonRight);
m_node->getEventDispatcher()->addEventListenerWithSceneGraphPriority(m_listener->clone(), m_buttonAction);
```

Otra posible alternativa habría sido añadir una lista (`Vector`) con todos los botones del mando, y un único _listener_ que recorra la lista y los compruebe todos. 

Necesitaremos definir además una función que dé acceso al estado de los botones en cualquier momento:

```cpp
bool VirtualPad::isButtonPressed(PadButton button) {
    return buttonState[button];
}
```

Mostramos a continuación el código completo de esta implementación sencilla de un _pad_ virtual:

```cpp
#define kNUM_BOTONES  3
#define kMARGEN_MANDO 20

enum PadButton {
    BUTTON_LEFT, BUTTON_RIGHT, BUTTON_ACTION
};

class VirtualPad: public GameEntity {
public:
    
    bool init();

    void preloadResources();
    Node* getNode();
    
    bool isButtonPressed(PadButton button);
    
    std::function<void(PadButton)> onButtonPressed;
    std::function<void(PadButton)> onButtonReleased;
    
    CREATE_FUNC(VirtualPad);
    
private:
    cocos2d::Sprite *m_buttonAction;
    cocos2d::Sprite *m_buttonLeft;
    cocos2d::Sprite *m_buttonRight;
    
    cocos2d::EventListenerTouchOneByOne *m_listener;
    
    bool buttonState[kNUM_BOTONES];
};
#endif
```

```cpp
bool VirtualPad::init(){
    GameEntity::init();
    
    m_buttonAction = Sprite::create();
    m_buttonLeft = Sprite::create();
    m_buttonRight = Sprite::create();

    for(int i=0;i<kNUM_BOTONES;i++) {
        buttonState[i] = false;
    }
    
    return true;
}

void VirtualPad::preloadResources(){
    
    //Cache de sprites
    auto spriteFrameCache = SpriteFrameCache::getInstance();
    
    //Si no estaba el spritesheet en la caché lo cargo
    if(!spriteFrameCache->getSpriteFrameByName("boton-direccion.png")) {
        spriteFrameCache->addSpriteFramesWithFile("mando.plist");
    }
}

Node* VirtualPad::getNode(){
    if(m_node==NULL) {
        
        Size visibleSize = Director::getInstance()->getVisibleSize();
        Vec2 visibleOrigin = Director::getInstance()->getVisibleOrigin();
        
        m_buttonLeft = Sprite::createWithSpriteFrameName("boton-direccion.png");
        m_buttonLeft->setAnchorPoint(Vec2(0,0));
        m_buttonLeft->setPosition(visibleOrigin.x+kMARGEN_MANDO, visibleOrigin.y+kMARGEN_MANDO);
        m_buttonLeft->setOpacity(127);
        m_buttonLeft->setTag(PadButton::BUTTON_LEFT);
        
        m_buttonRight = Sprite::createWithSpriteFrameName("boton-direccion.png");
        m_buttonRight->setAnchorPoint(Vec2(1,0));
        m_buttonRight->setScaleX(-1);
        m_buttonRight->setOpacity(127);
        m_buttonRight->setPosition(visibleOrigin.x+ kMARGEN_MANDO + m_buttonLeft->getContentSize().width + kMARGEN_MANDO, visibleOrigin.y+kMARGEN_MANDO);
        m_buttonRight->setTag(PadButton::BUTTON_RIGHT);

        m_buttonAction = Sprite::createWithSpriteFrameName("boton-accion.png");
        m_buttonAction->setAnchorPoint(Vec2(1,0));
        m_buttonAction->setPosition(visibleOrigin.x + visibleSize.width - kMARGEN_MANDO, visibleOrigin.y+kMARGEN_MANDO);
        m_buttonAction->setOpacity(127);
        m_buttonAction->setTag(PadButton::BUTTON_ACTION);
        
        m_node= Node::create();
        m_node->addChild(m_buttonLeft,0);
        m_node->addChild(m_buttonRight,0);
        m_node->addChild(m_buttonAction,0);
        m_node->setLocalZOrder(100);
        
        m_listener = EventListenerTouchOneByOne::create();
        m_listener->setSwallowTouches(true);
        
        m_listener->onTouchBegan = [=](Touch* touch, Event* event) {

            auto target = static_cast<Sprite*>(event->getCurrentTarget());
            Point locationInNode = target->convertToNodeSpace(touch->getLocation());
            
            Size s = target->getContentSize();
            Rect rect = Rect(0, 0, s.width, s.height);
            
            if(rect.containsPoint(locationInNode)) {
                buttonState[target->getTag()] = true;
                if(onButtonPressed) {
                    onButtonPressed((PadButton)target->getTag());
                }
                target->setOpacity(255);
                return true;
            }
            
            return false;
        };
        
        m_listener->onTouchEnded = [=](Touch* touch, Event* event) {
            auto target = static_cast<Sprite*>(event->getCurrentTarget());
            target->setOpacity(127);
            buttonState[target->getTag()] = false;
            if(onButtonReleased) {
                onButtonReleased((PadButton)target->getTag());
            }
        };
        
        m_node->getEventDispatcher()->addEventListenerWithSceneGraphPriority(m_listener, m_buttonLeft);
        m_node->getEventDispatcher()->addEventListenerWithSceneGraphPriority(m_listener->clone(), m_buttonRight);
        m_node->getEventDispatcher()->addEventListenerWithSceneGraphPriority(m_listener->clone(), m_buttonAction);

    }
    
    return m_node;
}

bool VirtualPad::isButtonPressed(PadButton button) {
    return buttonState[button];
}
```

### Stick virtual

El _stick_ virtual emula el _stick_ analógico de un mando. Podremos pulsar sobre él y arrastrar para así graduar cuánto queremos moverlo en una determinada dirección. En el caso del _pad_ por ejemplo la dirección izquierda puede estar pulsada o no estarlo. En el _stick_ podemos moverlo más o menos a la izquierda, tomando valores reales entre -1 y 0. 



### Stick virtual con posicionamiento automático

El _stick_ virtual tiene el problema de no tener _feedback_ físico, por lo que si tenemos la atención centrada en la escena del juego es posible que no sepamos si estamos tocando en el centro del mando o no, al intentar hacer un moviemiento. Para evitar esto podemos hacer que al tocar sobre la pantalla el _stick_ se sitúe automáticamente centrado en la posición donde hemos tocado. Así sabremos que siempre tocamos en el centro, y sólo tendremos que arrastrar.

### Stick virtual con posicionamiento automático y deslizamiento


## Soporte de mandos físicos

Vamos a ver en esta sección diferentes tipos de mandos _hardware_ que podremos integrar en nuestros videojuegos.

### Controladores oficiales iOS

La especificación de mandos para dispositivos iOS aparece a partir de iOS 7. En dicha versión del SDK se incorpora el _framework_ `GameController` que nos permitirá añadir soporte para este tipo de mandos, que llevan la etiqueta MFI (_Made for iPhone/iPod/iPad_), la cual se refiere a todos los dispositivos _hardware_ diseñados para estos dispositivos iOS. 

https://developer.apple.com/library/ios/documentation/ServicesDiscovery/Conceptual/GameControllerPG/Introduction/Introduction.html

### Controladores oficiales Android

El soporte para controladores de juego en Android está presente a partir de la API 9, aunque se han ido incorporando mejoras en APIs sucesivas.

http://developer.android.com/training/game-controllers/index.html

Encontramos en Android diferentes mandos que soportan el estándar definido en esta plataforma, como es el caso de Amazon fire TV. También tenemos otros tipos de mandos distintos a los oficiales, como los mandos de tipo Ouya TV, Moga y Nibiru.

### Controladores iCade

Estos controladores no utilizan la API oficial, ya que salieron a la venta antes de que ésta existiese. Se comportan como un teclado _bluetooth_, por lo que para utilizarlos simplemente deberemos conocer a qué tecla está mapeado cada botón. Está diseñado para ser utilizado con el iPad, pero puede utilizarse en cualquier dispositivo móvil que lo reconozca como teclado _bluetooth_.

En los siguientes enlaces se puede encontrar documentación para integrar estos controladores en nuestras aplicaciones:

http://www.ionaudio.com/downloads/ION%20Arcade%20Dev%20Resource%20v1.5.pdf

http://www.raywenderlich.com/8618/adding-icade-support-to-your-game


## Controladores oficiales en Cocos2d-x

Cocos2d-x soporta tanto los mandos oficiales de Android como los oficiales de iOS, ofreciéndonos una API única para utilizarlos en cualquiera de estas plataformas. 

Vamos a centrarnos en la API común de Cocos2d-x y en las cuestiones específicas para utilizarla en Android e iOS.

### Eventos del mando

En Cocos2d-x encontramos el _listener_ `EventListenerController` que nos permite incorporar soporte para mandos físicos de forma sencilla. Este _listener_ nos permite recibir los siguientes eventos:

* `onConnected`: Se ha conectado un mando.
* `onDisconnected`: Se ha desconectado un mando.
* `onKeyDown`: Se ha pulsado un botón del mando.
* `onKeyUp`: Se ha soltado un botón del mando.
* `onKeyRepeat`: Se mantiene pulsado un botón.
* `onAxisEvent`: Notifica cambios en el _stick_ analógico.

A continuación vemos el esqueleto de la clase de una escena de nuestro juego en la que utilizamos como entrada el mando. Al iniciar la escena registraremos el _listener_ de eventos del mando y configuraremos los _callbacks_ necesarios para cada uno de los eventos anteriores:

```cpp
bool MiEscena::init()
{
    if ( !Layer::init() )
    {
        return false;
    }       
    
    configuraMandos();

    return true;
}

void MiEscena::configurarMando()
{
    _listener = EventListenerController::create();

    // Registramos callbacks
    _listener->onConnected = CC_CALLBACK_2(MiEscena::onConnectController,this);
    _listener->onDisconnected = CC_CALLBACK_2(MiEscena::onDisconnectedController,this);
    _listener->onKeyDown = CC_CALLBACK_3(MiEscena::onKeyDown, this);
    _listener->onKeyUp = CC_CALLBACK_3(MiEscena::onKeyUp, this);
    _listener->onAxisEvent = CC_CALLBACK_3(MiEscena::onAxisEvent, this);

    // Añadimos el listener el mando al gestor de eventos
    _eventDispatcher->addEventListenerWithSceneGraphPriority(_listener, this);

    // Inicia búsqueda de controladores (necesario en iOS)
    Controller::startDiscoveryController();
}

void MiEscena::onKeyDown(cocos2d::Controller *controller, int keyCode, cocos2d::Event *event) { }   

void MiEscena::onKeyUp(cocos2d::Controller *controller, int keyCode, cocos2d::Event *event) { }

void MiEscena::onAxisEvent(cocos2d::Controller* controller, int keyCode, cocos2d::Event* event) { }   

void MiEscena::onConnectController(Controller* controller, Event* event) { }

void MiEscena::onDisconnectedController(Controller* controller, Event* event) { }
```

A continuación veremos con más detalle estos eventos.

### Conexión y desconexión del mando

Los mandos se conectarán de forma inalámbrica al móvil, por lo que deberemos poder conectar nuevos mandos, o desconectar los que tenemos conectados. 

Podemos estar al tanto de los eventos de conexión y desconexión de mandos. A partir del parámetros `Controller` que nos proporcionan estos eventos podremos saber además datos sobre el mando que se ha conectado:

```cpp
void MiEscena::onConnectController(Controller* controller, Event* event) { 
    CCLOG("Tag:%d", controller->getTag());
    CCLOG("Id:%d", controller->getDeviceId());
    CCLOG("Nombre:%s", controller->getDeviceName().c_str());
}

void MiEscena::onDisconnectedController(Controller* controller, Event* event) { 

}
```

Como vemos, una propiedad de los controladores es su etiqueta (_tag_). Podemos poner una etiqueta a los mandos para poder acceder a ellos de forma sencilla con `setTag` y consultarla con `getTag`. Esta etiqueta será un número entero. Por ejemplo, podríamos utilizar las etiquetas `1` y `2` para identificar los mandos para el primer y segundo jugador respectivamente. Podremos localizar uno de estos mandos de forma inmediata con el método estático `Controller::getControllerByTag`.

```cpp
Controller* primerJugador = Controller::getControllerByTag(1);
```


### Pulsación de teclas

A partir de un objeto `Controller` podremos conocer el estado de sus botones con el método `getKeyStatus`. Este método recibe como parámetro el código del botón que queremos consultar. En la siguiente imagen mostramos los grupos de botones que encontramos en los mandos para móviles:

![Botones de los mandos](imagenes/mandos/controller-num.png)

Los códigos para los botones de cada grupo se encuentran en la enumeración `Key` y son:

1. Analógico izquierdo: `JOYSTICK_LEFT_X`, `JOYSTICK_LEFT_Y`, `BUTTON_LEFT_THUMBSTICK`
2. Analógico derecho: `JOYSTICK_RIGHT_X`, `JOYSTICK_RIGHT_Y`, `BUTTON_RIGHT_THUMBSTICK`
3. Pad digital: `BUTTON_DPAD_UP`, `BUTTON_DPAD_DOWN`, 
`BUTTON_DPAD_LEFT`, `BUTTON_DPAD_RIGHT`, `BUTTON_DPAD_CENTER`
4. Botones frontales: `BUTTON_A`, `BUTTON_B`, `BUTTON_C`, `BUTTON_X`, `BUTTON_Y`, `BUTTON_Z`, `BUTTON_START`, `BUTTON_SELECT`, `BUTTON_PAUSE`
5. Gatillos: `AXIS_LEFT_TRIGGER`, `AXIS_RIGHT_TRIGGER`
6. Botones superiores: `BUTTON_LEFT_SHOULDER`, 
`BUTTON_RIGHT_SHOULDER`

Por ejemplo, si queremos consultar el estado del botón `A` en el mando del primer jugador haremos lo siguiente:

```cpp
KeyStatus estado = primerJugador->getKeyStatus(BUTTON_A);
```

El estado es una estructura que nos da la siguiente información:

* `isPressed`: Booleano que nos indica si está presionado el botón (para el caso de botones digitales).
* `isAnalog`: Nos indica si el botón es analógico (_sticks_ analógicos o gatillos).
* `value`: Nos indica el valor del estado del botón como número flotante. Dependerá del tipo de botón. Por ejemplo en caso de _sticks_ analógicos nos dará un valor entre `-1` y `1`. En caso de gatillos será entre `0` y `1`. En otros botones nos puede dar valores concretos como `0` ó `1`.

Por ejemplo, podemos hacer que al pulsar el botón `A` nuestro personaje dispare y que con el _stick_ izquierdo se mueva horizontalmente:

```cpp
KeyStatus estadoA = primerJugador->getKeyStatus(BUTTON_A);
if(estado.isPressed) {
    player->dispara();
}

KeyStatus estadoHorizontal = primerJugador->getKeyStatus(JOYSTICK_LEFT_X);
player->setVelocity(estadoHorizontal.value);
```

### Configuración de mandos para Android

Cocos2d-x en Android soporta tanto los mandos oficiales (como Amazon fire TV), como los mandos de tipo Ouya TV, Moga y Nibiru. Deberemos hacer algunos cambios en el proyecto Android para soportar estos mandos.

En primer lugar, deberemos añadir al _workspace_ de Eclipse la librería `libControllerManualAdapter` y añadirla como librería de nuestro proyecto Cocos2d-x. Esta librería la podremos encontrar en el directorio `$COCOS_HOME/platform/android/ControllerManualAdapter`.

Una vez añadida la librería, añadiremos los siguientes cambios a la actividad `AppActivity`:

* Haremos que la actividad herede de `GameControllerActivity`.
* En caso de querer utilizar controladores diferentes de los oficiales, deberemos especificarlo de forma explícita en `onCreate`:

```java
this.connectController(DRIVERTYPE_NIBIRU);
this.connectController(DRIVERTYPE_MOGA);
this.connectController(DRIVERTYPE_OUYA);
```
 
Por ejemplo, para dar soporte a controladores de tipo OUYA tendríamos:

```java
public class AppActivity extends GameControllerActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.connectController(DRIVERTYPE_OUYA);
    }
}
```
También será necesario que en nuestro dispositivo Android descarguemos los _drivers_ para el controlador que vayamos a utilizar y lo conectemos al dispositivo.

### Configuración de mandos para iOS

En el caso de iOS, para que nuestro proyecto soporte los mandos oficiales aparecidos a partir de iOS 7, tendremos que añadir el _framework_ `GameController.Framework` a nuestro proyecto.

Además, será importante que en nuestro proyecto llamemos a `Controller::startDiscoveryController()` para que inicie la búsqueda de mandos y establezca una conexión con ellos, tal como hemos indicado anteriormente.

## Eventos de teclado en Cocos2d-x
 
Cocos2d-x soporta eventos de teclado, pero éstos no funcionan en plataformas móviles. Aunque nuestro proyecto esté orientado exclusivamente a estas plataformas, si el control de nuestro juego se realiza mediante mando es recomendable que implementemos también la posibilidad de controlarlo mediante teclado. Esto será de gran utilidad durante el desarrollo, ya que no existe forma de emular un mando, y la forma más parecida al mando para manejar nuestro juego en las pruebas que hagamos durante el desarrollo es el control mediante teclado. 
 
Para leer los eventos de teclado desde Cocos2d-x podemos utilizar la clase `EventListenerKeyboard` como se muestra a continuación: 

```cpp
bool MiEscena::init()
{
    if ( !Layer::init() )
    {
        return false;
    }       
    
    configuraTeclado();

    return true;
}

void MiEscena::configurarTeclado()
{
    _listener = EventListenerKeyboard::create();

    // Registramos callbacks
    _listener->onKeyPressed = CC_CALLBACK_2(MiEscena::onConnectController,this);
    _listener->onReleased = CC_CALLBACK_2(MiEscena::onDisconnectedController,this);

    // Añadimos el listener el mando al gestor de eventos
    _eventDispatcher->addEventListenerWithSceneGraphPriority(_listener, this);
}

void MiEscena::onKeyDown(EventKeyboard::KeyCode code, Event *event) { }   

void MiEscena::onKeyUp(EventKeyboard::KeyCode code, Event *event) { }
```

Por ejemplo, para reconocer los controles izquierda-derecha mediante las teclas A-D podríamos escribir los métodos `onKeyDown` y `onKeyUp` como se muestra a continuación:

```cpp
void MiEscena::onKeyDown(EventKeyboard::KeyCode code, Event *event) { 
    switch(keyCode){
        case EventKeyboard::KeyCode::KEY_A:
            _izquierdaPulsado = true;
            break;
        case EventKeyboard::KeyCode::KEY_D:
            _derechaPulsado = true;
            break;
    }
}  

void MiEscena::onKeyUp(EventKeyboard::KeyCode code, Event *event) { 
    switch(keyCode){
        case EventKeyboard::KeyCode::KEY_A:
            _izquierdaPulsado = false;
            break;
        case EventKeyboard::KeyCode::KEY_D:
            _derechaPulsado = false;
            break;
    }
}  
```