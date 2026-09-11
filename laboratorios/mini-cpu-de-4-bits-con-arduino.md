# Mini CPU de 4 bits con Arduino

### 1. Introducción

En los laboratorios anteriores se estudió el funcionamiento de una **Unidad Aritmética Lógica (ALU)** y se construyó una ALU básica capaz de realizar operaciones aritméticas y lógicas.

Sin embargo, una ALU por sí sola **no constituye una computadora**.

Para ejecutar un programa se necesitan otros componentes que permitan almacenar instrucciones y datos, controlar el orden de ejecución y coordinar las operaciones realizadas por la ALU.

En este laboratorio utilizaremos un **Arduino Uno como núcleo de una CPU didáctica** y construiremos alrededor de él algunos elementos que nos permitirán observar de forma simplificada el funcionamiento de una computadora.

El sistema implementará los siguientes conceptos:

* Memoria de programa.
* Program Counter (PC).
* Registros A, B y OUT.
* Unidad Aritmética Lógica.
* Ciclo **Fetch – Decode – Execute**.
* Reloj manual.
* Registro de salida de 4 bits.

El resultado de las operaciones será almacenado en un registro externo **74HC595** y visualizado mediante cuatro LEDs.

## 2. Objetivos

Al finalizar el laboratorio, el estudiante será capaz de relacionar los componentes básicos de la arquitectura Von Neumann con la ejecución de un programa y observar paso a paso el ciclo de instrucción de una CPU.

También deberá identificar la función del **Program Counter**, los registros, la memoria y la ALU durante la ejecución de instrucciones.

## 3. Arquitectura que construiremos

Nuestro sistema tendrá la siguiente estructura conceptual:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FJrsnLFjgEsHWOIw9Qxij%2Fimage.png?alt=media&amp;token=4bd698d6-051b-4e34-876b-1ad29a6034c5" alt=""><figcaption></figcaption></figure>

| Elemento        | Implementación                     |
| --------------- | ---------------------------------- |
| CPU             | Arduino Uno                        |
| Memoria         | Arreglo dentro del programa        |
| Program Counter | Variable `PC`                      |
| Registro A      | Variable `A`                       |
| Registro B      | Variable `B`                       |
| ALU             | Operaciones realizadas por Arduino |
| Registro OUT    | 74HC595                            |
| Reloj           | Pulsador CLOCK                     |
| Reset           | Pulsador RESET                     |
| Salida          | 4 LEDs                             |

{% hint style="info" %}
**Importante:** se trata de un **modelo didáctico simplificado**. El Arduino internamente ya contiene CPU, memoria, registros y otros elementos. En este laboratorio se representan algunos de estos componentes explícitamente para poder estudiar su interacción.
{% endhint %}

## 4. Materiales

Utilice:

* Arduino Uno.
* Protoboard.
* 1 registro 74HC595.
* 4 LEDs.
* 4 resistencias de 220 Ω.
* 2 pulsadores.
* Cables de conexión.

Estos componentes se encuentran disponibles en el kit utilizado en el curso.

## 5. Construcción del circuito

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FcX97X6l0urfzprXBvii6%2Fimage.png?alt=media&amp;token=e2aeb85c-3948-4dbb-bbf5-55cb65ba1176" alt=""><figcaption></figcaption></figure>

### Paso 1. Alimentar la protoboard

Conecte:

```
Arduino 5V  → riel positivo (+)Arduino GND → riel negativo (-)
```

Todos los componentes compartirán la misma tierra.

## 6. Colocar el registro 74HC595

Coloque el 74HC595 atravesando la división central de la protoboard.

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FLa1tFn10eW77HuLKAiFn%2Fimage.png?alt=media&amp;token=f17a1842-9654-453d-8892-7b669c95e779" alt="" width="375"><figcaption></figcaption></figure>

Realice las siguientes conexiones:

| 74HC595 | Conectar a |
| ------- | ---------- |
| VCC     | 5V         |
| GND     | GND        |
| SRCLR   | 5V         |
| OE      | GND        |
| SER     | Arduino D2 |
| SRCLK   | Arduino D3 |
| RCLK    | Arduino D4 |

Deje **Q7' sin conectar**.

Las salidas Q4-Q7 tampoco serán utilizadas en este laboratorio.

#### ¿Por qué utilizaremos el 74HC595?

El **74HC595 es un registro de desplazamiento de 8 bits**. Su función es recibir información en forma serial, almacenarla y presentarla simultáneamente en sus salidas.

En este laboratorio lo utilizaremos como nuestro **registro de salida (OUT)**. El Arduino enviará al 74HC595 el resultado producido por la ALU y este mantendrá almacenados los bits que posteriormente observaremos mediante cuatro LEDs.

Por ejemplo, si la ALU obtiene el resultado decimal **5**:

```
OUT = 0101
8   4   2   1
0   1   0   1
```

Aunque el 74HC595 dispone de **8 salidas**, utilizaremos únicamente cuatro porque nuestra CPU didáctica trabajará con datos de **4 bits**.

En aplicaciones reales, este tipo de registro también se utiliza para **aumentar la cantidad de salidas disponibles de un microcontrolador**, por ejemplo para controlar múltiples LEDs, displays o indicadores utilizando pocos pines de control. Además, varios 74HC595 pueden conectarse en cascada para disponer de más salidas.

{% hint style="info" %}
**En nuestro modelo:** la ALU calcula el resultado y el 74HC595 permite representar físicamente el concepto de un registro que **conserva ese resultado.**
{% endhint %}

## 7. Conectar los LEDs

Nuestra computadora trabajará con resultados de **4 bits**.

Utilizaremos:

```
Q3 → bit 3 → valor 8
Q2 → bit 2 → valor 4
Q1 → bit 1 → valor 2
Q0 → bit 0 → valor 1
```

Conecte cada salida a una resistencia de 220 Ω y posteriormente a un LED:

```
Q3 ── 220Ω ── LED ── GND
Q2 ── 220Ω ── LED ── GND
Q1 ── 220Ω ── LED ── GND
Q0 ── 220Ω ── LED ── GND
```

Coloque los LEDs preferiblemente en el siguiente orden:

```
     RESULTADO DE LA ALU       
     8    4    2    1       
     ●    ●    ●    ●       
     Q3   Q2   Q1   Q0
```

De esta manera será sencillo interpretar los resultados.

Por ejemplo:

```
0101 = 5
8    4    2    1
○    ●    ○    ●
```

## 8. Conectar el botón CLOCK

Coloque un pulsador en la protoboard.

Conecte:

```
Arduino D7 ── pulsador ── GND
```

El programa utilizará:

```cpp
pinMode(BTN_CLOCK, INPUT_PULLUP);
```

Por lo tanto, no es necesario agregar una resistencia externa al pulsador.

Este botón tendrá una función especial:

{% hint style="info" %}
**Cada pulsación de CLOCK permitirá ejecutar una instrucción del programa.**
{% endhint %}

## 9. Conectar el botón RESET

Coloque un segundo pulsador y conecte:

```
Arduino D8 ── pulsador ── GND
```

RESET permitirá regresar nuestra CPU a su estado inicial:

```
PC  = 0
A   = 0000
B   = 0000
OUT = 0000
```

## 10. La memoria de nuestra computadora

Nuestra CPU ejecutará un pequeño programa almacenado en memoria.

Una instrucción tendrá dos componentes:

```
OPERACIÓN + DATO
```

Por ejemplo:

```
LOAD A, 5
```

Significa: *Cargar el valor 5 en el registro A.*

Nuestra memoria contendrá inicialmente:

| Dirección | Instrucción |
| --------- | ----------- |
| 0         | LOAD A, 5   |
| 1         | LOAD B, 3   |
| 2         | ADD         |
| 3         | LOAD A, 7   |
| 4         | LOAD B, 2   |
| 5         | SUB         |
| 6         | LOAD A, 12  |
| 7         | LOAD B, 10  |
| 8         | AND         |
| 9         | LOAD A, 12  |
| 10        | LOAD B, 3   |
| 11        | OR          |

Observe que la memoria contiene tanto **la operación que debe realizarse como los valores utilizados por algunas instrucciones.**

## 11. Program Counter

Nuestra CPU necesita saber qué instrucción debe ejecutar.

Para ello utilizaremos:

```cpp
int PC = 0;
```

`PC` significa **Program Counter**.

Inicialmente:

```cpp
PC = 0
```

por lo que la CPU ejecutará:

```cpp
MEMORIA[0]LOAD A,5
```

Después de ejecutar la instrucción:

```cpp
PC = PC + 1
```

por lo que:

```cpp
PC = 1
```

y la siguiente instrucción será:

```cpp
LOAD B,3
```

***

## 12. Registros de la CPU

Nuestra CPU tendrá tres registros principales:

```cpp
byte A   = 0;
byte B   = 0;
byte OUT = 0;
```

Los registros **A y B** almacenarán los operandos.

`OUT` almacenará el resultado de la ALU.

Por ejemplo:

```
A = 0101
B = 0011      

ALU -> A + B = 1000

OUT = 1000
```

## 13. Ciclo de instrucción

Cada vez que se presione **CLOCK**, nuestra CPU realizará:

```
FETCH  
↓
DECODE  
↓
EXECUTE
```

#### FETCH

La CPU utiliza el PC para obtener una instrucción de memoria.

```
PC = 2      
↓
MEMORIA[2]      
↓
ADD
```

#### DECODE

La CPU determina qué operación representa la instrucción.

```
ADD 
↓
Operación de suma
```

#### EXECUTE

La ALU realiza:

```
A + B
```

y almacena el resultado en `OUT`.

## 14. Programa base

Utilice el siguiente programa para probar la computadora:

```cpp
// =====================================================
// MINI CPU DIDÁCTICA DE 4 BITS
// Arquitectura Von Neumann + ALU básica
// =====================================================

// ---------- 74HC595 ----------
const int DATA_PIN  = 2;   // SER
const int CLOCK_REG = 3;   // SRCLK
const int LATCH_PIN = 4;   // RCLK

// ---------- BOTONES ----------
const int BTN_CLOCK = 7;
const int BTN_RESET = 8;


// =====================================================
// CONJUNTO DE INSTRUCCIONES
// =====================================================

enum Operacion {
  LOAD_A,
  LOAD_B,
  ADD,
  SUB,
  AND_OP,
  OR_OP
};


// Una instrucción tiene:
// - operación
// - dato
struct Instruccion {
  Operacion op;
  byte dato;
};


// =====================================================
// MEMORIA DEL PROGRAMA
// =====================================================
//
// La memoria contiene instrucciones y datos.
//
// Dirección 0: LOAD A,5
// Dirección 1: LOAD B,3
// Dirección 2: ADD
//
// Dirección 3: LOAD A,7
// Dirección 4: LOAD B,2
// Dirección 5: SUB
//
// Dirección 6: LOAD A,12
// Dirección 7: LOAD B,10
// Dirección 8: AND
//
// Dirección 9: LOAD A,12
// Dirección 10: LOAD B,3
// Dirección 11: OR
//

Instruccion memoria[] = {

  {LOAD_A, 5},
  {LOAD_B, 3},
  {ADD, 0},

  {LOAD_A, 7},
  {LOAD_B, 2},
  {SUB, 0},

  {LOAD_A, 12},
  {LOAD_B, 10},
  {AND_OP, 0},

  {LOAD_A, 12},
  {LOAD_B, 3},
  {OR_OP, 0}
};


// Calculamos automáticamente cuántas instrucciones existen
const int TAM_MEMORIA =
  sizeof(memoria) / sizeof(memoria[0]);


// =====================================================
// REGISTROS DE NUESTRA CPU
// =====================================================

byte A   = 0;
byte B   = 0;
byte OUT = 0;

// Program Counter
int PC = 0;


// =====================================================
// SETUP
// =====================================================

void setup() {

  Serial.begin(9600);

  // 74HC595
  pinMode(DATA_PIN, OUTPUT);
  pinMode(CLOCK_REG, OUTPUT);
  pinMode(LATCH_PIN, OUTPUT);

  // Botones con resistencia pull-up interna
  pinMode(BTN_CLOCK, INPUT_PULLUP);
  pinMode(BTN_RESET, INPUT_PULLUP);

  // Registro de salida inicia en cero
  mostrarResultado(0);

  Serial.println("================================");
  Serial.println(" MINI CPU DIDACTICA DE 4 BITS");
  Serial.println(" Von Neumann + ALU");
  Serial.println("================================");
  Serial.println();
  Serial.println("Presione CLOCK para ejecutar.");
}


// =====================================================
// LOOP
// =====================================================

void loop() {

  // ---------------- RESET ----------------

  if (digitalRead(BTN_RESET) == LOW) {

    resetCPU();

    esperarLiberacion(BTN_RESET);
  }


  // ---------------- CLOCK ----------------

  if (digitalRead(BTN_CLOCK) == LOW) {

    ejecutarSiguienteInstruccion();

    esperarLiberacion(BTN_CLOCK);
  }
}


// =====================================================
// CICLO FETCH - DECODE - EXECUTE
// =====================================================

void ejecutarSiguienteInstruccion() {

  // Verificar si terminó el programa
  if (PC >= TAM_MEMORIA) {

    Serial.println();
    Serial.println("=== FIN DEL PROGRAMA ===");

    return;
  }

  Serial.println();
  Serial.println("----------------------------");

  Serial.print("PC = ");
  Serial.println(PC);


  // ==================================================
  // FETCH
  // ==================================================

  Instruccion instruccion = memoria[PC];

  Serial.println("FETCH: instruccion obtenida");


  // ==================================================
  // DECODE + EXECUTE
  // ==================================================

  switch (instruccion.op) {


    // ---------- LOAD A ----------

    case LOAD_A:

      Serial.print("DECODE: LOAD A, ");
      Serial.println(instruccion.dato);

      A = instruccion.dato;

      Serial.println("EXECUTE: dato cargado en A");

      break;


    // ---------- LOAD B ----------

    case LOAD_B:

      Serial.print("DECODE: LOAD B, ");
      Serial.println(instruccion.dato);

      B = instruccion.dato;

      Serial.println("EXECUTE: dato cargado en B");

      break;


    // ---------- SUMA ----------

    case ADD:

      Serial.println("DECODE: ADD");
      Serial.println("EXECUTE: A + B");

      OUT = A + B;

      break;


    // ---------- RESTA ----------

    case SUB:

      Serial.println("DECODE: SUB");
      Serial.println("EXECUTE: A - B");

      OUT = A - B;

      break;


    // ---------- AND ----------

    case AND_OP:

      Serial.println("DECODE: AND");
      Serial.println("EXECUTE: A AND B");

      OUT = A & B;

      break;


    // ---------- OR ----------

    case OR_OP:

      Serial.println("DECODE: OR");
      Serial.println("EXECUTE: A OR B");

      OUT = A | B;

      break;
  }


  // ==================================================
  // LIMITAMOS TODO A 4 BITS
  // ==================================================

  A   = A   & 0b00001111;
  B   = B   & 0b00001111;
  OUT = OUT & 0b00001111;


  // ==================================================
  // ACTUALIZAR REGISTRO DE SALIDA
  // ==================================================

  mostrarResultado(OUT);


  // Mostrar registros por Serial
  imprimirEstado();


  // ==================================================
  // PROGRAM COUNTER
  // ==================================================

  PC++;

  Serial.print("Siguiente PC = ");
  Serial.println(PC);
}


// =====================================================
// ENVIAR RESULTADO AL 74HC595
// =====================================================

void mostrarResultado(byte valor) {

  digitalWrite(LATCH_PIN, LOW);

  shiftOut(
    DATA_PIN,
    CLOCK_REG,
    MSBFIRST,
    valor
  );

  digitalWrite(LATCH_PIN, HIGH);
}


// =====================================================
// MOSTRAR ESTADO DE LOS REGISTROS
// =====================================================

void imprimirEstado() {

  Serial.println();

  Serial.print("A   = ");
  imprimir4Bits(A);

  Serial.print("B   = ");
  imprimir4Bits(B);

  Serial.print("OUT = ");
  imprimir4Bits(OUT);
}


// =====================================================
// IMPRIMIR UN VALOR COMO 4 BITS
// =====================================================

void imprimir4Bits(byte valor) {

  for (int i = 3; i >= 0; i--) {

    Serial.print(
      bitRead(valor, i)
    );
  }

  Serial.println();
}


// =====================================================
// RESET DE LA CPU
// =====================================================

void resetCPU() {

  A   = 0;
  B   = 0;
  OUT = 0;

  PC = 0;

  mostrarResultado(0);

  Serial.println();
  Serial.println("============================");
  Serial.println("CPU REINICIADA");
  Serial.println("PC  = 0");
  Serial.println("A   = 0000");
  Serial.println("B   = 0000");
  Serial.println("OUT = 0000");
  Serial.println("============================");
}


// =====================================================
// ESPERAR QUE SE LIBERE UN BOTON
// Evita múltiples pulsos por una sola pulsación
// =====================================================

void esperarLiberacion(int pin) {

  while (digitalRead(pin) == LOW) {
    // esperar
  }

  delay(50);
}
```

***

## 15. Prueba del programa

Inicie la simulación y abra el **Monitor Serial**.

Presione RESET antes de comenzar.

El estado inicial será:

```
PC  = 0
A   = 0000
B   = 0000
OUT = 0000
```

Ahora presione CLOCK.

#### CLOCK 1

Se ejecuta:

```
LOAD A,5
```

Estado:

```
A   = 0101
B   = 0000
OUT = 0000
```

#### CLOCK 2

Se ejecuta:

```
LOAD B,3
```

Estado:

```
A   = 0101
B   = 0011
OUT = 0000
```

#### CLOCK 3

Se ejecuta:

```
ADD
```

La ALU realiza:

```
  0101+ 0011
  ------  
  1000
```

Por lo tanto:

```
OUT = 1000
```

Los LEDs deberán mostrar:

```
8    4    2    1
●    ○    ○    ○
```

El resultado es:

**5 + 3 = 8**

***

## 16. Continúe la ejecución

Siga presionando CLOCK.

Cada tres pulsaciones se completará otra operación.

Debe comprobar los siguientes resultados:

| Operación | Resultado decimal | Resultado binario |
| --------- | ----------------- | ----------------- |
| 5 + 3     | 8                 | `1000`            |
| 7 - 2     | 5                 | `0101`            |
| 12 AND 10 | 8                 | `1000`            |
| 12 OR 3   | 15                | `1111`            |

Observe simultáneamente los LEDs y el Monitor Serial.

***

## 17. Entregable 1

Una vez comprobado el funcionamiento del programa original, modifique la memoria para ejecutar las siguientes operaciones:

```
9 + 4
14 - 5
6 AND 3
8 OR 5
```

No modifique el funcionamiento del CLOCK, el Program Counter ni el registro 74HC595.

Modifique únicamente el **programa almacenado en memoria**.

Para cada operación registre:

* Valor de A.
* Valor de B.
* Operación ejecutada.
* Resultado decimal esperado.
* Resultado binario obtenido.
* Estado de los cuatro LEDs.

Finalmente responda:

**¿Qué función cumple el Program Counter?**

**¿Qué ocurre durante FETCH?**

**¿Qué diferencia existe entre los registros A/B y el registro OUT?**

**¿Por qué una ALU por sí sola no constituye una CPU?**

**¿Qué relación observa entre este laboratorio y la arquitectura Von Neumann estudiada en teoría?**

## 18. Entregable 2

Amplíe el conjunto de instrucciones de la CPU agregando una nueva instrucción llamada **`PLAY`**, cuya función será reproducir una nota musical mediante el **buzzer pasivo incluido en el kit**.&#x20;

Conecte el terminal positivo del buzzer al pin **D9** del Arduino y el negativo a **GND**.&#x20;

La instrucción deberá recibir un valor entre **1 y 8**, almacenado como un dato de 4 bits, y convertirlo en la frecuencia de la nota correspondiente:&#x20;

1. DO (262 Hz)
2. RE (294 Hz)
3. MI (330 Hz)
4. FA (349 Hz)
5. SOL (392 Hz)
6. LA (440 Hz)
7. SI (494 Hz)&#x20;
8. DO (523 Hz)&#x20;

Modifique el conjunto de instrucciones, el proceso de **DECODE/EXECUTE** y la memoria del programa para reconocer `PLAY` y utilizar `tone()` para generar el sonido.&#x20;

Finalmente, programe una secuencia de instrucciones que permita a la CPU **reproducir una melodía sencilla de 10 a 15 notas musicales al avanzar con el botón CLOCK**. Suba un video al portal del curso donde se reproduce la melodía programada en la CPU.&#x20;