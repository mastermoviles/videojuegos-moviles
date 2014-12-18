# Motores de físicas

Un tipo de juegos que ha tenido una gran proliferación en el mercado de aplicaciones
para móviles son aquellos juegos basados en físicas. Estos juegos son aquellos en los que
el motor realiza una simulación física de los objetos en pantalla, siguiendo las leyes
de la cinemática y la dinámica. Es decir, los objetos de la pantalla están sujetos a 
gravedad, cada uno de ellos tiene una masa, y cuando se produce una colisión entre ellos
se produce una fuerza que dependerá de su velocidad y su masa. El motor físico se 
encarga de realizar toda esta simulación, y nosotros sólo deberemos encargarnos de
proporcionar las propiedades de los objetos en pantalla. Uno de los motores físicos más
utilizados es Box2D, originalmente implementado en C++. Se ha utilizado para implementar juegos tan conocidos y exitosos
como Angry Birds. Podemos encontrar ports de este motor para las distintas
plataformas móviles. Motores como Cocos2D y libgdx incluyen una implementación de este
motor de físicas.


![Angry Birds, implementado con Box2D](imagenes/fisicas/box2d_angry.jpg)

	



## Motor de físicas Box2D

Vamos ahora a estudiar el motor de físicas Box2D. Es importante destacar que este motor sólo
	se encargará de simular la física de los objetos, no de dibujarlos. Será nuestra responsabilidad
	mostrar los objetos en la escena de forma adecuada según los datos obtenidos de la simulación física.
	Comenzaremos viendo los principales componentes de esta librería.

	
	## Componentes de Box2D
	
Los componentes básicos que nos permiten realizar la simulación física con Box2D son:
	
	
* `Body`: Representa un cuerpo rígido. Estos son los tipos de objetos que tendremos en el 
	mundo 2D simulado. Cada cuerpo tendrá una posición y velocidad. Los cuerpos se verán afectados por
	la gravedad del mundo, y por la interacción con los otros cuerpos. Cada cuerpo tendrá una serie de
	propiedades físicas, como su masa o su centro de gravedad.
	
* `Fixture`: Es el objeto que se encarga de fijar las propiedades de un cuerpo, como
	por ejemplo su forma, coeficiente de rozamiento o densidad. 
	
* `Shape`: Sirve para especificar la forma de un cuerpo. Hay distintos tipos de formas (subclases de
	`Shape`), como por ejemplo `CircleShape` y `PolygonShape`, para crear cuerpos
	con formar circulares o poligonales respectivamente.
	
* `Constraint`: Nos permite limitar la libertad de un cuerpo. Por ejemplo podemos utilizar una 
	restricción que impida que el cuerpo pueda rotar, o para que se mueva siguiendo sólo una línea (por ejemplo
	un objeto montado en un rail).
	
* `Joint`: Nos permite definir uniones entre diferentes cuerpos.
	
* `World`: Representa el mundo 2D en el que tendrá lugar la simulación. Podemos añadir una serie
	de cuerpos al mundo. Una de las principales propiedades del mundo es la gravedad.
	
	
	
Todas las clases de la librería Box 2D tienen el prefijo `b2`. Hay que tener en cuenta
	que se trata de clases C++, y no Objective-C.
	
	
	
Lo primero que deberemos hacer es crear el mundo en el que se realizará la simulación física. Como parámetro
	deberemos proporcionar un vector 2D con la gravedad del mundo:
	
```cpp
b2Vec2 gravity;
gravity.Set(0, -10);
b2World *world = new b2World(gravity);
```

	
	## Unidades de medida
	
Antes de crear cuerpos en el mundo, debemos entender el sistema de coordenadas de Box2D y sus unidades
	de medida. Los objetos de Box2D se miden en metros, y la librería está optimizada para objetos de 1m, por lo que
	deberemos hacer que los objetos que aparezcan con más frecuencia tengan esta medida. 
	
Sin embargo, los gráficos en pantalla se miden en píxeles (o puntos). Deberemos por lo tanto fijar
	el ratio de conversión entre pixeles y metros. Por ejemplo, si los objetos con los que trabajamos normalmente
	miden 32 pixeles, haremos que 32 pixeles equivalgan a un metro. Definimos el siguiente ratio de conversión:

```cpp
const float PTM_RATIO = 32.0;
```
	
![Métricas de Box2D](imagenes/fisicas/box2d_metricas.jpg)

	
	
Para todas las unidades de medida Box2D utiliza el sistema métrico. Por ejemplo, para la masa de los objetos
	utiliza Kg.
	
	
	
	
	## Tipos de cuerpos
	
Encontramos tres tipos diferentes de cuerpos en Box2D según la forma en la que queremos que se realice 
	la simulación con ellos:
	
	
* **Dinámicos**: Están sometidos a las leyes físicas, y tienen una masa concreta y finita. Estos
	cuerpos se ven afectados por la gravedad y por la interacción con los demás cuerpos.
* **Estáticos**: Son cuerpos que permanecen siempre en la misma posición. Equivalen a cuerpos
	con masa infinita. Por ejemplo, podemos hacer que el escenario sea estático.
* **Cinemáticos**: Al igual que los cuerpos estáticos tienen masa infinita y no se ven afectados
	por otros cuerpos ni por la gravedad. Sin embargo, en esta caso no tienen una posición fija, sino que tienen
	una velocidad constante. Nos son útiles por ejemplo para proyectiles.
	
	
![Tipos de cuerpos en Box2D](imagenes/fisicas/box2d_cuerpos.jpg)

	
	
	
	
	
	## Creación de cuerpos
	
Con todo lo visto anteriormente ya podemos crear distintos cuerpos. Para crear un cuerpo
	primero debemos crear un objeto de tipo `BodyDef` con las propiedades del cuerpo
	a crear, como por ejemplo su posición en el mundo, su velocidad, o su tipo. Una vez hecho esto, 
	crearemos el cuerpo a partir del mundo (`World`) y 
	de la definición del cuerpo que acabamos de crear. Una vez creado el cuerpo, podremos asignarle
	una forma y densidad mediante _fixtures_. Por ejemplo, en el siguiente caso creamos un cuerpo dinámico con forma
	rectangular:
	
```cpp
b2BodyDef bodyDef;
bodyDef.type = b2_dynamicBody;	
bodyDef.position.Set(x / PTM_RATIO, y / PTM_RATIO);

b2Body *body = world->CreateBody(&bodyDef);
		
b2PolygonShape bodyShape;
bodyShape.SetAsBox((width/2) / PTM_RATIO, (height/2) / PTM_RATIO);

body->CreateFixture(&bodyShape, 1.0f);
```
	
	
Podemos también crear un cuerpo de forma circular con:
	
```cpp
b2BodyDef bodyDef;
bodyDef.type = b2_dynamicBody;
bodyDef.position.Set(x / PTM_RATIO, y / PTM_RATIO);
		
b2Body *body = world->CreateBody(&bodyDef);
		
b2CircleShape bodyShape;
bodyShape.m_radius = radius / PTM_RATIO;		

b2Fixture *bodyFixture = body->CreateFixture(&bodyShape, 1.0f);
```
	
También podemos crear los límites del escenario mediante cuerpos de tipo
	estático y con forma de arista (_edge_):
	
```cpp
b2BodyDef limitesBodyDef;
limitesBodyDef.position.Set(x, y);
				
b2Body *limitesBody = world->CreateBody(&limitesBodyDef);
b2EdgeShape limitesShape;
b2FixtureDef fixtureDef;
fixtureDef.shape = &limitesShape;

limitesShape.Set(b2Vec2(0.0f / PTM_RATIO, 0.0f / PTM_RATIO), 
                 b2Vec2(width / PTM_RATIO, 0.0f / PTM_RATIO));
limitesBody->CreateFixture(&fixtureDef);

limitesShape.Set(b2Vec2(width / PTM_RATIO, 0.0f / PTM_RATIO), 
                 b2Vec2(width / PTM_RATIO, height / PTM_RATIO));
limitesBody->CreateFixture(&fixtureDef);

limitesShape.Set(b2Vec2(width / PTM_RATIO, height / PTM_RATIO), 
                 b2Vec2(0.0f / PTM_RATIO, height / PTM_RATIO));
limitesBody->CreateFixture(&fixtureDef);

limitesShape.Set(b2Vec2(0.0f / PTM_RATIO, height / PTM_RATIO), 
                 b2Vec2(0.0f / PTM_RATIO, 0.0f / PTM_RATIO));
limitesBody->CreateFixture(&fixtureDef);
```
	
Los cuerpos tienen una propiedad `userData` que nos permite
	vincular cualquier objeto con el cuerpo. Por ejemplo, podríamos vincular a
	un cuerpo físico el `Sprite` que queremos utilizar para 
	mostrarlo en pantalla:
	
```cpp
bodyDef.userData = sprite;
```
	
De esta forma, cuando realicemos la simulación podemos obtener
	el _sprite_ vinculado al cuerpo físico y mostrarlo en pantalla
	en la posición que corresponda.
	
	
	
	
	## Simulación
	
Ya hemos visto cómo crear el mundo 2D y los cuerpos rígidos. Vamos a ver ahora cómo realizar
	la simulación física dentro de este mundo. Para realizar la simulación deberemos llamar al
	método `step` sobre el mundo, proporcionando el _delta time_ transcurrido
	desde la última actualización del mismo:
	
```cpp
world->Step(delta, 6, 2);
world->ClearForces();
```
	
Además, los algoritmos de simulación física son iterativos. Cuantas más iteraciones se realicen
	mayor precisión se obtendrá en los resultados, pero mayor coste tendrán. El segundo y el tercer
	parámetro de `step` nos permiten establecer el número de veces que debe iterar el algoritmo
	para resolver la posición y la velocidad de los cuerpos respectivamente. Tras hacer la simulación,
	deberemos limpiar las fuerzas acumuladas sobre los objetos, para que no se arrastren estos resultados
	a próximas simulaciones.
	
Tras hacer la simulación deberemos actualizar las posiciones de los _sprites_ en pantalla
	y mostrarlos. Por ejemplo, si hemos vinculado el `Sprite` al cuerpo mediante la propiedad
	`userData`, podemos recuperarlo y actualizarlo de la siguiente forma:
	
```cpp
CCSprite *sprite = (CCSprite *)body->GetUserData();
b2Vec2 pos = body->GetPosition();
CGFloat rot = -1 * CC_RADIANS_TO_DEGREES(b->GetAngle());

sprite->setPosition(ccp(pos.x*PTM_RATIO, pos.y*PTM_RATIO));
sprite->setRotation(rot);
```

	
	
	
	## Detección de colisiones
	
Hemos comentado que dentro de la simulación física existen interacciones entre los diferentes
	objetos del mundo. Podemos recibir notificaciones cada vez que se produzca un contacto entre objetos,
	para así por ejemplo aumentar el daño recibido.
	
Podremos recibir notificaciones mediante un objeto que implemente la interfaz `ContactListener`. 
	Esta interfaz nos forzará a definir los siguientes métodos:
	
```cpp
class MiContactListener : public b2ContactListener {

public:
    MiContactListener();
    ~MiContactListener();
    
    // Se produce un contacto entre dos cuerpos
	virtual void BeginContact(b2Contact* contact);

    // El contacto entre los cuerpos ha finalizado		
	virtual void EndContact(b2Contact* contact);

    // Se ejecuta antes de resolver el contacto. 
    // Podemos evitar que se procese	
	virtual void PreSolve(b2Contact* contact, 
	                      const b2Manifold* oldManifold);    

    // Podemos obtener el impulso aplicado sobre los cuerpos en contacto
	virtual void PostSolve(b2Contact* contact, 
	                       const b2ContactImpulse* impulse);    
};
```
	
	
	
Podemos obtener los cuerpos implicados en el contacto a partir del parámetro `Contact`.
	También podemos obtener información sobre los puntos de contacto mediante la información proporcionada
	por `WorldManifold`:
	
```cpp
void MiContactListener::BeginContact(b2Contact* contact) {
 
    b2Body *bodyA = contact.fixtureA->GetBody();
    b2Body *bodyB = contact.fixtureB->GetBody(); 
 		
    // Obtiene el punto de contacto
    b2WorldManifold worldManifold; 
    contact->GetWorldManifold(&worldManifold);
    
    b2Vec2 point = worldManifold.points[0];
		
    // Calcula la velocidad a la que se produce el impacto
    b2Vec2 vA = bodyA->GetLinearVelocityFromWorldPoint(point); 
    b2Vec2 vB = bodyB->GetLinearVelocityFromWorldPoint(point);

    float32 vel = b2Dot(vB - vA, worldManifold.normal);		
 
    ...
}
```
	
De esta forma, además de detectar colisiones podemos también saber la velocidad a la que han chocado,
	para así poder aplicar un diferente nivel de daño según la fuerza del impacto.
	
También podemos utilizar `postSolve` para obtener el impulso ejercido sobre los cuerpos
    en contacto en cada instante:

```cpp
void MiContactListener::PostSolve(b2Contact* contact, 
                                  const b2ContactImpulse* impulse) {
	
    b2Body *bodyA = contact.fixtureA->GetBody();
    b2Body *bodyB = contact.fixtureB->GetBody(); 
		
    float impulso = impulse->GetNormalImpulses()[0];
}
```

Debemos tener en cuenta que `BeginContact` sólo será llamado una vez, al comienzo del
    contacto, mientras que `PostSolve` nos informa en cada iteración de las fuerzas ejercidas
    entre los cuerpos en contacto.




## Gestión de físicas con PhysicsEditor

Hasta ahora hemos visto que es sencillo crear con Box 2D formas rectangulares y circulares, pero si
tenemos objetos más complejos la tarea se complicará notablemente. Tendremos que definir la forma
del objeto mediante un polígono, pero definir este polígono en código es una tarea altamente
tediosa.

Podemos hacer esto de forma bastante más sencilla con herramientas como **Physics Editor**.
Se trata de una aplicación de pago, pero podemos obtener de forma gratuita una versión limitada. La aplicación
puede descargarse de:

```
http://www.codeandweb.com/physicseditor
```

Con esta herramienta podremos abrir determinados _sprites_, y obtener de forma automática su
contorno. Cuenta con una herramienta similar a la "varita mágica" de Photoshop, con la que podremos
hacer que sea la propia aplicación la que determine el contorno de nuestros _sprites_. A
continuación vemos el entorno de la herramienta con el contorno que ha detectado automáticamente
para nuestro _sprite_:

![Entorno de Physics Editor](imagenes/pe/pe_entorno.jpg)

	

En el lateral derecho podemos seleccionar el formato en el que queremos exportar el contorno detectado. 
En nuestro caso utilizaremos el formato de Box 2D genérico (se exporta como `plist`). También
debemos especificar el _ratio_ de píxeles a metros que queremos utilizar en nuestra aplicación
(_PTM-Ratio_).

En dicho panel también podemos establecer una serie de propiedades de la forma (_fixture_) que estamos
definiendo (densidad, fricción, etc).

Una vez establecidos los datos anteriores podemos exportar el contorno del objeto pulsando el botón
_Publish_. Con esto generaremos un fichero `plist` que podremos importar desde nuestro
juego Cocos2D. Para ello necesitaremos añadir la clase `GB2ShapeCache` a nuestro proyecto. 
Esta clase viene incluida en el instalador de Physics Editor (tenemos tanto versión para Cocos2D como
para Cocos2D-X).

Para utilizar las formas definidas primero deberemos cargar el contenido del fichero `plist`
en la caché de formas mediante la clase anterior:

```cpp
#include "GB2ShapeCache-x.h"

...

GB2ShapeCache::sharedGB2ShapeCache()->addShapesWithFile("formas.plist");
```

Una vez cargadas las formas en la caché, podremos asignar las propiedades de las _fixtures_
definidas a nuestros objetos de Box2D:

```cpp
b2Body *body = ... // Inicializar body

GB2ShapeCache::sharedGB2ShapeCache()->addFixturesToBody(body, "roca");
```

<!-- 
> Es importante utilizar en este editor la versión básica de nuestros _sprites_
(no la versión HD), para así obtener las coordenadas de las formas en puntos. Al
tratarse las coordenadas como puntos, será suficiente con hacer una única versión de este fichero.
 -->
