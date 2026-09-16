# Lectura RFID y control de acceso con servomotor

## Lectura RFID y control de acceso con servomotor

### Introducción

En este laboratorio se estudiarán dos dispositivos incluidos en el kit de Arduino Uno:

* El lector RFID RC522.
* El servomotor SG90.

El RC522 permitirá identificar una tarjeta por medio de su UID, mientras que el servomotor representará el mecanismo de apertura y cierre de una puerta.

El laboratorio se desarrollará en dos partes:

* **Parte I:** soldadura, explicación y pruebas independientes del RC522 y del servomotor.
* **Parte II:** integración de ambos dispositivos y desarrollo del reto.

### Objetivos

Al completar el laboratorio, el estudiante será capaz de:

1. Comprender el funcionamiento general de un sistema RFID.
2. Identificar los pines y señales principales del módulo RC522.
3. Comprender la comunicación SPI entre el RC522 y el Arduino Uno.
4. Leer y mostrar el UID de una tarjeta RFID.
5. Comprender cómo una señal periódica controla la posición de un servomotor.
6. Controlar diferentes posiciones del servomotor SG90.
7. Integrar un dispositivo de entrada con un actuador.
8. Relacionar el sistema construido con el modelo de entrada, procesamiento y salida de un computador.

### Materiales

* 1 Arduino Uno.
* 1 módulo RFID RC522.
* 1 tarjeta o llavero RFID compatible.
* 1 servomotor SG90.
* 1 protoboard.
* Cables Dupont.
* Cable USB.
* Computadora con Arduino IDE.
* Cautín y estaño.
* Equipo de protección para soldadura.
* Opcional para el reto: buzzer pasivo.

{% hint style="danger" %}
El módulo RC522 debe alimentarse con **3.3 V**. Conectarlo directamente a 5 V puede dañarlo.
{% endhint %}

## Parte I: Conocimiento y pruebas de los dispositivos

### Preparación y soldadura del módulo RC522

El módulo RC522 normalmente incluye una tira de pines sin soldar. Antes de conectarlo al Arduino será necesario soldar sus terminales.

#### Recomendaciones de seguridad

1. Trabaje en un lugar ventilado.
2. No toque la punta metálica del cautín.
3. Coloque el módulo sobre una superficie estable.
4. Caliente simultáneamente el pin y la pista metálica durante pocos segundos.
5. Aplique solamente una pequeña cantidad de estaño.
6. Verifique que no existan puentes de estaño entre pines vecinos.
7. Desconecte el cautín al finalizar.

Antes de energizar el circuito, el docente deberá revisar visualmente las soldaduras.

Una soldadura correcta debe cubrir el punto de contacto sin formar una esfera demasiado grande ni tocar los pines vecinos.

### ¿Qué es RFID?

RFID significa **Radio Frequency Identification**, o identificación por radiofrecuencia. Esta tecnología permite intercambiar información sin contacto físico entre un lector y una tarjeta o etiqueta electrónica.

El sistema utilizado en este laboratorio tiene dos elementos:

* **RC522:** genera un campo electromagnético y recibe la información.
* **Tarjeta o llavero:** contiene un circuito integrado y una pequeña antena.

Al acercar la tarjeta, la energía del campo generado por el lector permite que la tarjeta responda. El RC522 recibe la respuesta y entrega los datos al Arduino.

<figure><img src="../.gitbook/assets/image (45).png" alt="" width="375"><figcaption></figcaption></figure>

El RC522 trabaja con tarjetas sin contacto de **13.56 MHz**, compatibles con el estándar ISO/IEC 14443 tipo A y con diferentes productos de las familias MIFARE y NTAG.

#### ¿Qué es el UID?

UID significa **Unique Identifier**, o identificador único. Es una secuencia de bytes utilizada durante el proceso de detección, anticolisión y selección de una tarjeta. Cuando varias tarjetas se encuentran cerca del lector, el UID permite que el lector seleccione una tarjeta específica para comunicarse con ella.

Un UID puede verse de esta manera:

```
B3 7A 21 0F
```

Cada pareja de caracteres representa un byte:

```
Byte 0    Byte 1    Byte 2    Byte 3
  B3        7A        21        0F
```

En C++ podría representarse mediante un arreglo:

```cpp
byte uidTarjeta[4] = {
  0xB3,
  0x7A,
  0x21,
  0x0F
};
```

#### ¿A qué estándar pertenece el UID?

El formato del UID utilizado por estas tarjetas está definido por el estándar **ISO/IEC 14443-3**.

El estándar admite tres tamaños:

| Tipo       |   Tamaño | Cantidad de bits |
| ---------- | -------: | ---------------: |
| UID simple |  4 bytes |          32 bits |
| UID doble  |  7 bytes |          56 bits |
| UID triple | 10 bytes |          80 bits |

Por tanto, no todas las tarjetas poseen un UID de cuatro bytes.

La biblioteca MFRC522 permite conocer el tamaño real mediante:

```cpp
rfid.uid.size
```

Los bytes recibidos se encuentran en:

```cpp
rfid.uid.uidByte
```

Por ejemplo:

```
rfid.uid.uidByte[0] → B3
rfid.uid.uidByte[1] → 7A
rfid.uid.uidByte[2] → 21
rfid.uid.uidByte[3] → 0F
```

La documentación técnica recomienda que un lector pueda procesar UID de 4, 7 y 10 bytes.

#### ¿De dónde proviene el UID?

Normalmente, el UID es asignado por el fabricante del circuito integrado y queda almacenado dentro de la tarjeta durante su fabricación o personalización. En los UID de **7 y 10 bytes**, el primer byte contiene un código que identifica al fabricante del chip.

Por ejemplo:

```
04 A3 7B 92 15 68 80
```

El valor `04` corresponde a NXP Semiconductors.

Los UID de cuatro bytes no necesariamente incluyen un código de fabricante. Este formato fue común en tarjetas MIFARE antiguas, pero el espacio disponible para identificadores de cuatro bytes es limitado. Por ello, los productos más recientes utilizan principalmente UID de siete bytes.

**¿Por qué se muestra en hexadecimal?**

El UID no está almacenado internamente como texto hexadecimal. La tarjeta transmite una secuencia de bits que Arduino agrupa en bytes. Un byte contiene ocho bits y puede representar valores entre 0 y 255. El hexadecimal se utiliza porque permite representar cada byte mediante solamente dos caracteres.

| Binario    | Decimal | Hexadecimal |
| ---------- | ------: | ----------: |
| `10110011` |     179 |        `B3` |
| `01111010` |     122 |        `7A` |
| `00100001` |      33 |        `21` |
| `00001111` |      15 |        `0F` |

En el programa del laboratorio se utiliza la constante `HEX` para mostrar el byte en hexadecimal:

```cpp
Serial.print(rfid.uid.uidByte[i], HEX);
```

La constante `HEX` solamente cambia la forma de mostrar el número. No modifica su valor interno.

La siguiente condición agrega un cero a la izquierda:

```cpp
if (rfid.uid.uidByte[i] < 0x10)
{
  Serial.print("0");
}
```

Sin esa condición, el byte `0x0F` aparecería como:

```
F
```

Con la condición aparece como un byte hexadecimal completo:

```
0F
```

#### ¿El UID es fijo o variable?

En la mayoría de las tarjetas utilizadas en sistemas sencillos, el UID está almacenado dentro del circuito integrado y permanece fijo. Sin embargo, no siempre debe asumirse que es permanente o verdaderamente único.

Existen diferentes casos:

* Tarjetas con UID fijo.
* Identificadores de cuatro bytes que pueden reutilizarse.
* Tarjetas que generan identificadores aleatorios.
* Tarjetas especiales cuyo UID puede modificarse.
* Dispositivos que pueden emular una tarjeta y presentar otro UID.

Por esta razón, el nombre UID no garantiza completamente que el identificador sea:

* Único en todo el mundo.
* Imposible de copiar.
* Permanente en todos los modelos.

Para este laboratorio utilizaremos el UID como identificador porque permite estudiar arreglos, bytes, memoria, comparación y toma de decisiones. Sin embargo, un sistema de seguridad real no debería autorizar el acceso utilizando solamente el UID.

### Comunicación SPI

El RC522 se comunica con Arduino mediante el protocolo **SPI**, que significa _Serial Peripheral Interface_.

SPI utiliza varias señales:

| Señal  | Función                                                                                           |
| ------ | ------------------------------------------------------------------------------------------------- |
| SCK    | La señal SCK funciona como reloj. Sus pulsos indican cuándo debe enviarse o recibirse cada bit    |
| MOSI   | MOSI significa **Master Out – Slave In** y transporta información desde el Arduino hacia el RC522 |
| MISO   | MISO significa **Master In – Slave Out** y transporta información desde el RC522 hacia el Arduino |
| SS/SDA | La señal SS permite seleccionar el dispositivo con el que Arduino desea comunicarse               |
| RST    | Reinicia el RC522                                                                                 |

En esta comunicación, el Arduino actúa como controlador y el RC522 como periférico.

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

#### Conexión del RC522

| Pin RC522 |  Arduino Uno | Función                   |
| --------- | -----------: | ------------------------- |
| SDA/SS    |          D10 | Selección del módulo      |
| SCK       |          D13 | Reloj SPI                 |
| MOSI      |          D11 | Datos hacia el RC522      |
| MISO      |          D12 | Datos hacia Arduino       |
| IRQ       | Sin conectar | Interrupción no utilizada |
| GND       |          GND | Tierra                    |
| RST       |           D9 | Reinicio                  |
| 3.3V      |         3.3V | Alimentación              |

Revise todas las conexiones antes de conectar el cable USB.

{% hint style="danger" %}
**No conecte el pin de alimentación del RC522 al pin de 5 V.**
{% endhint %}

#### Instalación de la biblioteca MFRC522

En Arduino IDE:

1. Abra **Herramientas > Administrar bibliotecas**.
2. Busque `MFRC522`.
3. Instale la biblioteca **MFRC522 by GithubCommunity**.
4. Espere a que termine la instalación.

La biblioteca `SPI` ya forma parte del entorno de Arduino.

Las bibliotecas se incluyen al inicio del programa:

```cpp
#include <SPI.h>
#include <MFRC522.h>
```

### Prueba 1: lectura del UID

El siguiente programa detecta una tarjeta y muestra su UID en el monitor serial.

```cpp
#include <SPI.h>
#include <MFRC522.h>

const byte PIN_SS  = 10;
const byte PIN_RST = 9;

MFRC522 rfid(PIN_SS, PIN_RST);

void setup()
{
  Serial.begin(9600);

  SPI.begin();
  rfid.PCD_Init();

  Serial.println("Lector RFID listo");
  Serial.println("Acerque una tarjeta...");
}

void loop()
{
  // Verificar si hay una tarjeta nueva.
  if (!rfid.PICC_IsNewCardPresent())
  {
    return;
  }

  // Intentar leer la información de la tarjeta.
  if (!rfid.PICC_ReadCardSerial())
  {
    return;
  }

  Serial.print("Tamaño del UID: ");
  Serial.print(rfid.uid.size);
  Serial.println(" bytes");

  Serial.print("UID detectado: ");

  // Recorrer todos los bytes del UID.
  for (byte i = 0; i < rfid.uid.size; i++)
  {
    // Agregar un cero para mostrar siempre dos dígitos.
    if (rfid.uid.uidByte[i] < 0x10)
    {
      Serial.print("0");
    }

    Serial.print(rfid.uid.uidByte[i], HEX);
    Serial.print(" ");
  }

  Serial.println();

  // Finalizar la comunicación con la tarjeta actual.
  rfid.PICC_HaltA();
  rfid.PCD_StopCrypto1();

  delay(1000);
}
```

#### Procedimiento

1. Compile el programa.
2. Cargue el programa en el Arduino.
3. Abra el monitor serial.
4. Configure una velocidad de 9600 baudios.
5. Acerque la tarjeta al RC522.
6. Anote el tamaño y los bytes del UID.
7. Aleje la tarjeta.
8. Vuelva a acercarla.
9. Compruebe que el UID se mantiene igual.
10. Si dispone de otra tarjeta, compare ambos identificadores.

#### Resultado esperado

```
Lector RFID listo
Acerque una tarjeta...
Tamaño del UID: 4 bytes
UID detectado: B3 7A 21 0F
```

{% hint style="warning" %}
El UID mostrado será diferente para cada tarjeta.
{% endhint %}

### ¿Qué es un servomotor?

Un servomotor es un actuador capaz de colocar su eje en una posición determinada. El SG90 utilizado en este laboratorio normalmente puede moverse dentro de un rango cercano a 0°–180°.

En su interior contiene:

* Un motor de corriente continua.
* Un conjunto de engranajes.
* Un potenciómetro o sensor de posición.
* Un circuito electrónico de control.

El circuito compara la posición solicitada con la posición real. Si ambas son diferentes, activa el motor hasta reducir el error.

#### Señal de control del servomotor

Arduino envía pulsos periódicos al servomotor y la duración de cada pulso representa aproximadamente la posición solicitada.

| Duración aproximada | Posición aproximada |
| ------------------: | ------------------: |
|                1 ms |                  0° |
|              1.5 ms |                 90° |
|                2 ms |                180° |

Los valores pueden variar ligeramente entre diferentes servomotores. La señal enviada por Arduino solamente comunica la posición deseada. La alimentación eléctrica proporciona la energía necesaria para mover el motor. El servomotor recibe continuamente los pulsos y trata de mantener la posición solicitada.

#### Conexión del servomotor SG90

| Cable del servo            | Arduino Uno | Función          |
| -------------------------- | ----------: | ---------------- |
| Marrón o negro             |         GND | Tierra           |
| Rojo                       |          5V | Alimentación     |
| Naranja, amarillo o blanco |          D6 | Señal de control |

Para esta prueba, el servo debe estar sin carga mecánica.

{% hint style="danger" %}
No fuerce manualmente su eje. Un servomotor puede consumir más corriente de la que el Arduino puede proporcionar de manera estable. Para la prueba individual puede utilizarse el pin de 5 V si el servo está sin carga. Para el proyecto final se recomienda una fuente externa regulada de 5 V.
{% endhint %}

Cuando se utilice una fuente externa, su tierra debe conectarse con la tierra del Arduino, esto proporciona una referencia eléctrica común para interpretar correctamente la señal de control.

### Prueba 2: Posiciones del servomotor

La biblioteca `Servo` ya está incluida en Arduino IDE.

```cpp
#include <Servo.h>

const byte PIN_SERVO = 6;

Servo puerta;

void setup()
{
  puerta.attach(PIN_SERVO);
}

void loop()
{
  puerta.write(0);
  delay(2000);

  puerta.write(90);
  delay(2000);

  puerta.write(180);
  delay(2000);
}
```

#### Procedimiento

1. Desconecte el Arduino antes de modificar el circuito.
2. Conecte el servomotor.
3. Compile y cargue el programa.
4. Observe las tres posiciones.
5. Determine cuáles posiciones podrían representar una puerta cerrada y una puerta abierta.
6. Compruebe si el servo alcanza los extremos sin vibrar.

Si el servo vibra, se calienta o intenta continuar después de alcanzar un límite, desconecte la alimentación y reduzca el rango.

Por ejemplo:

```cpp
const byte PUERTA_CERRADA = 10;
const byte PUERTA_ABIERTA = 100;
```

No es obligatorio utilizar exactamente 0° y 180°. Los ángulos deben seleccionarse de acuerdo con la geometría del mecanismo.

## Parte II: integración del RFID y el servomotor

### Modelo de entrada, procesamiento y salida

En el sistema integrado:

* El RC522 funciona como dispositivo de entrada.
* El Arduino procesa la información.
* El servomotor funciona como dispositivo de salida.

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

La primera integración tendrá el siguiente comportamiento:

1. El sistema espera una tarjeta.
2. El RC522 detecta la tarjeta.
3. Arduino obtiene el UID.
4. El UID aparece en el monitor serial.
5. El servomotor abre la puerta.
6. La puerta permanece abierta durante cinco segundos.
7. El servomotor cierra la puerta.

En esta primera versión, **cualquier tarjeta detectada abrirá la puerta** y la validación del UID se implementará posteriormente como reto.

### Flujo del programa

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

### Conexiones del sistema integrado

#### RC522

| Pin RC522 | Arduino Uno |
| --------- | ----------: |
| SDA/SS    |         D10 |
| SCK       |         D13 |
| MOSI      |         D11 |
| MISO      |         D12 |
| RST       |          D9 |
| 3.3V      |        3.3V |
| GND       |         GND |

#### Servomotor SG90

| Cable SG90   |                  Arduino Uno |
| ------------ | ---------------------------: |
| Señal        |                           D6 |
| Alimentación | 5V o fuente externa regulada |
| Tierra       |                    GND común |

### Programa integrado

```cpp
#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>

const byte PIN_SS    = 10;
const byte PIN_RST   = 9;
const byte PIN_SERVO = 6;

const byte PUERTA_CERRADA = 10;
const byte PUERTA_ABIERTA = 100;

MFRC522 rfid(PIN_SS, PIN_RST);
Servo puerta;

void mostrarUID()
{
  Serial.print("UID detectado: ");

  for (byte i = 0; i < rfid.uid.size; i++)
  {
    if (rfid.uid.uidByte[i] < 0x10)
    {
      Serial.print("0");
    }

    Serial.print(rfid.uid.uidByte[i], HEX);
    Serial.print(" ");
  }

  Serial.println();
}

void abrirPuerta()
{
  Serial.println("Abriendo puerta...");

  puerta.write(PUERTA_ABIERTA);

  delay(5000);

  Serial.println("Cerrando puerta...");

  puerta.write(PUERTA_CERRADA);
}

void setup()
{
  Serial.begin(9600);

  SPI.begin();
  rfid.PCD_Init();

  puerta.attach(PIN_SERVO);
  puerta.write(PUERTA_CERRADA);

  Serial.println("Sistema de acceso listo");
  Serial.println("Acerque una tarjeta...");
}

void loop()
{
  // Comprobar si se acercó una tarjeta.
  if (!rfid.PICC_IsNewCardPresent())
  {
    return;
  }

  // Leer la tarjeta.
  if (!rfid.PICC_ReadCardSerial())
  {
    return;
  }

  mostrarUID();
  abrirPuerta();

  // Finalizar la comunicación con la tarjeta.
  rfid.PICC_HaltA();
  rfid.PCD_StopCrypto1();

  Serial.println("Acerque una tarjeta...");

  delay(1000);
}
```

### Prueba de integración

Realice las siguientes pruebas:

1. Encienda o reinicie el sistema.
2. Compruebe que la puerta se encuentre inicialmente cerrada.
3. No acerque ninguna tarjeta.
4. Verifique que el sistema permanezca esperando.
5. Acerque una tarjeta.
6. Verifique que el UID aparezca en el monitor serial.
7. Compruebe que el servo mueva la puerta a la posición abierta.
8. Espere cinco segundos.
9. Compruebe que la puerta regrese a la posición cerrada.
10. Repita la prueba al menos tres veces.

#### Resultado esperado

```
Sistema de acceso listo
Acerque una tarjeta...
UID detectado: B3 7A 21 0F
Abriendo puerta...
Cerrando puerta...
Acerque una tarjeta...
```

## Reto: Sistema de Acceso Autorizado

### Descripción

Modifique el programa integrado para que el servomotor abra la puerta únicamente cuando se presente una tarjeta autorizada. El sistema deberá almacenar el UID autorizado y comparar todos sus bytes con el UID leído y el estudiante deberá utilizar el UID obtenido durante la primera prueba.

### Comportamiento requerido

#### Tarjeta autorizada

Cuando se presente la tarjeta autorizada, el sistema deberá:

1. Mostrar el UID en el monitor serial.
2. Mostrar el mensaje `ACCESO AUTORIZADO`.
3. Mover el servo a la posición abierta.
4. Mantener la puerta abierta durante cinco segundos.
5. Cerrar la puerta automáticamente.

#### Tarjeta no autorizada

Cuando se presente una tarjeta diferente, el sistema deberá:

1. Mostrar el UID en el monitor serial.
2. Mostrar el mensaje `ACCESO DENEGADO`.
3. Mantener el servomotor en la posición cerrada.
4. Regresar al estado de espera.

### Almacenamiento del UID autorizado

Reemplace los valores del siguiente arreglo con el UID de su tarjeta:

```cpp
const byte TAMANO_UID = 4;

byte uidAutorizado[TAMANO_UID] = {
  0xB3,
  0x7A,
  0x21,
  0x0F
};
```

> \[!IMPORTANT] El tamaño del arreglo debe coincidir con la cantidad de bytes del UID de la tarjeta utilizada.

Si la tarjeta posee un UID de siete bytes, deberá modificarse de esta manera:

```cpp
const byte TAMANO_UID = 7;

byte uidAutorizado[TAMANO_UID] = {
  0x04,
  0xA3,
  0x7B,
  0x92,
  0x15,
  0x68,
  0x80
};
```

### Función que deberá completar

Deberá completar la siguiente función:

```cpp
bool esTarjetaAutorizada()
{
  // Verificar que el tamaño del UID leído
  // sea igual al tamaño del UID autorizado.

  // Recorrer todos los bytes del UID.

  // Comparar cada byte leído con el byte
  // correspondiente del arreglo uidAutorizado.

  // Retornar false si algún byte es diferente.

  // Retornar true si todos los bytes coinciden.
}
```

La comparación deberá analizar todos los bytes, comparar solamente el primer byte no es suficiente:

### Integración pendiente dentro de `loop()`

Después de leer y mostrar el UID, utilice la función de validación:

```cpp
if (esTarjetaAutorizada())
{
  Serial.println("ACCESO AUTORIZADO");

  // Abrir la puerta.
}
else
{
  Serial.println("ACCESO DENEGADO");

  // Mantener la puerta cerrada.
}
```

El comportamiento general deberá ser:

<figure><img src="../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

## Señalización con buzzer

Agregue el buzzer pasivo al sistema.

| Terminal del buzzer | Arduino |
| ------------------- | ------: |
| Positivo            |      D3 |
| Negativo            |     GND |

Defina el pin:

```cpp
const byte PIN_BUZZER = 3;
```

Configure su funcionamiento:

```cpp
pinMode(PIN_BUZZER, OUTPUT);
```

El sistema deberá producir:

* Un sonido agudo y largo cuando el acceso sea autorizado.
* Dos sonidos graves y cortos cuando el acceso sea denegado.

Ejemplo para acceso autorizado:

```cpp
tone(PIN_BUZZER, 2000, 700);
```

Ejemplo para acceso denegado:

```cpp
tone(PIN_BUZZER, 400, 250);
delay(350);

tone(PIN_BUZZER, 400, 250);
delay(350);
```

El sonido deberá permitir identificar el resultado sin observar el monitor serial.

## Entregables

Cada pareja deberá presentar:

1. Captura del monitor serial mostrando el UID de su tarjeta.
2. Código fuente del sistema integrado (.ino).
3. Código fuente del reto resuelto (.ino).
4. Video mostrando una tarjeta autorizada y una tarjeta no autorizada.
5. Respuestas a las preguntas de análisis.

## Preguntas de análisis

1. ¿Qué función cumple el UID dentro del sistema RFID?
2. ¿Por qué el RC522 debe conectarse a 3.3 V y no a 5 V?
3. ¿Qué información transportan las señales MOSI y MISO?
4. ¿Por qué el UID se almacena como un arreglo de bytes?
5. ¿Por qué se utiliza hexadecimal para mostrar los bytes del UID?
6. ¿Todos los UID tienen el mismo tamaño? Explique su respuesta.
7. ¿Cómo indica Arduino al servomotor la posición deseada?
8. ¿Qué función cumple el sensor de posición interno del servomotor?
9. ¿Qué sucedería si solamente se comparara el primer byte del UID?
10. Identifique la entrada, el procesamiento y la salida del sistema construido.

## Conclusión

En este laboratorio se integró un dispositivo de entrada basado en radiofrecuencia con un actuador electromecánico.

El RC522 permitió obtener el UID de una tarjeta, la CPU del Arduino procesó sus bytes y el servomotor convirtió una decisión del programa en una acción física.

<figure><img src="../.gitbook/assets/image (50).png" alt="" width="375"><figcaption></figcaption></figure>

Este ejercicio constituye la base del sistema final de control y seguridad. En las siguientes etapas, la identificación RFID podrá combinarse con el PIN almacenado en EEPROM, el teclado matricial, la pantalla LCD y el buzzer para implementar un proceso de autenticación más completo.
