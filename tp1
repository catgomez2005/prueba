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

**Conclusiones**: No hubo ningún error. Se realizó el preprocesamiento correctamente y se incluyo las declaraciones de la bibloteca `stdio.h`. Además se eliminaron los comentarios (desaparecio el comentario medio).

3) Escribimos `hello3.c`.
4) Análisis de la semántica:
   1) `int`: Tipo de retorno: printf devuelve un entero que representa el número de caracteres impresos, o un valor negativo si hay error.
   2) `printf`: Nombre de la función, estándar en C, usada para imprimir en la salida estándar (stdout).
   3) `const char *` Primer argumento: un puntero a una cadena constante (el formato), no modificable dentro de la función.
   4) `restrict`:Indica que el puntero s es el único puntero que accede a esa región de memoria durante la ejecución de printf. Permite optimizaciones por parte del compilador.
   5) `...`	Argumentos variádicos: permite pasar una cantidad indefinida de parámetros después de la cadena de formato, como en `printf("%d", x);`.

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

2) Para corregir el error lo que hicimos fue agregar `}` al final del archivo y corregir la declaración de la función a `prontf` (con solo la llave no fue suficiente).
3) El objetivo del código ensamblador es ser una representación de bajo nivel del programa que está más cercana al lenguaje máquina, pero todavía legible por humanos.
4) Ensamblar `hello4.s` en `hello4.o`, no vincular. Este proceso sale y no genera ningún error.


### Parte 3, Vinculación

1) Vincular `hello4.o` con la biblioteca estándar y generar el ejecutable. Esta parte nos muestra el siguiente error:

``` powershell
D:/MSYS2/ucrt64/bin/../lib/gcc/x86_64-w64-mingw32/14.2.0/../../../../x86_64-w64-mingw32/bin/ld.exe: .\hello4.o:hello4.c:(.text+0x1f): undefined reference to `prontf'
collect2.exe: error: ld returned 1 exit status
```

2) Para la corrección cambiamos los `prontf` por `printf`, que en esta ocasión funciona correctamente. 

3) Para ejecutarlo hicimos: `./hello5.exe`. Pero esto no imprimio el resultado que esperabamos sino que mostro: `La respuesta es 899871184`.

**Análisis**: La respuesta no fue la esperada ya que no usamos correctamente la función `printf`, deberíamos pasarle el `resultado` que queríamos mostrar.

### Parte 4, Corrección del Bug

1) Para corregir el bug cambiamos la llamada a la función `printf` por: `printf("La respuesta es %d\n", i);`. Ahora si genera la respuesta que esperabamos.

### Parte 5, Remoción del prototipo

1) Escribimos la nueva variante `hello7.c`.
2) En nuestro caso no se pudo compilar el archivo:

Lo que nos llevó a responder las siguiente preguntas:

1)  ¿Arroja error o warning? Arroja un error y un warning:

El primero se debe a que estamos utilizando una función perteneciente a la biblioteca estándar, sin haberla incluido previamente y por eso nos arroja una nota sugiriendo incluirla. Además de un warning.

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

El protipo se refiere a la declaración previa de una función que indica su firma. Se puede incluir manualmente o a través de un archivo `.h`.

3)  ¿Qué es una declaración implícita de una función? 

Es la declaración de una función o variable sin especificar su tipo de retorno o tipo de dato, el compilador asume por defecto que es del tipo `Int` (generalmente),pero esto puede llevar a errores de comportamiento si la función no se define de manera compatible con esta inferencia. 

4)  ¿Qué indica la especificación?

La especificación establece el **que** hace una función, pero no el como.

5)  ¿Cómo se comportan las principales implementaciones? 

En general, los compiladores de C más utilizados hoy en día, como GCC (GNU Compiler Collection), Clang, y MSVC (Microsoft Visual C++), se adhieren al estándar C99 (o posteriores) con respecto a la declaración de funciones. Esto significa que, en la mayoría de los casos, emitirán un error cuando se encuentra una llamada a una función sin una declaración previa.
GCC: Como vimos con el ejemplo de hello7.c, GCC produce un error (`implicit declaration of function`) y una advertencia (`incompatible implicit declaration of built-in function`). Por defecto, este error impedirá la generación del ejecutable. Sin embargo, GCC tiene opciones de compilación que pueden relajar esta restricción por compatibilidad con código antiguo, permitiendo que se genere el ejecutable con la suposición implícita (devolviendo `int`, argumentos sin tipo específico). Esto generalmente se desaconseja en código nuevo.

6)  ¿Qué es una función built-in? 

Son funciones que están integradas al compilador y que no requieren linkeo, en general suelen estar optimizadas.

7)  ¿Conjeture la razón por la cual gcc se comporta como se comporta? ¿Va realmente contra la especificación?

No necesariamente va contra la especificacion; El comportamiento de GCC parece estar motivado por un equilibrio entre la necesidad de adherirse a los estándares modernos de C para promover la seguridad y la portabilidad, y la practicidad de ayudar a los desarrolladores (incluyendo aquellos que trabajan con código más antiguo) a entender y corregir problemas. En el caso de la declaración implícita en código moderno, GCC se alinea con la especificación al emitir un error. Las advertencias y notas adicionales son parte de su esfuerzo por proporcionar diagnósticos útiles.

### Parte 6, Compilación Separada: Contratos y Módulos

1) Creamos los archivos `hello8.c` y `studio1.c`.
2) No pudimos compilar el archivo `hello8.c` ya que nos muestra el mismo error de antes (declaración implicita de la función). Por lo que tuvimos que agregar el prototipo de `prontf` e incluir `stdio.h` al archivo `studio1.c`. De esta forma logramos generar el ejecutable y comprobamos que la respuesta es correcta.

```powershell
gcc hello8.o studio1.o -o programa1
```

3) Lo que ocurriría si eliminaramos parametros o agregaramos parametros en `prontf` es que habría un problema en la vinculación ya que no encuentra una función con esa firma, por lo tanto el programa dejaría de funcionar.
4) Escribirmos el contrato `studio.h` y ambos archivos `hello9.c` y `studio2.c`. La ventaja que da incluir el contrato en los clientes y en el proveedor es que da la **libertad a ambas** partes de tomar decisiones independientes (como el tiempo de compilacion o algoritmos que se utilizan) simpre y cuando se respete el contrato.

***

### Extra

#### Bibloteca

> Una biblioteca es un conjunto de funciones, procedimientos o rutinas ya compiladas y listas para ser usadas en otros programas.

En C/C++, las bibliotecas agregan funcionalidad sin tener que reescribir código. Estas biblotecas son distribuibles, pero **pueden** ser portables dependiendo de si están bien adaptadas a distintos sistemas operativos.

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
