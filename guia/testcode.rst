Normalmente, una JVM (Java Virtual Machine) utiliza estas reglas para la gestión de memoria:

Cuando se invoca una JVM para ejecutar una aplicación, pedirá al sistema operativo suficiente memoria para ejecutar la JVM propiamente dichas y algo de memoria libre para que la aplicación pueda crear nuevos objetos.
Cuando se crea un nuevo objeto, la JVM reservará memoria para ese objeto dentro del área de memoria libre.
Cuando el área de memoria libre se vuelve demasiado pequeño, la JVM pedirá al sistema operativo más.
Cuando un objeto no se utiliza más, será destruido. Su memoria será liberada y devuelta al área de memoria libre.
Cuando se utiliza todo el área de memoria libre, y no hay más memoria disponible del sistema operativo, la JVM detendrá la aplicación y provocará un “Out of memory error”.
Prueba de fallo de memoria

Podemos utilizar el siguiente programa para pedir memoria a la JVM en un bucle:


package jvm.memory;

/**
* Prueba de gestion de memoria.
*
* @author rubensa
* @version 1.1 (rubensa)
*
*/
public class MemoryCrash {

/**
* Realiza reservas de memoria, de 10MB en 10MB, en un bucle;
* y va mostrando el estado de la memoria de la JVM.
*
* @param args parametros de linea de comandos (no usado)
*/
public static void main(String[] args) {
int max = 100;
Runtime rt = Runtime.getRuntime();
Object[] objects = new Object[max];
for (int i=0; i<max; i++) {
System.out.println("Tenemos " + i*10 + "MB y añadimos 10MB");
objects[i] = new long[10*1024*128];
System.out.println(" Memoria libre: " + rt.freeMemory());
System.out.println(" Memoria total: " + rt.totalMemory());
try {
Thread.sleep(2*1000);
} catch (InterruptedException ie) {
System.out.println("Interrumpido.");
}
}
}

}

Salida de la JRE 1.4.2_11 con los parámetros -Xms2m -Xmx64m


Tenemos 0MB y añadimos 10MB
Memoria libre: 1949472
Memoria total: 12521472
Tenemos 10MB y añadimos 10MB
Memoria libre: 8429240
Memoria total: 29487104
Tenemos 20MB y añadimos 10MB
Memoria libre: 6241888
Memoria total: 37785600
Tenemos 30MB y añadimos 10MB
Memoria libre: 14540720
Memoria total: 56569856
Tenemos 40MB y añadimos 10MB
Memoria libre: 4054176
Memoria total: 56569856
Tenemos 50MB y añadimos 10MB
Exception in thread "main" java.lang.OutOfMemoryError

Fíjate que:

El array de 10MB es ligeramente mayor de 10MB a causa de la sobrecarga del propio array.
Probando con -Xmx512m, en un equipo con 128MB de memoria física y 288MB de tamaño del archivo de intercambio se reservan 200MB antes del “out of memory”.
Prueba del recolector de basura

Podemos utilizar el siguiente programa para comprobar cómo funciona el proceso de recolección de basura:

package jvm.memory;

/**
* Prueba de recoleccin de basura.
*
* @author rubensa
* @version 1.1 (rubensa)
*
*/
public class GarbageCollection {

/**
* Realiza reservas de memoria, en bloques de 1MB, guardando la
* referencia a 32 bloques y reservando un máximo de 10.000;
* y va mostrando el estado de la memoria de la JVM.
*
* @param args parametros de linea de comandos (no usado)
*/
public static void main(String[] args) {
int max = 10000;
int min = 32;
Object[] arr = new Object[min];
Runtime rt = Runtime.getRuntime();
System.out.println("Memoria libre/total:");
for (int i=0; i<max; i++) {
// para añadir al principio
// desplazamos los bloques existentes
// descartando el último
for (int j=0; j<min-1; j++)
arr[min-j-1] = arr[min-j-2];
// añadimos al principio
arr[0] = getOneMega();
System.out.println(rt.freeMemory()+" "+rt.totalMemory());
try {
Thread.sleep(1000/10);
} catch (InterruptedException e) {
System.out.println("Interrumpido.");
}
}
}

/**
* Devuelve un array varios objetos que ocupan, en conjunto, 1MB.
*
* @return
*/
private static Object getOneMega() {
// return new long[1024*128]; // 1MB
Object[] lst = new Object[10];
lst[0] = new long[256 * 128];
lst[1] = new int[256 * 256];
lst[2] = new double[256 * 128];
lst[3] = new float[64 * 256];
lst[4] = new byte[64 * 1024];
String[] l = new String[64 * 64];
for (int i = 0; i < 64 * 64; i++)
l[i] = new String("12345678"); // 16B
lst[5] = l;
lst[6] = new char[64 * 512];
return lst;
}
}


Memoria libre/total.
..
27515784 64094208
26417624 64094208
25319464 64094208
24222096 64094208
23123936 64094208
22025776 64094208
20927616 64094208
19830512 64094208
18732352 64094208
17634192 64094208
16536032 64094208
15438928 64094208
14340768 64094208
13242608 64094208
12145240 64094208
11047080 64094208
9948920 64094208
8850760 64094208
7753656 64094208
6655496 64094208
5557336 64094208
4459968 64094208
3361808 64094208
2263648 64094208
1165488 64094208
28875896 64094208
27777736 64094208
...

Solamente se muestra una parte de la salida. Se puede observar que el recolector de basura solamente comienza a funcionar cuando la memoria ocupada se acerca al máximo especificado de 64MB (la memoria libre se acerca a 0, de echo entra en funcionamiento cuando queda poco más de 1MB de memoria libre).


.::

	java -XX:+PrintFlagsFinal -version

Without using JMX, which is what most tools use, all you can do is use::

	jps -lvm

I need it to be commandline because I don't have access to a windowing environment, and I want script based on the value , the application is running in Jetty Application server::

	jstat -gc pid
