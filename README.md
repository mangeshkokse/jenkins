# jenkins
## Stage-by-Stage Summary
```markdown
| **Stage**                         | **Description**                                                                 |
|----------------------------------|---------------------------------------------------------------------------------|
| Git Checkout                     | Clones the project from GitHub (main branch) using credentials                  |
| Compile                          | Compiles the Java code using Maven                                              |
| Test                             | Runs the project's unit tests                                                   |
| Trivy File System Scan           | Scans the project directory for vulnerabilities (Trivy FS scan)                |
| SonarQube Analysis               | Performs static code analysis with SonarQube                                    |
| Quality Gate                     | Waits for SonarQube’s quality gate result (pass/fail)                           |
| Build                            | Packages the project into a .jar using Maven                                    |
| Publish Artifacts to Nexus       | Publishes built artifacts to the Nexus repository using Maven                   |
| Build and Tag Docker Image       | Creates a Docker image for the app and tags it                                  |
| Docker Image Scan                | Runs a Trivy security scan on the Docker image                                  |
| Push Docker Image                | Pushes the image to DockerHub                                                   |
| Deploy to Kubernetes             | Deploys the app using `kubectl apply` to a Kubernetes cluster                   |
----------------------------------------------------------------------------------------------------------------------
```
  
- https://www.youtube.com/watch?v=Ww-Hh_WdzWw&t=1104s
- https://www.youtube.com/watch?v=Ww-Hh_WdzWw


- **Q:** Where does Jenkins store the default password?
- **Q:** Who is the user of Jenkins to execute shell?  
  **A:** Jenkins user
- **Q:** What is the path or working directory of Jenkins shell?  
  **A:** `/var/lib/Jenkins/workspace/<job-name>`
- **Q:** What is the home directory of Jenkins?  
  **A:** `/var/lib/Jenkins`
- **Q:** How can you change plugins in Jenkins?  
  **A:** Manage Jenkins → System Configuration
- **Q:** What is Global Tool Configuration?
- **Q:** What is Manage Plugins?
- **Q:** What is Configure Global Security?
- **Q:** What is Manage Credentials?
- **Q:** What is Manage Users?
- **Q:** What is the number of executors in Jenkins?
- **Q:** What is SCM Checkout Retry Count?
- **Q:** What is the use case of Role-Based Authorization Strategy Plugin?
- **Q:** What is "Delete Workspace Before Start"?
- **Q:** Explain Build Trigger and its types.
- **Q:** What is Build Authorization Token?
- **Q:** What is Poll SCM?
- **Q:** What is the difference between Build Periodically and Poll SCM?
- **Q:** What are Environment Variables in Jenkins?
- **Q:** What is a Parameterized Job?
- **Q:** What are Global Variables and how can we create custom global variables?
- **Q:** What is a Build Parameter and its use case?
- **Q:** Types of Build Parameters.
- **Q:** What is Timeout in a Jenkins job?
- **Q:** How to run parallel jobs or concurrent jobs in Jenkins
- **Q:** What is Workspace in Jenkins and its use case?
- **Q:** What is Upstream and Downstream in Jenkins?
- **Q:** How does Artifact Management work in Jenkins?
- **Q:** What is Master-Slave in Jenkins and its use case?
- **Q:** How to achieve a particular job running on a specific slave node in Jenkins?
- **Q:** What is Shared Library in Jenkins?


## Q. How are artifacts managed by jenkins.
- In Jenkins, artifacts are the files generated as part of a build, such as binaries, reports, logs, or compiled code. Here's how artifacts are typically managed in Jenkins:
   1. Archive Artifacts
      - Jenkins allows you to archive artifacts using the "Archive the artifacts" post-build action in job configuration by UI
      - Use the archiveArtifacts step to store build outputs (e.g., JARs, logs, reports) in Jenkins:
      ```groovy
        archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
        ```      
      - `artifacts:` Glob pattern of files to archive.
      - `fingerprint: true:` Enables tracking across builds.

  2. Stashing & Unstashing (for multi-stage pipelines)
      - Use stash to temporarily save files, and unstash to retrieve them in another stage:
```groovy
stage('Build') {
  steps {
    sh 'make build'
    stash includes: 'build/**', name: 'build-artifacts'
  }
}

stage('Test') {
  steps {
    unstash 'build-artifacts'
    sh 'make test'
  }
}
```
 3. Publishing to Artifact Repositories
```groovy
sh '''
  curl -u user:pass -T build/libs/app.jar https://nexus.example.com/repository/maven-releases/
'''
```
 - Or use plugins:
    - Nexus Artifact Uploader Plugin
    - jfrog Artifactory Plugin
    - S3 Publisher Plugin
      
4. Artifacts in Declarative Pipelines configure Nexus in a Jenkins DSL pipeline,
   - Option 1: Using sh or curl to upload to Nexus
```groovy
pipeline {
  agent any

  environment {
    NEXUS_URL = 'http://nexus.example.com/repository/maven-releases/'
    NEXUS_USER = credentials('nexus-username')   // Jenkins credential ID
    NEXUS_PASSWORD = credentials('nexus-password')
  }

  stages {
    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Upload to Nexus') {
      steps {
        sh '''
          curl -u $NEXUS_USER:$NEXUS_PASSWORD \
            --upload-file target/my-app.jar \
            $NEXUS_URL/com/example/my-app/1.0.0/my-app-1.0.0.jar
        '''
      }
    }
  }
}
```
- Option 2: Using the Nexus Artifact Uploader Plugin
   - If you're using classic jobs or scripted pipelines, you can use the plugin. First, install Nexus Artifact Uploader Plugin.
```groovy
node {
  stage('Build') {
    sh 'mvn clean package'
  }

  stage('Upload to Nexus') {
    nexusArtifactUploader(
      nexusVersion: 'nexus3',
      protocol: 'http',
      nexusUrl: 'nexus.example.com',
      groupId: 'com.example',
      version: '1.0.0',
      repository: 'maven-releases',
      credentialsId: 'nexus-creds',
      artifacts: [
        [artifactId: 'my-app', classifier: '', file: 'target/my-app.jar', type: 'jar']
      ]
    )
  }
}
```

 
