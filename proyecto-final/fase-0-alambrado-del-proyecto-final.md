# Fase 0: Alambrado del Proyecto de Referencia

A partir de esta etapa del curso se trabajará sobre un único circuito que irá evolucionando hasta convertirse en el **Sistema de Control de Acceso y Seguridad** del proyecto final.

El circuito integra los diferentes dispositivos estudiados durante los laboratorios y permitirá observar cómo un sistema computacional recibe información desde dispositivos de entrada, la procesa, consulta información almacenada en memoria y finalmente controla diferentes dispositivos de salida.

Durante los siguientes laboratorios **no será necesario reconstruir el circuito desde cero**. Cada nueva funcionalidad se podrá incorporar progresivamente.

### Diagrama general del circuito de referencia

El siguiente diagrama realizado en Tinkercad muestra un alambrado base que **podrá usarse como referencia** durante el desarrollo del proyecto. De acuerdo con la variante elegida, _**habrá que modificar la distribución de ciertos pines y componentes así como realizar modificaciones en el código fuente base.**_&#x20;

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

También encontrará un diagrama esquemático con el detalle de los pines en: [https://github.com/memepelle/arduino-security-project/blob/main/pinout.pdf](https://github.com/memepelle/arduino-security-project/blob/main/pinout.pdf)

Es importante respetar la distribución de pines durante su proyecto.&#x20;

El Arduino Uno será el elemento central del sistema y está conectados los siguientes dispositivos:

| Dispositivo           | Función dentro del sistema         |
| --------------------- | ---------------------------------- |
| RFID RC522            | Identificación mediante tarjeta    |
| Teclado matricial 4×4 | Ingreso del PIN                    |
| LCD 16×2 I2C          | Interfaz con el usuario            |
| 74HC595               | Expansión de las salidas digitales |
| LEDs                  | Indicación del estado del sistema  |
| Buzzer pasivo         | Señalización audible               |
| Servo SG90            | Mecanismo de apertura y cierre     |

El `74HC595` se utiliza tanto para controlar las cuatro filas del teclado como para controlar los cuatro LEDs:

| Salida 74HC595 | Dispositivo             | Función                        |
| -------------- | ----------------------- | ------------------------------ |
| **Q0**         | Teclado matricial       | Fila 1                         |
| **Q1**         | Teclado matricial       | Fila 2                         |
| **Q2**         | Teclado matricial       | Fila 3                         |
| **Q3**         | Teclado matricial       | Fila 4                         |
| **Q4**         | LED verde               | Acceso autorizado              |
| **Q5**         | LED rojo                | Acceso denegado / error        |
| **Q6**         | LED amarillo            | Estado de espera / advertencia |
| **Q7**         | LED azul (u otro color) | Estado auxiliar del sistema    |

Las cuatro columnas del teclado estarán conectadas directamente a las entradas `A0–A3` del Arduino.

### Distribución de pines del Arduino Uno

La distribución utilizada es la siguiente:

<table><thead><tr><th width="159.0489501953125">Pin Arduino Uno</th><th width="190.803955078125">Dispositivo / Señal</th><th>Función</th></tr></thead><tbody><tr><td><strong>D0</strong></td><td>Serial RX</td><td>Recepción de comunicación serial</td></tr><tr><td><strong>D1</strong></td><td>Serial TX</td><td>Transmisión de comunicación serial</td></tr><tr><td><strong>D2</strong></td><td>Buzzer pasivo</td><td>Generación de señales audibles</td></tr><tr><td><strong>D3</strong></td><td>Botón de diagnóstico</td><td>Ejecución de pruebas de diagnóstico</td></tr><tr><td><strong>D4</strong></td><td>Servo SG90</td><td>Control del mecanismo de apertura</td></tr><tr><td><strong>D5</strong></td><td>Libre</td><td>Reservado para futuras ampliaciones</td></tr><tr><td><strong>D6</strong></td><td>RFID RST</td><td>Reinicio del módulo RC522</td></tr><tr><td><strong>D7</strong></td><td>74HC595 DATA</td><td>Datos enviados al registro de desplazamiento</td></tr><tr><td><strong>D8</strong></td><td>74HC595 CLOCK</td><td>Señal de reloj del registro de desplazamiento</td></tr><tr><td><strong>D9</strong></td><td>74HC595 LATCH</td><td>Actualización de las salidas del registro</td></tr><tr><td><strong>D10</strong></td><td>RFID SDA / SS</td><td>Selección del módulo RC522 en el bus SPI</td></tr><tr><td><strong>D11</strong></td><td>RFID MOSI</td><td>Datos Arduino → RC522</td></tr><tr><td><strong>D12</strong></td><td>RFID MISO</td><td>Datos RC522 → Arduino</td></tr><tr><td><strong>D13</strong></td><td>RFID SCK</td><td>Señal de reloj del bus SPI</td></tr><tr><td><strong>A0</strong></td><td>Keypad C1</td><td>Columna 1 del teclado matricial</td></tr><tr><td><strong>A1</strong></td><td>Keypad C2</td><td>Columna 2 del teclado matricial</td></tr><tr><td><strong>A2</strong></td><td>Keypad C3</td><td>Columna 3 del teclado matricial</td></tr><tr><td><strong>A3</strong></td><td>Keypad C4</td><td>Columna 4 del teclado matricial</td></tr><tr><td><strong>A4</strong></td><td>LCD SDA</td><td>Datos del bus I²C</td></tr><tr><td><strong>A5</strong></td><td>LCD SCL</td><td>Reloj del bus I²C</td></tr></tbody></table>

### Conexión del módulo RFID RC522

El módulo RFID RC522 no se encuentra disponible para simulación dentro del circuito utilizado en Tinkercad. Por esta razón, deberá agregarse físicamente al montaje siguiendo el siguiente alambrado:

<table><thead><tr><th width="137.6640625">Pin RC522</th><th width="157.0631103515625">Pin Arduino Uno</th><th>Función</th></tr></thead><tbody><tr><td><strong>SDA / SS</strong></td><td><strong>D10</strong></td><td>Selección del módulo RFID en el bus SPI</td></tr><tr><td><strong>SCK</strong></td><td><strong>D13</strong></td><td>Señal de reloj del bus SPI</td></tr><tr><td><strong>MOSI</strong></td><td><strong>D11</strong></td><td>Datos enviados desde Arduino hacia el RC522</td></tr><tr><td><strong>MISO</strong></td><td><strong>D12</strong></td><td>Datos enviados desde el RC522 hacia Arduino</td></tr><tr><td><strong>IRQ</strong></td><td><strong>No conectar</strong></td><td>Interrupción; no será utilizada en el proyecto</td></tr><tr><td><strong>GND</strong></td><td><strong>GND</strong></td><td>Tierra común</td></tr><tr><td><strong>RST</strong></td><td><strong>D6</strong></td><td>Reinicio del módulo RC522</td></tr><tr><td><strong>3.3V</strong></td><td><strong>3.3V</strong></td><td>Alimentación del módulo</td></tr></tbody></table>

El RC522 se comunica con el ATmega328P mediante el bus **SPI**. Por esta razón, las señales `MOSI`, `MISO` y `SCK` utilizan los pines SPI correspondientes del Arduino Uno, mientras que `SDA/SS` se utiliza para seleccionar el dispositivo.

{% hint style="danger" %}
**Atención con la alimentación del RC522:** el módulo debe conectarse a **3.3 V y no a 5 V**.
{% endhint %}

### Código base del proyecto

El código inicial se encuentra disponible en el repositorio de GitHub del curso:

**Código base:** [https://github.com/memepelle/arduino-security-project/tree/main/base](https://github.com/memepelle/arduino-security-project/tree/main/base)

Este programa contiene **la estructura general de este sistema**, incluyendo la configuración de los dispositivos y la máquina de estados que controla este proyecto.

A lo largo del código encontrarán funciones marcadas con `TODO`. Estas funciones deben ser implementadas progresivamente.&#x20;

Por ejemplo:

```cpp
String leerTarjeta()
{
    /*
       TODO - LABORATORIO RFID
    */

    return "";
}
```

El objetivo **no es crear un programa nuevo en cada laboratorio**. El mismo programa base evolucionará durante el resto del curso. Cada laboratorio completará una parte del sistema hasta obtener el proyecto funcional.

#### Importante sobre el programa principal

Las funciones `setup()` y `loop()` proporcionadas en el código base representan la estructura principal del sistema. El trabajo deberá concentrarse en implementar las funciones correspondientes a cada etapa. Esto permitirá que cada nueva funcionalidad pueda probarse utilizando el flujo normal del sistema. De esta forma, al finalizar cada laboratorio no tendrán programas independientes, sino una **nueva versión funcional del mismo sistema computacional**.

### Código de diagnóstico

Además del código base del proyecto, se proporciona un **programa de diagnóstico para este sistema** cuyo propósito es verificar que el circuito se encuentre correctamente armado y que cada uno de sus dispositivos funcione antes de comenzar a depurar la lógica del proyecto.

{% hint style="danger" %}
Este programa es una **herramienta de prueba del hardware** y no forma parte de la solución del proyecto. Por esta razón, **no debe utilizarse como sustituto del código base ni modificarse para implementar las funcionalidades del sistema**.
{% endhint %}

Al ejecutar el programa de diagnóstico se podrán comprobar los principales componentes del circuito, incluyendo:

* LCD 16×2.
* LEDs conectados mediante el registro `74HC595`.
* Teclado matricial 4×4.
* Buzzer pasivo.
* Servo SG90.
* Lector RFID RC522.

El diagnóstico permite distinguir dos tipos de problemas durante el desarrollo.

Por ejemplo, si durante el proyecto una tarjeta RFID no es detectada, primero se deberá ejecutar el código de diagnóstico. Si el diagnóstico tampoco detecta la tarjeta, probablemente existe un problema de **alambrado, alimentación o conexión con el RC522**. Si el diagnóstico funciona correctamente, el problema deberá buscarse en la **implementación realizada en el código del proyecto**.

#### Código de diagnóstico

El programa se encuentra disponible en el repositorio de GitHub del curso:

**Código de diagnóstico:** [https://github.com/memepelle/arduino-security-project/tree/main/diagnostics](https://github.com/memepelle/arduino-security-project/tree/main/diagnostics)

{% hint style="warning" %}
**Importante:** el programa de diagnóstico y el programa base cumplen funciones diferentes. El diagnóstico responde a la pregunta **“¿Funciona correctamente el hardware?”**, mientras que `base.ino` permitirá responder **“¿Funciona correctamente la lógica del sistema?”**.
{% endhint %}

Se recomienda conservar siempre una copia sin modificaciones del código de diagnóstico. Esta podrá utilizarse durante todo el proyecto como un **punto de referencia conocido y funcional** para localizar problemas en el circuito.

### Antes de continuar

Recuerden la diferencia entre el programa diagnóstico y el programa base, si bien código del diagnóstico puede dar idea de cómo funciona el hardware, no basta simplemente con copiarlo y pegarlo para que funcione en el programa base.

Esta distinción será especialmente útil durante la depuración: **primero se comprueba el hardware y después se investiga la lógica del programa**.
