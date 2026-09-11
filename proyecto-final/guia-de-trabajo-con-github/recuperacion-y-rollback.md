# Recuperación y rollback

Existen ocasiones en que deseamos regresar a un estado anterior, ya sea porque perdimos el seguimiento del trabajo realizado o simplemente queremos iniciar desde 0 una nueva funcionalidad tomando como punto de partida el último código estable.&#x20;

Antes de ejecutar cualquier procedimiento de recuperación, se debe revisar el estado:

```shellscript
git status
```

También es conveniente consultar los últimos commits:

```shellscript
git log --oneline --decorate -10
```

Esto permite conocer la rama activa, los cambios pendientes, los últimos commits y las etiquetas existentes.

### Caso A: descartar cambios de un archivo sin commit

Si se modificó un archivo, pero todavía no se creó un commit:

```shellscript
git restore nombre_del_archivo
```

Ejemplo:

```shellscript
git restore proyecto_final.ino
```

El archivo regresará al estado de su último commit.

{% hint style="warning" %}
**Advertencia:** las modificaciones locales de ese archivo se perderán.
{% endhint %}

### Caso B: descartar todos los cambios locales sin commit

```shellscript
git restore .
```

Este comando restaura todos los archivos controlados por Git.

{% hint style="warning" %}
**Advertencia:** se perderán todas las modificaciones que todavía no hayan sido guardadas en un commit.
{% endhint %}

Antes de ejecutarlo, revise:

```
git status
```

### Caso C: deshacer el último commit y conservar los cambios

Si se creó un commit incorrecto, pero todavía no se ha enviado a GitHub:

```shellscript
git reset --soft HEAD~1
```

Este comando elimina el último commit, pero conserva los cambios realizados.

Después se puede corregir el código y crear nuevamente el commit:

```shellscript
git add .git commit -m "Mensaje corregido"
```

Este procedimiento debe utilizarse solamente si el commit todavía no fue enviado a GitHub.

### Caso D: deshacer un commit que ya fue enviado

Si el commit ya fue enviado a GitHub, no se debe borrar ni reescribir el historial.

Primero buscamos su identificador:

```shellscript
git log --oneline
```

El resultado será parecido a:

```shellscript
a1b2c3d Agregar movimiento del servof
4e5d6c Agregar lectura RFID
```

Para deshacer el commit:

```shellscript
git revert a1b2c3d
```

Luego se envía el nuevo commit:

```shellscript
git push
```

`git revert` no elimina el commit original. Crea un nuevo commit que aplica los cambios contrarios.

Esto permite mantener un historial claro de lo ocurrido.

### Caso E: abandonar el laboratorio y comenzar nuevamente

Si la rama del laboratorio tiene demasiados problemas, se puede comenzar otra vez desde `main`.

Primero debemos asegurarnos de que no existan cambios que deseemos conservar:

```shellscript
git status
```

Cambiamos a `main`:

```shellscript
git switch main
```

Actualizamos:

```shellscript
git pull origin main
```

Eliminamos la rama local:

```shellscript
git branch -D lab/04-rfid-servo
```

Volvemos a crearla desde `main`:

```shellscript
git switch -c lab/04-rfid-servo
```

Como la rama anterior todavía existe en GitHub, actualizamos solamente esa rama:

```shellscript
git push --force-with-lease -u origin lab/04-rfid-servo
```

{% hint style="danger" %}
`--force-with-lease` debe utilizarse únicamente sobre la rama del laboratorio y solo cuando el equipo haya decidido comenzar nuevamente. Nunca debe utilizarse sobre `main`.
{% endhint %}

### Caso F: revertir un laboratorio ya mezclado con `main`

Si una funcionalidad ya fue integrada y posteriormente se descubre que afecta el proyecto, no se debe borrar el historial de `main`.

La forma más sencilla es utilizar GitHub:

1. Abra el Pull Request que fue mezclado.
2. Presione **Revert**.
3. GitHub creará un nuevo Pull Request.
4. Ese nuevo Pull Request contendrá los cambios inversos.
5. Revise el código.
6. Realice nuevamente las pruebas.
7. Mezcle el Pull Request de reversión.

Este procedimiento permite deshacer un laboratorio completo manteniendo evidencia de qué se integró, cuándo se integró, por qué se revirtió y quién realizó la reversión.

Si el botón **Revert** no está disponible, se debe solicitar apoyo al catedrático.

No se debe ejecutar sobre `main`:

```shellscript
git reset --hardgit push --force
```

Estos comandos podrían borrar el historial compartido del proyecto.

### Caso G: consultar una versión anterior

Primero se muestran las etiquetas disponibles:

```shellscript
git tag
```

Por ejemplo:

```shellscript
inicio-proyecto
lab-01-completo
lab-02-completo
lab-03-completo
```

Para observar temporalmente una versión anterior:

```shellscript
git switch --detach lab-02-completo
```

En este modo se puede revisar el código, compilarlo, cargarlo al Arduino y compararlo con la versión actual.

{% hint style="danger" %}
No se recomienda desarrollar directamente en este estado.
{% endhint %}

Para regresar a la versión actual:

```shellscript
git switch main
```

### Caso H: recuperar una versión anterior en una rama nueva

Si se desea modificar o recuperar una versión anterior, se debe crear una rama desde su etiqueta:

```shellscript
git switch -c recuperacion/lab-02 lab-02-completo
```

Después se puede publicar:

```shellscript
git push -u origin recuperacion/lab-02
```

Así se puede trabajar sobre la versión anterior sin modificar ni destruir `main`.

### Caso I: volver al inicio del proyecto

Para recuperar exactamente el esqueleto inicial:

```shellscript
git switch main
git status
git switch -c recuperacion/inicio inicio-proyecto
git push -u origin recuperacion/inicio
```

La rama:

```shellscript
recuperacion/inicio
```

Contendrá exactamente el estado marcado al comenzar el proyecto.

Desde esa rama se puede inspeccionar el esqueleto, realizar pruebas, comparar el código o comenzar una recuperación completa.

{% hint style="warning" %}
No se debe retroceder `main` hasta el inicio. La recuperación se realiza mediante una rama nueva para conservar todo el historial.
{% endhint %}
