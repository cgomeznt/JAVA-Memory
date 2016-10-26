Find out your Java heap memory size
=====================================

Mostraremos como se usa el -XX:+PrintFlagsFinal para encontrar el tamaño de la heap y de la permgen, hay un algoritmo para tener las mejores practicas a la hora de colocar los tamaños de memoria.

Heap sizes
Initial heap size of 1/64 of physical memory up to 1Gbyte
Maximum heap size of 1/4 of physical memory up to 1Gbyte

1. Java Heap Size
+++++++++++++++++

Es el lugar donde se almacena los objetos creados por una aplicacion java, este es donde el minor Garbage Collection actua, para liberar memoria usada por la aplicacion java. En procesos java muy pesados, tener una insuficiente memoria Heap, causa "java.lang.OutOfMemoryError: Java heap space.".::


	-Xms<size> - Set initial Java heap size
	-Xmx<size> - Set maximum Java heap size

	$ java -Xms512m -Xmx1024m JavaApp

	$ java -XX:+PrintFlagsFinal -Xms64m -Xmx1024m -version | grep -i heap


2. Perm Gen Size
++++++++++++++++++

Es el lugar donde se almacena las clases cargadas, las definiciones de metadata. Si un proyecto muy grande de codigo base es cargado y tenemos insuficiente memoria PermGen, causa "Java.Lang.OutOfMemoryError: PermGen.".::

	-XX:PermSize<size> - Set initial PermGen Size.
	-XX:MaxPermSize<size> - Set the maximum PermGen Size.

	$ java -XX:PermSize=64m -XX:MaxPermSize=128m JavaApp


3. Java Stack Size
+++++++++++++++++++

Tamaño de Java Thread. Si un proyecto tiene muchos threads, Se intenta reducir el tamaño de este stack para permitir correr fuera de memoria.::

	-Xss = set java thread stack size

	$ java -Xss512k JavaApp


::

	Nota: El valor del tamaño por defecto de for heap size, perm gen, o stack, es diferente por JVMs. Las mejores practicas es definirlas.


Este es el ambiente de prueba.::

	Distributor ID:	Debian
	Description:	Debian GNU/Linux 8.0 (jessie)
	Release:	8.0
	Codename:	jessie
	Memory:		8G
	CPU:		Intel(R) Core(TM) i5 CPU         650  @ 3.20GHz
	JDK:		build 1.8.0_60-b27 y OpenJDK 1.7.0_75

.::

	$ java -XX:+PrintFlagsFinal -version | grep -iE 'HeapSize|PermSize|ThreadStackSize'
		 intx CompilerThreadStackSize                   = 0                                   {pd product}
		uintx ErgoHeapSizeLimit                         = 0                                   {product}
		uintx HeapSizePerGCThread                       = 87241520                            {product}
		uintx InitialHeapSize                          := 130023424                           {product}
		uintx LargePageHeapSizeThreshold                = 134217728                           {product}
		uintx MaxHeapSize                              := 2080374784                          {product}
		 intx ThreadStackSize                           = 1024                                {pd product}
		 intx VMThreadStackSize                         = 1024                                {pd product}
	java version "1.8.0_60"
	Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
	Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)

En el ambiente de arriba, el JVM esta localizado con los valores por defecto con el Java de Oracle.::

	Java heap size
	InitialHeapSize = 130023424 bytes (124M) and MaxHeapSize = 2080374784 bytes (1984M).
	Thread Stack Size
	ThreadStackSize = 1024 kilobytes (1M)

NOTA: EL soporte de PermSize fue removido de la version 1.8.

.::

	$ java -XX:+PrintFlagsFinal -version | grep -iE 'HeapSize|PermSize|ThreadStackSize'
		uintx AdaptivePermSizeWeight                    = 20              {product}           
		 intx CompilerThreadStackSize                   = 0               {pd product}        
		uintx ErgoHeapSizeLimit                         = 0               {product}           
		uintx HeapSizePerGCThread                       = 87241520        {product}           
		uintx InitialHeapSize                          := 129969600       {product}           
		uintx LargePageHeapSizeThreshold                = 134217728       {product}           
		uintx MaxHeapSize                              := 2080374784      {product}           
		uintx MaxPermSize                               = 174063616       {pd product}        
		uintx PermSize                                  = 21757952        {pd product}        
		 intx ThreadStackSize                           = 1024            {pd product}        
		 intx VMThreadStackSize                         = 1024            {pd product}        
	java version "1.7.0_75"
	OpenJDK Runtime Environment (IcedTea 2.5.4) (7u75-2.5.4-2)
	OpenJDK 64-Bit Server VM (build 24.75-b04, mixed mode)

En el ambiente de arriba, el JVM esta localizado con los valores por defecto con el OpenJDK.::
	
	Java heap size
	InitialHeapSize = 130023424 bytes (124M) and MaxHeapSize = 2080374784 bytes (1984M).
	PermGen Size
	PermSize = 21757952 bytes (20.75M), MaxPermSize = 174063616 bytes (166M)
	Thread Stack Size
	ThreadStackSize = 1024 kilobytes (1M)

Los tamaños de la memoria heap estan cercanos a los valores del algoritmo ergonomico.::

	#ergonomics algorithm
	Initial heap size = 7932M/64 = 124M
	Maximum heap size = 7932M/4 = 1984M











