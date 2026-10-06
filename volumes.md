# Docker Volumes

## Why persist data outside a container?

Files written to a container's writable layer remain there when the container is stopped or restarted. That layer is removed when the container is removed, so it is not a good place for data that must outlive the container. Docker volumes and bind mounts store data outside that writable layer.

Docker has two common ways to mount persistent data into a container:

1. **Volumes** are managed by Docker and are the usual choice for data that containers create and use, such as a database's files.
2. **Bind mounts** connect a specific file or directory on the host to a path in the container. They are useful when a container needs to work with host files, such as source code during development.

## Use a named volume

Create a volume:

```sh
docker volume create app-data
```

Write a file into it from a temporary container:

```sh
docker run --rm --mount type=volume,source=app-data,target=/data busybox sh -c "echo hello > /data/message.txt"
```

Read the file from another container:

```sh
docker run --rm --mount type=volume,source=app-data,target=/data,readonly busybox cat /data/message.txt
```

The second command prints `hello`. The `--rm` option removes each temporary container after it exits; it does not remove the volume. A named volume also remains when a container that used it is removed.

Inspect the volume and, when its data is no longer needed, remove it:

```sh
docker volume inspect app-data
docker volume rm app-data
```

Docker won't remove a volume that is still in use by a container. Stop and remove the container first.

## Use a bind mount

Replace the source with an existing absolute path on the host. This example is for Linux or macOS:

```sh
docker run --rm --mount type=bind,source=/absolute/path/to/data,target=/data busybox ls /data
```

In Windows PowerShell, quote the mount specification and use an existing Windows path:

```powershell
docker run --rm --mount "type=bind,source=C:\Users\me\project\data,target=/data" busybox ls /data
```

The container sees the host directory at `/data`. Changes made through a read-write bind mount are changes to the host files too. Add `readonly` to the mount, for example `--mount type=bind,source=/absolute/path/to/data,target=/data,readonly`, if the container only needs to read the files.

## Choosing between them

Use a **named volume** when Docker should manage the storage location, especially for persistent application data. Use a **bind mount** when the host path itself matters, such as sharing project files with a development container. Neither option automatically backs up or replicates data; plan backups separately when the data is important.
