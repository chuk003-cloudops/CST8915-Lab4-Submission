# CST8915 Lab 4: Introduction to Docker

Chukwudubem Chukwurah | Student ID: 041285726

This lab runs a Flask visit counter and Redis together with Docker Compose on an Azure Ubuntu VM.

## Part 6 demo

[Watch the Docker Compose demo on YouTube](https://youtu.be/BSsgFTbECs0) (2 minutes 44 seconds).

The live terminal recording shows the Compose configuration, both services running, increasing HTTP hit counts, web and Redis logs, stop/start/restart, persistence after `down`, three web instances with dynamic ports, and a counter reset after `down -v`. It records Part 6 only.

## Reflection questions

### 1. What are the main differences between a Docker image and a Docker container?

A Docker image is a read-only package containing the application, dependencies, and settings used to create containers. A container is an instance of that image with its own processes, network configuration, and writable layer. For example, `my-python-app:v3` was one image, while `app1`, `app2`, and `app3` were separate containers created from it. Stopping a container does not delete its writable layer; removing the container does.

### 2. Explain how Docker's layered architecture improves efficiency.

Docker reuses unchanged image layers instead of rebuilding and storing everything again. In this lab, copying `requirements.txt` and installing packages came before copying `app.py`. When the hello message changed, the dependency installation stayed cached, while the application-copy step was rebuilt. Images and containers can also share their unchanged base layers, reducing duplicate storage and downloads. Instructions such as `CMD` update image metadata rather than adding file content.

### 3. Why does each container get its own writable layer?

Each container needs a place to store changes without altering the shared image or another container's files. The three containers each had their own `data1.txt`, `data2.txt`, or `data3.txt`; the other containers could not see those files. Modifying `app.py` inside `app1` left `app2` unchanged, demonstrating copy-on-write. This layer is temporary: it disappears when the container is removed. Data that must survive removal belongs in a volume, such as the Redis volume used in Part 6.

### 4. What are the benefits of using Docker Compose over running containers individually?

Docker Compose keeps related services, ports, networks, and volumes in one configuration file. A single `docker compose up -d` built the web image and started Flask and Redis on the same network, where Flask could reach Redis by the service name `redis`. Compose also made it easier to inspect logs, stop or restart the whole application, and scale the web service to three instances. `depends_on` controls startup order but does not guarantee application readiness, so readiness still needs checking.

## Setup and checks

- Azure host: Ubuntu 24.04 LTS, **Standard_B2s**, 2 vCPUs and 4 GB configured RAM, in Denmark East on the student subscription.
- VS Code Remote SSH connected to the VM; Docker Engine was installed from Docker's official Ubuntu APT repository.
- Docker Engine 29.9.0; Docker Compose plugin 5.6.0.
- `hello-world`, NGINX lifecycle commands, Flask `/` and `/health`, image history, cached rebuilds, separate writable layers, and copy-on-write were exercised before the Part 6 recording.
- The Compose counter increased across requests and persisted through stop/start, restart, and container recreation without volume removal. Removing the volume reset it to 1.
- Three dynamically mapped web instances shared the same Redis counter.

The original B2s size was available in Denmark East, so no larger VM was used. Initial HTTP requests sometimes arrived before the container was ready; retrying the real HTTP request resolved that startup timing issue. The obsolete Compose `version` field was omitted, with the same services, network, and volume configuration retained.

## Reproduce the Compose application

`lab-source.zip` contains the Flask examples, Dockerfiles, Compose files, and exercise scripts. Run it only on a disposable lab host: the scripts create and remove lab containers and volumes, and Docker commands have privileged access to the host. Images and packages are downloaded from Docker Hub, Docker's APT repository, and PyPI. The Flask development server is for this lab, not production hosting.

After extracting the archive on a host with Docker installed:

```bash
cd docker-compose-lab
docker compose up -d --build
curl http://localhost/
docker compose ps
```

To scale, first remove the fixed-port stack, then use the dynamic-port configuration:

```bash
docker compose down
docker compose -f compose.scale.yml up -d --scale web=3
docker compose -f compose.scale.yml ps
```

To remove the lab application and its saved Redis data:

```bash
docker compose -f compose.scale.yml down -v
docker compose down -v
```

## References

- [Professor's lab instructions](https://github.com/ramymohamed10/26F_CST8915_Lab4)
- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose documentation](https://docs.docker.com/compose/)
