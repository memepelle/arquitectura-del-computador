# ADC + PWM con Arduino

## Introducción

Hasta este punto hemos estudiado cómo un sistema de cómputo procesa información representada mediante valores digitales. Sin embargo, el mundo físico que rodea a un computador no es exclusivamente digital. Magnitudes como temperatura, iluminación, presión, sonido o posición pueden variar de manera continua.

Un microcontrolador necesita mecanismos que permitan relacionar estas señales físicas con los valores binarios que puede procesar internamente.

En este laboratorio se estudiarán dos mecanismos importantes:

* **ADC (Analog-to-Digital Converter): Convertidor Analógico-Digital.**
* **PWM (Pulse Width Modulation): Modulación por Ancho de Pulso.**

Utilizando un **Arduino Uno**, un potenciómetro, un LED y una pantalla LCD 16×2 con comunicación I²C, construiremos el siguiente sistema:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2Fx8P2V9nmxkq1ym3xOSV7%2Fimage.png?alt=media&amp;token=1102ff56-39b1-4a00-8e79-53bd46d27533" alt=""><figcaption></figcaption></figure>

El objetivo no es únicamente controlar el brillo de un LED, sino comprender **cómo un computador recibe información del mundo físico, la representa digitalmente, la procesa y finalmente produce una salida**.

## Objetivos

Al finalizar el laboratorio, el estudiante será capaz de:

* Diferenciar una señal analógica de una señal digital.
* Comprender la función de un ADC dentro de un microcontrolador.
* Interpretar la resolución de un ADC de 10 bits.
* Relacionar un voltaje de entrada con su representación digital.
* Identificar registros relacionados con el ADC del ATmega328P.
* Comprender el funcionamiento de PWM.
* Relacionar PWM con los temporizadores internos del microcontrolador.
* Comprender la función de registros de control y comparación.
* Diferenciar PWM de una verdadera salida analógica.
* Comprender el concepto de **Duty Cycle o ciclo de trabajo**.
* Explicar por qué un LED controlado mediante PWM parece cambiar de brillo.
* Convertir información representada con 10 bits a una representación de 8 bits.
* Utilizar comunicación I²C para mostrar información en una pantalla LCD.
* Relacionar periféricos de entrada y salida con el funcionamiento interno del microcontrolador.

## Materiales

Para realizar el laboratorio se utilizarán:

* 1 Arduino Uno R3.
* 1 protoboard.
* 1 potenciómetro de 10 kΩ.
* 1 LED.
* 1 resistencia de 220 Ω.
* 1 pantalla LCD 16×2 con módulo I²C.
* Cables de conexión.
* Cable USB.

Todos estos componentes forman parte del kit utilizado durante el curso.

## ¿Qué es una señal analógica?

Una **señal analógica** puede tomar una gran cantidad de valores dentro de un determinado intervalo.

Por ejemplo, el voltaje producido por nuestro potenciómetro podría encontrarse aproximadamente en:

```
0 V
1.27 V
2.03 V
2.51 V
3.84 V
4.72 V
5 V
```

No estamos limitados únicamente a dos estados.

Esto es diferente de una señal digital, donde normalmente trabajamos con:

```
LOW  → 0
HIGH → 1
```

El problema es que el procesador trabaja internamente utilizando información digital.

Necesitamos entonces convertir:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FBYewMruKue5BDS1fyEJJ%2Fimage.png?alt=media&amp;token=2ccd5748-721a-4e4d-a0ce-91886892bad9" alt=""><figcaption></figcaption></figure>

## ADC: Analog-to-Digital Converter

**ADC** significa **Analog-to-Digital Converter** o **Convertidor Analógico-Digital.**

Su función es convertir una señal analógica, como un voltaje, en un número que pueda ser procesado digitalmente. El Arduino Uno utiliza un microcontrolador **ATmega328P**, que dispone de un ADC con una resolución de **10 bits**.

Con 10 bits pueden representarse:

```
2¹⁰ = 1024 niveles
```

Por lo tanto:

```
Valor mínimo = 0
Valor máximo = 1023
```

En binario:

```
0000000000 =    0

...

1111111111 = 1023
```

Por esta razón, cuando ejecutamos:

```cpp
analogRead(A0);
```

Arduino devuelve un número comprendido entre 0 y 1023.

## ¿Qué ocurre internamente cuando usamos `analogRead()`?

La instrucción:

```cpp
analogRead(A0);
```

parece sencilla, pero internamente intervienen varios componentes del microcontrolador.

Una representación simplificada es:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2Fe962i74LXbTXBWt3OsbY%2Fimage.png?alt=media&amp;token=5587aad4-aea1-4610-9071-5ac23f71a4bd" alt=""><figcaption></figcaption></figure>

El resultado de la conversión no aparece mágicamente dentro de una variable de C/C++. Primero queda almacenado en **registros de hardware**.

## Registros relacionados con el ADC

Un **registro** es una pequeña ubicación de almacenamiento dentro del microcontrolador utilizada para guardar datos o controlar hardware. Para el ADC del ATmega328P existen varios registros importantes.

### ADMUX

**ADMUX** significa **ADC Multiplexer Selection Register** o **Registro de Selección del Multiplexor del ADC**.

Este registro permite, entre otras funciones, seleccionar **qué entrada analógica será convertida**.

Conceptualmente:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FIXkdopVXwKfJKVuVtDpW%2Fimage.png?alt=media&amp;token=b466182b-c2d3-4db7-acc3-eef68ba187d5" alt=""><figcaption></figcaption></figure>

Cuando utilizamos:

```cpp
analogRead(A0);
```

Arduino configura internamente la selección correspondiente para que el ADC observe A0.

### ADCSRA

**ADCSRA** significa **ADC Control and Status Register A** o **Registro A de Control y Estado del ADC**.

Contiene bits utilizados para controlar el ADC.

Entre sus funciones se encuentran:

* Habilitar el ADC.
* Iniciar una conversión.
* Indicar cuándo una conversión ha terminado.
* Configurar aspectos relacionados con su funcionamiento.

Por tanto, una instrucción de alto nivel como:

```cpp
analogRead(A0);
```

Termina provocando operaciones sobre registros internos del microcontrolador.

### ADCL y ADCH

El ADC produce **10 bits**, pero el ATmega328P es un microcontrolador de arquitectura de datos de 8 bits.

Por ello el resultado se distribuye utilizando dos registros:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FGGzPyp9p1mWbuasr2dbR%2Fimage.png?alt=media&amp;token=3c4e77da-b7d7-4a00-9900-92865f733190" alt=""><figcaption></figcaption></figure>

**ADCL** significa **ADC Data Register Low**.

**ADCH** significa **ADC Data Register High**.

Arduino se encarga de leer estos registros y reconstruir el valor que finalmente recibimos en:

```cpp
int valorADC = analogRead(A0);
```

Por ejemplo:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FXEaT1PIPKrOFm5vEvhCM%2Fimage.png?alt=media&amp;token=b8ebeb73-6d1c-4a59-9550-e9032af0f3de" alt=""><figcaption></figcaption></figure>

Esta es una distinción importante:

```
valorADC
```

es una **variable del programa**. Mientras que:

```
ADMUX
ADCSRA
ADCL
ADCH
```

son **registros de hardware del microcontrolador**.

## Relación entre voltaje y ADC

En nuestro experimento utilizaremos aproximadamente el intervalo de **0 V a 5 V** que será convertido **a 0 - 1023.** Podemos visualizar algunos valores aproximados:

| Voltaje |  ADC |
| ------: | ---: |
|     0 V |    0 |
|  1.25 V |  256 |
|  2.50 V |  512 |
|  3.75 V |  767 |
|  5.00 V | 1023 |

Por ejemplo:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FrFYgEMe6ZQz4xiKz4ACE%2Fimage.png?alt=media&amp;token=9f0a730e-0c67-4034-baf1-78ea26e9eb40" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Observe algo importante: **el procesador no necesita trabajar con “2.5 V” internamente**. El ADC transforma esa magnitud física en una representación numérica que posteriormente puede ser procesada.
{% endhint %}

## El potenciómetro

Un potenciómetro es una resistencia variable.

Posee tres terminales:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FeGNNK61iFe1KECsK4BbB%2Fimage.png?alt=media&amp;token=9474cc4e-d382-4e5b-84fa-ea59624c2e9e" alt="" width="375"><figcaption></figcaption></figure>

En nuestro circuito conectaremos:

```
Terminal exterior ─────► 5 V
Terminal central  ─────► A0
Terminal exterior ─────► GND
```

Al girar el potenciómetro, el voltaje presente en el terminal central cambiará aproximadamente entre 0 V y 5 V. Ese voltaje será recibido por **A0**, una de las entradas analógicas del Arduino.

## PWM: Pulse Width Modulation

Una vez que Arduino ha recibido y procesado la información necesitamos producir una salida. Para controlar el brillo del LED utilizaremos **PWM**. PWM significa **Pulse Width Modulation** o **Modulación por Ancho de Pulso.** Aquí es fundamental comprender que PWM **no produce diferentes voltajes continuos en el pin**.

Cuando escribimos:

```cpp
analogWrite(9, 127);
```

Podría parecer intuitivo pensar:

```
5 V / 2 ≈ 2.5 V
```

Y concluir que Arduino está aplicando 2.5 V al LED. **Eso no es lo que ocurre.**

El pin continúa produciendo únicamente dos niveles:

```
LOW  ≈ 0 V
HIGH ≈ 5 V
```

Lo que Arduino modifica es el **tiempo que permanece en cada estado**.

## El LED realmente estará parpadeando

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FJmTK9etUeR3lPrVjYdM8%2Fimage.png?alt=media&amp;token=dc610dde-9601-488b-9bdc-66248327fccd" alt=""><figcaption></figcaption></figure>

### Entonces, ¿por qué parece más tenue?

Nuestro sistema visual no responde instantáneamente a cada cambio rápido de iluminación. Cuando el LED se enciende y apaga suficientemente rápido, no percibimos fácilmente cada pulso individual.

En lugar de observar:

```
ON OFF ON OFF ON OFF ON OFF...
```

Percibimos una sensación de iluminación aproximadamente relacionada con la fracción de tiempo durante la cual el LED permanece encendido.

Por ejemplo:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2Fo6DAuQ19BQGs6JZb7tNj%2Fimage.png?alt=media&amp;token=9d7fe472-0966-47c7-acc8-5d7334eddeff" alt=""><figcaption></figcaption></figure>

### Duty Cycle

**Duty Cycle** significa **ciclo de trabajo**. Indica qué porcentaje de un período permanece la señal en HIGH. Podemos expresarlo como:

```
             tiempo en HIGH
Duty Cycle = ─────────────── × 100 %
              período total
```

### ¿Quién genera esos pulsos?

Podríamos intentar generar PWM manualmente:

```cpp
digitalWrite(9, HIGH);
delay(...);

digitalWrite(9, LOW);
delay(...);
```

Pero eso obligaría al procesador a ejecutar continuamente instrucciones para encender y apagar el LED. El ATmega328P dispone de hardware especializado denominado **Timer/Counter**. Un **Timer/Counter** es un periférico interno capaz de contar pulsos de reloj. Esto permite generar PWM mediante hardware.

Conceptualmente:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FlS9W3aAFpdFB2CCBbh7a%2Fimage.png?alt=media&amp;token=3376f30c-fc79-40f8-a962-475bdbba3dac" alt="" width="375"><figcaption></figcaption></figure>

Una vez configurado, el temporizador puede continuar generando los pulsos sin que nuestro programa tenga que ejecutar manualmente cada transición HIGH/LOW.

{% hint style="info" %}
Esto demuestra una característica fundamental de un microcontrolador: **La CPU no realiza directamente todas las tareas. Existen periféricos de hardware especializados que trabajan junto con ella.**
{% endhint %}

### D9 y Timer1

En nuestro laboratorio utilizamos:

```cpp
const int LED = 9;
```

En el Arduino Uno, el pin **D9** puede utilizar la salida PWM asociada al **Timer/Counter1** del ATmega328P.

Por eso Arduino identifica este pin con el símbolo:

```
~
```

La salida correspondiente está relacionada con **OC1A**:

**Output Compare 1 A**

De manera conceptual:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2F0FrehR83IOEZVM4QeG92%2Fimage.png?alt=media&amp;token=9a524d33-0b7d-484c-81c1-297df226ec0c" alt="" width="375"><figcaption></figcaption></figure>

## Registros involucrados en PWM

Existen varios registros asociados al Timer1. Para este laboratorio interesa comprender principalmente tres conceptos.

### TCCR1A

**Timer/Counter1 Control Register A** es uno de los registros utilizados para configurar el comportamiento del Timer1 y de sus salidas.

### TCCR1B

**Timer/Counter1 Control Register B** complementa la configuración del temporizador, incluyendo aspectos relacionados con el reloj utilizado por el contador y su modo de operación.

### OCR1A

**Output Compare Register 1 A** es un registro de comparación asociado con la salida OC1A. El temporizador cuenta y el hardware compara su valor con registros configurados para determinar cuándo debe cambiar el estado de la salida.

Conceptualmente:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FvZCRRuXCFV86PGpJceXH%2Fimage.png?alt=media&amp;token=6ab654b4-2fbb-4898-a271-1047f46cad9d" alt="" width="375"><figcaption></figcaption></figure>

La implementación concreta utilizada por el núcleo de Arduino configura estos recursos automáticamente cuando usamos sus funciones.

## `analogWrite()` ya no es una caja negra

Cuando escribimos:

```cpp
analogWrite(9, valorPWM);
```

Desde C/C++, vemos una sola línea.

Pero conceptualmente estamos solicitando:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FqgdI5pvQPZElIt1JLlAN%2Fimage.png?alt=media&amp;token=7a716405-aa00-4d4c-bfb1-a4da00858736" alt=""><figcaption></figcaption></figure>

Esta relación entre **software y hardware** es precisamente uno de los objetivos de Arquitectura del Computador.

## Un problema interesante: 10 bits contra 8 bits

El ADC produce:

```
ADC
10 bits
0 – 1023
```

Mientras que el valor utilizado por `analogWrite()` en nuestro programa se expresa en:

```
PWM
8 bits
0 – 255
```

Tenemos entonces:

```
ADC                         PWM

10 bits                     8 bits

0 ─────── 1023      →      0 ─────── 255

0000000000                  00000000
     ↓                          ↓
1111111111                  11111111
```

Por lo tanto necesitamos realizar una transformación:

```cpp
int valorPWM = map(valorADC, 0, 1023, 0, 255);
```

Esto transforma proporcionalmente:

```
ADC             PWM

0       ─────►   0
256     ─────►  63
512     ─────► 127
768     ─────► 191
1023    ─────► 255
```

Estamos pasando de **1024 posibles valores de entrada a 256 posibles valores de salida**.

Por tanto, varios valores distintos del ADC terminarán correspondiendo al mismo nivel PWM.

## Pantalla LCD e I²C

Para observar lo que ocurre utilizaremos una pantalla **LCD 16×2**. **LCD** significa **Liquid Crystal Display**, o pantalla de cristal líquido. Nuestra pantalla incorpora comunicación **I²C (Inter-Integrated Circuit)**. Las dos señales principales son:

* **SDA — Serial Data:** datos.
* **SCL — Serial Clock:** reloj.

En Arduino Uno:

```
A4 → SDA
A5 → SCL
```

Esto permite que el microcontrolador transmita información al LCD utilizando un bus serial en lugar de dedicar numerosos pines independientes a la pantalla.

## Construcción del circuito

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FmCUOIN9Nk5Uixj9XQjSH%2Fimage.png?alt=media&amp;token=b5322473-12cb-4824-b97c-457704870d14" alt=""><figcaption></figcaption></figure>

### Potenciómetro

```
5V  ─────► extremo
A0  ─────► centro
GND ─────► extremo
```

### LED PWM

```
D9 ── 220 Ω ──► LED ──► GND
```

### LCD I²C

```
LCD                 Arduino Uno

VCC ───────────────► 5V
GND ───────────────► GND
SDA ───────────────► A4
SCL ───────────────► A5
```

## Programa

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int POT = A0;
const int LED = 9;

void setup()
{
  pinMode(LED, OUTPUT);

  Serial.begin(9600);

  lcd.init();
  lcd.backlight();
}

void loop()
{
  // 1. ADC: convertir el voltaje de A0
  // en un valor digital de 10 bits.
  int valorADC = analogRead(POT);

  // 2. Convertir 0-1023 a 0-255.
  int valorPWM = map(valorADC, 0, 1023, 0, 255);

  // 3. Estimar el voltaje de entrada.
  float voltaje = valorADC * (5.0 / 1023.0);

  // 4. Configurar el ciclo de trabajo PWM.
  analogWrite(LED, valorPWM);

  // 5. Mostrar datos.
  lcd.setCursor(0, 0);
  lcd.print("ADC:");
  lcd.print(valorADC);
  lcd.print("       ");

  lcd.setCursor(0, 1);
  lcd.print("PWM:");
  lcd.print(valorPWM);
  lcd.print(" ");
  lcd.print(voltaje, 1);
  lcd.print("V   ");

  Serial.print("ADC: ");
  Serial.print(valorADC);

  Serial.print(" PWM: ");
  Serial.print(valorPWM);

  Serial.print(" Voltaje: ");
  Serial.println(voltaje);

  delay(200);
}
```

## De la instrucción al hardware

Ahora podemos reinterpretar dos líneas fundamentales del programa.

Cuando ejecutamos:

```cpp
int valorADC = analogRead(POT);
```

Podemos pensar:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FcPEcwRzjgLmVxHO2LlxT%2Fimage.png?alt=media&amp;token=2c48e877-05e9-4ba5-bf1d-eea16bba7291" alt="" width="375"><figcaption></figcaption></figure>

Mientras que:

```cpp
analogWrite(LED, valorPWM);
```

puede visualizarse como:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FJuNMnylhsWzBRWDyQo8H%2Fimage.png?alt=media&amp;token=d986dab5-c74b-4936-9898-075a7a4f587e" alt="" width="375"><figcaption></figcaption></figure>

Ahora las funciones de Arduino dejan de ser simples instrucciones abstractas: representan operaciones realizadas por hardware específico dentro del microcontrolador.

## Prueba y observación

Gire lentamente el potenciómetro.

Observe simultáneamente:

* Valor ADC.
* voltaje.
* Valor PWM.
* Brillo aparente del LED.

Aproximadamente:

```
ADC = 0
PWM = 0
Duty Cycle ≈ 0 %
LED apagado
```

```
ADC ≈ 512
PWM ≈ 127
Duty Cycle ≈ 50 %

El LED NO recibe constantemente 2.5 V.

Recibe aproximadamente:

5V → 0V → 5V → 0V → 5V → 0V ...
```

Finalmente:

```
ADC ≈ 1023
PWM = 255
Duty Cycle = 100 %

LED permanentemente encendido
```

## Entregable — Sistema de indicadores por niveles

Una vez que el circuito base funcione, deberá modificar **hardware y software**.

Agregue:

```
D7 → resistencia 220 Ω → LED amarillo → GND

D8 → resistencia 220 Ω → LED rojo → GND
```

Estos dos LEDs utilizarán salidas digitales convencionales, mientras D9 continuará utilizando PWM.

El sistema deberá comportarse de acuerdo con:

|      ADC | Amarillo | Rojo | Estado      |
| -------: | :------: | :--: | ----------- |
|    0–511 |    OFF   |  OFF | NORMAL      |
|  512–767 |    ON    |  OFF | ADVERTENCIA |
| 768–1023 |    OFF   |  ON  | CRÍTICO     |

El LED de D9 deberá **continuar variando su brillo mediante PWM en todos los rangos**. El LCD deberá mostrar el valor ADC, PWM y estado actual.

Ejemplo:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FVnHckda3tTnYk3cAvQU8%2Fimage.png?alt=media&amp;token=4aff7536-c6bf-401d-9aa9-9f9e873cd77d" alt="" width="375"><figcaption></figcaption></figure>

{% hint style="danger" %}
**No se proporciona el código del reto.** El estudiante deberá implementar las condiciones necesarias para determinar cada estado.
{% endhint %}

### Comparación final de las salidas

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FrWcSBRA1sULu61o3PxKY%2Fimage.png?alt=media&amp;token=169e4d81-4c30-4274-9692-04992fb52f28" alt=""><figcaption></figcaption></figure>

El estudiante deberá presentar el sistema completo funcionando físicamente.

Deberá incluir:

1. Potenciómetro conectado al ADC.
2. LCD funcionando mediante I²C.
3. LED principal controlado mediante PWM.
4. LED amarillo para ADVERTENCIA.
5. LED rojo para CRÍTICO.
6. Funcionamiento correcto de los tres rangos.
7. Código completo y comentado.
8. Demostración en vídeo de NORMAL, ADVERTENCIA y CRÍTICO.
9. Explicación de la diferencia entre una salida digital convencional y PWM.
10. Explicación de por qué el LED PWM parece cambiar de brillo aunque el pin continúa generando pulsos HIGH/LOW.

Durante la demostración (en vídeo), el estudiante deberá ser capaz de explicar el recorrido:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FFVPloEi5dWD0faOKNyTO%2Fimage.png?alt=media&amp;token=e55f75fb-dd21-424a-b45e-10d64bb0518c" alt="" width="375"><figcaption></figcaption></figure>

### Preguntas de comprensión

El estudiante deberá poder responder:

**¿Por qué el LED conectado a D9 parece estar más tenue cuando utilizamos un PWM de 50 %, si el pin continúa produciendo pulsos de aproximadamente 5 V?**

**¿Qué ventaja existe en utilizar un Timer/Counter para generar PWM en lugar de hacer que la CPU cambie continuamente el pin entre HIGH y LOW?**

**¿Cuál es la diferencia entre una variable del programa como `valorADC` y registros de hardware como ADMUX, ADCSRA, ADCL y ADCH?**

**¿Por qué al convertir el valor ADC de 10 bits a un valor PWM de 8 bits necesariamente se pierde resolución?**