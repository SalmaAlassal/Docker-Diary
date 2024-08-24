# Docker Volumes

## Docker File System

In order to understand Docker volumes, it is important to first understand how the Docker file system works.

- **Docker containers consist of two layers:** the Docker **image layer** and the **container layer** which is also known as the **container filesystem**.
    - The **image layer** is usually made up of multiple layers, but it is a **read-only layer**.
    - The **container layer** is a **read-write layer**. Any changes are made in the container layer.

- A **Docker image** is a collection of **read-only layers**. When you launch a container from an image, Docker adds a **read-write layer** (the container layer) to the top of that stack of read-only layers. Any time a file is changed, Docker makes a copy of the file from the read-only layers up into the top read-write layer. This leaves the original (read-only) file unchanged. When a container is deleted, that top read-write layer(the container layer) is lost. This means that any changes made after the container was launched are now gone. But the image layer remains on the host. This is why you can create a new container from the same image and have the same filesystem as the previous container. To solve this issue, Docker provides volumes.


- The location of all data that Docker handles is `/var/lib/docker` on Linux systems. This directory contains all the images, containers, volumes, and other data that Docker uses to run your applications.

--------------------------------------------

# Why Do We Need Docker Volumes?

Docker containers are ephemeral. That means the containers last for a short period. Containers are so flexible. They can be easily created, deleted and scaled as per the need. **But what if your application needs to store some data?** And the container gets removed, or even worse if it somehow crashes? Will you be able to get your data back? The answer is **NO!** The data inside the container gets lost if it gets deleted or re-created.

While docker’s flexible and ephemeral nature makes it easy to scale, it also creates a problem for data persistence. To address this issue of data persistence **docker volumes come to rescue.**


## What are Docker Volumes?

**A volume is a way to persist data outside the container’s filesystem.** Instead of writing data inside the container filesystem, the data is written to a location that resides outside the container, called a **Docker volume.** This ensures that even if you remove the container or the image, the data will still persist.

All volumes are **managed by Docker** and **stored** in a dedicated **directory on your host**, usually `/var/lib/docker/volumes` for Linux systems.

A file can never be a volume. **A volume is always a directory**, and it is a directory which is **created by Docker and handled by Docker** throughout the entire lifetime of the volume. The main purpose of a volume is to populate it with the content of the directory to which you mount it in the container.


## Benefits of Docker Volumes

- **Data Persistence**: Volumes ensure that important data, like databases, configuration files, or user uploads, are retained even when the container is updated, replaced, or removed.

- **Data Sharing**: Volumes can be shared among multiple containers.

- **Backup and Restore**: Since data in volumes is decoupled from the container, you can easily back up and restore data independently of the container’s life-cycle.


## Types of Docker Volumes

### 1. **Named Volumes** `-v some-name:/some/path/in/container`

Named volumes are volumes that are given a specific name when they are created. You can create a named volume using the `-v` flag in the `docker run` command or by using the `VOLUME` instruction in a Dockerfile.

```bash
docker run -v my-volume:/app/data my-image
```

In this example, a named volume called `my-volume` is created and mounted at the `/app/data` path inside the container. The volume is given a specific name, so you can reference it by name in other Docker commands.

Named volumes persist even if you remove the container that uses them. This means that you can create a new container and mount the same named volume to access the data that was stored in the volume by the previous container.

### 2. **Anonymous Volumes** `-v /some/path/in/container`

Anonymous volumes are created **specifically for a single container.** They are typically used when you want temporary storage that is associated with a particular container. Data stored in anonymous volumes will be deleted when the container stops.

They are created when you use the `-v` flag without specifying a name or when you use the `VOLUME` instruction in a Dockerfile.

**Anonymous volumes are given a random name** that's guaranteed to be unique within a given Docker host.

```bash
docker run -v /app/data my-image
```

In this example, an anonymous volume is created at the `/app/data` path inside the container. The volume is not given a specific name, so Docker generates a random name for it.

**Anonymous Volumes can be useful** for ensuring that some Container-internal folder **is not overwritten by a “Bind Mount” for example.** If you want to ensure that a certain folder is always available in the container, you can use an anonymous volume with the same name as the folder you want to ensure is available. Since anonymous volumes are created by Docker, they are guaranteed to be available even if the container is started with a bind mount that would otherwise overwrite the folder.

They are tied to the container's lifecycle and are automatically removed when the container is removed because of `--rm` added on the `docker run` command.

--------------------------------------------