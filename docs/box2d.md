# Motores de físicas

Un tipo de juegos que ha tenido una gran proliferación en el mercado de aplicaciones
para móviles son aquellos juegos basados en físicas. Estos juegos son aquellos en los que
el motor realiza una simulación física de los objetos en pantalla, siguiendo las leyes
de la cinemática y la dinámica. Es decir, los objetos de la pantalla están sujetos a
gravedad, cada uno de ellos tiene una masa, y cuando se produce una colisión entre ellos
se produce una fuerza de reacción que dependerá de su velocidad y su masa. El motor de físicas se
encarga de realizar toda esta simulación, y nosotros sólo deberemos encargarnos de
proporcionar las propiedades de los objetos del mundo. Uno de los motores físicos más
utilizados es Box2D, originalmente implementado en C++. Se ha utilizado para implementar juegos tan conocidos y exitosos
como Angry Birds. Podemos encontrar ports de este motor para las distintas
plataformas móviles. Motores como Cocos2D, libgdx y Unity incluyen una implementación de este
motor de físicas.


![Angry Birds, implementado con Box2D](imagenes/fisicas/box2d_angry.jpg)





## Motor de físicas Box2D

Vamos ahora a estudiar el motor de físicas Box2D. Es importante destacar que este motor sólo
	se encargará de simular la física de los objetos, no de dibujarlos. Será nuestra responsabilidad
	mostrar los objetos en la escena de forma adecuada según los datos obtenidos de la simulación física.
	Comenzaremos viendo los principales componentes de esta librería.


### Componentes de Box2D

Los componentes básicos que nos permiten realizar la simulación física con Box2D son:


* `Body`: Representa un cuerpo rígido. Estos son los tipos de objetos que tendremos en el
	mundo 2D simulado. Cada cuerpo tendrá una posición y velocidad. Los cuerpos se verán afectados por
	la gravedad del mundo, y por la interacción con los otros cuerpos. Cada cuerpo tendrá una serie de
	propiedades físicas, como su masa o su centro de gravedad.

* `Fixture`: Es el objeto que se encarga de fijar las propiedades de un cuerpo, como su forma, coeficiente de rozamiento o densidad. Un cuerpo podría contener varias _fixtures_, para así poder crear formas más complejas combinando formas básicas.

* `Shape`: Sirve para especificar la forma de una _fixture_. Hay distintos tipos de formas (subclases de
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


### Unidades de medida

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




### Tipos de cuerpos

Encontramos tres tipos diferentes de cuerpos en Box2D según la forma en la que queremos que se realice
	la simulación con ellos:


* **Dinámicos**: Están sometidos a las leyes físicas, y tienen una masa concreta y finita. Estos
	cuerpos se ven afectados por la gravedad y por la interacción con los demás cuerpos.
* **Estáticos**: Son cuerpos que permanecen siempre en la misma posición. Equivalen a cuerpos
	con masa infinita. Por ejemplo, podemos hacer que el escenario sea estático. Es importante no mover aquellos cuerpos que hayan sido marcados como estáticos, ya que el motor podría no responder de forma correcta.
* **Cinemáticos**: Al igual que los cuerpos estáticos tienen masa infinita y no se ven afectados
	por otros cuerpos ni por la gravedad. Sin embargo, en esta caso no tienen una posición fija, sino que podemos moverlos por el mundo. Nos son útiles por ejemplo para proyectiles.


![Tipos de cuerpos en Box2D](imagenes/fisicas/box2d_cuerpos.jpg)






### Creación de cuerpos

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

En este caso hemos creado un cuerpo con una única _fixture_ con forma de caja y densidad 1.0 $\frac{kg}{m^2}$. La masa del cuerpo sera calculada de forma automática a partir de la forma y densidad de sus _fixtures_.

De forma similar podemos también crear un cuerpo dinámico de forma circular con:

```cpp
b2BodyDef bodyDef;
bodyDef.type = b2_dynamicBody;
bodyDef.position.Set(x / PTM_RATIO, y / PTM_RATIO);

b2Body *body = world->CreateBody(&bodyDef);

b2CircleShape bodyShape;
bodyShape.m_radius = radius / PTM_RATIO;		

b2Fixture *bodyFixture = body->CreateFixture(&bodyShape, 1.0f);
```

Para definir los límites del escenario utilizaremos un cuerpo de tipo estático compuesto de varias _fixtures_ con forma de arista (_edge_). En este caso en lugar de utilizar el atajo `CreateFixture(shape, density)` de los ejemplos anteriores utilizaremos la versión `CreateFixture(fixtureDef)` que crea la _fixture_ a partir de las propiedades definidas en una estructura de tipo `b2FixtureDef`, lo cual nos dará mayor flexibilidad:

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

En este último caso vemos que no hemos indicado ni el tipo de cuerpo ni la masa. Si no indicamos nada por defecto el cuerpo será estático y su masa será infinita.

En la propiedad `type` de la estructura `b2BodyDef` podemos especificar de forma explícita el tipo de cuerpo que queremos crear, que puede ser:

* `b2_staticBody`: Cuerpo estático (valor por defecto). Podemos moverlos manualmente, pero el motor no los mueve.
* `b2_kinematicBody`: Cuerpo cinemática. Podemos darle una velocidad pero las fuerzas no tienen efecto sobre él.
* `b2_dinamicBody`: Cuerpo dinámico. Sometido totalmente a simulación física.

Los cuerpos tienen además una propiedad `userData` que nos permite vincular cualquier objeto con el cuerpo. Por ejemplo, podríamos vincular a un cuerpo físico el `Sprite` que queremos utilizar para mostrarlo en pantalla:

```cpp
bodyDef.userData = sprite;
```

De esta forma, cuando realicemos la simulación podemos obtener
	el _sprite_ vinculado al cuerpo físico y mostrarlo en pantalla
	en la posición que corresponda.




### Simulación

Ya hemos visto cómo crear el mundo 2D y los cuerpos rígidos. Vamos a ver ahora cómo realizar
	la simulación física dentro de este mundo. Para realizar la simulación deberemos llamar al
	método `step` sobre el mundo, proporcionando el _delta time_ transcurrido
	desde la última actualización del mismo:

```cpp
world->Step(delta, 6, 2);
world->ClearForces();
```

> **Recomendación**: Conviene utilizar un _delta time_ fijo para el motor de físicas, para así obtener resultados predecibles en la simulación (por ejemplo 60 fps). Si el _frame rate_ del _render_ es distinto podemos interpolar las posiciones.

Además, los algoritmos de simulación física son iterativos. Con cada iteración se busca resolver las colisiones y restricciones de los objetos del mundo para aproximar su posición y velocidad. Cuantas más iteraciones se realicen mayor precisión se obtendrá en los resultados, pero mayor coste tendrán. El segundo y el tercer	parámetro de `step` nos permiten establecer el número de veces que debe iterar el algoritmo	para resolver la posición y la velocidad de los cuerpos respectivamente. Tras hacer la simulación, deberemos limpiar las fuerzas acumuladas sobre los objetos, para que no se arrastren estos resultados a próximas simulaciones.

> **Recomendación**: Un valor recomendable para las iteraciones de posición y velocidad es 8 y 3 respectivamente.

Tras hacer la simulación deberemos actualizar las posiciones de los _sprites_ en pantalla y mostrarlos. Por ejemplo, si hemos vinculado el `Sprite` al cuerpo mediante la propiedad `userData`, podemos recuperarlo y actualizarlo de la siguiente forma:

```cpp
Sprite *sprite = (Sprite *)body->GetUserData();
b2Vec2 pos = body->GetPosition();
float rot = -1 * CC_RADIANS_TO_DEGREES(b->GetAngle());

sprite->setPosition(Vec2(pos.x*PTM_RATIO, pos.y*PTM_RATIO));
sprite->setRotation(rot);
```

### Formas de los objetos

Hemos visto que mediante _fixtures_ podemos asignar diferentes formas a los objetos del mundo, como círculos, polígonos y aristas.

#### Círculos

Es la forma más sencilla. Se crea simplemente indicando su centro y su radio, y el cálculo de colisiones con ellos es muy eficiente.

```cpp
b2CircleShape circle;
circle.m_p.Set(0.0f, 0.0f); // Centro
circle.m_radius = 0.5f;     // Radio
```

#### Polígonos

Nos permite crear formas arbitrarias convexas. Es importante destacar que los polígonos siempre serán convexos y cerrados, y sus vértices se definirán en sentido contrario a las agujas del reloj (CCW). El cálculo de colisiones con formas cóncavas es demasiado complejo para el motor de físicas.

```cpp
b2Vec2 vertices[kNUM_VERTICES];  // Vertices definidos en orden CCW
vertices[0].Set(-1.0f, 0.0f);
vertices[1].Set(1.0f, 0.0f);
vertices[2].Set(0.0f, 2.0f);

b2PolygonShape polygon;
polygon.Set(vertices, kNUM_VERTICES);
```

Un caso particular de los polígonos son las cajas. Al ser este tipo de polígonos muy común, se proporciona un método para crearlas de forma automática a partir de su media altura y anchura:

```cpp
b2PolygonShape box;
bodyShape.SetAsBox(0.5, 0.5); // Crea una caja de 1m x 1m
```

#### Aristas

Las aristas (_edges_) son segmentos de línea que normalmente se utilizan para construir la geometría del escenario estático, que podrá tener una forma arbitraria. Podemos

```cpp
b2Vec2 v1(0.0f, 0.0f); // Inicio del segmento
b2Vec2 v2(1.0f, 0.0f); // Fin del segmento

b2EdgeShape edge;
edge.Set(v1, v2);
```

#### Cadenas

Las cadenas nos permiten unir varias aristas para así definir la geometría estática del escenario y evitar que se puedan producir "baches" en las juntas entre diferentes aristas.

```cpp
b2Vec2 v[kNUM_VERTICES];
v[0].Set(0.0f, 0.0f);
v[1].Set(1.0f, 0.25f);
v[2].Set(2.0f, 1.0f);
v[3].Set(3.0f, 1.25f);

b2ChainShape chain;
chain.CreateChain(vs, kNUM_VERTICES);
```

> **Cuidado**: Las aristas de la cadena no deben intersectar entre si. Esto no está previsto por el motor, por lo que puede producir efectos inesperados.


#### Formas compuestas

Si ninguno de los tipos anteriores de formas se adapta a nuestras necesidades, como por ejemplo en el caso de necesitar una forma cóncava, podemos definir la forma del cuerpo como una composición de formás básicas. Esto lo podemos conseguir añadiendo múltiples _fixtures_ a un cuerpo, cada una de ellas con una forma distinta. Esto será útil para cuerpos dinámicos con formas complejas.


### Propiedades de los cuerpos

Los cuerpos y _fixtures_ tienen una serie de propiedades que nos permiten definir su comportamiento en la simulación física. Hemos visto algunas básicas como la masa y la forma, que se indican en el momento de crear una _fixture_. Vamos a ver ahora otras propiedades físicas de los objetos.

#### Resistencia al aire

Para cada cuerpo podemos indicar una constante de resistencia al aire (_damping_), tanto lineal como angular. La resistencia al aire es la fuerza que hará que la velocidad del objeto disminuya, aunque no esté en contacto con ningún otro cuerpo. Cuánta mayor sea la velocidad, más fuerza ejercerá la resistencia al aire para pararlo. Es recomendable indicar una resistencia al aire para que los cuerpos no se muevan (o roten) de forma indefinida:

```cpp
bodyDef.linearDamping = 0.1f;
bodyDef.angularDamping = 0.25f;
```

#### Fricción

La fricción es la fuerza que hace que un objeto se pare al deslizarse sobre otro, debido a rugosidades de la superficie. A diferencia de la resistencia al aire, esta fuerza sólo se ejercerá cuando dos _fixtures_ estén en contacto. La fricción se define a nivel de _fixture_:

```cpp
fixtureDef.friction = 0.25f;
```

#### Restitución

La restitución nos indica la forma en la que responderá un objeto al colisionar con otro, permitiendo que los objetos permanezcan juntos o reboten. Una restitución 0 indica que el objeto no rebotará el colisionar, mientras que el valor 1 indica que al colisionar rebota y en el rebote se restituye toda la velocidad que tenía en el momento previo a la colisión.

```cpp
fixtureDef.restitution = 0.5f;
```

### Dinámica de cuerpos rígidos

Vamos a ver con mayor detalle la forma en la que se aplican fuerzas e impulsos sobre los cuerpos del mundo.

#### Fuerza y masa

Siguiendo la segunda ley de Newton, la fuerza que se debe aplicar sobre un objeto para producir una determinada aceleración se calcula de la siguiente forma:

$$\mathbf{f} = m\mathbf{a}$$

Sin embargo, en nuestro motor de físicas lo que realmente nos interesa es conocer la aceleración producida tras aplicar una fuerza, calculada como:

$$\mathbf{a} = \frac{1}{m}\mathbf{f}$$

Podemos ver que aquí multiplicamos la fuerza por la **inversa de la masa**. Dado que este cálculo es frecuente, para evitar tener que calcular la inversa en cada momento, normalmente los motores almacenan la masa inversa de los cuerpos, en lugar de almacenar la masa.

Almacenar la masa inversa tiene una ventaja importante. Para hacer que un cuerpo sea estático (que no se vea afectado por las fuerzas que sobre él se ejerzan) lo que haremos es dar a ese cuerpo masa infinita. Este valor infinito podría crear dificultades en el código, y la necesidad de tratar casos especiales. Si trabajamos únicamente con masa inversa, bastará con darle un valor 0 a la masa inversa para hacer el cuerpo estático.

En el caso de Box2D en lugar de indicar la masa al crear una _fixture_ indicamos su densidad (medida en $\frac{kg}{m^2}$). En función del tamaño de la forma y de su densidad la librería calculará de forma automática la masa.

```cpp
fixtureDef.density = 1.0;
```

Podemos modificar las propiedades de masa de un cuerpo con el método `SetMassData`.

```cpp
b2MassData md;
md.mass = 2.0;
md.center = b2Vec(1.0, 0.0);
md.I = 1.0;

body.SetMassData(md);
```

De esta forma además de la masa podremos especificar el centro de masas y el momento de inercia. El momento de inercia nos permitirá indicar qué par de fuerzas (_torque_) deberemos ejercer para producir una determinada aceleración angular, de la misma forma que la masa nos indica qué fuerza debemos ejercer para producir una determinada aceleración lineal.

#### _Torque_ y momento de inercia

El _torque_ $\tau$ es a la aceleración angular $\alpha$ lo que la fuerza es a la aceleración lineal. En este caso, en lugar de tener en cuenta únicamente la masa del objeto, deberemos tener en cuenta su momento de inercia $I$, en el que no sólo tenemos la masa, sino cómo está repartida a lo largo del cuerpo, lo cual influirá en cómo las fuerzas afectarán a la rotación.

$$\tau = I \alpha$$

Por ejemplo, si tenemos un objeto con forma de bastón, habrá que hacer menos fuerza para que gire alrededor de su eje principal que alrededor de otro eje. Por lo tanto, el momento de inercia no tendrá siempre el mismo valor para un determinado objeto, sino que dependerá del eje de rotación.

El momento de inercia codifica cómo está repartida la masa del objeto alrededor de su centro. Para simplificar, supongamos que nuestro cuerpo rígido está compuesto de $n$ partículas cada una de ellas con una determinada masa $m_i$, y situada en una posición $(x_i, y_i)$ respecto al centro de masas del cuerpo. El momento de inercia se calcularía de la siguiente forma (medido en $kg·m$):

$$I = \sum^{n}_{i=1} m_i \sqrt{x_i^2 + y_i^2}$$

Es decir, este coeficiente no tiene en cuenta sólo la masa, sino también lo alejada que está la masa respecto del centro del centro. De esta forma, hará falta hacer más fuerza para girar un cuerpo cuando la distribución de masa esté alejada del centro.

> Box2D calculará de forma automática tanto el centro de masas como el momento de inercia a partir de la densidad, forma y posición de cada _fixture_ que compone un cuerpo, y normalmente no necesitaremos establecer estos datos de forma manual.

#### Acumulador de fuerzas

Normalmente sobre un cuerpo actuarán varias fuerzas. Siguiendo el principio de D'Alembert,  un conjunto de fuerzas

$$F=\{f_1, f_2, ... f_{|F|}\}$$

actuando sobre un objeto pueden ser sustituidas por una única fuerza calculada como la suma de las fuerzas de $F$:

$$f = \sum^{|F|}_{i=1} f_i$$

Para ello, cada objeto contará con un acumulador de fuerzas f donde se irán sumando todas las fuerzas que actúan sobre él (gravedad, interacción con otros objetos, suelo, etc). Cuando llegue el momento de realizar la actualización de posición y velocidad, la aceleración del objeto se calculará a partir de la fuerza que indique dicho acumulador $f$.

> **Poner a cero el acumulador.** Una vez finalizado un paso de la simulación deberemos poner a cero los acumuladores de fuerzas de cada objeto del mundo. Por este motivo Box2D tiene un método `clearForces` que deberemos llamar antes de realizar cada paso de la simulación.

Deberemos llevar cuidado con la discretización del tiempo. Si una gran fuerza se aplica durante un periodo de tiempo muy breve (por ejemplo para disparar una bala), si la aceleración producida se extiende a todo el _delta time_ el incremento de velocidad producido puede ser desmesurado. Por este motivo, estas fuerzas que se aplican en un breve instante puntual de tiempo se tratarán como **impulsos**.

#### Aplicación de fuerzas

El caso más común de fuerza aplicada a los objetos es la **gravedad**. Si queremos hacer una simulación realista deberíamos aplicar una fuerza que produzca una aceleración de

$$a_{gravedad}=-9.8 \frac{m}{s^2}$$

sobre nuestros objetos en el eje $y$ (normalmente se redondea en $a_{gravedad}=10$. Considerando el vector

$$\mathbf{a}_{gravedad} = (0, a_{gravedad})$$

tenemos:

$$\mathbf{f}_{gravedad} = \mathbf{a}_{gravedad}m$$

Los cuerpos de Box2D tienen una propiedad `gravityScale` que nos permite aplicar una gravedad distinta a cada cuerpo. Podemos especificarlo al crear el cuerpo:

```cpp
bodyDef.gravityScale = 5.0;
```

También se puede tratar como una fuerza la **"resistencia al aire"** (_damping_) que produce que los objetos vayan frenando y no se muevan indefinidamente. Un modelo simplificado para esta fuerza que se suele utilizar en videojuegos es el siguiente:

$$\mathbf{f}_{resistencia} = -\mathbf{\hat{v}}(k_{damping} |\mathbf{v}| $$

Donde $k_{damping}$ es la constante de _damping_ especificada para el cuerpo, y $\mathbf{\hat{v}}$ el vector de velocidad normalizado (vector unitario con la dirección de la velocidad). Podemos ver que la fuerza actúa en el sentido opuesto a la velocidad del objeto (lo frena), y con una magnitud proporcional a la velocidad.

A parte de las fuerzas de gravedad, resistencia al aire, y las fuerzas ejercidas entre cuerpos en contacto, también podemos aplicar una fuerza manualmente sobre un determinado cuerpo. Para ello deberemos indicar el vector de fuerza y el punto del objeto donde se aplicará dicha fuerza:

```cpp
body.ApplyForce(b2Vec(5.0, 2.0), body.GetPosition());
```

Las unidades en las que especificaremos la fuerza son _Newtons_ ($N = \frac{kg·m}{s^2}$).

Si el punto del objeto al que aplicamos la fuerza no es su centro de masas, la fuerza producirá además que el objeto rote (a no ser que en su definición hayamos dado valor `TRUE` a su propiedad `fixedRotation`, que evitará que rote).

Si nos interesa siempre aplicar la fuerza en el centro, podemos utilizar el método:

```cpp
body.ApplyForceToCenter(b2Vec(5.0, 2.0));
```

#### Aplicación de un par de fuerzas (_torque_)

Podemos también aplicar un par de fuerzas (_torque_) para producir una rotación del objeto alrededor de su centro de masas sin producir una traslación:

```cpp
body.ApplyTorque(2.0);
```
El _torque_ se indica en $N·m$.

#### Impulsos

Los impulsos producen un cambio instantáneo en la velocidad de un objeto. Podemos ver los impulsos respecto a la velocidad como vemos a las fuerzas respecto a la aceleración. Si aplicar una fuerza a un cuerpo produce una aceleración, aplicar un impulso produce un cambio de velocidad. Una diferencia importante es que no puede haber aceleración si no se aplica ninguna fuerza, mientras que si que puede haber velocidad si no se aplican impulsos, un impulso lo que provoca es un cambio en la velocidad. El impulso $g$ necesario para producir un cambio de velocidad $\Delta v$ será proporcional a la masa del objeto:

\begin{equation}
g = m\Delta v
\end{equation}

Al igual que en el caso de las fuerzas, el cálculo que nos interesará realizar es la obtención del cambio de velocidad a partir del impulso:

\begin{equation}
\Delta v = \frac{1}{m}g
\end{equation}

Considerando $\Delta v = v' - v$, donde $v$ es la velocidad previa a la aplicación del impulso, y $v'$ es la velocidad resultante, tenemos:

\begin{equation}
v' = v + \frac{1}{m}g
\end{equation}

#### Aplicación de impulsos

En Box2D podremos aplicar un impulso sobre un punto de un cuerpo (al igual que en el caso de la fuerza) con:

```cpp
body.ApplyImpulse(b2Vec(5.0, 2.0, body.GetPosition());
```

El impulso lineal se especifica en $N·s$. Podemos también aplicar un impulso angular con:

```
body.ApplyAngularImpulse(2.0);
```

Las unidades en este caso son $N·m·s$ (es decir, $kg\frac{m^2}{s}$).

#### Velocidad

Además de aplicar fuerzas e impulsos sobre los cuerpos, también podemos consultar o modificar su velocidad con `GetVelocity` y `SetVelocity`. En el caso de la velocidad trabajaremos con $\frac{m}{s}$.

Esto puede ser útil en cuerpos de tipo _kinematic_, en los que las fuerzas nos tienen efecto (al tener masa infinita), pero que si que pueden mantener una velocidad constante, como por ejemplo un proyectil.

```cpp
body.SetVelocity(b2Vec(5.0, 0.0));
```

De la misma forma, también podemos consultar y modificar la velocidad angular con `GetAngularVelocity` y `SetAngularVelocity` respectivamente. En estos casos las unidades son $\frac{radianes}{s}$.

#### Posición

Dado un cuerpo, cuyo centro de masas está posicionado en $\mathbf{p}_0$ y con rotación $\Theta$ (matriz de rotación), puede interesarnos determinar la posición de cualquier otro punto del objeto en el mundo. Supongamos que queremos conocer la posición de un punto cuyas coordenadas locales (respecto al centro de masas) son $\mathbf{p}_{local}$. La posición global de dicho punto vendrá determinada por:

$$\mathbf{p}_{global} = \Theta \mathbf{p}_{local} + \mathbf{p}_0$$

Para simplificar este cálculo, Box2D nos proporciona una serie de métodos con los que podemos convertir entre coordenadas locales del objeto y coordenadas globales del mundo, teniendo en cuenta la posición o orientación del objeto. Con `GetWorldPoint` podremos obtener las coordenadas globales a partir de la coordenadas locales del objeto, y con `GetLocalPoint` podremos hacer la transformación inversa.

```cpp
b2Vec globalPos = body->GetWorldPoint(b2Vec(0.0, 1.0));
```


### Detección de colisiones

Hemos comentado que dentro de la simulación física existen interacciones entre los diferentes objetos del mundo. Encontramos diferentes formas de consultar las colisiones de los objetos del mundo con otros objetos y otros elementos.

#### Colisión con un punto del mundo

Un _test_ sencillo consiste en comprobar si la forma de una _fixture_ ocupa un determinado punto del mundo. Esto es útil por ejemplo cuando tocamos sobre la pantalla táctil, para comprobar si en el punto sobre el que hemos pulsado hay un determinado objeto. Este método se aplica sobre una _fixture_ concreta:

```cpp
b2Transfrom transform;
transform.SetIdentity();
b2Vec2 point(touch_x / PTM_RATIO, touch_y / PTM_RATIO);

bool hit = fixture->TestPoint(transform, point);
```

#### Trazado de rayos

Otro _test_ disponible es el trazado de rayos. Consiste en lanzar un rayo desde una determinada posición del mundo en una determinada dirección y comprobar cuál es el primer objeto del mundo físico con el que impacta.

Esto es especialmente útil para implementar por ejemplo los disparos de nuestro personaje. Al ser la bala un objeto extremadamente rápido, no es conveniente simular su movimiento con el motor de físicas, ya que podría producirse el efecto conocido como _tunneling_, atravesando objetos al dar un gran salto en su posición de una iteración a la siguiente. En este caso es mejor simplemente considerar la bala como algo instantáneo, y encontrar en el mismo momento en que se dispara el objeto con el que impactaría lanzando un rayo.

Puede aplicarse para una _fixture_ concreta para saber si el rayo impacta con ella:

```cpp
b2RayCastInput input;
input.p1.Set(0.0f, 0.0f, 0.0f); // Punto inicial del rayo
input.p2.Set(1.0f, 0.0f, 0.0f); // Punto final del rayo
input.maxFraction = 1.0f;

b2RayCastOutput output;
bool hit = fixture->RayCast(&output, input, 0);
if (hit) {
    b2Vec2 hitPoint = input.p1 + output.fraction * (input.p2 – input.p1);
    b2Vec2 normal = output.normal;
}
```

Como salida tenemos la siguiente información del punto de impacto:

* _Fracción_: Tomando como referencia el vector desde el punto inicial al final del rayo, nos indica por cuánto debemos multiplicar dicho vector para encontrar el punto de impacto. Como entrada debemos especificar la fracción máxima hasta la que vamos a buscar el impacto. Por ejemplo, si la fracción es 1.0 el punto de impacto coincidirá con el punto final del rayo, mientras que si es 0.5 el impacto estaría justo a la mitad del vector del rayo.
* _Normal_: Nos indica la dirección normal de la superficie sobre la que ha impactado el rayo. De esta forma podremos saber si hemos impactado de lado o de frente, y así aplicar distinto nivel de daño en cada caso, o aplicar una fuerza al objeto en la dirección en la que haya recibido el impacto.

También podría aplicarse sobre el mundo, para buscar la primera _fixture_ con la que impacte. En este caso necesitaremos utilizar un objeto `b2RayCastCallback` para obtener la información del primer _fixture_ con el que impacte y los datos del impacto.


#### Colisiones entre cuerpos

Podemos recibir notificaciones cada vez que se produzca un contacto entre objetos del mundo, para así por ejemplo aumentar el daño recibido.

Podremos recibir notificaciones mediante un objeto que implemente la interfaz `ContactListener`. Esta interfaz nos forzará a definir los siguientes métodos:

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

El objeto _manifold_ nos da el conjunto de puntos que define el contacto. En el caso de la colisión de una esfera con una superficie siempre será un único punto, pero en el caso de una caja puede ocurrir que toda una cara de la caja colisione con la superficie. En este caso el _manifold_ nos devolerá los puntos de los extremos de la cara que colisiones con la superficie.

También podemos utilizar `PostSolve` para obtener el impulso ejercido sobre los cuerpos en contacto en cada instante:

```cpp
void MiContactListener::PostSolve(b2Contact* contact,
                                  const b2ContactImpulse* impulse) {

    b2Body *bodyA = contact.fixtureA->GetBody();
    b2Body *bodyB = contact.fixtureB->GetBody();

    float impulso = impulse->GetNormalImpulses()[0];
}
```

Debemos tener en cuenta que `BeginContact` sólo será llamado una vez, al comienzo del contacto, mientras que `PostSolve` nos informa en cada iteración de las fuerzas ejercidas entre los cuerpos en contacto para mantener uno en reposo sobre otro.


#### Sensores

En el punto anterior hemos visto cómo detectar colisiones entre cuerpos que producen una respuesta (fuerza de reacción). En algunos casos nos interesa que en el motor de físicas se detecten colisiones con un cuerpo, pero que no produzcan una respuesta en la simulación física. Por ejemplo, podríamos tener una zona en la que al entrar algún cuerpo queramos que se abra alguna puerta. Esto podemos conseguirlo mediante sensores. Podemos hacer que una _fixture_ se comporte como sensor mediante su propiedad `isSensor`:

```cpp
fixtureDef.isSensor = TRUE;
```

Al ser un sensor otros objetos atravesará esta _fixture_, pero podremos detectar las colisiones mediante los métodos `BeginContact` y `EndContact` de una `ContactListener`.

## Controlador de personaje

En un videojuego de plataformas debemos aplicar físicas y control de colisiones a nuestro personaje para así por ejemplo controlar el salto y las caídas, y evitar que pueda atravesar muros o el suelo. Este control suele hacerse con físicas simplificadas que aplican una gravedad al personaje y comprueban si entra en colisión con el suelo (parte inferior) o con muros (laterales). Sin embargo, podría interesarnos delegar todo este control a un motor de físicas como Box2D. Vamos a ver cómo podríamos utilizar este motor para implementar un controlador de personaje.

En primer lugar deberemos crear un cuerpo físico para nuestro personaje y su geometría de colisión. Podemos para ello utilizar una caja con las dimensiones de su nodo gráfico (`m_playerSprite`). Es importante bloquear la rotación del cuerpo físico, ya que normalmente buscaremos que nuestro personaje esté siempre _de pie_:

```cpp
b2BodyDef bodyDef;
bodyDef.type = b2BodyType::b2_dynamicBody;
bodyDef.fixedRotation = true;

b2PolygonShape shapeBoundingBox;
shapeBoundingBox.SetAsBox(m_playerSprite->getContentSize().width / PTM_RATIO,
                          m_playerSprite->getContentSize().height / PTM_RATIO);

m_body = world->CreateBody(&bodyDef);
b2Fixture *fixture = m_body->CreateFixture(&shapeBoundingBox, 1.0);
fixture->SetFriction(0.0f);
```

Además de crear la geometría de colisión del _sprite_, es importante saber cuándo estamos pisando suelo y cuándo estamos en el aire, para determinar así si podemos saltar o no. Para ello podemos utilizar un sensor añadido bajo los pies del personaje:

```cpp
b2CircleShape shapeSensor;
shapeSensor.m_radius = GROUND_TEST_RADIUS;
shapeSensor.m_p = b2Vec2(0, -m_playerSprite->getContentSize().height * 0.5 / PTM_RATIO);

m_groundTest = m_body->CreateFixture(&shapeSensor, 1.0);
m_groundTest->SetSensor(true);
```

Al marcar esta forma circular como _sensor_, no causará reacción de colisión con el suelo pero si que detectará cuando está solapado con él. De esta forma podremos saber si estamos sobre una superficie o en el aire. Podemos comprobar las colisiones de nuestro cuerpo con el siguiente código:

```cpp
bool checkGrounded() {
    b2ContactEdge *edge = m_body->GetContactList();
    while ( edge != NULL ) {
        if ( edge->contact->GetFixtureA()==m_groundTest ||
             edge->contact->GetFixtureB()==m_groundTest) {
            return true;
        }
        edge = edge->next;
    }
    return false;
}
```

Con este método obtenemos todos los contactos existentes con el cuerpo de nuestro personaje, y filtramos sólo aquellos que se producen con el sensor (`m_groundTest`). En caso de existir alguno, es que estamos pisando sobre alguna superficie.

Podríamos implementar el salto aplicando una velocidad vertical (`m_jump`) y conservando la velocidad horizontal del personaje. Haremos esto sólo cuando el personaje esté sobre una superficie sólida:

```cpp
if(checkGrounded()) {
    m_body->SetLinearVelocity(b2Vec2(m_body->GetLinearVelocity().x, m_jump));
}
```

Para mover el personaje a izquierda o derecha lo único que deberemos hacer es establecer su velocidad en _x_ a partir del valor del eje horizontal de mando, conservando su velocidad vertical (determinada por la fuerza de la gravedad):

```cpp
m_body->SetLinearVelocity(b2Vec2(m_horizontalAxis * m_vel / PTM_RATIO,
                                 m_body->GetLinearVelocity().y));
```

Con esto ya tendremos nuestro _sprite_ en movimiento utilizando el motor de físicas. Ya sólo quedaría controlar las animaciones de nuestro personaje, aunque esto ya no es responsabilidad del motor de físicas. Por ejemplo, deberemos hacer que mire en la dirección en la que estemos moviendo el mando:

```cpp
if(m_horizontalAxis < 0 && !m_playerSprite->isFlippedX()) {
    m_playerSprite->setFlippedX(true);
} else if(m_horizontalAxis > 0 && m_playerSprite->isFlippedX()) {
    m_playerSprite->setFlippedX(false);
}
```

Si queremos controlar la velocidad de la animación de fotogramas en función de la velocidad a la que estemos moviendo el personaje podemos añadir una acción de tipo `Speed`:

```cpp
Animate* actionAnimate = Animate::create(AnimationCache::getInstance()->getAnimation("animAndar"));
RepeatForever* actionRepeat = RepeatForever::create(actionAnimate);
m_actionAndar = Speed::create(actionRepeat, 1.0f);
```

Con esta acción podremos controlar la velocidad a la que se reproduce la animación en cada momento con:

```cpp
if(fabsf(m_horizontalAxis) < 0.1) {
    // Paramos al personaje
    m_actionAndar->setSpeed(0.0f);
    m_playerSprite->setSpriteFrame("idle.png");
} else {
    // Establecemos la velocidad de la animación
    m_actionAndar->setSpeed(fabsf(m_horizontalAxis));
}
```

## Depuración de las físicas

Dado que los cuerpos físicos se tratan de forma independiente de los nodos gráficos, a veces resulta conveniente poder visualizar qué ocurre en el mundo físico para poder depurar de forma correcta el comportamiento de las entidades de nuestro juego.

El motor Box2D nos ofrece facilidades para hacer esto. La clase `b2World` ofrece la funcionalidad de "dibujar" los objetos del mundo físico, como las formas de las _fixtures_, los AABB, o los centros de masas de los objetos.

Para poder dibujar estos contenidos deberemos proporcionar a Box2D una clase que le indique cómo dibujar cada elemento con nuestro motor gráfico (en nuestro caso Cocos2d-x). Esta clase deberá heredar de `b2Draw`, y deberá implementar una serie de métodos en los que indicaremos cómo dibujar diferentes primitivas gráficas (círculos, rectángulos, polilíneas, etc). Afortunadamente, Cocos2d-x cuenta con el nodo de tipo `DrawNode` que nos facilitará dibujar dichas primitivas. A continuación mostrarmos un ejemplo de implementación de una clase que nos permita depurar la física de Box2D en Cocos2d-x:

```cpp
class CocosDebugDraw : public b2Draw
{
    float32 mRatio;
    cocos2d::DrawNode *mNode;

public:
    CocosDebugDraw();
    CocosDebugDraw( float32 ratio );
    ~CocosDebugDraw();

    cocos2d::Node* GetNode();
    void Clear();

    virtual void DrawPolygon(const b2Vec2* vertices, int vertexCount, const b2Color& color);
    virtual void DrawSolidPolygon(const b2Vec2* vertices, int vertexCount, const b2Color& color);
    virtual void DrawCircle(const b2Vec2& center, float32 radius, const b2Color& color);
    virtual void DrawSolidCircle(const b2Vec2& center, float32 radius, const b2Vec2& axis, const b2Color& color);
    virtual void DrawSegment(const b2Vec2& p1, const b2Vec2& p2, const b2Color& color);
    virtual void DrawTransform(const b2Transform& xf);
    virtual void DrawPoint(const b2Vec2& p, float32 size, const b2Color& color);
    virtual void DrawString(int x, int y, const char* string, ...);
    virtual void DrawAABB(b2AABB* aabb, const b2Color& color);
};
```

```cpp
CocosDebugDraw::CocosDebugDraw()
: mRatio( 1.0f )
{
    mNode = DrawNode::create();
    mNode->retain();
}

CocosDebugDraw::CocosDebugDraw( float32 ratio )
: mRatio( ratio )
{
    mNode = DrawNode::create();
    mNode->retain();
}

CocosDebugDraw::~CocosDebugDraw()
{
    mNode->release();
}

void CocosDebugDraw::Clear()
{
    mNode->clear();
}

Node* CocosDebugDraw::GetNode() {
    return mNode;
}

void CocosDebugDraw::DrawPolygon(const b2Vec2* old_vertices, int vertexCount, const b2Color& color)
{

    Vec2 *vertices = new Vec2[vertexCount];
    for( int i=0;i<vertexCount;i++)
    {
        vertices[i] = Vec2(old_vertices[i].x * mRatio, old_vertices[i].y * mRatio);
    }

    mNode->drawPoly(vertices, vertexCount, false, Color4F(color.r, color.g, color.b, 1.0f));

    delete[] vertices;
}

void CocosDebugDraw::DrawSolidPolygon(const b2Vec2* old_vertices, int vertexCount, const b2Color& color)
{

    Vec2 *vertices = new Vec2[vertexCount];
    for( int i=0;i<vertexCount;i++)
    {
        vertices[i] = Vec2(old_vertices[i].x * mRatio, old_vertices[i].y * mRatio);
    }

    mNode->drawSolidPoly(vertices, vertexCount, Color4F(color.r, color.g, color.b, 1.0f));

    delete[] vertices;
}

void CocosDebugDraw::DrawCircle(const b2Vec2& center, float32 radius, const b2Color& color)
{
    mNode->drawCircle(Vec2(center.x * mRatio, center.y * mRatio), radius * mRatio, 0.0f, 16.0f, false, 1.0f, 1.0f, Color4F(color.r, color.g, color.b, 1.0f));
}

void CocosDebugDraw::DrawSolidCircle(const b2Vec2& center, float32 radius, const b2Vec2& axis, const b2Color& color)
{
    mNode->drawSolidCircle(Vec2(center.x * mRatio, center.y * mRatio), radius * mRatio, 0.0f, 16.0f, 1.0f, 1.0f, Color4F(color.r, color.g, color.b, 1.0f));
}

void CocosDebugDraw::DrawSegment(const b2Vec2& p1, const b2Vec2& p2, const b2Color& color)
{
    mNode->drawSegment(Vec2(p1.x * mRatio, p1.y * mRatio), Vec2(p2.x * mRatio , p2.y * mRatio), 1.0f, Color4F(color.r, color.g, color.b, 1.0f));
}

void CocosDebugDraw::DrawTransform(const b2Transform& xf)
{
    b2Vec2 p1 = xf.p, p2;
    const float32 k_axisScale = 0.4f;
    p2 = p1 + k_axisScale * xf.q.GetXAxis();
    DrawSegment(p1, p2, b2Color(1,0,0));

    p2 = p1 + k_axisScale * xf.q.GetYAxis();
    DrawSegment(p1,p2,b2Color(0,1,0));
}

void CocosDebugDraw::DrawPoint(const b2Vec2& p, float32 size, const b2Color& color)
{
    mNode->drawPoint(Vec2(p.x * mRatio, p.y * mRatio), size * mRatio, Color4F(color.r, color.g, color.b, 1.0f));
}

void CocosDebugDraw::DrawString(int x, int y, const char *string, ...)
{
    // No soportado
}

void CocosDebugDraw::DrawAABB(b2AABB* aabb, const b2Color& color)
{
    mNode->drawRect(Vec2(aabb->lowerBound.x * mRatio, aabb->lowerBound.y * mRatio), Vec2(aabb->upperBound.x * mRatio, aabb->upperBound.y * mRatio), Color4F(color.r, color.g, color.b, 1.0f));
}
```

Una vez definida dicha clase, la añadiremos al mundo físico (`b2World`):

```cpp
m_world = new b2World(b2Vec2(0,-10));

m_debugDraw = new CocosDebugDraw(PTM_RATIO);
m_world->SetDebugDraw(m_debugDraw);
```

Será importante tras esto indicar qué tipos de elementos del motor físico queremos que se muestre en la capa de depuración:

```cpp
uint32 flags = 0;
flags += b2Draw::e_shapeBit;
flags += b2Draw::e_jointBit;
flags += b2Draw::e_aabbBit;
flags += b2Draw::e_pairBit;
flags += b2Draw::e_centerOfMassBit;

m_debugDraw->SetFlags(flags);
```

Tras esto, añadiremos el nodo de depuración a nuestra escena. Haremos que quede por delante del resto de capas:

```cpp
m_node->addChild(m_debugDraw->GetNode(),9999);
```

Lo último que deberemos hacer es que en cada iteración, tras actualizar el estado del mundo físico, redibujaremos la capa de depuración:

```cpp
void Mundo::update(float delta){
    m_world->ClearForces();
    m_world->Step(1.0f/60.0f, 6, 2);

    m_debugDraw->Clear();
    m_world->DrawDebugData();

    // ...        
}
```

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

## Referencias

* [(Gamasutra) How to create 2D Physics Games with Box2D library](http://www.gamasutra.com/blogs/JuanBelonPerez/20130826/198897/How_to_create_2D_Physics_Games_with_Box2D_Library.php)
