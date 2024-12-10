# Teoría y Uso de Docker

Docker es una herramienta poderosa para desarrollar, desplegar y ejecutar aplicaciones en contenedores, garantizando portabilidad y consistencia en diferentes entornos.
## Índice
1. [¿Por qué se creó Docker?](#por-qué-se-creó-docker)
2. [Tipos de Escalabilidad](#tipos-de-escalabilidad)
3. [¿Qué es Docker?](#qué-es-docker)
4. [Instalación de Docker en Linux](#instalación-de-docker-en-linux)
5. [Comandos Comunes de Docker CLI](#comandos-comunes-de-docker-cli)
   - [Imágenes](#imágenes)
   - [Contenedores](#contenedores)
   - [Inspección y gestión](#inspección-y-gestión)
   - [Limpieza](#limpieza)
6. [Dockerización de Aplicaciones](#dockerización-de-aplicaciones)
   - [Ejemplo para una aplicación Node.js](#ejemplo-para-una-aplicación-nodejs)
   - [Redes en docker](#redes-en-docker)
7. [Conceptos avanzados](#conceptos-avanzados)
   - [Multi-stages en Docker](#multi-stages-en-docker)
   - [CMD: Shell Form vs Exec Form](#cmd-shell-form-vs-exec-form)
8. [Crear contenedores para desarrollo con Docker](#crear-contenedores-para-desarrollo-con-docker)
   - [Usando la extensión oficial de Dev Containers en VSCode](#usando-la-extensión-oficial-de-dev-containers-en-vscode)
   - [Usando la CLI de Docker](#usando-la-cli-de-docker)
9. [Consejos para despliegues a producción](#consejos-para-despliegues-a-producción)
10. [Desplegar una Aplicación en DigitalOcean con Docker](#desplegar-una-aplicación-en-digitalocean-con-docker)
    - [Crear cuentas necesarias](#crear-cuentas-necesarias)
    - [Construir y subir tu imagen a Docker Hub](#construir-y-subir-tu-imagen-a-docker-hub)
    - [Configurar y elegir los recursos en DigitalOcean](#configurar-y-elegir-los-recursos-en-digitalocean)
    - [Configurar el plan y realizar el pago](#configurar-el-plan-y-realizar-el-pago)
    - [Desplegar la aplicación](#desplegar-la-aplicación)

## ¿Por qué se creó Docker?

En el pasado, mover software entre máquinas requería garantizar que el entorno del sistema operativo y sus dependencias fueran idénticos. Este proceso, conocido como _software shipping_, era propenso a errores.

**Docker automatiza esta tarea, garantizando que las aplicaciones puedan ejecutarse en cualquier entorno sin importar el proveedor de infraestructura.**

## Tipos de Escalabilidad

- **Horizontal:** Agregar más servidores con las mismas especificaciones.
- **Vertical:** Incrementar la capacidad de los servidores existentes.


## ¿Por qué se creó Docker?

En el pasado, mover software entre máquinas requería garantizar que el entorno del sistema operativo y sus dependencias fueran idénticos. Este proceso, conocido como _software shipping_, era propenso a errores.

**Docker automatiza esta tarea, garantizando que las aplicaciones puedan ejecutarse en cualquier entorno sin importar el proveedor de infraestructura.**
## Tipos de Escalabilidad

- **Horizontal:** Agregar más servidores con las mismas especificaciones.
- **Vertical:** Incrementar la capacidad de los servidores existentes.

## ¿Qué es Docker?

Docker **no** es una máquina virtual. En lugar de interpretar cada instrucción para el sistema operativo, como lo hacen las máquinas virtuales, Docker utiliza **contenedores**.

- Un **contenedor** es un paquete que incluye todo lo necesario para ejecutar una aplicación: sistema de archivos, bibliotecas, dependencias, etc. Todo su sistema de ficheros queda aislado por defecto.
- Los contenedores se basan en imágenes, que actúan como un plano para definir su contenido.

## Instalación de Docker en Linux

1. [Guía oficial de instalación](https://docs.docker.com/engine/install/)
2. Post-instalación:
    - Asignar un usuario a Docker.
    - Asegurarse de que el demonio de Docker esté en ejecución: 
	     `sudo systemctl start docker`

## Comandos Comunes de Docker CLI

### Imágenes

- Descargar imagen:
    
    ```bash
    docker pull debian
    ```
    
- Listar imágenes:
    
    ```bash
    docker images
    ```
    
### Contenedores

- Listar contenedores:
    
    ```bash
    docker ps -a
    ```
    
- Crear e iniciar un contenedor interactivo:
    
    ```bash
    docker container create --interactive --tty --name [name] [image]
    docker container start --interactive [name]
    ```
    
- Ejecutar un contenedor y adjuntarse a él:
    
    ```bash
    docker container attach [name]
    ```
    
- Copiar archivos:
    
    ```bash
    docker cp ./local-file.txt debian-console:/destination-path
    ```
    
### Inspección y gestión

- Inspeccionar un contenedor:
    
    ```bash
    docker container inspect [name]
    ```

- Ejecutar un comando de un contenedor sin entrar en el:
    
    ```bash
    docker exec --tty [name] apt list installed
    ```
    
- Formas de parar contenedores:
    
    ```bash
    docker stop $(docker ps --quiet)  
	docker stop $(docker ps --filter "name=debian-" --quiet)  
	docker stop $(docker ps --filter "ancestor=fedora-" --quiet)
	docker kill [name] # lo detiene inmediatamente 
    ```

### Limpieza

- Eliminar contenedores :
    
    ```bash
    docker container rm [name | id]
    ```
    
- Eliminar imagen :
    
    ```bash
    docker container rmi [image]
    ```

- Eliminar imágenes no usadas:
    
    ```bash
    docker image prune --all
    ```
    
- Limpieza total del sistema:
    
    ```bash
    docker system prune
    ```
    
**Nota importante**:
- eliminar los contenedores antes de eliminar imágenes 


## Dockerización de Aplicaciones

El proceso de _dockerización_ consiste en encapsular todos los requisitos de tu aplicación dentro de una imagen.

### Ejemplo para una aplicación Node.js:

1. Crear un archivo `Dockerfile` en el directorio del proyecto. 
    
    ```Dockerfile
    FROM ubuntu:22.04
    
    RUN apt update && apt install -y curl \
        && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
        && apt-get install -y nodejs 
    
    WORKDIR /app
    COPY index.js ./
    
    EXPOSE 3000
    CMD node index.js
    ```
    
2. Construir la imagen:
    
    ```bash
    docker build -t usuario/nombre-app:0.1.0 .
    ```
    
3. Ejecutar el contenedor:
    
    ```bash
    docker run -it -p 8010:3000 --name app-node usuario/nombre-app:0.1.0
    ```
    

**Consejos:**

- Usa imágenes ligeras como `alpine` para reducir el tamaño.
- Separa la instalación de dependencias de la copia del código para optimizar el caché.


### Redes en docker 

 **Bridge** es la red por defecto que se asigna a los contenedores y al compartir la pueden comunicarse entre sí, no serán accesibles desde fuera sin configuración adicional. Tendremos que compartir los puertos:
		    
```bash
    docker run -it --publish 5000:3000 --name app-node ubuntu:22.04
```

Ademas de compartir los puertos debemos saber que IP tiene ese contenedor para poder acceder a su servicio, un comando rápido nos facilita esto:
    
```bash
    docker inspect --format "{{.NetworkSettings.IPAddress}}" [container_name]
```
    
Como medida adicional es recomendable que nuestra aplicación del contenedor este configurada con la **IPV4 en la dirección 0.0.0.0** para aceptar conexiones externas.


## Conceptos avanzados 
### Multi-stages en Docker

La idea es tener dos o mas etapas, la primera imagen se encarga de descargar, instalar y compilar. La segunda etapa se usa para ejecutar, esto ahorra en peso en la imagen final. Puedes ver como se hace uso de alias para acceder a recursos de la imagen anterior que se encuentra cacheada.

Aquí puedes ver con un ejemplo:

```Dockerfile
# First stage
FROM ubuntu:22.04 AS builder
# install go deps ... 

# Second stage
FROM ubuntu:22.04
WORKDIR /app
COPY --from=builder /app/app-go ./
EXPOSE 8080
CMD ./app-go
```

De esta forma, la lógica que hay en la primera etapa sirve de forma temporal para instalar las dependencias necesarias para generar el binario. Mientras que la segunda solo se copiara el ejecutable necesario para el sistema.

**Nota Importante:**  Es mejor si usamos la misma distro de linux en ambos etapas.

### CMD: Shell Form vs Exec Form

| Característica       | Shell Form      | Exec Form        |
| -------------------- | --------------- | ---------------- |
| Número de procesos   | 2               | 1                |
| Ctrl + C             | Para el proceso | No para          |
| Señal SIGTERM        | No recibe       | Recibe señales   |
| Variables de entorno | Se sustituyen   | No se sustituyen |

#### Otros parámetros: 

**Entrypoint:** Ejecutable obligatorio que recibe la imagen, puede recibir argumentos.

```Dockerfile
# ...
ENTRYPOINT ["node"]
```


## Crear contenedores para desarrollo con Docker

En esta guía exploraremos dos formas de crear contenedores para desarrollo utilizando Docker. Esto es útil para configurar entornos aislados y reproducibles, ideales para trabajar en proyectos sin necesidad de instalar herramientas directamente en tu sistema.

### 1. Usando la extensión oficial de Dev Containers en VSCode

Visual Studio Code (VSCode) ofrece una integración sencilla para trabajar con contenedores mediante la extensión oficial de Microsoft: **Dev Containers**.

#### Pasos:
1. **Instalar las extensiones necesarias**:
   - Abre VSCode.
   - Ve a la sección de extensiones (`Ctrl+Shift+X` o `Cmd+Shift+X` en macOS).
   - Busca e instala las siguientes extensiones:
     - [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)
     - [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

2. **Abrir tu proyecto en un contenedor de desarrollo**:
   - En VSCode, abre tu proyecto o carpeta deseada.
   - Usa el comando `Ctrl+Shift+P` (o `Cmd+Shift+P` en macOS) para abrir la paleta de comandos.
   - Escribe y selecciona `Dev Containers: Open Folder in Container...`.
   - Sigue las instrucciones para configurar un contenedor. Si no tienes un archivo de configuración, VSCode te guiará para crear uno. En este punto podrás elegir contenedores ya predefinidos para tu lenguaje y tus necesidades.

3. **Personalizar el contenedor** (opcional):
   - VSCode utiliza un archivo llamado `.devcontainer/devcontainer.json` para configurar el entorno del contenedor.
   - Puedes definir la imagen base, extensiones específicas, y más.
  
4. **Para tener soporte y auto completado**:
   - Usa el comando `Ctrl+Shift+P` (o `Cmd+Shift+P` en macOS) y filtra con `>Python: select interpreter`.
   - Selecciona manualmente el path de tu intérprete de Python.
   - Esto puede variar según el lenguaje

5. **Configurar el depurador**:
   - Ve al apartado de "Run & Debug" en la barra lateral.
   - Ejecuta el depurador una vez.
   - Selecciona la configuración sugerida por defecto y asegúrate de elegir la que corresponda a tu aplicación (por ejemplo, Flask si estás trabajando con este framework).

6. **Salir del contenedor**:
   - Para salir del contenedor y volver al entorno local, usa el menú `File > Close Remote Connection`.
### 2. Usando la CLI de Docker

La línea de comandos de Docker (CLI) permite crear y gestionar contenedores de manera directa. Este enfoque es ideal si prefieres trabajar sin interfaces gráficas o deseas un mayor control. 

#### Ejemplo básico:

Para crear un contenedor primero que nos situamos en una carpeta de desarrollo en nuestro sistema. Después puedes usar el siguiente comando:

```bash
docker run -it --workdir /app --name python-dev \
  --mount type=bind,source="$(pwd)",target=/app \
  ubuntu:22.04
```

En este comando:
- `-it`: Ejecuta el contenedor en modo interactivo con un terminal.
- `--workdir /app`: Establece el directorio de trabajo dentro del contenedor.
- `--name python-dev`: Asigna un nombre al contenedor.
- `--mount`: Monta un volumen compartido:
  - `type=bind`: Define un enlace entre el sistema de archivos del host y el contenedor.
  - `source=$(pwd)`: Especifica el directorio actual como origen.
  - `target=/app`: Define el punto de montaje en el contenedor.
- `ubuntu:22.04`: Especifica la imagen base y su versión.

#### Solución a problemas de permisos:

Si encuentras problemas de permisos al acceder a los archivos montados desde el host, puedes resolverlos creando un usuario en el contenedor que coincida con el usuario del host.

1. Identifica tu usuario:
   ```bash
   cat /etc/passwd | grep $(whoami)
   ```

2. Dentro del contenedor, crea un usuario con el mismo UID:
   ```bash
   useradd -m -s "/bin/bash" usuario
   ```

3. Restablecer los permisos sobre archivos existentes:
   ```bash
   chown -R usuario:usuario ./
   ```

## Consejos para despliegues a producción

1. **Evitar que los ficheros del proyecto pertenezcan a `root`**  
    Asegúrate de que ningún fichero del proyecto sea propiedad del usuario `root`. Utiliza un usuario pre configurado en la imagen y cambia su contraseña si es necesario. Por ejemplo:
    
    ```dockerfile
    RUN useradd -m basic-user
    COPY --chown=basic-user:basic-user package*.json ./
    ```
    
2. **Excluir la carpeta `.git` en producción**  
    Añade la carpeta `.git` en el archivo `.dockerignore` para evitar que se incluya en las imágenes de producción.
    
3. **Configurar una variable de entorno para producción**  
    Define una variable de entorno que indique el entorno de producción. Esto permite, entre otras cosas, deshabilitar los logs no necesarios:
    
    ```dockerfile
    ENV NODE_ENV=production
    ```
    
4. **Usar una versión específica de la imagen base**  
    
    ```dockerfile
    FROM node:19.0
    ```
    
5. **Utilizar `exec form` para iniciar procesos**
    - Gestiona correctamente las señales, como `SIGTERM`, para permitir una finalización controlada del proceso.
    - Cierra las conexiones abiertas, como las de bases de datos, antes de finalizar la aplicación.  
        Por ejemplo:
    
    ```dockerfile
    CMD ["node", "app.js"]
    ```
    
6. **Seleccionar un servidor de producción adecuado**  
    Elige un servidor optimizado para entornos de producción, como **Nginx**, **Gunicorn**, entre otros, dependiendo de las necesidades de tu proyecto.

## Desplegar una Aplicación en DigitalOcean con Docker

### 1. Crear cuentas necesarias

Antes de iniciar el despliegue, es indispensable crear las cuentas necesarias:
- **DigitalOcean**: Dirígete a [DigitalOcean](https://www.digitalocean.com/) y regístrate para obtener una cuenta. Esta plataforma será el proveedor de infraestructura donde se ejecutará tu aplicación.
- **Docker Hub**: Accede a [Docker Hub](https://hub.docker.com/) y crea una cuenta. Docker Hub sirve como repositorio centralizado para alojar y gestionar tus imágenes Docker.

### 2. Construir y subir tu imagen a Docker Hub

1. **Construir la imagen Docker**:
   - Desde tu entorno de desarrollo local, asegúrate de tener un `Dockerfile` configurado correctamente.
   - Ejecuta el siguiente comando para construir tu imagen:
     ```bash
     docker build -t nombre_usuario/nombre_imagen:tag .
     ```
     Donde:
     - `nombre_usuario` es tu nombre de usuario en Docker Hub.
     - `nombre_imagen` es el nombre que deseas asignar a la imagen.
     - `tag` es una etiqueta opcional para identificar la versión (por ejemplo, `latest`).

2. **Iniciar sesión en Docker Hub**:
   ```bash
   docker login
   ```
   Ingresa tus credenciales cuando se te solicite.

3. **Subir la imagen a Docker Hub**:
   ```bash
   docker push nombre_usuario/nombre_imagen:tag
   ```

### 3. Configurar y elegir los recursos en DigitalOcean

1. **Crear un droplet en DigitalOcean**:
   - Inicia sesión en tu cuenta de DigitalOcean.
   - Dirígete al apartado de "Droplets" y selecciona la opción de crear un nuevo droplet.

2. **Seleccionar la imagen desde Docker Hub**:
   - En la sección de imágenes, selecciona la pestaña "Container Distributions".
   - Busca tu imagen cargada en Docker Hub utilizando el formato `nombre_usuario/nombre_imagen`.

3. **Asignar recursos de CPU y memoria**:
   - Define los recursos necesarios para tu aplicación, como CPU, memoria y almacenamiento. Asegúrate de elegir un plan que cumpla con los requisitos de tu aplicación para un rendimiento óptimo.

### 4. Configurar el plan y realizar el pago

- DigitalOcean ofrece diferentes planes con cuotas mensuales según los recursos asignados.
- Revisa los costos asociados al plan seleccionado y completa el pago mediante los métodos aceptados por la plataforma.

### 5. Desplegar la aplicación

1. **Iniciar el droplet**:
   - Una vez configurados los recursos y seleccionada la imagen, despliega el droplet.
   - DigitalOcean provisionará el entorno automáticamente.

2. **Probar la aplicación**:
   - Accede a la dirección IP pública del droplet para verificar que tu aplicación esté funcionando correctamente.
   - Si es necesario, realiza ajustes en la configuración del firewall o en las variables de entorno.

3. **Configurar dominio personalizado (opcional)**:
   - Asocia un dominio a tu droplet mediante la configuración DNS en DigitalOcean.

**Notas Importantes:**
- Mantén actualizadas tus imágenes en Docker Hub para garantizar que los cambios y actualizaciones se reflejen en producción.
- Revisa regularmente los logs del sistema y de la aplicación para monitorear el rendimiento y solucionar posibles problemas.


