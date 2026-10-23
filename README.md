# HelloDevOps

**A beginner-friendly CI/CD pipeline using Java, GitHub, Jenkins, and Docker.**

HelloDevOps demonstrates how a source-code change can be checked out, compiled, executed, packaged into a Docker image, and verified by an automated Jenkins pipeline.

## Pipeline overview

```text
GitHub repository
      |
      v
Jenkins: Checkout
      |
      v
Compile Java -----> Run Java + verify output
      |
      v
Build Docker image
      |
      v
Run container + verify output
      |
      v
Build SUCCESS
```

## Repository structure

```text
helloDevops/
├── HelloDevOps.java       # Java application
├── Dockerfile             # Builds the application image
├── Jenkinsfile            # CI pipeline as code
├── compose.yaml           # Local Jenkins + Docker-in-Docker setup
├── .gitignore
├── README.md
└── jenkins/
    └── Dockerfile         # Jenkins image with Git and Docker CLI
```

## Technology stack

| Technology | Purpose |
| --- | --- |
| Java 21 | Sample application |
| Git and GitHub | Source control and remote repository |
| Jenkins | CI pipeline orchestration |
| Docker | Container image build and runtime |
| Docker Compose | Local development environment |

## Quick start (Windows)

### Prerequisites

- Docker Desktop running with Linux containers enabled
- Git for Windows
- Internet access to download the required container images

A local JDK 21 is optional; Jenkins and the application image each include a Java 21 runtime/toolchain.

### 1. Clone the repository

```powershell
git clone https://github.com/YashLakkas24/helloDevops.git
cd helloDevops
```

### 2. Start Jenkins

Run these commands from the repository root, where `compose.yaml` is located:

```powershell
docker compose up -d --build
docker compose ps
```

Wait until both services are running and the Docker service is healthy.

Open [http://localhost:8080](http://localhost:8080). Retrieve the initial Jenkins administrator password with:

```powershell
docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Complete the Jenkins setup and create the administrator account.

### 3. Configure the Jenkins job

Create a **Pipeline** job named `HelloDevOps-Pipeline` with the following settings:

| Setting | Value |
| --- | --- |
| Definition | Pipeline script from SCM |
| SCM | Git |
| Repository URL | `https://github.com/YashLakkas24/helloDevops.git` |
| Branch Specifier | `*/main` |
| Script Path | `Jenkinsfile` |

Save the job and select **Build Now**. Open **Console Output** to inspect each stage.

### 4. Test the Java app locally (optional)

With JDK 21 installed, run from the repository root:

```powershell
javac HelloDevOps.java
java HelloDevOps
```

Expected output:

```text
Hello DevOps!
Java application running successfully.
```

### 5. Test Docker locally (optional)

With Docker Desktop running:

```powershell
docker build -t hello-devops:local .
docker run --rm hello-devops:local
```

The container should print the same two lines.

## What the pipeline verifies

1. **Checkout from GitHub** — retrieves the configured source branch.
2. **Verify Docker Access** — checks that Jenkins can communicate with the Docker daemon.
3. **Build Java** — compiles `HelloDevOps.java` and archives the generated class.
4. **Run Java and Verify Output** — compares the application's output with the expected result.
5. **Build Docker Image** — creates `hello-devops:<build-number>` and `hello-devops:latest`.
6. **Run Container and Verify Output** — runs the build-specific image and validates its output.

Any failed command or output check fails the pipeline.

## Troubleshooting

- **Docker is unavailable:** Start Docker Desktop, wait for the engine, then rerun `docker compose up -d`.
- **Port 8080 is occupied:** Change the Compose port mapping from `8080:8080` to `8081:8080`, restart the stack, and open `http://localhost:8081`.
- **Jenkins cannot access Docker:** Check `docker compose ps` and `docker compose logs docker`. Verify the Docker service is healthy.
- **Git checkout fails:** Confirm the repository URL and branch. Public repository checkout does not need credentials; private repositories do.
- **Docker build fails:** Inspect the Jenkins Console Output and verify the Docker service can download `eclipse-temurin:21-jdk`.

## Stop the environment

Stop the containers while retaining the Jenkins configuration and Docker image cache:

```powershell
docker compose down
```

To also delete the named data volumes (including Jenkins configuration and cached Docker images), run `docker compose down -v` only when you intentionally want a complete reset.

## Security note

This is a local educational configuration. The Docker-in-Docker service uses privileged mode. Do not expose Jenkins or the Docker daemon to the public internet, and do not use this configuration for untrusted multi-user workloads.
