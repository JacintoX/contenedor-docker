# Practicamos con Docker: Contenedores efímeros

** Jacinto Rodríguez Lucin ** 


---

## Introducción

En esta práctica trabajo por primera vez con Docker usando **contenedores efímeros**, que hacen una tarea concreta y se detienen en cuanto la terminan. El objetivo es seguir el ciclo de vida completo de un contenedor sencillo: descargar la imagen, ejecutar el contenedor, comprobar su estado, eliminar el contenedor y, por último, eliminar la imagen.

> **Nota:** si en las capturas no aparece mi nombre de usuario en el prompt, ejecuto antes `echo "Tu nombre completo"` para que salga en cada captura.

---

## Parte 1: Ejecutar el primer contenedor y observar su ciclo de vida

### 1.1. Ejecución y descarga

**Explicación:** `docker run` crea un contenedor a partir de una imagen y lo arranca. Si la imagen no está en mi equipo, Docker la descarga primero desde Docker Hub. Uso `hello-world`, una imagen mínima que solo sirve para comprobar que Docker funciona.

```bash
docker run hello-world
```


![Captura 1](capturas\2026-11-04-165936.png)

### 1.2. Reflexión: ¿qué mensaje aparece?

**Acciones que Docker reporta:**

1. **Buscar la imagen en local:** indica que no encuentra `hello-world:latest` en el equipo (`Unable to find image ... locally`).
2. **Descargar:** la descarga desde Docker Hub (`Pulling from library/hello-world`).
3. **Verificar:** muestra el `Digest: sha256:...`, que es la huella que identifica de forma única la imagen y garantiza que se ha descargado íntegra, y confirma `Status: Downloaded newer image`.
4. **Ejecutar:** crea un contenedor a partir de la imagen y este imprime el mensaje `Hello from Docker!`, explicando los pasos que ha seguido (el cliente contacta con el demonio, el demonio descarga la imagen, crea el contenedor y devuelve la salida a la terminal).
5. **Salir:** al terminar de imprimir el mensaje, el programa acaba y el contenedor se detiene solo.

**¿Dónde reside la imagen ahora? ¿Se ha eliminado?**
La imagen queda guardada en el almacenamiento local de Docker de mi equipo. **No se elimina** al terminar la ejecución: se conserva para que las próximas veces no haya que descargarla de nuevo.

**¿Sigue "vivo" o terminó?**
Terminó. El contenedor ejecutó su única tarea (mostrar el mensaje) y se detuvo. Su estado final es **Exited (0)**, donde el `0` significa que finalizó correctamente, sin errores.

### 1.3. Verificación del contenedor detenido

**Explicación:** `docker ps` lista contenedores. Por defecto solo muestra los que están **en ejecución**; como el contenedor de `hello-world` ya se detuvo, no aparecería. Por eso hay que añadir `-a` (*all*), que muestra **todos** los contenedores, también los detenidos.

```bash
docker ps -a
```

> ⚠️ Escribir `-a` con un guion normal. Si se copia de las diapositivas puede salir un guion largo (`–a`) y Docker da error.


![Captura 2](capturas\2026-11-04-175157.png)

**Anotaciones:**

| Dato | Valor |
|---|---|
| CONTAINER ID | a97359a456ed |
| STATUS | Exited (0) 25 minutes ago |
| NAMES (aleatorio) | gifted_germain |

---

## Parte 2: Nombrar contenedores para una mejor gestión

### 2.1. Ejecutar con nombre personalizado

**Explicación:** el flag `--name` permite elegir el nombre del contenedor en lugar de que Docker invente uno aleatorio. Esta vez la imagen ya está en local, así que Docker no la descarga y ejecuta directamente.

```bash
docker run --name saludo hello-world
```


![Captura 3](capturas\2026-11-05-184657.png)

> En esta salida ya no aparecen las líneas de descarga (`Pulling...`), porque la imagen se reutiliza desde el equipo.

### 2.2. Verificación

```bash
docker ps -a
```


![Captura 4](capturas\2026-11-05-185035.png)

### 2.3. Reflexión

**¿Qué diferencias noto en la columna NAMES?**
Ahora hay dos contenedores detenidos de la misma imagen. El primero tiene un nombre aleatorio generado por Docker (formado por un adjetivo y el apellido de un científico, como gifted_germain, y el segundo se llama exactamente `saludo`, el nombre que yo elegí.

**¿Por qué es útil asignar un nombre?**
Porque es mucho más fácil de recordar y escribir que un ID hexadecimal o un nombre aleatorio. Además, el nombre lo decido yo y es predecible, así que puedo usarlo en comandos (`docker rm saludo1`, `docker logs saludo1`...) o en scripts sin tener que consultar antes `docker ps -a`. También ayuda a saber de un vistazo para qué sirve cada contenedor cuando hay muchos.

---

## Parte 3: Limpieza de contenedores detenidos

Un contenedor detenido apenas consume recursos, pero su sistema de archivos sigue ocupando espacio en disco, por lo que conviene eliminarlo cuando ya no se necesita.

### 3.1. Eliminación por nombre

**Explicación:** `docker rm` elimina un contenedor detenido. Gracias a haberle puesto nombre, puedo indicarlo directamente.

```bash
docker rm saludo
```



![Captura 5](capturas\2026-11-05-185731.png)

### 3.2. Verificación de la eliminación

```bash
docker ps -a
```



![Captura 6](capturas\2026-11-05-185927.png)

> Sigue apareciendo el contenedor con nombre aleatorio de la Parte 1, porque solo he borrado `saludo`.

---

## Parte 4: Limpieza de imágenes

Aunque haya borrado el contenedor, la imagen `hello-world` sigue en mi equipo para futuras ejecuciones.

### 4.1. Listar imágenes locales

**Explicación:** `docker images` muestra las imágenes descargadas en local, con su repositorio, etiqueta (TAG), ID, fecha de creación y tamaño.

```bash
docker images
```


![Captura 7](capturas\2026-11-05-190211.png)

### 4.2. Eliminación de la imagen

**Explicación:** `docker rmi` elimina una imagen local.



```bash
docker rmi hello-world
```



![Captura 8](capturas\2026-11-05-191040.png)

> ⚠️ **Importante:** Docker no deja borrar una imagen si algún contenedor (aunque esté detenido) la está usando. Como el contenedor de la Parte 1 sigue existiendo, `docker rmi hello-world` daría un error del tipo `conflict: unable to remove repository reference "hello-world" ... container ... is using its referenced image`. Por eso primero elimino ese contenedor por su nombre aleatorio o su ID:
```bash
docker rm gifted_germain
```
![Captura 9](capturas\2026-11-05-191320.png)

```bash
docker rmi hello-world
```
![Captura 10](capturas\2026-11-05-191705.png)

### 4.3. Verificación final

```bash
docker images
```



![Captura 9](capturas\2026-11-05-191902.png)

### 4.4. Reflexión

**¿Diferencia entre `docker rm` y `docker rmi`?**
- `docker rm` elimina **contenedores**, es decir, las instancias concretas que se crearon al ejecutar una imagen (con su capa de escritura y su estado).
- `docker rmi` elimina **imágenes**, la plantilla de solo lectura a partir de la cual se crean los contenedores.

Una imagen es como el "molde" y el contenedor es lo que se fabrica con él. Primero hay que borrar los contenedores que dependen de una imagen para poder borrar la imagen.

**¿Qué ocurriría si ejecutara `docker run hello-world` de nuevo?**
Al no estar la imagen en local, Docker volvería a hacer lo mismo que en la Parte 1: mostraría que no la encuentra, la descargaría de nuevo desde Docker Hub, la verificaría y crearía un contenedor nuevo (con otro ID y otro nombre aleatorio) que mostraría el mensaje y se detendría.

---

## Resumen de comandos

| Comando | Concepto principal |
|---|---|
| `docker run` | Descargar la imagen (si hace falta) y ejecutar un contenedor |
| `docker ps -a` | Mostrar todos los contenedores y su estado, incluidos los detenidos |
| `docker rm` | Eliminar un contenedor detenido |
| `docker images` | Listar las imágenes descargadas en local |
| `docker rmi` | Eliminar una imagen local |

## Conclusión

He aprendido cosas básicas como que un contenedor efímero se detiene al acabar su tarea pero no desaparece hasta que lo borro, y que la imagen y el contenedor son cosas distintas que se gestionan por separado.
