# Del código Arduino al lenguaje máquina

### Objetivo

Comprender cómo un programa escrito en Arduino C/C++ es transformado por el compilador en instrucciones que pueden ser ejecutadas por el microcontrolador.

Durante el laboratorio se seguirá el recorrido:

**Código Arduino → Compilación → Ensamblador AVR → Código hexadecimal → Ejecución en el microcontrolador → LED**

Al finalizar, podrá relacionar conceptos estudiados anteriormente como **registros, ALU, memoria, instrucciones, puertos de entrada/salida y arquitectura del computador** con un microcontrolador real.

## Parte 1. Conociendo el procesador de nuestro Arduino

El Arduino Uno utiliza un microcontrolador de la familia AVR. En el Arduino Uno R3 trabajaremos con el **ATmega328P**.

Este pequeño circuito integrado contiene prácticamente todos los elementos necesarios para ejecutar un programa:

* CPU.
* ALU.
* Registros.
* Contador de programa.
* Memoria Flash.
* Memoria SRAM.
* Puertos de entrada/salida.
* Temporizadores.
* Interfaces de comunicación.

Cuando programamos Arduino normalmente no interactuamos directamente con estos componentes. El entorno de Arduino y su compilador realizan gran parte del trabajo por nosotros.

En este laboratorio observaremos qué ocurre debajo de esa capa de abstracción.

## Parte 2. Nuestro primer programa

Conecte el Arduino Uno a la computadora y abra **Arduino IDE**.

Seleccione la tarjeta correspondiente:

**Tools → Board → Arduino AVR Boards → Arduino Uno**

Luego escriba el siguiente programa:

```cpp
void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH);
}
```

Compile y cargue el programa en el Arduino.

#### ¿Qué hace el programa?

La instrucción:

```cpp
pinMode(13, OUTPUT);
```

Configura el pin digital 13 como una **salida**.

La instrucción:

```cpp
digitalWrite(13, HIGH);
```

Coloca un nivel lógico alto en ese pin.

En el Arduino Uno existe un LED integrado conectado al pin 13, por lo que deberá observar que el LED marcado normalmente con la letra **L** permanece encendido.

Hasta este momento estamos observando el programa desde el punto de vista del programador:

```
digitalWrite(13, HIGH)
          ↓
       LED encendido
```

Pero el microcontrolador **no ejecuta directamente la instrucción `digitalWrite()`**.

El procesador necesita instrucciones de máquina.

Nuestro siguiente objetivo será descubrirlas.

## Parte 3. ¿Qué ocurre cuando presionamos Compile?

Arduino utiliza un conjunto de herramientas para transformar nuestro programa.

De forma simplificada:

<figure><img src="../.gitbook/assets/image (13).png" alt="" width="375"><figcaption></figcaption></figure>

El compilador transforma nuestro programa en instrucciones correspondientes a la arquitectura AVR utilizada por el microcontrolador.

Estas instrucciones tienen nombres como:

```
LDI
MOV
ADD
SUB
OUT
IN
SBI
CBI
RJMP
CALL
RET
```

Cada una representa una operación que el procesador puede realizar.

## Parte 4. Mostrar información detallada de la compilación

Necesitamos encontrar los archivos que Arduino genera durante la compilación.

Abra:

**File → Preferences**

o **Settings**, dependiendo de la versión del Arduino IDE.

Busque la opción:

**Show verbose output during:**

Active:

**Compilation**

Guarde los cambios.

Ahora vuelva a presionar **Verify/Compile**.

Observe la consola ubicada en la parte inferior del Arduino IDE. Esta vez aparecerá mucha más información. No se preocupe si todavía no comprende todo lo que aparece. El objetivo es encontrar dónde Arduino almacenó temporalmente nuestro programa compilado.

## Parte 5. Localizar el archivo ELF

Durante la compilación se genera un archivo con extensión:

```
.elf
```

Busque en las últimas líneas de la consola una referencia similar a:

```
sketch_aug18a.ino.elf
```

El nombre exacto y la ubicación pueden variar según la computadora.

El archivo **ELF (Executable and Linkable Format)** contiene el programa ya compilado junto con información que permite analizarlo. Este archivo será especialmente importante porque podemos pedirle a las herramientas de AVR que nos muestren las instrucciones que contiene.

Copie o identifique la ruta donde se encuentra el archivo `.elf`.

## Parte 6. Del ELF al ensamblador

Ahora utilizaremos una herramienta llamada:

```
avr-objdump
```

Esta herramienta permite inspeccionar el programa compilado.

Abra una terminal o consola de comandos. Dependiendo la versión de Arduino IDE la herramienta se encontrará en un path similar al siguiente:

```
\\AppData\\Local\\Arduino15\\packages\\arduino\\tools\\avr-gcc\\7.3.0-atmel3.6.1-arduino7/bin/
```

Ejecute:

```bash
avr-objdump -d ruta_del_programa.ino.elf
```

Sustituya `ruta_del_programa.ino.elf` por la ruta obtenida anteriormente.

Por ejemplo:

```bash
avr-objdump -d sketch_aug18a.ino.elf
```

Aparecerá una gran cantidad de información. Esto es normal. El pequeño programa que escribimos utiliza funciones proporcionadas por Arduino. Por esa razón, unas pocas líneas de C/C++ pueden convertirse en muchas instrucciones.

Busque instrucciones con nombres como:

```
ldi
mov
out
call
ret
rjmp
```

Estas son instrucciones reales de la arquitectura AVR.

## Parte 7. Interpretando una instrucción

Podrá encontrar líneas con una estructura similar a:

```
80:   25 9a        sbi  0x04, 5
```

{% hint style="info" %}
**Nota:** esta línea es solamente un ejemplo de cómo leer la salida. Los valores concretos que aparezcan en su programa pueden ser diferentes.
{% endhint %}

Separemos sus partes:

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

La parte:

```
sbi
```

es una instrucción en **lenguaje ensamblador**.

`SBI` significa:

**Set Bit in I/O Register**

Su función es colocar en `1` un bit determinado de un registro de entrada/salida.

Observe algo importante:

El procesador no recibe el texto:

```
sbi
```

El nombre `sbi` solamente existe para que los humanos podamos comprender la instrucción.

El procesador recibe su **código máquina**.

## Parte 8. Del hexadecimal al binario

Supongamos que encontramos los bytes:

```
25 9A
```

Estos números están representados en **hexadecimal**.

Cada dígito hexadecimal representa cuatro bits.

Por ejemplo:

```
HEX        BINARIO

2    →     0010
5    →     0101

9    →     1001
A    →     1010
```

Por lo tanto:

```
25 9A
```

puede representarse byte a byte como:

```
0010 0101   1001 1010
```

{% hint style="info" %}
**Importante:** AVR maneja instrucciones de varios tamaños y existe un orden específico para almacenar sus bytes. Por ahora no intentaremos decodificar manualmente toda la instrucción. Nuestro objetivo es reconocer que los bytes mostrados por `objdump` son la representación del código que ejecutará el procesador.
{% endhint %}

Acabamos de recorrer tres niveles diferentes:

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

## Parte 9. ¿Dónde está Von Neumann?

Regrese mentalmente a la arquitectura estudiada anteriormente.

Cuando el programa está almacenado en el microcontrolador ocurre, de forma simplificada:

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

El pin digital **13** del Arduino Uno está relacionado con el pin **PB5** del microcontrolador.

Por lo tanto, cuando nuestro programa modifica la salida correspondiente, estamos modificando hardware interno del microcontrolador que finalmente controla un pin físico.

Ya no estamos trabajando únicamente con una simulación de registros, memoria y unidad de control.

Estamos observando estos conceptos en un procesador real.

## Parte 10. Eliminando parte de la abstracción de Arduino

`digitalWrite()` es muy cómodo para programar Arduino, pero oculta muchos detalles.

Ahora sustituya el programa anterior por:

```cpp
void setup() {
  DDRB |= (1 << 5);
}

void loop() {
  PORTB |= (1 << 5);
}
```

Compile y cargue nuevamente el programa.

El LED integrado deberá permanecer encendido igual que antes.

Entonces:

**¿Qué cambió?**

No cambió el resultado.

Cambió **cómo accedemos al hardware**.

En el primer programa utilizábamos:

```cpp
digitalWrite(13, HIGH);
```

Ahora utilizamos directamente:

```cpp
PORTB |= (1 << 5);
```

`PORTB` representa uno de los registros de entrada/salida del microcontrolador.

El bit 5 corresponde a **PB5**.

Podemos visualizarlo como:

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

La expresión:

```cpp
(1 << 5)
```

produce:

```
00100000
```

y permite seleccionar el bit número 5.

## Parte 11. Compare nuevamente el ensamblador

Obtenga nuevamente el archivo `.elf` y ejecute:

```bash
avr-objdump -d ruta_del_programa.ino.elf
```

Compare el resultado con el obtenido utilizando `digitalWrite()`.

Observe especialmente:

* Cantidad de instrucciones.
* Llamadas a funciones.
* Acceso a registros.
* Instrucciones de entrada/salida.
* Tamaño final del programa mostrado por Arduino IDE.

El objetivo no es comprender todavía **cada instrucción**.

El objetivo es descubrir que una misma operación puede expresarse con diferentes niveles de abstracción.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

## Reto Final

Realice ahora el proceso inverso.

Modifique el programa para que el LED integrado se **apague** mediante acceso directo al registro `PORTB`.

No utilice:

```cpp
digitalWrite()
```

Después:

1. Compile el programa.
2. Compruebe físicamente que el LED se apaga.
3. Obtenga nuevamente el ensamblador.
4. Identifique qué cambió respecto al programa que encendía el LED.
5. Localice al menos una instrucción AVR relacionada con el cambio.
6. Identifique los bytes hexadecimales asociados a esa instrucción.
7. Convierta esos bytes a binario.

Finalmente explique con sus propias palabras el recorrido:

**Código C/C++ → Ensamblador → Código máquina → CPU → Registro de E/S → Pin → LED.**

## Entregable del laboratorio

Como evidencia del laboratorio, deberá completar el **reto final**: modificar el programa para **apagar el LED integrado mediante acceso directo al registro `PORTB`, sin utilizar `digitalWrite()`**, comprobar su funcionamiento y analizar el código ensamblador generado por el compilador. Deberá entregar capturas que demuestren la ejecución del programa y el análisis realizado, identificando al menos una instrucción AVR relacionada con el cambio del LED, sus bytes en hexadecimal y su representación en binario. Finalmente, deberá explicar brevemente el recorrido.

**Código C/C++ → Ensamblador → Código máquina → CPU → Registro de E/S → Pin → LED.**

Adicionalmente, deberá adjuntar **un archivo de texto (`.txt`)** que contenga los códigos ensambladores completos obtenidos durante el laboratorio. El archivo deberá incluir claramente identificadas las dos versiones:&#x20;

* **Ensamblador generado para el programa que utiliza `digitalWrite(13, HIGH)`**&#x20;
* **Ensamblador generado para el programa que accede directamente a `PORTB`**.&#x20;

No deberá copiar únicamente algunas instrucciones: deberá incluir la salida completa generada por `avr-objdump` para cada programa. Esto permitirá comparar cómo el nivel de abstracción utilizado en C/C++ afecta las instrucciones que finalmente ejecuta el microcontrolador.
