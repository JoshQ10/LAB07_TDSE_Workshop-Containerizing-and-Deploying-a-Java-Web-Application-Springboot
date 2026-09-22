# Containerizing and Deploying a Java Web Application

Workshop that builds a minimal Spring Boot web application, packages it as a Docker image, runs isolated container instances locally, publishes the image to Docker Hub, and deploys it on an Amazon EC2 instance.

## Technology stack

- Java 21 LTS
- Maven 3.9+
- Spring Boot 4.1.1
- Docker Desktop with Docker Compose v2
- Docker Hub
- Amazon Linux 2023 on AWS EC2
- Amazon Corretto 21 container image

## Project structure

```
.
├── pom.xml
├── Dockerfile
├── compose.yaml
└── src/main/java/co/edu/escuelaing/virtualizationlab/
    ├── RestServiceApplication.java
    └── HelloRestController.java
```

## Part 1 — Web application

`GET /greeting?name=<name>` returns `Hello, <name>!`. The port is read from the `PORT` environment variable, defaulting to `9000`.

### Build and run locally

```bash
mvn clean package
java -jar target/*.jar
```

### Verify

```
http://localhost:9000/greeting?name=Pedro
```

Expected response:

```
Hello, Pedro!
```

**Evidence — local execution:**



## Part 2 — Docker image

Build the image (replace `<dockerhub-user>` with your Docker Hub username):

```bash
docker build -t <dockerhub-user>/virtualization-lab:1.0 .
docker images
```

Run one container:

```bash
docker run -d \
  --name virtualization-lab-1 \
  -e PORT=9000 \
  -p 34000:9000 \
  <dockerhub-user>/virtualization-lab:1.0

docker ps
```

Verify:

```
http://localhost:34000/greeting?name=Container
```

**Evidence — image built and container running (`docker images`, `docker ps`):**



### Container isolation

Run two more instances of the same image:

```bash
docker run -d --name virtualization-lab-2 -p 34001:9000 <dockerhub-user>/virtualization-lab:1.0
docker run -d --name virtualization-lab-3 -p 34002:9000 <dockerhub-user>/virtualization-lab:1.0
```

Each container responds independently:

```
http://localhost:34001/greeting?name=Container2
http://localhost:34002/greeting?name=Container3
```

**Evidence — three isolated containers responding independently:**



## Part 3 — Docker Compose

```bash
docker compose up -d --build
docker compose ps
docker compose logs web
```

Verify:

```
http://localhost:8087/greeting?name=Compose
```

**Evidence — Compose environment running:**



## Part 4 — Docker Hub

```bash
docker login
docker tag <dockerhub-user>/virtualization-lab:1.0 <dockerhub-user>/virtualization-lab:latest
docker push <dockerhub-user>/virtualization-lab:1.0
docker push <dockerhub-user>/virtualization-lab:latest
```

**Docker Hub repository URL:**



**Evidence — Docker Hub repository with both tags:**



## Part 5 — AWS EC2 deployment

1. Launch an EC2 instance with Amazon Linux 2023.
2. Security group:
   - SSH (22) restricted to your public IP.
   - Application port (e.g. 8080) restricted to the network that needs access.
3. Connect via SSH and install Docker:

```bash
sudo yum update -y
sudo yum install -y docker
sudo service docker start
sudo usermod -a -G docker ec2-user
```

Reconnect, then pull and run the image:

```bash
docker pull <dockerhub-user>/virtualization-lab:1.0

docker run -d \
  --name virtualization-lab \
  --restart unless-stopped \
  -e PORT=9000 \
  -p 8080:9000 \
  <dockerhub-user>/virtualization-lab:1.0

docker ps
docker logs virtualization-lab
```

Verify:

```
http://<ec2-public-dns>:8080/greeting?name=AWS
```

Expected response:

```
Hello, AWS!
```

**Public deployment URL:**



**Evidence — EC2 deployment (`docker ps`, `docker logs`, browser response):**



## Part 6 — Deployment model and cost analysis

### Deployment model

```
Client
  ↓ HTTP request
EC2 virtual machine
  ↓
Docker Engine
  ↓
Java web application container
```

| Layer | Responsibility |
|---|---|
| EC2 virtual machine | Isolated compute, memory, storage, and network resources rented by the hour. |
| Docker container | Portable execution environment containing the application and its runtime dependencies. |
| Java web application | Receives HTTP requests and provides the business functionality. |
| Security group | Controls which inbound traffic can reach the virtual machine. |

### Workload assumptions

| Scenario | Requests/month | AWS Region | Instance type | Instances | Monthly runtime (h) | EBS storage | Outbound transfer | Avg. request/response size | Runs continuously? | High availability? |
|---|---|---|---|---|---|---|---|---|---|---|
| Small | 10,000 | | | | | | | | | |
| Medium | 100,000 | | | | | | | | | |
| Large | 1,000,000 | | | | | | | | | |

### Cost estimate

**AWS Pricing Calculator estimate:**



| Scenario | Monthly requests | Monthly infrastructure cost | Estimated cost per request | Main cost drivers |
|---|---|---|---|---|
| Small workload | 10,000 | | | EC2 runtime and storage |
| Medium workload | 100,000 | | | EC2 runtime, storage, and network transfer |
| Large workload | 1,000,000 | | | Instance capacity, transfer, and scaling needs |

Estimated cost per request = monthly infrastructure cost / monthly requests.

**Assumptions behind the estimate:**



### Architectural discussion

**Why does an EC2-based deployment have a baseline monthly cost even when the application receives few requests?**



**At which workload level does the fixed cost become less significant per request?**



**What would force you to move from one EC2 instance to multiple instances?**



**Which additional services would a production deployment likely require?**



**Would a serverless deployment be more cost-effective for the small-workload scenario?**



### Conclusion



## Deliverables checklist

- [x] Spring Boot source code
- [x] Dockerfile and compose.yaml
- [x] Build, execution, containerization, and deployment instructions
- [ ] Docker Hub repository URL
- [ ] Evidence of local execution
- [ ] Evidence of Docker image and running containers
- [ ] Evidence of Docker Hub image
- [ ] Evidence of successful EC2 deployment
- [ ] Public deployment URL
- [ ] Deployment-model diagram and cost analysis

Imagen comando para que docker corra desde terminal: <img width="2200" height="454" alt="image" src="https://github.com/user-attachments/assets/e6707257-a52d-4840-a9e8-2333e0185e92" />
contenedor corriendo con prueba: <img width="1892" height="199" alt="image" src="https://github.com/user-attachments/assets/0eb77c49-6ff3-4a79-82ca-f8bff32adaf1" />
prueba contenedor corriendo: <img width="656" height="135" alt="image" src="https://github.com/user-attachments/assets/4668d974-7b75-4d66-bcad-d7940545d5f1" />
prueba 3 contenedores corriendo simultaneamente: <img width="2546" height="285" alt="image" src="https://github.com/user-attachments/assets/d90263b9-7e79-4e44-a3fc-2502fe2eef6b" />
prueba compose: <img width="559" height="158" alt="image" src="https://github.com/user-attachments/assets/27b904c2-3737-4d4b-bcc4-d7cefef4b1de" />
correccion nuevos tags con minusculas: <img width="1611" height="120" alt="image" src="https://github.com/user-attachments/assets/baa81bcb-b4b8-44d3-b495-276286a3ae0a" />
push lab 1.0: <img width="1757" height="161" alt="image" src="https://github.com/user-attachments/assets/f4a115f3-2bfd-433c-bf7c-9c6528193ce7" />




