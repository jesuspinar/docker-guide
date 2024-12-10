# Guía Completa de Docker

## Índice

1. [Introducción a Docker](#introducción-a-docker)
   - [¿Por qué se construyó Docker?](#por-qué-se-construyó-docker)
   - [Tipos de Escalabilidad](#tipos-de-escalabilidad)
   - [Docker vs Máquina Virtual](#docker-no-es-una-máquina-virtual)

2. [Instalación](#guía-para-instalar-docker-en-linux)
   - [Instalación en Linux](#guía-para-instalar-docker-en-linux)
   - [Configuración Post-Instalación](#configuración-post-instalación)

3. [Comandos de CLI](#cli---comandos-comunes)
   - [Gestión de Imágenes](#gestión-de-imágenes)
   - [Gestión de Contenedores](#gestión-de-contenedores)
   - [Comandos de Red](#comandos-de-red)
   - [Limpieza de Recursos](#limpieza-de-recursos)

4. [Dockerización](#dockerización)
   - [Creando una Imagen](#creando-una-imagen)
   - [Dockerfile](#crear-un-archivo-dockerfile)

5. [Conceptos Avanzados](#conceptos-avanzados-de-docker)
   - [CMD: Shell vs Exec Form](#cmd-shell-form-vs-cmd-exec-form)
   - [Entrypoint](#entrypoint)
   - [Multi-Stage Builds](#multi-stages-en-docker)

6. [Entorno de Desarrollo](#creando-un-entorno-de-desarrollo-con-docker)
   - [Configuración en VS Code](#en-vscode)
   - [Configuración Manual](#si-quieres-hacer-lo-sin-extensiones)
   - [Soporte y Autocompletado](#para-tener-soporte-y-autocompletado)

7. [Despliegue en Producción](#despliegues-a-producción-consejos)
   - [Mejores Prácticas](#despliegues-a-producción-consejos)
   - [Despliegue en Digital Ocean](#desplegar-en-digital-ocean)

## Introducción a Docker

### ¿Por qué se construyó Docker?

El sistema operativo es parte de tu aplicación. Los programas condicionan cómo se ejecuta tu app. Tradicionalmente, transportar software entre máquinas era complicado debido a diferencias en versiones de sistema operativo y dependencias. Este proceso, conocido como *software shipping*, es lo que Docker busca automatizar.

### Tipos de Escalabilidad

- **Escalabilidad Horizontal**: Añadir más servidores con el mismo número de especificaciones.
- **Escalabilidad Vertical**: Aumentar la potencia de los servidores actuales.

### Docker no es una Máquina Virtual

**Máquina Virtual**: Por cada instrucción que necesita la CPU, el supervisor hace una interpretación para el otro sistema.

**Docker**: Un gestor de **contenedores**. Cada contenedor viene con el software empaquetado, listo para transportarse a cualquier sistema operativo sin necesidad de pedir dependencias. Su sistema de ficheros queda **completamente aislado**.

Cada contenedor se crea a partir de una imagen, que es una especificación de lo que contiene, similar a un albarán.

## Guía para Instalar Docker en Linux

### Instalación

Sigue la guía oficial: [Documentación de Instalación de Docker](https://docs.docker.com/engine/install/)

### Configuración Post-Instalación

- Si tienes problemas para arrancar Docker, verifica:
  - Asignación de usuario
  - Iniciar el demonio con `sudo systemctl start docker`

**Nota Importante**: Al desplegar un proyecto, evita dar privilegios de root al contenedor. Es mejor crear un usuario con permisos específicos.

## CLI - Comandos Comunes

### Gestión de Imágenes y Contenedores

```bash
# Descargar imagen
docker pull imagen debian

# Listar imágenes
docker images

# Listar contenedores
docker ps -a

# Crear contenedor interactivo
docker run -it --name debian-console debian

# Copiar archivos
docker cp ./fichero-host.txt debian-console:/ruta-destino
docker cp debian-console:/fichero-host.txt /ruta-destino

# Iniciar/Detener contenedores
docker start -i debian-console
docker stop debian-console
docker kill debian-console

# Eliminar contenedores
docker container rm -f nombre-contenedor
docker container prune
```

### Redes en Docker

- Los contenedores nuevos están en la misma red (interfaz *bridge*)
- Para acceder desde el host, necesitas publicar puertos

```bash
# Publicar puerto
docker run -p 5000:3000 --name mi-app ubuntu:22.04

# Obtener dirección IP
docker inspect --format "{{.NetworkSettings.IPAddress}}" nombre-contenedor
```

## Creando una Imagen: Dockerfile

```dockerfile
FROM ubuntu:22.04

RUN apt update && apt install -y curl \
    && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs 

WORKDIR /app
COPY index.js ./

EXPOSE 3000

CMD node index.js
```

### Construcción de Imagen

```bash
docker image build --tag usuario/app:version
docker run -p 8010:3000 usuario/app:version
```

## Conceptos Avanzados

### CMD: Shell Form vs Exec Form

| Característica | Shell Form | Exec Form |
|---------------|------------|-----------|
| Número de procesos | 2 | 1 |
| Ctrl + C | Para el proceso | No para |
| Señal SIGTERM | No recibe | Recibe señal |
| Variables de entorno | Se sustituyen | No se sustituyen |

### Multi-Stage Builds

```dockerfile
# Etapa de construcción
FROM ubuntu:22.04 AS builder
# Instalar dependencias...

# Etapa de ejecución
FROM ubuntu:22.04 AS runner
WORKDIR /app
COPY --from=builder /app/binario ./
CMD ./binario
```

## Despliegues a Producción: Mejores Prácticas

- No usar archivos con pertenencia a root
- Crear usuario específico
- No incluir `.git` en producción
- Configurar variables de entorno
- Usar versiones específicas de imágenes
- Manejar señales de cierre correctamente

### Desplegar en Digital Ocean

1. Crear cuenta en Digital Ocean y Docker Hub
2. Subir imagen a Docker Hub
3. Seleccionar imagen y recursos
4. Configurar y desplegar

**Consejo**: Elige un servidor de producción como Nginx o Gunicorn.
