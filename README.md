# HelloDevOps — CI/CD Automation Mini-Project

A Java 21 mini-project demonstrating a basic CI/CD workflow with Git, GitHub, Jenkins, and Docker.

## Repository layout

The application and local Jenkins setup are in [`hellodevops-mini-project/`](hellodevops-mini-project/). The detailed installation guide is in [`hellodevops-mini-project/README.md`](hellodevops-mini-project/README.md).

## Quick start (Windows + Docker Desktop)

1. Clone this repository and open it in VS Code.
2. Start Docker Desktop and wait for the engine to be ready.
3. In a VS Code terminal, change to the project directory:

   ```powershell
   cd hellodevops-mini-project
   ```

4. Build and start Jenkins and its Docker-in-Docker service:

   ```powershell
   docker compose up -d --build
   docker compose ps
   ```

5. Open [http://localhost:8080](http://localhost:8080). Retrieve the initial setup password with:

   ```powershell
   docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```

6. Finish Jenkins setup, then create a **Pipeline** job named `HelloDevOps-Pipeline`.
7. Configure **Pipeline script from SCM** → **Git**, set the repository URL to `https://github.com/YashLakkas24/helloDevops.git`, branch to `*/main`, and **Script Path** to `hellodevops-mini-project/Jenkinsfile`.
8. Save the job and click **Build Now**. Check **Console Output** for each stage.

The pipeline checks out the repository at its root, compiles and runs the Java class from the nested project directory, builds the Docker image, runs the container, and verifies that both executions produce the expected output.

## Expected output

```text
Hello DevOps!
Java application running successfully.
```

## Local project commands

After changing to `hellodevops-mini-project/`, you can test the program directly:

```powershell
javac HelloDevOps.java
java HelloDevOps
```

Or test the Docker image independently:

```powershell
docker build -t hello-devops:local .
docker run --rm hello-devops:local
```

## Troubleshooting

- **Docker command fails:** confirm Docker Desktop is running and configured for Linux containers.
- **Port 8080 is occupied:** change the port mapping in `compose.yaml` from `8080:8080` to `8081:8080`, restart the stack, then visit `http://localhost:8081`.
- **Jenkins checkout fails:** verify the repository URL and `main` branch; configure credentials in Jenkins if the repository is private.
- **Docker stage fails:** check `docker compose ps` and `docker compose logs docker`; the Docker service should be healthy.

## Security note

This configuration is intended for a local educational demonstration. Docker-in-Docker runs in privileged mode. Do not expose Jenkins or the Docker daemon to the public internet, and do not use this configuration for untrusted multi-user workloads.
