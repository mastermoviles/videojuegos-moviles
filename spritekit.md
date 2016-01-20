# Desarrollo de juegos con SpriteKit

**SpriteKit** es un _framework_ nativo de la plataforma iOS destinado al desarrollo de aplicaciones que muestren cualquier tipo de gráficos 2D animados, como es el caso de los videojuegos. Todas las clases de este _framework_ tienen el prefijo `SK`.

Su API es muy parecida a la del motor de videojuegos Cocos2d-x, por lo que la transición entre estas tecnologías será muy sencilla. 

## La escena de SpriteKit

Al igual que en dicho motor, utiliza una organización **jerárquica** de los elementos **escena**, es decir, todos los elementos de la escena son **nodos** que se organizan en forma de de árbol. 

Para crear una escena normalmente crearemos una clase que herede de `SKScene`, pero para poder mostrar esa escena necesitaremos previamente contar con una vista de tipo `SKView`. Por ejemplo, podemos introducir una vista de este tipo dentro de nuestro controlador en el _storyboard_.

