# 01 - Fases de Traducción y Errores

## Investigación

> Considerando GCC 14.2.0

Para limitar el proceso de compilación a fases específicas podemos usar

- `-E`: Ejecuta el preprocesador y muestra la salida. No compila.

```
gcc -E archivo.c -o archivo.i
```

- `-S `: Traduce a ensamblador. No ensambla ni enlaza.

```
gcc -S archivo.c -o archivo.s
gcc -S archivo.i -o archivo.s
```

- `-c` : Traduce a codigo objeto. No enlaza.

```
gcc -c archivo.c -o archivo.o
```

- Para generar el vincular y generar el ejecutable: `gcc archivo.o -o ejecutable`

***

Algunas opciones útiles:

- `-nostartfiles, -nodefaultlibs, -nostdlib, -Wl,...` : Controlan la etapa de enlace.
- `-v` : Muestra que hace el compilador paso a paso.
- `save-temps`: Guarda todos los archivos intermedios (.i, .s, .o).
- `fdump-*`: Para depurar el estado del codigo en distintas fases internas del compilador.

## Ejecución

### Parte 1, Preprocesador

1) Creamos el archivo `hello2.c`.
2) Preprocesamos el archivo usando el comando:

```
gcc -E hello2.c -o hello2.i
```

**Conclusiones**
El archivo preprocesado se diferencia del archivo fuente, pues:<br> 
- El preprocesador responde a la directiva #include <stdio.h>, incluyendo el contenido del header, es decir, los prototipos de las funciones de la librería estándar.<br>
- Se reemplaza (no elimina) el comentario entre el tipo de dato int y la función main por un espacio en blanco.<br>

Además, ignora los problemas sintácticos (la llave faltante que cierra la función main) y semánticos (la ausencia de la declaración de la función prontf) del archivo fuente. Esto se debe a que el preprocesador no conoce la sintáctica y la semántica de C.<br>


3) Escribimos `hello3.c`.
4) Análisis de la semántica:
```
int printf(const char * restrict s, ...);
```
Es el prototipo de la función printf que recibe como argumento un puntero a un carácter s:<br>

1) `int`: Indica que el valor de retorno es un entero
2) `printf`: Nombre de la función, estándar en C, usada para imprimir en la salida estándar (stdout).
3) `const char *` La palabra const es un calificador de tipo, en este caso, aplicado a un puntero a un char, que indica que no se puede modificar el caracter al que apunta. Esto sin embargo no me impide modificar el valor del puntero en sí
4) `restrict s`:Con el calificador restrict me aseguro de que el puntero s no podrá ser modificado una vez que se inicializa.
5) `...`	Argumentos variádicos: permite pasar una cantidad indefinida de parámetros después de la cadena de formato.

**Conclusiones**: Al no estar el `#include <stdio.h>` no se incluyó la librería estándar lo que generó un archivo mucho más pequeño que incluia directivas de control de línea generadas por el preprocesador: 
```
# 0 ".\\hello3.c"
Informa que el compilador vuelve a comenzar desde línea 0 del archivo original hello3.c. El número 0 también suele ir acompañado de flags internos, como que comienza un nuevo archivo fuente.

# 0 "<built-in>"
Código incorporado por el compilador (por ejemplo, definiciones predefinidas por el compilador).

# 0 "<command-line>"
Código o macros definidos desde la línea de comandos con opciones como -D.

# 1 ".\\hello3.c"
El compilador regresa al archivo fuente original, y está comenzando la línea 1 del archivo hello3.c.
```

### Parte 2, Compilación

1) Compilar el resultado y generar hello3.s, no ensamblar. En este paso la terminal nos avisa de un error:

``` powershell
.\hello3.c: In function 'main':
.\hello3.c:4:2: error: implicit declaration of function 'prontf'; did you mean 'printf'? [-Wimplicit-function-declaration]
    4 |  prontf("La respuesta es %d\n");
      |  ^~~~~~
      |  printf
.\hello3.c:4:2: error: expected declaration or statement at end of input
```

2) El primer error indica que la función prontf no fue declarada, lo pudimos resolver corrigiendo la declaración de la función en la 1era línea a `prontf`
   El segundo error señala la ausencia del centinela, se soluciona agregando `}` al final del archivo

3) El objetivo del código ensamblador es ser una representación de bajo nivel del programa que está más cercana al lenguaje máquina, pero todavía legible por humanos.

4) Ensamblar `hello4.s` en `hello4.o`, no vincular. Este proceso sale y no genera ningún error.


### Parte 3, Vinculación

1) Vincular `hello4.o` con la biblioteca estándar y generar el ejecutable. Esta parte nos muestra el siguiente error:

``` powershell
D:/MSYS2/ucrt64/bin/../lib/gcc/x86_64-w64-mingw32/14.2.0/../../../../x86_64-w64-mingw32/bin/ld.exe: .\hello4.o:hello4.c:(.text+0x1f): undefined reference to `prontf'
collect2.exe: error: ld returned 1 exit status
```

2) Este error se debe a que en el proceso de vinculación, el linker responde a la llamada a prontf que se hace en hello4.o, buscando la función en la biblioteca estándar, sin embargo, al no encontrarla, falla el linkeo, no se genera el ejecutable.Para su corrección cambiamos los `prontf` por `printf`, función que si va a encontrar en la librería estándar. 

3) Para ejecutarlo hicimos: `./hello5.exe`. Pero esto no imprimio el resultado que esperabamos sino que mostro: `La respuesta es 899871184`.

**Análisis**: La respuesta no fue la esperada ya que la función printf no fue invocada correctamente, puesto que espera un número en decimal (esto se indica usando "%d"), el 42 almacenado en i, que nunca se pasa.

### Parte 4, Corrección del Bug

1) Para corregir el bug cambiamos la llamada a la función `printf` por: `printf("La respuesta es %d\n", i);`. Ahora si genera la respuesta que esperabamos.

### Parte 5, Remoción del prototipo

1) Escribimos la nueva variante `hello7.c`.

2) En nuestro caso no se pudo compilar el archivo:

Lo que nos llevó a responder las siguiente preguntas:

1)  ¿Arroja error o warning? Arroja un error y un warning:

El primero se debe a que estamos utilizando una función perteneciente a la biblioteca estándar, sin haberla incluido previamente y por eso nos sugiere incluirla. Además de un warning.

```powershell
   .\hello7.c:3:5: error: implicit declaration of function 'printf' [-Wimplicit-function-declaration]
3 |     printf("La respuesta es %d\n", i);
  |     ^~~~~~
```
```powershell 
   .\hello7.c:1:1: note: include '<stdio.h>' or provide a declaration of 'printf'
+++ |+#include <stdio.h>
   1 | int main(void){
```
```powershell
.\hello7.c:3:5: warning: incompatible implicit declaration of built-in function 'printf' [-Wbuiltin-declaration-mismatch]
    3 |     printf("La respuesta es %d\n", i);
      |     ^~~~~~
```

2)  ¿Qué es un prototipo y de qué maneras se puede generar? 

Un prototipo es la declaración de una función/procedimiento, se indica al principio de un programa, fuera de main, y debe indicar:
- El valor que retorna (en caso de ser una función)
- El o los parámetros que recibe, indicando el tipo de variable de cada uno y, si así se desea, sus identificadores
Estos pueden ser expresados en una misma unidad de traducción o colocados en un archivo de cabecera (header) distinto que se incluye con la directiva `#include` en la etapa de preprocesamiento


3)  ¿Qué es una declaración implícita de una función? 

Es la declaración de una función o variable sin especificar su tipo de retorno o tipo de dato, el compilador asume por defecto que es del tipo `Int` (generalmente),pero esto puede llevar a errores de comportamiento si la función no se define de manera compatible con esta inferencia. 

4)  ¿Qué indica la especificación?

La especificación de de C indica que para utilizar una función esta debe ser declarada previamente, se trata entonces de una especificación a nivel semántico.

5)  ¿Cómo se comportan las principales implementaciones? 

En general, los compiladores de C más utilizados hoy en día, como GCC (GNU Compiler Collection), Clang, y MSVC (Microsoft Visual C++), se adhieren al estándar C99 (o posteriores) con respecto a la declaración de funciones. Esto significa que, en la mayoría de los casos, emitirán un error cuando se encuentra una llamada a una función sin una declaración previa.<br>
GCC: Como vimos con el ejemplo de hello7.c, GCC produce un error (`implicit declaration of function`) y una advertencia (`incompatible implicit declaration of built-in function`). Por defecto, este error impedirá la generación del ejecutable. Sin embargo, GCC tiene opciones de compilación que pueden relajar esta restricción por compatibilidad con código antiguo, permitiendo que se genere el ejecutable con la suposición implícita (devolviendo `int`, argumentos sin tipo específico). Esto generalmente se desaconseja en código nuevo.

6)  ¿Qué es una función built-in? 

Son funciones que están integradas al compilador y que no requieren linkeo, en general suelen estar optimizadas.

7)  Conjeture la razón por la cual gcc se comporta como se comporta ¿Va realmente contra la especificación?

No necesariamente va contra la especificacion; El comportamiento de GCC parece estar motivado por un equilibrio entre la necesidad de adherirse a los estándares modernos de C para promover la seguridad y la portabilidad, y la practicidad de ayudar a los desarrolladores (incluyendo aquellos que trabajan con código más antiguo) a entender y corregir problemas. En el caso de la declaración implícita en código moderno, GCC se alinea con la especificación al emitir un error. Las advertencias y notas adicionales son parte de su esfuerzo por proporcionar diagnósticos útiles.

### Parte 6, Compilación Separada: Contratos y Módulos

1) Creamos los archivos `hello8.c` y `studio1.c`.

2) No pudimos compilar el archivo `hello8.c` ya que nos muestra el mismo error de antes (declaración implicita de la función). Por lo que deberíamos que agregar el prototipo de `prontf` a `hello8.c` e incluir `stdio.h` al archivo `studio1.c` (así tiene acceso a la declaración de printf). De esta forma logramos generar el ejecutable y comprobamos que la respuesta es correcta.

```powershell
gcc hello8.o studio1.o -o programa1
```

3) Si eliminamos o agregamos parámetros en `prontf` habría un problema en la vinculación, ya que el linker no encontraría una función con una declaración compatible a la que provee studio1.c, por lo tanto, la vinculación fallaría.

4) Escribirmos el contrato `studio.h` y ambos archivos `hello9.c` y `studio2.c`.
Cuando en contrato es incluido por ambas partes, es posible detectar errores **ANTES** del tiempo de ejecución, es decir, en **TIEMPO DE COMPILACIÓN**, tales como:
- Una INVOCACIÓN INCORRECTA por parte del consumidor
- Una DEFINICIÓN INCORRECTA por parte del proveedor

***

### Extra

#### Bibloteca

> Una biblioteca es un conjunto de funciones, procedimientos o rutinas ya compiladas y listas para ser usadas en otros programas.

En C/C++, las bibliotecas agregan funcionalidad sin tener que reescribir código. Estas biblotecas son distribuibles, pero **pueden** ser portables dependiendo de si están bien adaptadas a distintos sistemas operativos.

#### Tipos de Bibliotecas
- **Estáticas**: Se integran directamente en el ejecutable durante la compilación.
- **Dinámicas**: Se cargan en tiempo de ejecución, lo que permite compartirlas entre varios programas y reducir el tamaño del ejecutable.
##### Ventajas

- Reutilización de código: Evitás escribir desde cero funciones comunes.

- Modularidad: Separás lógica en componentes independientes.

- Mantenimiento más fácil: Si una biblioteca mejora, solo actualizás esa parte.

- Tamaño reducido (en bibliotecas dinámicas): El ejecutable no incluye todo el código.

##### Desventajas

- Las dinámicas pueden causar errores si faltan en tiempo de ejecución.

- Las estáticas aumentan el tamaño del ejecutable.

- Si no están bien documentadas, puede ser difícil entender cómo usarlas.

- En algunos casos, hay que tener cuidado con licencias si se redistribuye código que usa bibliotecas de terceros.
