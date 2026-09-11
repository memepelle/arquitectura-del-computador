# Memoria EEPROM y almacenamiento de datos

## Introducción

En el laboratorio anterior se programó el Arduino Uno para detectar las teclas de un teclado matricial 4×4 y mostrar información en una pantalla LCD.

Los datos capturados podían guardarse temporalmente en variables. Sin embargo, estas variables se almacenan en SRAM y desaparecen cuando se desconecta el Arduino.

En este laboratorio se utilizará la memoria EEPROM del ATmega328P para almacenar información que debe permanecer aun cuando el sistema deje de recibir energía.

Primero se escribirá y leerá un dato. Después, su equipo extenderá el procedimiento para almacenar varios números en direcciones consecutivas y finalmente aplicará este mecanismo para almacenar un PIN ingresado desde el teclado matricial.

## Objetivos

* Diferenciar la memoria volátil de la memoria no volátil.
* Explicar cómo se organiza la EEPROM del ATmega328P.
* Utilizar los registros EEAR, EEDR y EECR.
* Escribir y leer un byte.
* Almacenar varios números mediante arreglos, ciclos y direcciones consecutivas.
* Integrar como reto el teclado matricial, la pantalla LCD y la EEPROM.

## Materiales

Para la parte guiada únicamente se necesita el Arduino Uno y el cable USB.

Para el reto se reutilizará el circuito construido en el laboratorio anterior:

* Arduino Uno.
* Teclado matricial 4×4.
* Pantalla LCD 16×2 con módulo I²C.
* Cables de conexión.
* Cable USB.
* Computadora con Arduino IDE.

## Memoria volátil y no volátil

Una memoria es **volátil** cuando necesita alimentación eléctrica para conservar su contenido. Las variables del programa se almacenan normalmente en la memoria SRAM. Cuando se desconecta el Arduino, los valores almacenados en estas variables desaparecen.

Una memoria **no volátil** conserva la información incluso cuando deja de recibir energía. La EEPROM pertenece a esta segunda categoría.

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

_Figura 1. Diferencia entre SRAM y EEPROM._

| Memoria | Uso principal                       | ¿Conserva los datos? |
| ------- | ----------------------------------- | -------------------- |
| Flash   | Instrucciones del programa          | Sí                   |
| SRAM    | Variables durante la ejecución      | No                   |
| EEPROM  | Configuraciones y datos permanentes | Sí                   |

{% hint style="info" %}
**Importante:** En sistemas reales no debe almacenarse una contraseña sensible sin protección. En este laboratorio se trabaja con un PIN sencillo únicamente con fines didácticos.
{% endhint %}

## Organización de la EEPROM

El ATmega328P posee **1024 bytes de memoria EEPROM**. Las posiciones están numeradas desde la dirección `0` hasta la dirección `1023`. Cada dirección puede almacenar exactamente un byte, es decir, ocho bits.

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

_Figura 2. Organización de la EEPROM del ATmega328P._

| Dirección | Contenido |
| --------- | --------- |
| 0         | Un byte   |
| 1         | Un byte   |
| 2         | Un byte   |
| …         | …         |
| 1023      | Un byte   |

Por ejemplo, si la dirección `10` contiene el valor binario `00000111`, el número almacenado es el decimal `7`.

## Registros utilizados

La CPU no escribe directamente sobre una casilla de la EEPROM. Para realizar una operación utiliza registros especiales que permiten indicar la dirección, transferir el dato y controlar la operación.

| Registro | Función                                           |
| -------- | ------------------------------------------------- |
| EEAR     | Contiene la dirección de EEPROM que se utilizará. |
| EEDR     | Contiene el byte que se leerá o escribirá.        |
| EECR     | Controla las operaciones de lectura y escritura.  |

La relación entre los registros puede resumirse de la siguiente manera:

> Dirección → EEAR\
> Dato → EEDR\
> Orden → EECR\
> ↓\
> EEPROM

### Bits de control

Dentro del registro `EECR` se encuentran los bits que controlan las operaciones de la EEPROM.

| Bit   | Función                                     |
| ----- | ------------------------------------------- |
| EEPE  | Indica que existe una escritura en proceso. |
| EEMPE | Autoriza una nueva escritura.               |
| EERE  | Inicia una lectura.                         |

## Escritura de un byte

La siguiente función escribe un byte en una dirección de la EEPROM mediante los registros del ATmega328P:

```cpp
void escribirEEPROM(uint16_t direccion, uint8_t dato)
{
    while (EECR & (1 << EEPE))
    {
        // Esperar mientras la EEPROM está ocupada
    }

    EEAR = direccion;
    EEDR = dato;

    uint8_t estadoAnterior = SREG;

    cli();

    EECR |= (1 << EEMPE);
    EECR |= (1 << EEPE);

    SREG = estadoAnterior;

    while (EECR & (1 << EEPE))
    {
        // Esperar hasta que termine la escritura
    }
}
```

### Explicación del código

| Instrucción                  | Propósito                                           |
| ---------------------------- | --------------------------------------------------- |
| `while (EECR & (1 << EEPE))` | Esperar mientras la EEPROM está ocupada.            |
| `EEAR = direccion`           | Seleccionar la posición de memoria.                 |
| `EEDR = dato`                | Preparar el byte que se escribirá.                  |
| `cli()`                      | Desactivar temporalmente las interrupciones.        |
| `EECR \|= (1 << EEMPE)`      | Autorizar la escritura.                             |
| `EECR \|= (1 << EEPE)`       | Iniciar la escritura.                               |
| `SREG = estadoAnterior`      | Restaurar el estado anterior de las interrupciones. |

Las interrupciones se desactivan brevemente porque el bit `EEPE` debe activarse inmediatamente después del bit `EEMPE`.

La secuencia general de escritura es:

1. Esperar a que termine cualquier escritura anterior.
2. Seleccionar la dirección mediante `EEAR`.
3. Colocar el dato en `EEDR`.
4. Autorizar la escritura mediante `EEMPE`.
5. Iniciar la escritura mediante `EEPE`.
6. Esperar hasta que finalice el proceso.

## Lectura de un byte

La lectura de un byte requiere seleccionar la dirección y activar el bit `EERE`.

```cpp
uint8_t leerEEPROM(uint16_t direccion)
{
    while (EECR & (1 << EEPE))
    {
        // Esperar si existe una escritura en proceso
    }

    EEAR = direccion;
    EECR |= (1 << EERE);

    return EEDR;
}
```

Al activar `EERE`, el contenido de la dirección seleccionada se copia al registro `EEDR`.

La instrucción:

```cpp
return EEDR;
```

Devuelve el byte a la función que solicitó la lectura.

## Primera prueba

Cargue el siguiente programa para almacenar el número `7` en la dirección `10` y recuperarlo mediante el monitor serial.

```cpp
#include <avr/io.h>
#include <avr/interrupt.h>

const uint16_t DIRECCION_PRUEBA = 10;

void escribirEEPROM(uint16_t direccion, uint8_t dato)
{
    while (EECR & (1 << EEPE))
    {
    }

    EEAR = direccion;
    EEDR = dato;

    uint8_t estadoAnterior = SREG;

    cli();

    EECR |= (1 << EEMPE);
    EECR |= (1 << EEPE);

    SREG = estadoAnterior;

    while (EECR & (1 << EEPE))
    {
    }
}

uint8_t leerEEPROM(uint16_t direccion)
{
    while (EECR & (1 << EEPE))
    {
    }

    EEAR = direccion;
    EECR |= (1 << EERE);

    return EEDR;
}

void setup()
{
    Serial.begin(9600);

    escribirEEPROM(DIRECCION_PRUEBA, 7);

    uint8_t datoRecuperado =
        leerEEPROM(DIRECCION_PRUEBA);

    Serial.print("Dato recuperado: ");
    Serial.println(datoRecuperado);
}

void loop()
{
}
```

### Resultado esperado

En el monitor serial deberá aparecer:

```
Dato recuperado: 7
```

### Prueba de persistencia

Realice el siguiente procedimiento:

1. Ejecute el programa una vez.
2. Compruebe que aparece el número `7`.
3. Comente la instrucción de escritura:

```cpp
// escribirEEPROM(DIRECCION_PRUEBA, 7);
```

4. Cargue nuevamente el programa.
5. Compruebe que el monitor serial continúa mostrando el número `7`.

El dato continúa disponible porque fue almacenado en una memoria no volátil.

## Almacenar varios números

Cada dirección de EEPROM guarda solamente un byte. Para almacenar varios números es necesario utilizar varias direcciones y un arreglo temporal en SRAM.

Considere el siguiente arreglo:

```cpp
uint8_t numeros[4] = {25, 100, 7, 240};
```

En este ejemplo, la dirección `20` será la dirección inicial o dirección base.

| Posición del arreglo | Dirección EEPROM | Valor |
| -------------------- | ---------------- | ----- |
| `numeros[0]`         | 20               | 25    |
| `numeros[1]`         | 21               | 100   |
| `numeros[2]`         | 22               | 7     |
| `numeros[3]`         | 23               | 240   |

### Direcciones consecutivas

La expresión:

```
direccionInicial + i
```

calcula una dirección diferente en cada repetición.

Al mismo tiempo:

```
datos[i]
```

obtiene el elemento correspondiente del arreglo.

La relación entre el arreglo y la EEPROM será:

> `numeros[0]` → `EEPROM[20]`\
> `numeros[1]` → `EEPROM[21]`\
> `numeros[2]` → `EEPROM[22]`\
> `numeros[3]` → `EEPROM[23]`

En cada repetición, el índice `i` se utiliza tanto para seleccionar un elemento del arreglo como para calcular su dirección en la EEPROM.

## Función para guardar varios números

La siguiente función recibe un arreglo y almacena sus elementos en direcciones consecutivas.

```cpp
void guardarNumeros(
    const uint8_t datos[],
    uint8_t cantidad,
    uint16_t direccionInicial)
{
    for (uint8_t i = 0; i < cantidad; i++)
    {
        escribirEEPROM(
            direccionInicial + i,
            datos[i]
        );
    }
}
```

### Parámetros de la función

| Parámetro          | Función                                       |
| ------------------ | --------------------------------------------- |
| `datos[]`          | Arreglo que contiene los números.             |
| `cantidad`         | Número de elementos que se almacenarán.       |
| `direccionInicial` | Primera dirección de EEPROM que se utilizará. |

Si `direccionInicial` contiene el valor `20`, las direcciones calculadas serán:

```
i = 0 → 20 + 0 = 20
i = 1 → 20 + 1 = 21
i = 2 → 20 + 2 = 22
i = 3 → 20 + 3 = 23
```

## Función para leer varios números

La lectura utiliza el mismo principio. En cada repetición se lee una dirección consecutiva y el resultado se almacena en un elemento del arreglo.&#x20;

## Reto 1. Programa completo: almacenamiento de varios números

Complete el siguiente programa que integra las funciones de escritura y lectura de un byte con las funciones para almacenar varios números. Debe terminar la función `leerNumeros`

```cpp
#include <avr/io.h>
#include <avr/interrupt.h>

const uint16_t DIRECCION_INICIAL = 20;
const uint8_t CANTIDAD_NUMEROS = 4;

void escribirEEPROM(
    uint16_t direccion,
    uint8_t dato)
{
    while (EECR & (1 << EEPE))
    {
    }

    EEAR = direccion;
    EEDR = dato;

    uint8_t estadoAnterior = SREG;

    cli();

    EECR |= (1 << EEMPE);
    EECR |= (1 << EEPE);

    SREG = estadoAnterior;

    while (EECR & (1 << EEPE))
    {
    }
}

uint8_t leerEEPROM(uint16_t direccion)
{
    while (EECR & (1 << EEPE))
    {
    }

    EEAR = direccion;
    EECR |= (1 << EERE);

    return EEDR;
}

void guardarNumeros(
    const uint8_t datos[],
    uint8_t cantidad,
    uint16_t direccionInicial)
{
    for (uint8_t i = 0; i < cantidad; i++)
    {
        escribirEEPROM(
            direccionInicial + i,
            datos[i]
        );
    }
}

void leerNumeros(
    uint8_t datos[],
    uint8_t cantidad,
    uint16_t direccionInicial)
{
    for (uint8_t i = 0; i < cantidad; i++)
    {
        //TODO leer datos de la EEPROM. 
        //Tip: Revise las funciones que usan para leer un dato. 
    }
}

void setup()
{
    Serial.begin(9600);

    uint8_t numeros[CANTIDAD_NUMEROS] =
        {25, 100, 7, 240};

    uint8_t recuperados[CANTIDAD_NUMEROS];

    guardarNumeros(
        numeros,
        CANTIDAD_NUMEROS,
        DIRECCION_INICIAL
    );

    leerNumeros(
        recuperados,
        CANTIDAD_NUMEROS,
        DIRECCION_INICIAL
    );

    Serial.println("Datos recuperados:");

    for (uint8_t i = 0;
         i < CANTIDAD_NUMEROS;
         i++)
    {
        Serial.print("Direccion ");
        Serial.print(DIRECCION_INICIAL + i);
        Serial.print(": ");
        Serial.println(recuperados[i]);
    }
}

void loop()
{
}
```

### Resultado esperado

```
Datos recuperados:
Direccion 20: 25
Direccion 21: 100
Direccion 22: 7
Direccion 23: 240
```

### Prueba de persistencia

Después de ejecutar el programa:

1. Compruebe que aparecen los cuatro valores.
2. Comente la llamada a la función de escritura:

```cpp
// guardarNumeros(
//     numeros,
//     CANTIDAD_NUMEROS,
//     DIRECCION_INICIAL
// );
```

3. Cargue nuevamente el programa.
4. Compruebe que los cuatro valores continúan disponibles.

## Relación entre el arreglo y la EEPROM

| Característica             | Arreglo en SRAM                 | EEPROM                                 |
| -------------------------- | ------------------------------- | -------------------------------------- |
| Función                    | Guardar datos temporales        | Guardar datos permanentes              |
| Conserva datos sin energía | No                              | Sí                                     |
| Velocidad                  | Mayor                           | Menor                                  |
| Escrituras limitadas       | No es la preocupación principal | Deben evitarse escrituras innecesarias |

El proceso de almacenamiento puede resumirse así:

> Arreglo en SRAM\
> ↓ `datos[i]`\
> CPU\
> ↓ `direccionInicial + i`\
> EEAR + EEDR + EECR\
> ↓\
> Memoria EEPROM

## Límite de un byte

El tipo de dato `uint8_t` representa valores desde `0` hasta `255`. Este es el mismo intervalo que puede almacenarse en una posición de ocho bits:

```
00000000₂ = 011111111₂ = 255
```

Por esta razón, los números utilizados en el arreglo deben encontrarse entre `0` y `255`.

{% hint style="info" %}
**Para investigar:** Guardar números mayores que 255 requiere dividir el valor entre dos o más bytes. Ese procedimiento no forma parte de este laboratorio.
{% endhint %}

## Entregable 1: Almacenar un PIN desde el teclado

Partiendo del programa desarrollado en el laboratorio anterior, modifique el sistema para capturar un PIN de cuatro dígitos y almacenarlo en direcciones consecutivas de la EEPROM.

{% hint style="danger" %}
**No se proporciona el código resuelto:** El propósito del reto es aplicar el patrón arreglo → ciclo → direcciones consecutivas al teclado matricial y a la pantalla LCD.
{% endhint %}

### Funcionamiento requerido

El programa deberá:

* Leer los números desde el teclado matricial.
* Almacenar temporalmente cuatro dígitos en un arreglo.
* Mostrar un asterisco en el LCD por cada dígito ingresado.
* Impedir que se ingresen más de cuatro dígitos.
* Utilizar `*` para borrar la entrada actual.
* Utilizar `#` para confirmar el PIN.
* Rechazar un PIN que contenga menos de cuatro dígitos.
* Guardar los cuatro dígitos en direcciones consecutivas.
* Mostrar un mensaje al finalizar la escritura.
* Conservar el PIN después de desconectar el Arduino.

### Interfaz esperada

Mientras se ingresa el PIN:

```
Ingrese PIN:****
```

Al presionar `#` después de ingresar cuatro dígitos:

```
PIN guardado 
correctamente
```

Al presionar `#` con menos de cuatro dígitos:

```
PIN incompleto
Ingrese 4 nums
```

Al presionar `*`, el programa deberá borrar el PIN temporal y permitir que el estudiante comience nuevamente.

### Restricciones

* **NO UTILIZAR IA. Los laboratorios contienen la información suficiente que necesita.**&#x20;
* Utilizar las funciones construidas con `EEAR`, `EEDR` y `EECR`.
* Escribir en la EEPROM únicamente cuando se presione `#`.
* No mostrar los números reales del PIN en la pantalla LCD.
* El PIN debe contener exactamente cuatro dígitos.
* Las teclas `A`, `B`, `C` y `D` pueden ignorarse.

Cada equipo deberá presentar:

* Código completo y **bien** **comentado explicando lo que sucede** (subir el archivo .ino en el portal).
* Evidencia del ingreso de cuatro dígitos.
* Evidencia del borrado mediante la tecla `*`.
* Evidencia del almacenamiento mediante la tecla `#`.
* Evidencia del rechazo de un PIN incompleto.
* Evidencia de persistencia después de desconectar y volver a conectar el Arduino.
* Explicación del uso de direcciones consecutivas.

Toda la evidencia mostrada en un solo video. No es necesaria explicación.&#x20;

## Reto adicional opcional

Este reto le servirá para el proyecto final. Recupere el PIN almacenado en la EEPROM y compárelo con un nuevo PIN ingresado desde el teclado.

El sistema deberá mostrar:

```
ACCESO PERMITIDO
```

Si todos los dígitos coinciden.

Si existe al menos una diferencia, deberá mostrar:

```
ACCESO DENEGADO
```
