DevOps Engineering Challenge: End-to-End Delivery Pipeline
==========================================================
Welcome to the DevOps technical challenge! In this exercise, you will demonstrate your ability to package an application, write configuration as code, and build a fully automated CI/CD pipeline.
We are looking for clean code, best practices, and a solid understanding of containers and Kubernetes.
The Scenario
------------
You are tasked with deploying a Spring Boot application that connects to a MySQL database. You need to containerize this application, create a Helm chart for it, and write a GitHub Actions workflow that builds, deploys, and tests the application inside an ephemeral Kubernetes cluster using vind (vCluster in Docker).
Your Tasks
----------
### Phase 1: Repository & Code Setup
1.  Create a new public GitHub repository.
2.  Initialize it with two branches: main and dev.
3.  Generate the Spring Boot application using [Spring Initializr](https://start.spring.io/) with the exact following settings:
    *   **Project:** Maven
    *   **Language:** Java
    *   **Spring Boot:** (Leave as the default stable version)
    *   **Project Metadata:**
        *   Group: com.devops
        *   Artifact: app
        *   Packaging: Jar
        *   Java: 17 (or 21)
    *   **Dependencies:** Click "ADD DEPENDENCIES" and select:
        *   Spring Web
        *   Spring Data JPA
        *   MySQL Driver
4.  Click **GENERATE**, extract the downloaded .zip file, and commit the extracted contents to your repository.
5.  **Add an Endpoint:** Create a simple REST Controller class so the application has an endpoint you can test later.
    *   Create a new file at exactly this path: src/main/java/com/devops/app/HelloController.java
    ```

      Javapackage com.devops.app;

      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      @RestControllerpublic
      class HelloController { 
        @GetMapping("/")
          public String index() { 
                return "200 OK - Application is running!"; 
                }
        }
    ```
6. Configure the Database: Update the src/main/resources/application.properties file to connect to a MySQL database using environment variables. These will be injected via Kubernetes later:
```
spring.datasource.url=jdbc:mysql://${DB\_HOST}:${DB\_PORT}/${DB\_NAME}
spring.datasource.username=${DB\_USER}
spring.datasource.password=${DB\_PASSWORD}
spring.jpa.hibernate.ddl-auto=update
```


### Phase 2: Dockerization
Write a Dockerfile for the Spring Boot application.
*   **Complexity Requirement:** Use a **multi-stage build**. The first stage should compile the Java application using Maven, and the second stage should package the compiled .jar into a lightweight JRE base image.
### Phase 3: Helm Chart Creation
Create a Helm chart to deploy the application.
*   The chart must include a Deployment, a Service, and a Secret/ConfigMap for the database credentials.
*   **Complexity Requirement:** The application requires a MySQL database. Use Helm dependencies (via Chart.yaml) to pull down a standard MySQL chart (e.g., from Bitnami) as a subchart, ensuring the database spins up alongside the application.
### Phase 4: GitHub Actions CI/CD Pipeline
Create a single GitHub Actions workflow (.github/workflows/pipeline.yml) that executes on every push to the dev and main branches.
The pipeline must perform the following steps in order:
1.  **Build & Push:** Build the Docker image and push it to the GitHub Container Registry (GHCR).
    *   **Complexity:** Tag the image dynamically based on the branch (e.g., dev- for the dev branch and main- for the main branch).
2.  **Cluster Provisioning:** Install and create a local Kubernetes cluster using [vind](https://github.com/loft-sh/vind) directly within the GitHub Actions runner.
3.  **Deployment:** Use Helm to deploy your chart into the vind cluster.
    *   **Complexity:** Pass branch-specific values to the Helm deployment (e.g., overriding the image tag to match the branch that triggered the build).
4.  **Validation/Testing:** Test the application externally from the pipeline runner (not from a pod inside the cluster).
    *   **Networking:** Because vind seamlessly exposes cluster services directly to the Docker host, **do not** use kubectl port-forward. Configure your Helm chart to expose the service appropriately (e.g., NodePort or LoadBalancer) so it is immediately reachable from the GitHub Actions runner.
    *   **Scripting:** Write a bash script using curl or a short Python script to hit the application's root endpoint (/) and assert a successful response (checking for the "200 OK" text).
    *   **Complexity:** Applications take time to start, especially when waiting for a database to initialize. You must implement a **wait/retry polling mechanism** in your test script to ensure the deployment is fully ready before the test fails.
 