# Theory and Use of Docker

Docker is a powerful tool for developing, deploying, and running applications in containers, ensuring portability and consistency across different environments.

## Table of Contents
1. [Why was Docker created?](#why-was-docker-created)
2. [Types of Scalability](#types-of-scalability)
3. [What is Docker?](#what-is-docker)
4. [Installing Docker on Linux](#installing-docker-on-linux)
5. [Common Docker CLI Commands](#common-docker-cli-commands)
    - [Images](#images)
    - [Containers](#containers)
    - [Inspection and management](#inspection-and-management)
    - [Cleanup](#cleanup)
6. [Dockerizing Applications](#dockerizing-applications)
    - [Example for a Node.js application](#example-for-a-nodejs-application)
    - [Docker networks](#docker-networks)
7. [Advanced Concepts](#advanced-concepts)
    - [Multi-stage builds in Docker](#multi-stage-builds-in-docker)
    - [CMD: Shell Form vs Exec Form](#cmd-shell-form-vs-exec-form)
8. [Creating Development Containers with Docker](#creating-development-containers-with-docker)
    - [Using the official Dev Containers extension in VSCode](#using-the-official-dev-containers-extension-in-vscode)
    - [Using the Docker CLI](#using-the-docker-cli)
9. [Tips for Production Deployments](#tips-for-production-deployments)
10. [Deploying an Application on DigitalOcean with Docker](#deploying-an-application-on-digitalocean-with-docker)
     - [Create necessary accounts](#create-necessary-accounts)
     - [Build and push your image to Docker Hub](#build-and-push-your-image-to-docker-hub)
     - [Configure and choose resources on DigitalOcean](#configure-and-choose-resources-on-digitalocean)
     - [Configure the plan and make the payment](#configure-the-plan-and-make-the-payment)
     - [Deploy the application](#deploy-the-application)

## Why was Docker created?

In the past, moving software between machines required ensuring that the operating system environment and its dependencies were identical. This process, known as _software shipping_, was problematic and time-consuming. Docker automates this task, ensuring that applications can run in any environment regardless of the infrastructure provider.

## Types of Scalability

- **Horizontal:** Adding more servers with the same specifications.
- **Vertical:** Increasing the capacity of existing servers.

## What is Docker?

Docker is **not** a virtual machine. Instead of interpreting each instruction for the operating system like virtual machines do, Docker uses **containers**.

- A **container** is a package that includes everything needed to run an application: file system, libraries, dependencies, etc. Its file system is isolated by default.
- Containers are based on images, which act as a blueprint to define their content.

## Installing Docker on Linux

1. [Official installation guide](https://docs.docker.com/engine/install/)
2. Post-installation:
    - Assign a user to Docker.
    - Ensure the Docker daemon is running:
      `sudo systemctl start docker`

## Common Docker CLI Commands

### Images

- Pull an image:

    ```bash
    docker pull debian
    ```

- List images:

    ```bash
    docker images
    ```

### Containers

- List containers:

    ```bash
    docker ps -a
    ```

- Create and start an interactive container:

    ```bash
    docker container create --interactive --tty --name [name] [image]
    docker container start --interactive [name]
    ```

- Run a container and attach to it:

    ```bash
    docker container attach [name]
    ```

- Copy files:

    ```bash
    docker cp ./local-file.txt debian-console:/destination-path
    ```

### Inspection and management

- Inspect a container:

    ```bash
    docker container inspect [name]
    ```

- Execute a command in a container without entering it:

    ```bash
    docker exec --tty [name] apt list installed
    ```

- Ways to stop containers:

    ```bash
    docker stop $(docker ps --quiet)  
    docker stop $(docker ps --filter "name=debian-" --quiet)  
    docker stop $(docker ps --filter "ancestor=fedora-" --quiet)
    docker kill [name] # stops it immediately 
    ```

### Cleanup

- Remove containers:

    ```bash
    docker container rm [name | id]
    ```

- Remove image:

    ```bash
    docker container rmi [image]
    ```

- Remove unused images:

    ```bash
    docker image prune --all
    ```

- Complete system cleanup:

    ```bash
    docker system prune
    ```

**Important Note**:
- Remove containers before removing images.

## Dockerizing Applications

The process of _dockerizing_ involves encapsulating all your application's requirements within an image.

### Example for a Node.js application:

1. Create a `Dockerfile` in the project directory:

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

2. Build the image:

    ```bash
    docker build -t user/app-name:0.1.0 .
    ```

3. Run the container:

    ```bash
    docker run -it -p 8010:3000 --name app-node user/app-name:0.1.0
    ```

**Tips:**

- Use lightweight images like `alpine` to reduce size.
- Separate dependency installation from code copying to optimize cache.

### Docker networks

**Bridge** is the default network assigned to containers, allowing them to communicate with each other. They will not be accessible from the outside without additional configuration. You need to know the IP of the container to access its service, a quick command facilitates this:

```bash
docker inspect --format "{{.NetworkSettings.IPAddress}}" [container_name]
```

Additionally, it is recommended that your container application is configured with **IPV4 at address 0.0.0.0** to accept external connections.

## Advanced Concepts 

### Multi-stage builds in Docker

The idea is to have two or more stages, the first image handles downloading, installing, and compiling. The second stage is used for execution, saving space in the final image. An example:

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

The logic in the first stage is temporarily used to install the necessary dependencies to generate the binary. The second stage only copies the executable.

**Important Note:** Use the same Linux distro in both stages.

### CMD: Shell Form vs Exec Form

| Feature             | Shell Form      | Exec Form        |
| -------------------- | --------------- | ---------------- |
| Number of processes | 2               | 1                |
| Ctrl + C            | Stops process   | Doesn't stop     |
| SIGTERM signal      | Doesn't receive | Receives signals |
| Environment variables | Substituted    | Not substituted  |

#### Other parameters: 

**Entrypoint:** Mandatory executable that the image receives, can receive arguments.

```Dockerfile
# ...
ENTRYPOINT ["node"]
```

## Creating Development Containers with Docker

This guide explores two ways to create development containers using Docker. This is useful for setting up isolated and reproducible environments, ideal for working on projects.

### 1. Using the official Dev Containers extension in VSCode

Visual Studio Code (VSCode) offers easy integration for working with containers using Microsoft's official **Dev Containers** extension.

#### Steps:
1. **Install necessary extensions**:
    - Open VSCode.
    - Go to the extensions section (`Ctrl+Shift+X` or `Cmd+Shift+X` on macOS).
    - Search and install the following extensions:
      - [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)
      - [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

2. **Open your project in a development container**:
    - In VSCode, open your desired project or folder.
    - Use the `Ctrl+Shift+P` command (or `Cmd+Shift+P` on macOS) to open the command palette.
    - Type and select `Dev Containers: Open Folder in Container...`.
    - Follow the instructions to set up a container. If you don't have a configuration file, VSCode will guide you to create one. At this point, you can choose pre-defined containers.

3. **Customize the container** (optional):
    - VSCode uses a `.devcontainer/devcontainer.json` file to configure the container environment.
    - You can define the base image, specific extensions, and more.

4. **For support and autocomplete**:
    - Use the `Ctrl+Shift+P` command (or `Cmd+Shift+P` on macOS) and filter with `>Python: select interpreter`.
    - Manually select your Python interpreter path.
    - This may vary depending on the language.

5. **Configure the debugger**:
    - Go to the "Run & Debug" section in the sidebar.
    - Run the debugger once.
    - Select the suggested default configuration and make sure to choose the one that corresponds to your application (e.g., Flask if working with this framework).

6. **Exit the container**:
    - To exit the container and return to the local environment, use `File > Close Remote Connection`.

### 2. Using the Docker CLI

The Docker command line interface (CLI) allows you to create and manage containers directly. This approach is ideal if you prefer working without graphical interfaces or want more control.

#### Basic example:

To create a container, first navigate to a development folder on your system. Then you can use the following command:

```bash
docker run -it --workdir /app --name python-dev \
  --mount type=bind,source="$(pwd)",target=/app \
  ubuntu:22.04
```

In this command:
- `-it`: Runs the container in interactive mode with a terminal.
- `--workdir /app`: Sets the working directory inside the container.
- `--name python-dev`: Assigns a name to the container.
- `--mount`: Mounts a shared volume:
  - `type=bind`: Defines a link between the host's file system and the container.
  - `source=$(pwd)`: Specifies the current directory as the source.
  - `target=/app`: Defines the mount point in the container.
- `ubuntu:22.04`: Specifies the base image and its version.

#### Solving permission issues:

If you encounter permission issues accessing files mounted from the host, you can resolve them by creating a user in the container that matches the host user.

1. Identify your user:
    ```bash
    cat /etc/passwd | grep $(whoami)
    ```

2. Inside the container, create a user with the same UID:
    ```bash
    useradd -m -s "/bin/bash" user
    ```

3. Reset permissions on existing files:
    ```bash
    chown -R user:user ./
    ```

## Tips for Production Deployments

1. **Avoid project files belonging to `root`**  
    Ensure no project files are owned by the `root` user. Use a pre-configured user in the image and change its password if necessary. For example:

    ```dockerfile
    RUN useradd -m basic-user
    COPY --chown=basic-user:basic-user package*.json ./
    ```

2. **Exclude the `.git` folder in production**  
    Add the `.git` folder to the `.dockerignore` file to avoid including it in production images.

3. **Configure a production environment variable**  
    Define an environment variable that indicates the production environment. This allows, among other things, disabling unnecessary logs:

    ```dockerfile
    ENV NODE_ENV=production
    ```

4. **Use a specific version of the base image**  

    ```dockerfile
    FROM node:19.0
    ```

5. **Use `exec form` to start processes**
    - Properly handle signals like `SIGTERM` for controlled process termination.
    - Close open connections, such as database connections, before terminating the application.  
        For example:

    ```dockerfile
    CMD ["node", "app.js"]
    ```

6. **Select an appropriate production server**  
    Choose a server optimized for production environments, such as **Nginx**, **Gunicorn**, among others, depending on your project's needs.

## Deploying an Application on DigitalOcean with Docker

### 1. Create necessary accounts

Before starting the deployment, it is essential to create the necessary accounts:
- **DigitalOcean**: Go to [DigitalOcean](https://www.digitalocean.com/) and register for an account. This platform will be the infrastructure provider where your application will run.
- **Docker Hub**: Access [Docker Hub](https://hub.docker.com/) and create an account. Docker Hub serves as a centralized repository to host and manage your Docker images.

### 2. Build and push your image to Docker Hub

1. **Build the Docker image**:
    - From your local development environment, ensure you have a properly configured `Dockerfile`.
    - Run the following command to build your image:
      ```bash
      docker build -t username/image-name:tag .
      ```
      Where:
      - `username` is your Docker Hub username.
      - `image-name` is the name you want to assign to the image.
      - `tag` is an optional tag to identify the version (e.g., `latest`).

2. **Log in to Docker Hub**:
    ```bash
    docker login
    ```
    Enter your credentials when prompted.

3. **Push the image to Docker Hub**:
    ```bash
    docker push username/image-name:tag
    ```

### 3. Configure and choose resources on DigitalOcean

1. **Create a droplet on DigitalOcean**:
    - Log in to your DigitalOcean account.
    - Go to the "Droplets" section and select the option to create a new droplet.

2. **Select the image from Docker Hub**:
    - In the images section, select the "Container Distributions" tab.
    - Search for your uploaded image on Docker Hub using the format `username/image-name`.

3. **Assign CPU and memory resources**:
    - Define the necessary resources for your application, such as CPU, memory, and storage. Ensure to choose a plan that meets your application's requirements for optimal performance.

### 4. Configure the plan and make the payment

- DigitalOcean offers different plans with monthly fees based on the assigned resources.
- Review the costs associated with the selected plan and complete the payment using the accepted methods on the platform.

### 5. Deploy the application

1. **Start the droplet**:
    - Once the resources are configured and the image is selected, deploy the droplet.
    - DigitalOcean will automatically provision the environment.

2. **Test the application**:
    - Access the public IP address of the droplet to verify that your application is working correctly.
    - If necessary, make adjustments to the firewall configuration or environment variables.

3. **Configure a custom domain (optional)**:
    - Associate a domain with your droplet through the DNS configuration in DigitalOcean.

**Important Notes:**
- Keep your images on Docker Hub updated to ensure that changes and updates are reflected in production.
- Regularly check system and application logs to monitor performance and troubleshoot potential issues.
