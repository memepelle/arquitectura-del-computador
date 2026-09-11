# Resolución de conflictos

### ¿Cuándo aparece un conflicto?

Un conflicto puede ocurrir cuando otra persona modificó las mismas líneas, `main` cambió después de crear la rama, dos ramas modificaron la misma función o se eliminó un archivo que otra rama todavía utiliza.

GitHub indicará que la rama no puede mezclarse automáticamente.

### Incorporar los cambios recientes de `main`

Primero cambie a la rama del laboratorio:

```shellscript
git switch lab/04-rfid-servo
```

Descargue la información más reciente:

```shellscript
git fetch origin
```

Integre los cambios de `main`:

```shellscript
git merge origin/main
```

Si existen conflictos, Git marcará los archivos afectados.

### Identificar los conflictos

Dentro de los archivos aparecerán marcas similares a estas:

```
<<<<<<< HEAD
cambios de la rama del laboratorio
=======
cambios que ya existen en main
>>>>>>> origin/main
```

La sección superior contiene los cambios de la rama actual. La sección inferior contiene los cambios provenientes de `main`.

Los estudiantes deben comparar ambas versiones, decidir cuál conservar, combinar el código cuando sea necesario, eliminar las marcas de conflicto, guardar el archivo, compilar nuevamente y realizar las pruebas físicas.

### Guardar la resolución

Después de resolver todos los conflictos:

```shellscript
git add .
git commit -m "Resolver conflictos con la rama main"
git push
```

El Pull Request se actualizará automáticamente.&#x20;

{% hint style="warning" %}
La rama no debe mezclarse hasta comprobar que la resolución del conflicto no dañó ninguna funcionalidad anterior.
{% endhint %}