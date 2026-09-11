# Teclado Matricial 4x4

## Objetivos

Al finalizar el laboratorio, el estudiante será capaz de:

* Comprender el funcionamiento de un teclado matricial 4×4.
* Relacionar los pines digitales del Arduino Uno con los puertos físicos del ATmega328P.
* Configurar entradas y salidas utilizando registros.
* Utilizar los registros `DDRx`, `PORTx` y `PINx`.
* Comprender el funcionamiento de las resistencias **pull-up**.
* Manipular registros mediante máscaras y operadores binarios.
* Implementar el escaneo de un teclado matricial sin utilizar `Keypad.h`, `pinMode()`, `digitalWrite()` ni `digitalRead()`.
* Relacionar una entrada física con la información que finalmente procesa la CPU.

## Circuito del laboratorio

Para este laboratorio utilizaremos:

* Arduino Uno.
* Teclado matricial 4×4.
* Pantalla LCD 16×2 con interfaz I²C.
* Protoboard.
* Cables de conexión.

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FsHmJMZtbs70yM0ANv9Np%2Fimage.png?alt=media&amp;token=93f46fbd-1e1c-48e4-84bf-2817ca3abafc" alt=""><figcaption></figcaption></figure>

### Conexión del teclado

Conecte el teclado matricial de la siguiente manera:

| Teclado | Arduino Uno |
| ------- | ----------- |
| F1      | D2          |
| F2      | D3          |
| F3      | D4          |
| F4      | D5          |
| C1      | D6          |
| C2      | D7          |
| C3      | D8          |
| C4      | D9          |

### Conexión de la LCD I²C

| LCD I²C | Arduino Uno |
| ------- | ----------- |
| VCC     | 5 V         |
| GND     | GND         |
| SDA     | A4          |
| SCL     | A5          |

En este laboratorio **sí está permitido utilizar una biblioteca para controlar la LCD I²C**.

Nuestro objeto de estudio será específicamente cómo el microcontrolador obtiene información del teclado utilizando directamente sus registros de entrada/salida.

## Código base del laboratorio

A continuación se proporciona la estructura completa del programa.

{% hint style="danger" %}
**IMPORTANTE:** el código contiene espacios identificados mediante `____________`. El programa **no compilará ni funcionará correctamente hasta que estos espacios sean completados**.&#x20;

A medida que avance por el laboratorio encontrará la información necesaria para determinar qué registro, máscara, bit, función o variable corresponde en cada espacio.&#x20;

**No utilice prueba y error para completar los espacios.** Cada respuesta debe poder justificarse a partir del funcionamiento del hardware.
{% endhint %}

Copie inicialmente todo el siguiente programa en Arduino IDE:

{% code overflow="wrap" %}

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// ==================================================
// LCD I2C
// ==================================================
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ==================================================
// CONFIGURAR TECLADO
// ==================================================
void configurarTeclado()
{
    // D2-D5 = PD2-PD5
    // Configurar como SALIDAS
    DDRD |= 0b00111100;
    
    // D6-D7 = PD6-PD7
    // Configurar como ENTRADAS
    DDRD &= __________________;

    // D8-D9 = PB0-PB1
    // Configurar como ENTRADAS
    DDRB &= __________________;

    // Activar resistencias PULL-UP
    // en PD6 y PD7
    PORTD |= __________________;

    // Activar resistencias PULL-UP
    // en PB0 y PB1
    PORTB |= __________________;

    // Todas las filas inicialmente HIGH
    PORTD |= 0b00111100;
}

// ==================================================
// LEER COLUMNAS
// ==================================================

int leerColumna()
{
    // C1 = D6 = PD6
    if (!(PIND & (1 << PD6)))
    {
        return 0;
    }

    // C2 = D7 = PD7
    if (!(PIND & (1 << ______)))
    {
        return 1;
    }

    // C3 = D8 = PB0
    if (!(PINB & (1 << ______)))
    {
        return 2;
    }

    // C4 = D9 = PB1
    if (!(PINB & (1 << ______)))
    {
        return 3;
    }

    // Ninguna columna activa
    return -1;
}


// ==================================================
// LEER TECLADO
// ==================================================
char leerTeclado()
{
    const char mapa[4][4] = {
        {'1', '2', '3', 'A'},
        {'4', '5', '6', 'B'},
        {'7', '8', '9', 'C'},
        {'*', '0', '#', 'D'}
    };

    for (int fila = 0; fila < 4; fila++)
    {
        // Todas las filas HIGH
        PORTD |= 0b00111100;

        // Solamente la fila actual LOW
        PORTD &= ~(1 << (fila + 2));

        // Esperar estabilización de la señal
        delayMicroseconds(5);

        // Leer las columnas
        int columna = __________________;
        if (columna != -1)
        {
            // Esperar hasta que se libere la tecla
            while (__________________ != -1)
            {
            }
            // Reducir rebote mecánico
            delay(20);
            // Convertir fila + columna
            // al carácter correspondiente
            return mapa[________][________];
        }
    }
    // Ninguna tecla presionada
    return 0;
}

// ==================================================
// CONFIGURACIÓN INICIAL
// ==================================================
void setup()
{
    Serial.begin(9600);
    // Configurar registros utilizados
    // por el teclado
    __________________;

    // Inicializar LCD
    lcd.init();
    lcd.backlight();
    lcd.setCursor(0, 0);
    lcd.print("TECLA DETECTADA");
    lcd.setCursor(0, 1);
}


// ==================================================
// PROGRAMA PRINCIPAL
// ==================================================
void loop()
{
    // Obtener una tecla
    char tecla = __________________;
    if (tecla)
    {
        // Monitor Serial
        Serial.print("Tecla: ");
        Serial.println(tecla);
        // Limpiar segunda línea de LCD
        lcd.setCursor(0, 1);
        lcd.print("                ");
        // Mostrar tecla
        lcd.setCursor(7, 1);
        lcd.print(________);
    }
}
```

{% endcode %}

### Restricciones

Para resolver el laboratorio **no está permitido utilizar**:

```cpp
#include <Keypad.h>
```

Tampoco podrán utilizarse para controlar el teclado:

```cpp
pinMode()
digitalRead()
digitalWrite()
```

La interacción con el teclado deberá realizarse directamente mediante:

```
DDRx
PORTx
PINx
```

La biblioteca de la LCD I²C sí está permitida, ya que la pantalla se utilizará únicamente para visualizar el resultado.

## ¿Cómo funciona un teclado matricial?

Un teclado 4×4 contiene 16 teclas, pero no necesita 16 conexiones independientes.

Las teclas están organizadas en una matriz formada por **cuatro filas y cuatro columnas**:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FLBjfRXicVKPEmCScLvrt%2Fimage.png?alt=media&amp;token=15e8fb6c-aab6-464a-b86f-d33fda1ce05f" alt="" width="375"><figcaption></figcaption></figure>

Cada tecla funciona como un interruptor que conecta una **fila** con una **columna**.

Por ejemplo, al presionar la tecla `6`:

```
Fila    = F2
Columna = C3
```

El microcontrolador no recibe directamente:

```
'6'
```

El hardware únicamente permite detectar:

```
F2 + C3
```

Posteriormente, el software interpreta esa combinación como el carácter `'6'`.

Este proceso puede representarse como:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FGi4g3FhvZPUj3Wq1nyxS%2Fimage.png?alt=media&amp;token=a98b1705-951e-4b2c-a0f0-3422645cc421" alt="" width="375"><figcaption></figcaption></figure>

## Arduino Uno y ATmega328P

Cuando programamos normalmente un Arduino utilizamos nombres como:

```
D2
D3
D4
D5
...
```

Estos nombres forman parte de la abstracción proporcionada por la plataforma Arduino. Internamente, el ATmega328P organiza sus pines mediante **puertos de entrada/salida**.

Para nuestro circuito, la correspondencia es:

```
Arduino              ATmega328P

D2  ───────────────► PD2
D3  ───────────────► PD3
D4  ───────────────► PD4
D5  ───────────────► PD5

D6  ───────────────► PD6
D7  ───────────────► PD7

D8  ───────────────► PB0
D9  ───────────────► PB1
```

Por lo tanto, para controlar el teclado utilizaremos principalmente los puertos:

```
PORTD
PORTB
```

Observe que el teclado **no cabe completamente dentro de un solo puerto**.

Las filas y las primeras dos columnas utilizan el puerto D, mientras que las últimas dos columnas utilizan el puerto B.

## Registros de entrada/salida

Para controlar directamente los puertos del ATmega328P utilizaremos tres tipos de registros:

```
DDRx
PORTx
PINx
```

La letra `x` identifica el puerto.

Por ejemplo:

```
PUERTO D

DDRD
PORTD
PIND
```

y:

<pre><code>PUERTO B
<strong>
</strong><strong>DDRB
</strong>PORTB
PINB
</code></pre>

Cada registro cumple una función diferente.

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2Feu2jKRQJXCgZmtisRpcH%2Fimage.png?alt=media&amp;token=5f98a243-1360-47ee-b895-b425aad5d54d" alt=""><figcaption></figcaption></figure>

## Operadores binarios

Los registros están formados por varios bits. Normalmente no queremos modificar todos los bits de un registro. Queremos modificar **solamente determinados bits** sin alterar los demás. Para ello utilizaremos operadores **bit a bit** o *bitwise*.

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FbXSr4MmvEiRlyYo5f861%2Fimage.png?alt=media&amp;token=79532451-0082-4c6d-aeed-f7dc84b7bc52" alt=""><figcaption></figcaption></figure>

## Resistencia pull-up

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FIwC5tHsouQ3fJm5p1guQ%2Fimage.png?alt=media&amp;token=1a66cad7-3f39-4430-b4f6-98b936b10f63" alt=""><figcaption></figcaption></figure>

### Pull-up interno del ATmega328P

El ATmega328P posee resistencias pull-up internas. Por ello no necesitaremos colocar resistencias externas para cada columna. Cuando un pin está configurado como **entrada**, escribir un `1` en el correspondiente bit de `PORTx` habilita su resistencia pull-up interna.

## Configuración de filas y columnas

Para poder escanear el teclado necesitamos que las filas sean las salidas y las columnas sean las entradas. ¿Por qué? Porque la CPU debe **controlar las filas** y **leer las columnas**.

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2Fv788P4xZUhUQPF6FG8K8%2Fimage.png?alt=media&amp;token=bcbf6e01-b089-4d95-a05b-186d31e896b6" alt=""><figcaption></figcaption></figure>

Regrese vaya a la función:

```cpp
void configurarTeclado()
```

del código inicial.

Complete las cuatro máscaras faltantes.

Antes de escribir cada respuesta, dibuje:

```
Bit:  7  6  5  4  3  2  1  0
```

e identifique cuáles bits deben quedar en `0` y cuáles en `1`.

{% hint style="danger" %}
**No utilice prueba y error.**
{% endhint %}

## ¿Cómo encuentra la CPU una tecla?

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FN6BN71q9Ce7emJpXiKTb%2Fimage.png?alt=media&amp;token=78f4e846-8d70-4f51-ad44-57fb3a56e9b2" alt=""><figcaption></figcaption></figure>

## Lectura de las columnas mediante PINx

Para consultar un bit específico utilizamos una máscara.

Por ejemplo:

```cpp
PIND & (1 << PD6)
```

Primero:

```
1 << PD6
01000000
```

Supongamos que `PIND` contiene: 10110101&#x20;

Realizamos:

```
PIND       10110101
Máscara    01000000           
           ──────── AND
Resultado  00000000
```

De esta forma hemos aislado el estado de PD6. Como utilizamos resistencias pull-up, una tecla presionada produce un `0`.

Por eso encontramos:

```cpp
if (!(PIND & (1 << PD6)))
```

El operador `!` representa una negación lógica.

Conceptualmente podemos interpretar esta condición como **¿PD6 se encuentra en LOW?** Si es así, la columna C1 está activa.

Vaya ahora a:

```cpp
int leerColumna()
```

y complete los bits correspondientes a C2 C3 C4

Recuerde

```
C1 → PD6
C2 → PD7
C3 → PB0
C4 → PB1
```

## Decodificación de la tecla

Una vez identificadas la fila y la columna necesitamos convertir esa posición física en un carácter.

Para ello utilizamos:

```cpp
const char mapa[4][4] = {{'1','2','3','A'},    
                         {'4','5','6','B'},    
                         {'7','8','9','C'},    
                         {'*','0','#','D'}};
```

Este arreglo representa la distribución física del teclado. Por ejemplo, la fila = 1 y la columna 2 serían la tecla '6', de acuerdo al mapa. Recuerde que los índices de los arreglos comienzan en `0`.

Regrese ahora a la función:

```cpp
leerTeclado()
```

y complete:

* La función necesaria para leer las columnas.
* La condición que espera que la tecla sea liberada.
* Los índices utilizados para acceder a `mapa`.

### ¿Por qué aparece `fila + 2`?

Dentro del código encontramos:

```cpp
PORTD &= ~(1 << (fila + 2));
```

¿Por qué no utilizamos simplemente `fila`?

Observe la correspondencia:

```
Fila lógica       Bit de PORTD
     0       →        PD2
     1       →        PD3
     2       →        PD4
     3       →        PD5
```

Los índices de nuestro ciclo comienzan en `0`, pero las filas comienzan físicamente en PD2.

Por eso `fila + 2` entonces `1 << (fila + 2)` crea una máscara para la fila que estamos revisando.

Posteriormente `~` invierte la máscara y `&=` coloca únicamente esa fila en `LOW`.

### Rebote de tecla (*bounce)*

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FuyGHpeD12h7TKaPZIzVy%2Fimage.png?alt=media&amp;token=eb9055e5-364b-4690-bfc3-f1f3379c6fdc" alt=""><figcaption></figcaption></figure>

## Complete la integración

Hasta este punto ya posee la información necesaria para completar el programa.&#x20;

Regrese al código inicial y revise todos los espacios pendientes.

En `setup()` deberá identificar qué función configura el teclado.

En `loop()` deberá identificar:

* Qué función obtiene una tecla.
* Qué variable contiene el carácter detectado.
* Qué dato debe enviarse a la LCD.

Cuando todos los espacios hayan sido completados correctamente, compile el programa.

{% hint style="danger" %}
Si existe un error, **no sustituya el código por funciones de Arduino como `digitalRead()` o `digitalWrite()`**.
{% endhint %}

Revise qué registro, bit o máscara está utilizando.

## Prueba de funcionamiento

Ejecute el circuito y pruebe individualmente las 16 teclas:

```
1   2   3   A
4   5   6   B
7   8   9   C
*   0   #   D
```

Al presionar una tecla, la LCD deberá mostrar el carácter detectado.

Por ejemplo, al presionar `6`:

```
TECLA DETECTADA: 6
```

El Monitor Serial también deberá mostrar:

```
Tecla: 6
```

Pruebe las **16 teclas**.

No continúe hasta comprobar que todas funcionan correctamente.

## Trabajo de Laboratorio

Hasta este momento el programa solamente muestra la última tecla presionada. Modifique el sistema para permitir que el usuario ingrese una secuencia de **cuatro dígitos numéricos**.

Por ejemplo, si el usuario presiona:

```
2 → 5 → 8 → 0
```

la pantalla deberá mostrar:

```
ENTRADA: 2580
```

El sistema deberá:

* Aceptar únicamente los caracteres `0` a `9`.
* Permitir un máximo de **cuatro dígitos**.
* Mostrar en la LCD los dígitos ingresados.
* Utilizar `*` para borrar toda la entrada y comenzar nuevamente.
* Ignorar `A`, `B`, `C`, `D` y `#` en esta etapa.
* Continuar leyendo el teclado mediante registros.
* Mantener todas las restricciones establecidas en el laboratorio.

No está permitido utilizar:

```cpp
Keypad.h
```

ni:

```cpp
pinMode()
digitalRead()
digitalWrite()
```

### Pista

Hasta ahora solamente hemos necesitado almacenar un carácter:

```cpp
char tecla;
```

Para conservar varios caracteres puede investigar cómo utilizar un arreglo:

```cpp
char entrada[5];
```

También necesitará conocer en qué posición debe guardar el siguiente carácter.

Puede utilizar:

```cpp
byte posicion = 0;
```

**No se proporciona el código del reto.**

Deberá utilizar la función:

```cpp
leerTeclado()
```

desarrollada durante el laboratorio y construir a partir de ella el nuevo comportamiento.

## Preguntas de análisis

Responda las siguientes preguntas justificando técnicamente cada respuesta.

1\. ¿Qué significa `DDRx` y cuál es su función?

2\. ¿Cuál es la diferencia entre los registros DDRx PORTx PINx?

3\. ¿Por qué las filas del teclado se configuraron como **salidas** y las columnas como **entradas**?

4\. ¿Qué es una entrada flotante y qué problema podría ocasionar?

5\. ¿Qué función cumple una resistencia **pull-up**?

6\. ¿Por qué en nuestro circuito una tecla presionada se detecta como `LOW` y una tecla libre como `HIGH`?

7\. Explique bit por bit qué ocurre en:

```
DDRD |= 0b00111100;
```

Indique específicamente qué bits son afectados.

8. Explique paso a paso qué realiza:

```
PORTD &= ~(1 << (fila + 2));
```

Incluya en su explicación los operadores:

```
<< ~ & =
```

9. Si el usuario presiona la tecla `6`, explique el proceso completo desde el punto de vista del hardware y del software hasta obtener el carácter `'6'`.

10\. ¿Por qué un teclado con **16 teclas** puede funcionar utilizando solamente **8 líneas** de conexión?

## Entregables

### Entregable 1 — Código completado y preguntas

Entregue:

* Código fuente con todos los espacios correctamente completados.
* Respuestas a las preguntas de análisis.
* Justificación de las máscaras binarias utilizadas.

El estudiante deberá ser capaz de explicar qué bits modifica cada máscara.

### Entregable 2 — Funcionamiento del teclado

Demuestre con un video la lectura correcta de las **16 teclas**:

```
1  2  3  A
4  5  6  B
7  8  9  C
*  0  #  D
```

La tecla deberá visualizarse correctamente tanto en:

* LCD.
* Monitor Serial.

La lectura deberá realizarse mediante los registros estudiados.

### Entregable 3 — Código con funcionalidad extendida

Demuestre con un video el funcionamiento del reto:

* Ingreso de 4 dígitos
* Visualización en LCD
* Borrado mediante \*

El código deberá mantener todas las restricciones del laboratorio.

## Conclusión

En este laboratorio utilizamos un teclado matricial como dispositivo de entrada, pero el objetivo principal no fue aprender a utilizar un teclado.

El objetivo fue comprender el recorrido:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FqbFTCmPJ3VZZdIYVO8W9%2Fimage.png?alt=media&amp;token=1629a89e-51f1-41ed-9db3-a0e4b51491e5" alt=""><figcaption></figcaption></figure>

También observamos que los registros DDRx PORTx PINx permiten controlar directamente la interfaz entre el procesador y los dispositivos externos.

{% hint style="info" %}
Las funciones de alto nivel de Arduino facilitan enormemente el desarrollo de aplicaciones. Sin embargo, trabajar directamente con los registros nos permite comprender **qué ocurre realmente dentro del microcontrolador cuando una computadora recibe información del mundo exterior**.
{% endhint %}

En el siguiente laboratorio utilizaremos la secuencia de dígitos obtenida en el reto para estudiar otro componente fundamental de una computadora: **la memoria y el almacenamiento de información**.