# Guía de trabajo con GitHub

### Propósito

Durante el proyecto final trabajaremos con un flujo sencillo inspirado en **Gitflow**.Cada laboratorio se desarrollará en una rama independiente. Cuando la funcionalidad esté terminada y probada, se integrará a la rama principal.

El flujo será:

```
main actualizada
       ↓
rama del laboratorio
       ↓
desarrollo y commits
       ↓
pruebas físicas
       ↓
Pull Request
       ↓
integración a main
```

La rama `main` representará siempre la última versión estable y funcional del sistema.

### Reglas generales

1. No se debe trabajar directamente en `main`.
2. Cada laboratorio debe realizarse en una rama nueva de tipo `feature/`.
3. Antes de crear una rama, se debe actualizar `main`.
4. Los avances deben guardarse mediante commits pequeños.
5. Todo cambio debe subirse a GitHub.
6. Antes de integrar una rama, el programa debe compilar y probarse físicamente.
7. La integración se realizará mediante un **Pull Request**.
8. No se debe utilizar `push --force` sobre la rama `main`.
9. Si ocurre un error, no se borrará el historial de `main`; se utilizará un rollback seguro.

## Convención de ramas

### Nombre de las ramas

Cada laboratorio utilizará una rama con este formato:

```git-commit
feature/numero-descripcion
```

Ejemplos:

```
feature/01-esqueleto
feature/02-teclado
feature/03-eepromfeature
...
```

Los nombres deben escribirse:

* En minúsculas.
* Sin espacios.
* Sin tildes.
* Sin caracteres especiales.
* Utilizando guiones para separar palabras.

El prefijo `feature/` identifica el desarrollo de una funcionalidad nueva, como se acostumbra en muchos proyectos de software. Durante el curso, este será el único prefijo obligatorio.

También podrán encontrarse los siguientes prefijos opcionales:

| Prefijo     | Uso                                            |
| ----------- | ---------------------------------------------- |
| `feature/`  | Desarrollar la funcionalidad de un laboratorio |
| `fix/`      | Corregir un error específico                   |
| `recovery/` | Recuperar o inspeccionar una versión anterior  |

Ejemplos:

```
feature/04-rfid-servo
fix/servo-no-regresa
recovery/inicio-proyecto
```

## Preparación inicial del proyecto

### Marcar el punto de partida

Este procedimiento se realiza una sola vez, cuando el repositorio ya contiene el esqueleto inicial del proyecto.

Primero debemos asegurarnos de estar en la rama principal:

```bash
git switch main
```

Descargamos la versión más reciente:

```bash
git pull origin main
```

Creamos una etiqueta que identifique el inicio del proyecto:

```bash
git tag -a inicio-proyecto -m "Esqueleto inicial del proyecto"
```

Publicamos la etiqueta en GitHub:

```bash
git push origin inicio-proyecto
```

La etiqueta `inicio-proyecto` permitirá recuperar posteriormente el esqueleto original.

Esta etiqueta:

* No se debe eliminar.
* No se debe modificar.
* No se debe volver a crear.
* Representará siempre el estado inicial del proyecto.

## Procedimiento antes de cada laboratorio

### Revisar el estado del repositorio

Antes de comenzar, abra una terminal dentro de la carpeta del proyecto y ejecute:

```shellscript
git status
```

Idealmente, Git deberá mostrar un mensaje similar a:

```
nothing to commit, working tree clean
```

{% hint style="info" %}
Esto significa que no existen cambios pendientes. Si aparecen archivos modificados, no continúe hasta decidir si esos cambios deben guardarse mediante un commit, conservarse temporalmente o descartarse.
{% endhint %}

### Cambiar a la rama principal

```shellscript
git switch main
```

### Actualizar la rama principal

```shellscript
git pull origin main
```

{% hint style="danger" %}
Este paso es obligatorio porque garantiza que el nuevo laboratorio comience desde la versión estable más reciente. Si se crea una rama sin actualizar `main`, podrían faltar las funcionalidades integradas en el laboratorio anterior.
{% endhint %}

### Crear la rama del laboratorio

Para crear una nueva rama:

```shellscript
git switch -c feature/04-rfid-servo
```

La opción `-c` crea la rama y cambia inmediatamente hacia ella. El nombre deberá sustituirse por el número y la descripción del laboratorio correspondiente.

### Confirmar la rama activa

```shellscript
git branch --show-current
```

Git deberá mostrar:

```
feature/04-rfid-servo
```

Antes de modificar el código, es muy importante comprobar que estamos en la rama correcta.

### Publicar la rama en GitHub

```shellscript
git push -u origin feature/04-rfid-servo
```

La opción `-u` conecta la rama local con su correspondiente rama en GitHub.

Este comando completo se utiliza solamente la primera vez. Después será suficiente ejecutar:

```shellscript
git push
```

## Trabajo durante el laboratorio

### Realizar cambios en el programa

Los estudiantes pueden comenzar a desarrollar la funcionalidad correspondiente al laboratorio.

Por ejemplo:

* Lectura del teclado.
* Almacenamiento en EEPROM.
* Lectura de una tarjeta RFID.
* Movimiento del servo.
* Activación del buzzer.
* Integración con la pantalla LCD.

Todo el trabajo deberá realizarse dentro de la rama `feature/` correspondiente.

### Crear commits de avance

No es necesario esperar hasta terminar todo el laboratorio para crear un commit.

Cuando se complete una funcionalidad pequeña, primero se debe revisar el estado:

```shellscript
git status
```

Después se agregan los archivos modificados:

```shellscript
git add .
```

Se crea el commit:

```shellscript
git commit -m "Agregar lectura del UID de la tarjeta"
```

Finalmente, se envía el avance a GitHub:

```shellscript
git push
```

### Ejemplos de buenos mensajes de commit

```
Agregar lectura del UID de la tarjeta
Conectar y probar el servo
Validar tarjeta RFID autorizada
Mostrar estado de acceso en LCDA
```

Se deben evitar mensajes poco descriptivos como:

```
cambios
avance
prueba final
ya funciona
corrección
```

Un buen mensaje debe indicar claramente qué se hizo.

### ¿Cuándo crear un commit?

Se recomienda crear un commit cada vez que se alcance un avance que compile correctamente, implemente una función concreta, corrija un error identificable, se tenga un avance que no se desea perder y/o deje el programa en un estado estable.

Por ejemplo, durante el laboratorio de RFID y servo podrían existir estos commits:

```
Agregar librería y configuración del RFID
Mostrar UID en el monitor serial
Comparar UID con tarjeta autorizada
Mover servo al autorizar el acceso
Agregar confirmación mediante buzzer
```

Así, si el último cambio produce un error, será más fácil regresar a un punto anterior.

## Finalización del laboratorio

### Realizar las pruebas finales

Antes de integrar la rama a `main`, se debe comprobar lo siguiente:

* El programa compila sin errores.
* El programa puede cargarse en el Arduino.
* La nueva funcionalidad trabaja físicamente.
* Las funcionalidades anteriores continúan trabajando.
* No quedaron mensajes de depuración innecesarios.
* No quedaron archivos temporales.
* No quedaron contraseñas o información privada.
* Todos los cambios están guardados en commits.
* Todos los commits fueron enviados a GitHub.

En el proyecto final no bastará con demostrar que la funcionalidad nueva trabaja. También deberá comprobarse que no dañó lo desarrollado anteriormente.

### Guardar los últimos cambios

```shellscript
git status
git add .
git commit -m "Completar laboratorio RFID y servo"
git push
```

Después verificamos nuevamente:

```shellscript
git status
```

El resultado esperado es:

```
nothing to commit, working tree clean
```

## Crear el Pull Request

### ¿Qué es un Pull Request?

Un **Pull Request**, también llamado PR, es una solicitud para integrar los cambios de una rama en otra.

En nuestro caso, por ejemplo se podrí solicitará integrar:

```shellscript
feature/04-rfid-servo → main
```

El Pull Request permite revisar los cambios, verificar qué archivos fueron modificados, discutir correcciones, detectar conflictos, aprobar la integración y mantener evidencia del trabajo realizado.

Esto simula la forma de trabajo utilizada en proyectos reales.

### Crear el Pull Request en GitHub

En GitHub:

1. Abra el repositorio del proyecto.
2. Ingrese en la sección **Pull requests**.
3. Presione **New pull request**.
4. Seleccione las ramas:

```
base: main
compare: feature/04-rfid-servo
```

5. Presione **Create pull request**.
6. Escriba un título claro.
7. Complete la descripción.
8. Vuelva a presionar **Create pull request**.

Un título adecuado sería:

```
Integrar laboratorio 04: RFID y servo
```

### Descripción sugerida del Pull Request

#### Funcionalidades agregadas

* Lectura del UID mediante RFID.
* Validación de tarjetas autorizadas.
* Apertura y cierre mediante servo.
* Confirmación mediante buzzer.

#### Pruebas realizadas

* [ ] El programa compila.
* [ ] El programa carga correctamente en el Arduino.
* [ ] Reconoce una tarjeta autorizada.
* [ ] Rechaza una tarjeta no autorizada.
* [ ] El servo abre y vuelve a cerrar.
* [ ] El buzzer confirma el acceso.
* [ ] Las funciones anteriores siguen trabajando.

#### Observaciones

Indicar cualquier limitación, problema conocido o decisión importante tomada durante el laboratorio.

### Revisar los cambios

Dentro del Pull Request, abra la pestaña:

```
Files changed
```

Revise que:

* Solo aparecen cambios relacionados con el laboratorio.
* No se hayan eliminado funciones anteriores accidentalmente.
* No existen archivos innecesarios.
* El código está ordenado.
* No aparezcan credenciales o información privada.

Si todavía hay correcciones pendientes, no es necesario cerrar el Pull Request.

Se puede regresar al programa, corregirlo y ejecutar:

```shellscript
git add .
git commit -m "Corregir validación de tarjeta autorizada"
git push
```

El Pull Request se actualizará automáticamente.

## Mezclar la rama con `main`

### Integrar el laboratorio

Cuando el código esté completo, revisado y probado, se puede realizar la integración.

En GitHub, dentro del Pull Request, presione:

```
Merge pull request
```

Luego:

```
Confirm merge
```

Cuando esté disponible, se utilizará la opción:

```
Create a merge commit
```

Esta opción conserva claramente el momento en el que todo el laboratorio fue integrado. También facilita revertir posteriormente la integración completa.

### Actualizar la copia local

Después de realizar la mezcla en GitHub, la computadora todavía puede tener una versión anterior de `main`.

Cambie a la rama principal:

```shellscript
git switch main
```

Descargue la versión integrada:

```shellscript
git pull origin main
```

La rama `main` local contendrá ahora la funcionalidad del laboratorio.

### Marcar la versión estable del laboratorio

Después de probar la integración, se creará una etiqueta:

```shellscript
git tag -a lab-04-completo -m "Laboratorio 04 RFID y servo completo"
```

La etiqueta se publica con:

```shellscript
git push origin lab-04-completo
```

Ejemplos de etiquetas:

```
inicio-proyectolab-01-completo
lab-02-completo
lab-03-completo
lab-04-completo
```

Estas etiquetas funcionan como puntos de recuperación y también pueden servir para la evaluación del proyecto.

Es importante diferenciar:

* `feature/04-rfid-servo` es una rama temporal de trabajo.
* `lab-04-completo` es una etiqueta permanente que identifica una entrega estable.

## Resumen rápido

### Antes de cada laboratorio

```shellscript
git status
git switch main
git pull origin main
git switch -c lab/NN-descripcion
git push -u origin lab/NN-descripcion
```

### Durante el laboratorio

```shellscript
git status
git add .
git commit -m "Descripción concreta del avance"
git push
```

### Al finalizar

1. Compilar el programa.
2. Cargarlo en el Arduino.
3. Probar la nueva funcionalidad.
4. Probar las funcionalidades anteriores.
5. Guardar todos los cambios.
6. Crear el Pull Request.
7. Revisar `Files changed`.
8. Integrar mediante `Create a merge commit`.
9. Actualizar `main` local.
10. Crear la etiqueta del laboratorio.

```shellscript
git switch main
git pull origin main
git tag -a lab-NN-completo -m "Laboratorio NN completo"
git push origin lab-NN-completo
```

## Regla de oro

Si algo falla en una rama del laboratorio, el equipo puede corregirla, revertir un commit, recuperar una versión anterior o comenzar nuevamente.

La rama `main`, en cambio, representa la última versión funcional del proyecto y no debe modificarse de forma destructiva.

Nunca se debe utilizar `push --force` sobre `main`.