Java Memory PermGen y ClassLoader
======================================

"java.lang.OutOfMemoryError: PermGen space failure". Esta excepción es debida a que la máquina virtual de Java(JVM) se ha quedado sin memoria PermGen.

¿Cómo se distribuye la memoria en la JVM?
+++++++++++++++++++++++++++++++++++++++++

En la JVM podemos diferenciar entre tres tipos de memorias:

Memoria de almacenamiento dinámico o memoria Heap: Esta zona de la memoria es la encargada de almacenar las instancias de los objetos creados por las aplicaciones.

Memoria de almacenamiento estático o memoria PermGen: En esta zona de la memoria se almacenan las clases que han sido cargadas por la aplicación. Esta zona también es utilizada para almacenar la información para la optimización de la aplicación por parte de la JVM.Los elementos que se almacenan en esta zona de la memoria son:
Métodos de las clases(incluido bytecode).
Nombre de las clases.
Pool de Constantes de la JVM [1].
Array de Objetos y array de tipos asociados con una clase (Ej: Array de objetos con referencia a métodos).
Objetos internos creados por la JVM (java.lang.Object or java.lang.Exception).
Información utilizada para la optimización de los compiladores.
Cadenas ‘internas’. Éstas se almacenan para la optimización en el acceso a las mismas.

Memoria dinámica nativa: Es la zona de la memoria encargada de almacenar el código de la Interfaz Nativa de Java (JNI) o la bibliotecas nativas de una aplicación y en la puesta en práctica de la JVM asignan memoria del almacenamiento dinámico nativo.

En nuestro casos nos referiremos a la Memoria PermGen que es la que genera la excepción anteriormente indicada.

¿Qué puede  causar que se produzca esta excepción?
++++++++++++++++++++++++++++++++++++++++++++++++++

Nuestra aplicación carga demasiadas clases: A la hora de cargar las clases, la JVM lo hace bajo demanda. En la práctica esto se transforma en dos posibles casos:
Cuando se crea un objeto relativo a una clase, es decir, llamada mediante el operador new.
Cuando accedemos a algún método o atributo estático de la clase.

Hay que decir que una vez que una clase es cargada en el ClassLoader no será descargada del mismo y a su vez se mantendrá una referencia desde el ClassLoader a la Clase y desde la clase al ClassLoader que la ha cargado. Todo esto será almacenado en la memoria PermGen, de tal forma que conforme vamos accediendo a las diferentes clases que son utilizadas por nuestra aplicación aumentará el tamaño ocupado.

NOTA: Este caso puede ser especialmente dado en los JSP, ya que cada uno de ellos será convertido en un fichero de una clase Servlet. Otros casos se pueden dar en algunos frameworks como Hibernate, EJB … Debido a que estos frameworks generan clases auxiliares para la optimización de los mismos.

Nuestra aplicación utiliza muchas cadenas ‘internas’: String.intern [2] es una optimización de la JVM para permitir optimizar la comparación entre cadenas. La comparación carácter a carácter entre cadenas es muy lenta, por ello se Mantiene un lista de cadenas instanciadas, de tal forma que permite una comparación de identidades que es más rápida.
El problema de esto es que la lista utilizada para la comparación de cadenas es establecida en la memoria PermGen y en algunos casos el tamaño de la misma puede ser demasiado elevado. Un ejemplo de este caso se da en algunos parses de XML, los cuales genera cadenas ‘internas’ muy grandes, por ello en algunos casos pueden provocar este tipo de excepción.
Document doc = SAXParser.new().parse( cadenaXML );

Fugas de Memoria(Memory Leaks): Esto se produce cuando re-desplegamos una aplicación. En los servidores de aplicaciones se suele cargar el ClassLoader de cada una de las aplicaciones que se cargan. Si cuando re-desplegamos la aplicación no se elimina de forma correcta el ClassLoader anterior, este se mantendrá en la memoria PermGen y se sumará al espacio ocupado por la nueva aplicación. Cuando re-desplegamos un par de veces la aplicación, provocará que agotemos la memoria PermGen. Este caso se encuentra perfectamente explicado aquí [3].

¿Cómo solucionarlo?
+++++++++++++++++++

Un primera opción sería modificar nuestro código para evitar el llegar a alguno de los tres posibles casos. Esta opción muchas veces no es válida debido a que dependemos de terceros o no podemos modificar el código de nuestra aplicación. La segunda opción sería modificar los parámetros de la JVM.

A continuación detallo un conjunto de parámetros que pueden ser indicados a la JVM para paliar, en la medida de lo posible, la aparición de esta excepción:

Aumentar el tamaño de la memoria PermGen. Ésta suele ser la primera medida a tomar, aunque no siempre resuelve los problemas sino que los aplaza y a veces puede derivar en problemas con el colector de basura o en problemas con la escasez de memoria dinámica.::

	-XX:MaxPermSize=128m

NOTA: Por defecto el tamaño de la memoria PermGen es de 64MB.

En el caso de multiprocesadores podemos establecer un colector de basura concurrente, el cual puede ‘eliminar’ memoria PermGen, cosa que el colector por defecto no puede. Para ello primero cambiamos el algoritmo del colector de basura::

	-XX:+UseConcMarkSweepGC

Habilitamos en la JVM la posibilidad de ‘eliminar’ memoria PermGen::

	-XX:+CMSPermGenSweepingEnabled

Las clases son un caso particular a la hora de gestionarlas en la memoria PermGen, por ello debemos indicarle a la JVM la posibilidad de permitir que descargue las clases::
	
	-XX:+CMSClassUnloadingEnabled
