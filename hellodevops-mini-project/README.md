# HelloDevOps — Java CI/CD with GitHub, Jenkins, and Docker

This mini project implements the workflow described in the provided project report:

**Developer → Git/GitHub → Jenkins pipeline → compile and run Java → build Docker image → run container and verify output**

The application intentionally stays small so the focus is on CI/CD automation. The pipeline is stored in `Jenkinsfile` in the repository rather than being typed only into the Jenkins UI. It includes explicit output checks and tags the Docker image with both the Jenkins build number and `latest`.

## 1. Project files

```text
hellodevops-mini-project/
├── HelloDevOps.java       # Java application
├── Dockerfile             # Builds the container image
├── Jenkinsfile            # CI pipeline stored with source code
├── compose.yaml           # Local Jenkins + Docker-in-Docker setup
├── .gitignore
├── README.md
└── jenkins/
    └── Dockerfile         # Jenkins image with Git and Docker CLI
```

## 2. Requirements

- Windows with Docker Desktop configured for **Linux containers**
- Git for Windows
- A GitHub account and an empty GitHub repository
- Internet access to pull Jenkins, Docker, and Java container images

A local JDK 21 is optional but useful for testing the Java file directly. The application container and Jenkins image provide their own JDKs.

## 3. Test the Java program locally (optional)

Open PowerShell in this project directory:

```powershell
javac HelloDevOps.java
java HelloDevOps
```

Expected output:

```text
Hello DevOps!
Java application running successfully.
```

The `javac` command creates `HelloDevOps.class`; that generated file is excluded from Git by `.gitignore`.

## 4. Test the Docker image locally (optional)

Make sure Docker Desktop is running, then run:

```powershell
docker build -t hello-devops:local .
docker run --rm hello-devops:local
```

The container should print the same two lines shown above.

## 5. Push the project to GitHub

1. Create a **new empty repository** on GitHub, for example `HelloDevOps-CICD`. Do not initialize it with a separate README, license, or `.gitignore`, because this project already includes them.
2. From PowerShell, run these commands in the project directory, replacing the URL with your repository URL:

```powershell
git init
git add .
git commit -m "Add HelloDevOps CI/CD project"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/HelloDevOps-CICD.git
git push -u origin main
```

If Git asks you to identify yourself, configure your name and email with `git config --global user.name "Your Name"` and `git config --global user.email "you@example.com"`. Use GitHub's supported authentication method when prompted.

## 6. Start Jenkins with Docker Desktop

From the same project directory, run:

```powershell
docker compose up -d --build
```

This starts two local services:

- `jenkins`: Jenkins UI and pipeline execution environment
- `docker`: a separate Docker-in-Docker daemon that Jenkins uses to build and run the application image

The Docker daemon is reachable only on the internal Compose network; port `2376` is not published to Windows.

Check startup status:

```powershell
docker compose ps
docker compose logs jenkins
```

Open `http://localhost:8080` in your browser. Retrieve the initial administrator password with:

```powershell
docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Paste the password into the Jenkins setup screen, install the suggested plugins if asked, and create your administrator account. The supplied Jenkins image preinstalls the Git, Pipeline, and Docker Pipeline plugins required by this project.

## 7. Create the Jenkins pipeline job

1. In Jenkins, select **New Item**.
2. Enter `HelloDevOps-Pipeline`, select **Pipeline**, and click **OK**.
3. In the job configuration, find the **Pipeline** section.
4. Set **Definition** to **Pipeline script from SCM**.
5. Set **SCM** to **Git**.
6. Enter your GitHub repository URL, such as `https://github.com/YOUR-USERNAME/HelloDevOps-CICD.git`.
7. Set **Branch Specifier** to `*/main`.
8. Set **Script Path** to `Jenkinsfile`.
9. Save the job, then click **Build Now**.

For a public repository, credentials are not needed for checkout. For a private repository, add Jenkins credentials and select them in the job configuration.

## 8. What the pipeline does

| Stage | Action | Success condition |
|---|---|---|
| Checkout from GitHub | Checks out the configured repository | Checkout succeeds |
| Build Java | Runs `javac HelloDevOps.java` | Java source compiles; class file is archived |
| Run Java and Verify Output | Runs `java HelloDevOps` | Output matches the two expected lines |
| Build Docker Image | Builds `hello-devops:<build-number>` and `hello-devops:latest` | Docker image builds successfully |
| Run Container and Verify Output | Runs the new image | Container output matches the expected lines |

Open the job and select **Console Output** to inspect every command and stage. A successful run ends with a message stating that Java compiled, output was verified, and the Docker image was built and tested.

## 9. Common problems

### Docker Desktop is not running

Start Docker Desktop, wait until its engine is ready, then run `docker compose up -d` again.

### Port 8080 is already in use

Change the Jenkins port mapping in `compose.yaml` from `8080:8080` to `8081:8080`, restart with `docker compose up -d`, and open `http://localhost:8081`.

### Jenkins cannot connect to Docker

Check `docker compose ps` and `docker compose logs docker`. The `docker` service should become healthy. Confirm the Jenkins service has `DOCKER_HOST=tcp://docker:2376`, `DOCKER_CERT_PATH=/certs/client`, and `DOCKER_TLS_VERIFY=1` as configured in `compose.yaml`.

### Git checkout fails

Confirm that the GitHub URL is correct, the branch is `main`, the repository is reachable, and Jenkins has credentials if the repository is private.

### Java stage works but the Docker stage fails

Inspect the Docker build logs. It may need internet access to pull `eclipse-temurin:21-jdk`. Also confirm the Docker-in-Docker service is healthy.

### The Jenkins administrator password command fails

Wait until Jenkins has fully started and try again:

```powershell
docker compose logs jenkins
docker compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

## 10. Stop or reset the environment

Stop services while preserving Jenkins configuration and Docker images:

```powershell
docker compose down
```

Start them again with `docker compose up -d`.

To remove the containers **and all named data volumes** (this deletes Jenkins configuration and the Docker-in-Docker image cache), use:

```powershell
docker compose down -v
```

Use the destructive command only when you intentionally want a clean reset.

## Security note

This is a local educational setup, not a production deployment. The Docker-in-Docker service uses privileged mode, which grants elevated container capabilities. Do not expose this Jenkins instance or the Docker daemon to the public internet, and do not use this configuration for an untrusted multi-user environment.
