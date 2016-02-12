# Desarrollo de juegos con SpriteKit

**SpriteKit** es un _framework_ nativo de la plataforma iOS destinado al desarrollo de aplicaciones que muestren cualquier tipo de gráficos 2D animados, como es el caso de los videojuegos. Todas las clases de este _framework_ tienen el prefijo `SK`, y se encuentra disponible a partir de iOS 7.

Su API es muy parecida a la del motor de videojuegos Cocos2d-x, por lo que la transición entre estas tecnologías será muy sencilla. 

## La escena de SpriteKit

Al igual que en dicho motor, utiliza una organización **jerárquica** de los elementos **escena**, es decir, todos los elementos de la escena son **nodos** que se organizan en forma de de árbol. 

Tendremos una escena para cada estado (pantalla) del juego (menú principal, opciones, partida, etc). Para crear una escena normalmente crearemos una clase que herede de `SKScene`, por ejemplo:

```objc
#import <SpriteKit/SpriteKit.h>

@interface MainMenuScene : SKScene

@end
```

Para poder mostrar esa escena necesitaremos previamente contar con una vista de tipo `SKView`. Por ejemplo, podemos introducir una vista de este tipo dentro de nuestro controlador en el _storyboard_. Al cargar nuestro controlador podemos obtener dicha vista y cargar en ella la escena principal:

```objc
- (void)viewDidLoad
{
    [super viewDidLoad];

    // Obtenemos la vista del juego (suponemos que es la vista raíz de nuestro controlador).
    SKView * skView = (SKView *)self.view;

    // Creamos la escena.
    MainMenuScene *scene = [MainMenuScene nodeWithFileNamed:@"MainMenuScene"];

    // Mostramos la escena en la vista.
    [skView presentScene:scene];
}
```

Hemos de destacar que el constructor de la escena toma como parámetro el nombre de un fichero. El fichero que deberemos pasarle es un fichero de escena, con extensión `.sks` (_Scene Kit Scene_) que puede contener los elementos de la escena (_sprites_, etiquetas de texto, cámara, luces, etc). 

Podemos crear este fichero desde Xcode, con _File > New > File ... > iOS > Resource > SpriteKit Scene_:


Podemos dejar esta escena vacía y crear su contenido de forma programada. Vamos a hacerlo así de momento.

La clase escena tiene un método `-(void)didMoveToView:(SKView *)view` que se invocará cuando la escena se pase a ejecutar en la vista del juego. Podemos aprovechar este método para crear de forma programada el contenido de la escena. Este contenido se definirá como un árbol de nodos (objetos de tipo `SKNode` o de alguna de sus subclases).

## Nodos de la escena

La escena de SpriteKit se define como un árbol de nodos, todos ellos de tipo `SKNode` o alguna de sus subclases. Destacamos los siguiente tipos de nodos:

* `SKNode`: Clase de la que heredan todos los nodos. Podemos instanciarla directamente y añadir el nodo a la escena. Este tipo de nodos no mostrarán nada en la escena, pero son útiles para agrupar otros nodos en el árbol de la escena y poder moverlos de forma conjunta.
* `SKLabelNode`: Subclase de `SKNode` que mostrará una etiqueta de texto en la escena.
* `SKSpriteNode`: Subclase de `SKNode` que mostrará un _sprite_ en la escena. 
* 

Los nodos genéricos se instancian mediante el método factoría `node`:

```objc
SKNode *miNodo = [SKNode node];
```

Podremos cambiar la posición (x,y) de todos los nodos en la escena, así como su orden z, que determinará qué nodos se muestran por delante de otros:

```objc
miNodo.position = CGPointMake(100, 100);
miNodo.zPosition = 5;
```

También podremos cambiar su rotación y su escala (x,y):

```objc
miNodo.zRotation = 90;
miNodo.xScale = -1;
miNodo.yScale = 1;
```

> Es de especial interés la posibilidad de hacer un escalado de -1, para así crear un efecto "espejo".

Además de estas propiedades, otra característica importante de los nodos es su método `addChild:`, que nos permite añadir otro nodo como hijo:

```objc
SKNode *miGrupo = [SKNode node];
[miGrupo addChild: miNodo];
```

> El agrupamiento jerárquico de nodos es importante porque nos permitirá por ejemplo mover de forma conjunta todo un grupo de nodos cambiando únicamente la posición del nodo padre.

Encontraremos también métodos y propiedades para conocer quién es el padre de un nodo, eliminar un hijo de un nodo, o moverlo a otro nodo padre, entre otras funciones.

### Etiqueta de texto

Un tipo de nodo fundamental es la etiqueta de texto (`SKLabelNode`) que nos permitirá mostrar texto en la escena. Este tipo de nodo se puede crear a partir de la fuente a utilizar:

```objc
SKLabelNode *miEtiqueta = [SKLabelNode labelNodeWithFontNamed:@"Chalkduster"];
```

A parte de las propiedades generales de los nodos, la propiedad más importante de este tipo de nodo es `text`, que nos permite especificar el texto a mostrar por la etiqueta:

```objc
miEtiqueta.text = @"Super Mobile Game";
```
