# Introducción a Arduino y Control de un Display de 7 Segmentos

**Curso:** Arquitectura del Computador

**Duración Aproximada:** 2 horas

**Modalidad:** Parejas

## Objetivos

Al finalizar este laboratorio el estudiante será capaz de:

* Identificar los componentes básicos de un Arduino Uno.
* Comprender el funcionamiento de los pines digitales de entrada y salida.
* Configurar pines como salidas utilizando Arduino IDE.
* Controlar un display de siete segmentos mediante programación.
* Comprender la relación entre software y hardware.

## Competencias desarrolladas

Durante este laboratorio el estudiante desarrollará la capacidad de:

* Interpretar un diagrama electrónico.
* Construir un circuito sobre protoboard.
* Programar un microcontrolador.
* Verificar y depurar conexiones físicas.
* Resolver problemas básicos de integración hardware/software.

## Materiales

Cada pareja deberá contar con:

* Arduino Uno
* Protoboard
* Display de 7 segmentos (cátodo común)
* 7 resistencias de 220 Ω
* Cables Dupont
* Cable USB
* Computadora con Arduino IDE instalado

## Introducción

En los laboratorios anteriores se trabajó con circuitos digitales mediante simulación.

A partir de este laboratorio comenzaremos a trabajar sobre hardware real.

El Arduino Uno incorpora un microcontrolador que ejecuta instrucciones escritas en lenguaje C++. Dicho programa controla directamente los pines digitales del dispositivo, permitiendo interactuar con componentes electrónicos externos.

En esta práctica aprenderemos a controlar un display de siete segmentos.

## Parte I

### Armado del circuito

Utilice el siguiente diagrama para construir el circuito.

<figure><img src=".gitbook/assets/WhatsApp Image 2026-07-31 at 4.45.10 PM.jpeg" alt=""><figcaption></figcaption></figure>

Una vez finalizado:

* Revise todas las conexiones.
* Solicite autorización al docente antes de conectar el cable USB.

## Parte II

### Preparación del entorno

1. Conecte el Arduino al computador.
2. Abra Arduino IDE.
3. Cree un nuevo proyecto.
4. Guárdelo con el nombre

```
Laboratorio7Segmentos
```

5. Seleccione la placa

```
Arduino Uno
```

6. Seleccione el puerto COM correspondiente.

## Parte III

### Conociendo el programa

Todo programa de Arduino posee dos funciones principales.

#### setup()

```cpp
void setup(){}
```

Esta función se ejecuta una única vez cuando el Arduino recibe alimentación.

Generalmente se utiliza para:

* configurar pines
* inicializar variables
* iniciar comunicación serial

#### loop()

```cpp
void loop(){}
```

Esta función se ejecuta continuamente mientras el Arduino permanezca encendido.

Cuando llega al final vuelve a comenzar automáticamente.

## Parte IV

### Código base

Copie cuidadosamente el siguiente programa.

```cpp
// Pines del display
const int A = 2;
const int B = 3;
const int C = 4;
const int D = 5;
const int E = 6;
const int F = 7;
const int G = 8;

void setup() {
  pinMode(A, OUTPUT);
  pinMode(B, OUTPUT);
  pinMode(C, OUTPUT);
  pinMode(D, OUTPUT);
  pinMode(E, OUTPUT);
  pinMode(F, OUTPUT);
  pinMode(G, OUTPUT);
}

void apagarTodo() {
  digitalWrite(A, LOW);
  digitalWrite(B, LOW);
  digitalWrite(C, LOW);
  digitalWrite(D, LOW);
  digitalWrite(E, LOW);
  digitalWrite(F, LOW);
  digitalWrite(G, LOW);
}

void loop() {
  apagarTodo();
  digitalWrite(A, HIGH);
  delay(1000);

  apagarTodo();
  digitalWrite(B, HIGH);
  delay(1000);

  apagarTodo();
  digitalWrite(C, HIGH);
  delay(1000);

  apagarTodo();
  digitalWrite(D, HIGH);
  delay(1000);

  apagarTodo();
  digitalWrite(E, HIGH);
  delay(1000);

  apagarTodo();
  digitalWrite(F, HIGH);
  delay(1000);

  apagarTodo();
  digitalWrite(G, HIGH);
  delay(1000);

}
```

{% hint style="danger" %}
**Antes de ejecutarlo, lea cuidadosamente el programa y trate de comprender la función de cada sección. Valide también si los pines corresponden a los segmentos del display; es probable que no necesariamente coincidan.**&#x20;
{% endhint %}

## Explicación del código

Observe que el programa está dividido en cuatro partes principales.

### 1. Declaración de pines

En esta sección se asigna un nombre a cada pin utilizado.

```cpp
const int A = 2;
```

En lugar de recordar que el segmento A está conectado al pin 2, utilizaremos un nombre más fácil de leer.

### 2. setup()

Aquí configuramos todos los pines como salidas.

```cpp
pinMode(A, OUTPUT);
```

Si un pin no es configurado correctamente, el Arduino no podrá controlar el display.

### 3. Funciones

El programa utiliza funciones para evitar repetir código.

Por ejemplo:

```cpp
apagarDisplay();
```

En lugar de escribir siete instrucciones cada vez que queremos apagar el display, utilizamos una función que realiza ese trabajo.

### 4. loop()

Es el programa principal.

Aquí indicamos qué número queremos mostrar y durante cuánto tiempo.

## Ejecución

Compile el programa (Verify).

Si no existen errores, seleccione subir (Upload).

Espere a que aparezca el mensaje

```console
Done Uploading
```

Observe el comportamiento del display.

## Tarea del Laboratorio

Una vez verificado el funcionamiento del circuito y comprendida la estructura del programa, complete el código proporcionado por el docente para que el display de siete segmentos muestre la siguiente secuencia:

```
0123456789
```

Cada número deberá permanecer visible durante **2 segundos** antes de mostrar el siguiente.

Al finalizar el número **9**, el programa deberá regresar automáticamente al número **0** y repetir la secuencia de manera continua mientras el Arduino permanezca encendido.

Para completar este reto será necesario identificar qué segmentos deben encenderse para representar correctamente cada uno de los números del **0 al 9**.

{% hint style="danger" %}
**Importante:** No modifique el circuito físico. El reto consiste únicamente en completar el programa utilizando las funciones y estructuras vistas durante el laboratorio.
{% endhint %}

{% hint style="info" %}
Utilice funciones para mostrar cada número para que el código sea sencillo de leer.&#x20;
{% endhint %}

## Restricciones

Durante este laboratorio:

* **No utilizar librerías externas.**
* **No modificar el circuito.**
* **No cambiar la numeración de los pines.**
* **Resolver el problema utilizando únicamente las funciones vistas en clase.**
* **No utilizar inteligencia artificial para generar el código.**&#x20;

## Entregables

Cada pareja deberá entregar:

* Código fuente (.ino)
* Video demostrando el funcionamiento
* Explicación breve (máximo una página) respondiendo:

1. ¿Qué función cumple `setup()`?
2. ¿Qué función cumple `loop()`?
3. ¿Qué ocurre cuando ejecutamos `digitalWrite(pin, HIGH)`?
4. ¿Por qué apagamos todos los segmentos antes de mostrar un nuevo número?
5. ¿Qué dificultades encontraron durante el laboratorio?
