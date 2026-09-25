# Laboratorio Monitoreo ambiental con DHT11 y LED RGB

## Introducción

Los sistemas computacionales pueden recibir información del entorno mediante sensores. Estos dispositivos transforman una condición física, como la temperatura o la humedad, en datos que pueden ser procesados por un microcontrolador. Después de interpretar esos datos, el sistema puede tomar decisiones y controlar dispositivos de salida.

En este laboratorio se utilizará el sensor DHT11 para medir la temperatura y la humedad del ambiente. El Arduino Uno procesará las mediciones y utilizará un módulo LED RGB para representar visualmente el estado ambiental. Cuando la temperatura alcance un nivel considerado peligroso, también se activará un buzzer.

## Objetivos

Al finalizar el laboratorio, el estudiante será capaz de:

1. Explicar el funcionamiento general del sensor DHT11.
2. Diferenciar una señal analógica de una comunicación digital.
3. Obtener mediciones de temperatura y humedad mediante un Arduino Uno.
4. Interpretar los datos enviados por un sensor digital.
5. Representar diferentes estados mediante un LED RGB.
6. Implementar decisiones a partir de rangos de temperatura.
7. Detectar y manejar errores de lectura.
8. Relacionar sensores, procesamiento y actuadores con la arquitectura de un sistema computacional.

## Materiales

Para realizar el laboratorio se necesita:
* 1 Arduino Uno, un sensor DHT11
* 1 módulo LED RGB
* 1 buzzer 
* 1 protoboard
* Cables de conexión

El kit incluye un módulo LED RGB que puede incorporar resistencias (revise que el componente las tenga). Antes de conectarlo se deben revisar las letras impresas junto a sus terminales. Normalmente aparecen identificadas como `R`, `G`, `B` y `-`. Si se utiliza un LED RGB que no viene montado sobre un módulo, deberán colocarse resistencias de aproximadamente 220 ohmios en los canales rojo, verde y azul.

## El sensor DHT11

### ¿Qué es el DHT11?

El DHT11 es un sensor digital capaz de medir la temperatura y la humedad relativa del ambiente. En su interior contiene un elemento sensible a la humedad, un sensor de temperatura y un pequeño circuito integrado encargado de procesar las mediciones. A diferencia del potenciómetro, el LDR o el sensor LM35, el DHT11 no entrega un voltaje analógico que el Arduino deba convertir mediante el ADC. El sensor realiza internamente la medición y transmite el resultado como una secuencia de bits. Esta diferencia es importante, en un sensor analógico el microcontrolador recibe un nivel de voltaje y debe convertirlo en un valor numérico. En el DHT11, el sensor y el Arduino se comunican mediante un protocolo digital, el Arduino recibe una trama que contiene los valores de humedad y temperatura.

```text
CONDICIÓN AMBIENTAL
         │
         ▼
     SENSOR DHT11
         │
         ▼
CONVERSIÓN INTERNA
         │
         ▼
 TRAMA DIGITAL DE DATOS
         │
         ▼
      ARDUINO UNO
```

### Temperatura y humedad relativa

La temperatura representa qué tan caliente o frío se encuentra el ambiente. El DHT11 expresa esta medición en grados Celsius. La humedad relativa representa la cantidad de vapor de agua presente en el aire en comparación con la cantidad máxima que podría contener a esa temperatura y se expresa mediante un porcentaje. Una humedad relativa del 60 % no significa que el aire esté compuesto por 60 % de agua, sino que contiene aproximadamente el 60 % del vapor de agua que podría almacenar bajo esas condiciones.

La humedad relativa depende de la temperatura y el aire caliente puede contener una mayor cantidad de vapor de agua que el aire frío. Por esta razón, ambos valores deben analizarse como condiciones relacionadas, aunque el sensor los entregue como mediciones separadas.

### Características generales

El DHT11 es apropiado para actividades educativas y sistemas donde no se requiere una gran precisión. Puede medir temperaturas aproximadamente entre 0 y 50 °C y niveles de humedad entre 20 % y 90 %. Sus mediciones cambian lentamente y no debe leerse demasiadas veces por segundo.

Para este laboratorio se dejarán aproximadamente dos segundos entre cada lectura, si se intenta consultar el sensor con demasiada frecuencia, puede devolver valores repetidos o producir una lectura inválida.

El DHT11 no debe utilizarse como dispositivo de seguridad en aplicaciones reales donde una medición incorrecta pueda causar daños. Sin embargo, es adecuado para comprender la comunicación entre un sensor digital y un microcontrolador.

### Comunicación digital del DHT11

#### La trama de 40 bits

Cuando el Arduino solicita una medición, el DHT11 responde enviando una trama digital de 40 bits, estos bits se organizan en cinco grupos.

| Byte | Información |
|---:|---|
| 1 | Parte entera de la humedad |
| 2 | Parte decimal de la humedad |
| 3 | Parte entera de la temperatura |
| 4 | Parte decimal de la temperatura |
| 5 | Byte de comprobación |

En el DHT11, las partes decimales normalmente tienen poca información, ya que el sensor proporciona una resolución limitada. Aun así, el protocolo reserva estos bytes para mantener una estructura definida.

El quinto byte funciona como una suma de comprobación o `checksum`. El sensor calcula este valor a partir de los cuatro bytes anteriores. El Arduino puede realizar la misma operación y comparar el resultado recibido. Si los valores no coinciden, significa que probablemente ocurrió un error durante la comunicación. Esta verificación no corrige los datos dañados. Su función es permitir que el sistema detecte que la información recibida no es confiable.

#### Representación de cada bit

El DHT11 utiliza la duración de los pulsos para representar ceros y unos. Después de una breve señal inicial, cada bit comienza con un pulso bajo. La duración posterior del nivel alto permite identificar el valor transmitido.

Un **pulso alto relativamente corto representa un cero** y un **pulso alto más largo representa un uno**. El microcontrolador debe medir estas duraciones y reconstruir los 40 bits.

En este laboratorio no se programará manualmente todo el protocolo. Se utilizará una biblioteca que se encarga de generar la solicitud, medir los pulsos, reconstruir la trama y comprobar los datos. Sin embargo, es importante comprender que la biblioteca no obtiene mágicamente la temperatura, internamente ejecuta una secuencia de operaciones de entrada, salida, temporización y procesamiento de bits.

#### Preparación de la biblioteca

Para comunicarse con el DHT11 se utilizará la biblioteca `DHT sensor library` de Adafruit. En Arduino IDE, abra el administrador de bibliotecas y busque:

```text
DHT sensor library
```

Instale la biblioteca publicada por Adafruit. El Arduino IDE también puede solicitar la instalación de la dependencia `Adafruit Unified Sensor`. En ese caso, se debe aceptar la instalación de todas las dependencias necesarias.

Después de instalarla, la biblioteca podrá incluirse en el programa mediante la siguiente instrucción:

```cpp
#include <DHT.h>
```

Una biblioteca contiene código previamente desarrollado para realizar tareas específicas. Su uso permite concentrarse en la lógica principal del sistema. Esto no elimina la necesidad de comprender el proceso interno, pero evita tener que implementar nuevamente todas las temporizaciones del protocolo.

### Lectura de temperatura y humedad

#### Conexión del DHT11

Algunos sensores DHT11 vienen montados sobre un pequeño módulo de tres terminales. Estas suelen estar identificadas como `S`, `+` y `-`.

| DHT11 | Arduino Uno |
|---|---|
| S o DATA | D2 |
| + o VCC | 5V |
| - o GND | GND |

Si se utiliza el componente DHT11 individual de cuatro terminales, la conexión puede requerir una resistencia `pull-up` de aproximadamente 10 kohmios entre `DATA` y 5 V. El módulo incluido en muchos kits ya contiene esta resistencia.

Antes de alimentar el circuito se debe revisar cuidadosamente la identificación impresa en el módulo. No todos los fabricantes colocan las terminales en el mismo orden.

### Programa de prueba

```cpp
#include <DHT.h>

const byte PIN_DHT = 2;
const byte TIPO_DHT = DHT11;

DHT dht(PIN_DHT, TIPO_DHT);

void setup() {
  Serial.begin(9600);
  dht.begin();

  Serial.println("Prueba del sensor DHT11");
  Serial.println();
}

void loop() {
  float humedad = dht.readHumidity();
  float temperatura = dht.readTemperature();

  if (isnan(humedad) || isnan(temperatura)) {
    Serial.println("Error al leer el sensor DHT11");
  } else {
    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.println(" grados Celsius");

    Serial.print("Humedad: ");
    Serial.print(humedad);
    Serial.println(" %");
  }

  Serial.println();
  delay(2000);
}
```

### Explicación del programa

Las primeras instrucciones incluyen la biblioteca y definen el pin utilizado para la comunicación. También indican que el sensor conectado pertenece al modelo DHT11.

```cpp
const byte PIN_DHT = 2;
const byte TIPO_DHT = DHT11;

DHT dht(PIN_DHT, TIPO_DHT);
```

La instrucción que comienza con `DHT dht` crea un objeto llamado `dht`. Este objeto representa al sensor dentro del programa. A través de él se ejecutan las funciones necesarias para iniciar el dispositivo y obtener sus mediciones.

En `setup()` se inicia la comunicación serial y se prepara el sensor mediante `dht.begin()`.

La función `readHumidity()` solicita la humedad relativa y la función `readTemperature()` solicita la temperatura en grados Celsius.

```cpp
float humedad = dht.readHumidity();
float temperatura = dht.readTemperature();
```

Las variables se declaran como `float` porque pueden almacenar números con parte decimal.

Cuando una lectura falla, la biblioteca devuelve un valor especial llamado `NaN`, cuyas siglas significan `Not a Number`. Este valor indica que el resultado no representa una medición numérica válida.

La función `isnan()` permite comprobar si una variable contiene este valor. El programa no debe utilizar una lectura inválida para tomar decisiones, porque podría provocar un comportamiento incorrecto.

```cpp
if (isnan(humedad) || isnan(temperatura)) {
  Serial.println("Error al leer el sensor DHT11");
}
```

El operador `||` significa “o”. La condición será verdadera si falla la medición de humedad, la medición de temperatura o ambas.

Finalmente, el programa espera dos segundos antes de solicitar otra medición. Esta pausa es necesaria porque el DHT11 es un sensor relativamente lento.

### Experimentación con el sensor

Abra el monitor serial y configúrelo a 9600 baudios. Espere algunos segundos hasta observar las primeras mediciones. Los valores deberían cambiar lentamente, ya que la temperatura y la humedad de una habitación normalmente no varían de forma inmediata.

Para observar una modificación, puede acercar cuidadosamente la mano al sensor sin tocarlo. También puede soplar suavemente en su dirección. Al hacerlo, probablemente cambiarán tanto la temperatura como la humedad, porque el aire expulsado por una persona contiene calor y vapor de agua.

No se debe acercar fuego, líquidos ni objetos excesivamente calientes al DHT11. El propósito es observar cambios moderados sin dañar el componente.

Registre al menos cinco mediciones separadas por algunos segundos:

| Medición | Temperatura | Humedad |
|---:|---:|---:|
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |
| 5 | | |

Después de completar la tabla, compare los resultados. Si las mediciones permanecen relativamente estables, el sensor probablemente está funcionando correctamente. Si aparecen errores frecuentes, se deben revisar las conexiones, la alimentación, el modelo configurado y el tiempo entre lecturas.

## Control del módulo LED RGB

### Representación de estados mediante colores

El módulo LED RGB contiene tres luces dentro de un mismo dispositivo: roja, verde y azul. Al controlar la intensidad de cada canal se pueden producir diferentes colores.

En este laboratorio, los colores no se utilizarán solamente como decoración. Cada uno representará un estado lógico del sistema:

| Temperatura | Estado | Color |
|---:|---|---|
| Menor de 20 °C | Ambiente frío | Azul |
| Entre 20 y menos de 27 °C | Ambiente normal | Verde |
| Entre 27 y menos de 32 °C | Temperatura elevada | Amarillo |
| 32 °C o más | Alerta | Rojo |
| Error de lectura | Sensor no disponible | Magenta intermitente |

Los límites utilizados tienen una finalidad educativa y pueden modificarse según el ambiente donde se realice la práctica. No deben considerarse límites oficiales de seguridad.

### Conexión del módulo LED RGB

Para el programa de referencia se utilizarán tres pines con capacidad PWM:

| Módulo RGB | Arduino Uno |
|---|---|
| R | D5 |
| G | D6 |
| B | D10 |
| - | GND |

Se han elegido los pines D5, D6 y D10 para evitar interferencias con la función `tone()`. En el Arduino Uno, esta función utiliza internamente el temporizador 2 y puede afectar la señal PWM de los pines D3 y D11 mientras el buzzer está activo.

Si el módulo tiene una terminal marcada con `+` en lugar de `-`, podría tratarse de un dispositivo de ánodo común. En ese caso, el comportamiento de los valores estará invertido y será necesario adaptar la función de control.

### Programa de prueba

```cpp
const byte PIN_ROJO = 5;
const byte PIN_VERDE = 6;
const byte PIN_AZUL = 10;

void configurarColor(byte rojo, byte verde, byte azul) {
  analogWrite(PIN_ROJO, rojo);
  analogWrite(PIN_VERDE, verde);
  analogWrite(PIN_AZUL, azul);
}

void setup() {
  pinMode(PIN_ROJO, OUTPUT);
  pinMode(PIN_VERDE, OUTPUT);
  pinMode(PIN_AZUL, OUTPUT);
}

void loop() {
  configurarColor(255, 0, 0);
  delay(1000);

  configurarColor(0, 255, 0);
  delay(1000);

  configurarColor(0, 0, 255);
  delay(1000);

  configurarColor(255, 100, 0);
  delay(1000);

  configurarColor(255, 0, 255);
  delay(1000);

  configurarColor(0, 0, 0);
  delay(1000);
}
```

La función `configurarColor()` recibe tres valores entre 0 y 255. Cada valor determina la intensidad de uno de los canales. En un módulo de cátodo común, el cero apaga el canal y el valor 255 produce su intensidad máxima.

El color amarillo se obtiene combinando rojo y verde. Se utiliza un valor menor para el verde porque la intensidad percibida de cada canal puede ser diferente. Estos valores pueden ajustarse experimentalmente.

## Integración del sistema

### Estados ambientales

El programa integrado deberá leer el DHT11, interpretar la temperatura y seleccionar un estado. Después actualizará el LED RGB y el buzzer.

```text
LEER EL DHT11
      │
      ▼
¿LECTURA VÁLIDA?
      │
  ┌───┴────┐
  │        │
  NO       SÍ
  │        │
  ▼        ▼
 ERROR   COMPARAR TEMPERATURA
           │
     ┌─────┼────────┬─────────┐
     ▼     ▼        ▼         ▼
   FRÍO  NORMAL  ELEVADA    ALERTA
     │     │        │         │
     ▼     ▼        ▼         ▼
   AZUL  VERDE   AMARILLO   ROJO Y
                              BUZZER
```

### Conexión del buzzer

| Buzzer | Arduino Uno |
|---|---|
| Positivo | D9 |
| Negativo | GND |

El programa utilizará la función `tone()` para generar una señal audible. Esta función resulta apropiada para un buzzer pasivo. Si el kit contiene un buzzer activo, normalmente bastará con utilizar `digitalWrite()` para encenderlo y apagarlo.

### Programa integrado

```cpp
#include <DHT.h>

const byte PIN_DHT = 2;
const byte TIPO_DHT = DHT11;

const byte PIN_ROJO = 5;
const byte PIN_VERDE = 6;
const byte PIN_AZUL = 10;
const byte PIN_BUZZER = 9;

const unsigned long INTERVALO_LECTURA = 2000;

DHT dht(PIN_DHT, TIPO_DHT);

enum EstadoAmbiental {
  SENSOR_ERROR,
  AMBIENTE_FRIO,
  AMBIENTE_NORMAL,
  TEMPERATURA_ELEVADA,
  ALERTA_TEMPERATURA
};

unsigned long ultimaLectura = 0;

void configurarColor(byte rojo, byte verde, byte azul) {
  analogWrite(PIN_ROJO, rojo);
  analogWrite(PIN_VERDE, verde);
  analogWrite(PIN_AZUL, azul);
}

EstadoAmbiental determinarEstado(float temperatura) {
  if (temperatura < 20.0) {
    return AMBIENTE_FRIO;
  }

  if (temperatura < 27.0) {
    return AMBIENTE_NORMAL;
  }

  if (temperatura < 32.0) {
    return TEMPERATURA_ELEVADA;
  }

  return ALERTA_TEMPERATURA;
}

void aplicarEstado(EstadoAmbiental estado) {
  noTone(PIN_BUZZER);

  switch (estado) {
    case SENSOR_ERROR:
      configurarColor(255, 0, 255);
      Serial.println("Estado: error del sensor");
      break;

    case AMBIENTE_FRIO:
      configurarColor(0, 0, 255);
      Serial.println("Estado: ambiente frio");
      break;

    case AMBIENTE_NORMAL:
      configurarColor(0, 255, 0);
      Serial.println("Estado: ambiente normal");
      break;

    case TEMPERATURA_ELEVADA:
      configurarColor(255, 100, 0);
      Serial.println("Estado: temperatura elevada");
      break;

    case ALERTA_TEMPERATURA:
      configurarColor(255, 0, 0);
      tone(PIN_BUZZER, 1000);
      Serial.println("Estado: alerta de temperatura");
      break;
  }
}

void setup() {
  Serial.begin(9600);
  dht.begin();

  pinMode(PIN_ROJO, OUTPUT);
  pinMode(PIN_VERDE, OUTPUT);
  pinMode(PIN_AZUL, OUTPUT);
  pinMode(PIN_BUZZER, OUTPUT);

  configurarColor(0, 0, 0);

  Serial.println("Sistema de monitoreo ambiental");
  Serial.println();
}

void loop() {
  unsigned long tiempoActual = millis();

  if (tiempoActual - ultimaLectura >= INTERVALO_LECTURA) {
    ultimaLectura = tiempoActual;

    float humedad = dht.readHumidity();
    float temperatura = dht.readTemperature();

    if (isnan(humedad) || isnan(temperatura)) {
      aplicarEstado(SENSOR_ERROR);
      Serial.println();
      return;
    }

    Serial.print("Temperatura: ");
    Serial.print(temperatura);
    Serial.println(" grados Celsius");

    Serial.print("Humedad: ");
    Serial.print(humedad);
    Serial.println(" %");

    EstadoAmbiental estadoActual = determinarEstado(temperatura);

    aplicarEstado(estadoActual);
    Serial.println();
  }
}
```

## Análisis del programa integrado

### Separación de responsabilidades

El programa se encuentra dividido en funciones para separar las diferentes responsabilidades. La biblioteca se encarga de la comunicación con el sensor. La función `determinarEstado()` interpreta la temperatura y la función `aplicarEstado()` controla los dispositivos de salida.

Esta organización facilita la comprensión y modificación del programa. Por ejemplo, si posteriormente se cambian los límites de temperatura, solamente será necesario modificar la función `determinarEstado()`. Si se reemplaza el LED RGB por una pantalla, se podrá cambiar `aplicarEstado()` sin alterar la lectura del sensor.

```text
ADQUISICIÓN DE DATOS
   dht.readTemperature()
   dht.readHumidity()
            │
            ▼
PROCESAMIENTO
   determinarEstado()
            │
            ▼
CONTROL DE SALIDAS
     aplicarEstado()
```

### Representación de estados mediante `enum`

Los estados posibles se definen mediante una enumeración:

```cpp
enum EstadoAmbiental {
  SENSOR_ERROR,
  AMBIENTE_FRIO,
  AMBIENTE_NORMAL,
  TEMPERATURA_ELEVADA,
  ALERTA_TEMPERATURA
};
```

Una enumeración permite utilizar nombres descriptivos en lugar de números sin significado aparente. Internamente, el compilador representa los estados mediante valores enteros, pero el programador puede trabajar con nombres que expresan claramente la función de cada estado.

Esta técnica será especialmente importante en el proyecto final. Un sistema de seguridad puede encontrarse esperando una credencial, validando un acceso, abriendo una puerta, generando una alerta o bloqueado por una condición ambiental. Representar explícitamente cada estado facilita controlar el comportamiento general.

### Uso de `millis()`

En el primer programa se utilizó `delay(2000)` para esperar entre las mediciones. Aunque esta solución es sencilla, durante el `delay()` el programa no puede continuar con otras tareas.

El programa integrado utiliza `millis()` para determinar cuándo han transcurrido dos segundos:

```cpp
if (tiempoActual - ultimaLectura >= INTERVALO_LECTURA) {
```

La función `millis()` devuelve la cantidad de milisegundos transcurridos desde que el Arduino comenzó a ejecutar el programa. La CPU puede continuar recorriendo `loop()` y realizar otras tareas mientras espera el momento de leer nuevamente el sensor.

Esta diferencia será importante en el proyecto final. Mientras espera una nueva lectura del DHT11, el Arduino podrá atender el receptor infrarrojo, actualizar el LED RGB, leer otro sensor o controlar el motor paso a paso.

## Relación con la arquitectura del computador

El sistema construido sigue el modelo de entrada, procesamiento y salida. El DHT11 funciona como dispositivo de entrada. El ATmega328P del Arduino ejecuta las instrucciones del programa y procesa los datos. El LED RGB y el buzzer representan los dispositivos de salida.

```text
ENTRADA                  PROCESAMIENTO                   SALIDA

DHT11                Arduino ATmega328P              LED RGB
Temperatura   ─────► Lectura y comparación ─────►   Buzzer
Humedad              Determinación de estado
```

La CPU no percibe directamente la temperatura. El DHT11 transforma esa condición física en una trama de bits. Después, el programa interpreta los bits como números y compara esos números con los límites establecidos.

El funcionamiento general sigue una secuencia similar a la siguiente:

```text
SOLICITAR MEDICIÓN
        │
        ▼
RECIBIR 40 BITS
        │
        ▼
VALIDAR CHECKSUM
        │
        ▼
OBTENER TEMPERATURA Y HUMEDAD
        │
        ▼
COMPARAR CON LOS LÍMITES
        │
        ▼
DETERMINAR EL ESTADO
        │
        ▼
ACTUALIZAR LAS SALIDAS
```

Cada una de estas operaciones se convierte en instrucciones que la CPU debe buscar en la memoria, decodificar y ejecutar. Aunque el programa utilice funciones de alto nivel, el microcontrolador finalmente trabaja con registros, posiciones de memoria, operaciones lógicas y señales eléctricas.

## Reto del laboratorio

El programa integrado utiliza únicamente la temperatura para seleccionar el estado ambiental. Como reto, se deberá modificar la función `determinarEstado()` para considerar también la humedad.

El sistema deberá utilizar las siguientes condiciones:

| Temperatura | Humedad | Estado |
|---|---|---|
| Menor de 20 °C | Cualquier valor válido | Ambiente frío |
| De 20 a menos de 27 °C | Entre 30 % y 70 % | Ambiente normal |
| De 20 a menos de 27 °C | Menor de 30 % o mayor de 70 % | Humedad fuera del rango |
| De 27 a menos de 32 °C | Cualquier valor válido | Temperatura elevada |
| 32 °C o más | Cualquier valor válido | Alerta |

El estado `HUMEDAD_FUERA_RANGO` deberá representarse con el color celeste. Para producir este color se combinarán los canales verde y azul.

La función deberá recibir ahora las dos mediciones:

```cpp
EstadoAmbiental determinarEstado(
  float temperatura,
  float humedad
) {
  // Implementar las condiciones
}
```

También será necesario agregar el nuevo estado a la enumeración y configurar su comportamiento dentro de `aplicarEstado()`.

Una solución correctamente estructurada deberá conservar la separación entre lectura, procesamiento y salida. No se recomienda colocar todas las condiciones directamente dentro de `loop()`, porque esto dificultaría la integración posterior con el proyecto final.

## Preguntas de análisis

1. ¿Qué diferencia existe entre la salida del DHT11 y la salida de un sensor analógico?
2. ¿Por qué el DHT11 puede comunicarse utilizando un solo pin de datos?
3. ¿Cómo se encuentran organizados los 40 bits enviados por el sensor?
4. ¿Qué función cumple el byte de comprobación?
5. ¿Qué significa que una lectura contenga el valor `NaN`?
6. ¿Por qué no se debe utilizar una lectura inválida para determinar el estado del sistema?
7. ¿Por qué se recomienda esperar aproximadamente dos segundos entre las lecturas del DHT11?
8. ¿Qué ventaja ofrece `millis()` frente a `delay()` en un sistema que debe realizar varias tareas?
9. ¿Qué partes del circuito corresponden a entrada, procesamiento y salida?
10. ¿Por qué es conveniente separar el programa en funciones?

## Entrega

Los estudiantes deberán entregar el código fuente, fotografías claras del circuito, la tabla de mediciones y las respuestas a las preguntas de análisis. También deberán incluir un video corto donde se observe la lectura de temperatura y humedad, el cambio de estado del LED RGB y la activación del buzzer.

El reto deberá mostrar que la decisión considera tanto la temperatura como la humedad. Si las condiciones ambientales del laboratorio no permiten alcanzar todos los estados, los estudiantes podrán modificar temporalmente los límites para demostrar el funcionamiento. Antes de entregar, deberán restaurar los valores solicitados.