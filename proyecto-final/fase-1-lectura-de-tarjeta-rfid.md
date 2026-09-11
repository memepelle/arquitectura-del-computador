# Fase 1: Lectura de tarjeta RFID

En esta primera fase comenzaremos a implementar el flujo real del **Sistema de Control de Acceso y Seguridad**.

El objetivo será lograr que el sistema permanezca esperando una tarjeta RFID y que, cuando una tarjeta sea acercada al lector RC522, el Arduino pueda **detectarla, leer su identificador (UID) y continuar con la siguiente etapa del sistema**.

En esta fase **todavía no se determinará si la tarjeta está autorizada o no**. Esa funcionalidad será implementada posteriormente.

### ¿Qué debe hacer el sistema?

Al encender el sistema, la pantalla deberá indicar que se encuentra esperando una tarjeta:

```
SISTEMA URL
ACERQUE TARJETA
```

El programa permanecerá en el estado:

```
ESPERANDO_RFID
```

Cuando se acerque una tarjeta al RC522, el sistema deberá obtener su UID.

El flujo esperado en esta fase es:

<figure><img src="https://2529195988-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FN1KcgI4teUFIowGXx5QB%2Fuploads%2FwsMNmR3lFnS2sUgRNwC5%2Fimage.png?alt=media&amp;token=c32a17a0-b8bf-4ef6-a849-7318abb96381" alt="" width="375"><figcaption></figcaption></figure>

### Trabajo a realizar

En el código base del proyecto encontrarán la función:

```cpp
String leerTarjeta(){    
    /*       TODO - LABORATORIO RFID    */    
    return "";
}
```

Su trabajo consiste en implementar esta función para que se comunique con el módulo **RC522 mediante SPI**, detecte una tarjeta y obtenga su UID.

La función deberá respetar el siguiente comportamiento:

* Si **no existe una tarjeta**, deberá retornar una cadena vacía.
* Si se detecta correctamente una tarjeta, deberá retornar su **UID**.

No deberán modificar `setup()` ni `loop()` para realizar esta fase. El programa principal ya se encuentra preparado para utilizar el resultado de `leerTarjeta()`.

### Resultado esperado

Al finalizar esta fase deberá ser posible ejecutar la siguiente prueba:

1. Encender o reiniciar el sistema.&#x20;
2. Verificar: "ACERQUE TARJETA"&#x20;
3. No acercar ninguna tarjeta. → El sistema permanece esperando.&#x20;
4. Acercar una tarjeta RFID.&#x20;
   1. El RC522 detecta la tarjeta.
   2. El Arduino obtiene el UID.
   3. El sistema continúa a la siguiente etapa.

No importa qué tarjeta se utilice en esta fase. **Todas las tarjetas correctamente detectadas podrán continuar**, ya que la validación del UID se implementará en la siguiente fase.

### ¿Cuándo está terminada la Fase 1?

La fase se considera completada cuando:

**El sistema puede permanecer esperando una tarjeta, detectar una tarjeta real, obtener correctamente su UID y continuar el flujo del programa sin realizar modificaciones en `setup()` o `loop()`.**

Al finalizar el trabajo, verifiquen el funcionamiento utilizando el flujo normal del sistema y realicen el `commit` correspondiente en la rama de trabajo indicada de acuerdo con la guía de GitHub.

Una vez que la funcionalidad haya sido probada y se encuentre estable, podrán integrarla a la rama principal siguiendo el procedimiento establecido.

#### Entregable

Al finalizar esta fase, deberá realizar el Pull Request correspondiente siguiendo la guía de GitHub proporcionada y verificar que la funcionalidad haya sido integrada correctamente a la rama `main`.

En el portal de la universidad deberá entregar **el enlace al Pull Request de GitHub** y **un video corto demostrando el funcionamiento de la fase**. El video deberá mostrar el circuito físico y el resultado esperado descrito anteriormente.

No es necesario adjuntar nuevamente el código fuente al portal, ya que el historial de cambios, commits y código desarrollado será revisado directamente desde GitHub.