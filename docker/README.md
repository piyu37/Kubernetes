# Docker & Podman Command Reference

## 📌 Topics / Sections

1. [Create Dockerfile](#create-dockerfile)
2. [Build Docker Image](#build-docker-image)
3. [Run Container](#run-container)
4. [Tag Image](#tag-image)
5. [Push Image to Registry](#push-image-to-registry)
6. [View Container Logs](#view-container-logs)
7. [Save Image as TAR / OCI](#save-image-as-tar--oci)
8. [Run Container with Port Mapping](#run-container-with-port-mapping)
9. [Stop Container](#stop-container)
10. [Remove Images](#remove-images)
11. [Clean Up Unused Images](#clean-up-unused-images)

---

## Create Dockerfile

### Description

Create a Dockerfile that:

* Uses **bash** as the base image
* Runs `ping killercoda.com`

### Dockerfile

```dockerfile
FROM bash
CMD ["ping", "killercoda.com"]
```

Save it as:

```bash
/root/Dockerfile
```

---

## Build Docker Image

### Description

Build a container image from the Dockerfile and tag it.

### Commands

```bash
docker build -f Dockerfile -t piyush/my-app .
```

> `piyush` = Docker Hub account name
> `my-app` = Image name

```bash
podman build -t pinger .
```

Build with multiple tags:

```bash
docker build -t <name1>:<tag1> -t <name2>:<tag2> .
```

List images:

```bash
podman image ls
```

---

## Run Container

### Description

Create and run a container from the image.

### Command

```bash
podman run --name my-ping pinger
```

---

## Tag Image

### Description

Tag an existing image for a local registry.

### Command

```bash
podman tag pinger local-registry:5000/pinger
```

Verify:

```bash
podman image ls
```

---

## Push Image to Registry

### Description

Push image to a registry.

### Commands

Local registry:

```bash
podman push local-registry:5000/pinger
```

Docker Hub:

```bash
docker push <image-name>:<tag>
```

---

## View Container Logs

### Description

View logs from a container.

### Command

```bash
podman logs <container-name>
```

---

## Save Image as TAR / OCI

### Description

Export a Docker image to a tar file.

### Commands

Standard TAR:

```bash
docker save -o <file>.tar <image-name>:<tag>
```

OCI format:

```bash
docker image save --format oci -o /path/to/target/my-oci-image.tar my-oci-image:latest
```

---

## Run Container with Port Mapping

### Description

Run a container in detached mode with port mapping.

### Command

```bash
docker run -it --rm -d -p 8080:80 --name my-container my-image:v0.1
```

---

## Stop Container

### Description

Stop a running container.

### Command

```bash
docker container stop my-container
```

---

## Remove Images

### Description

Remove a specific Docker image.

### Command

```bash
docker image rm my-image:v0.1
```

---

## Clean Up Unused Images

### Description

Remove all unused images.

### Command

```bash
docker image prune -a
```
