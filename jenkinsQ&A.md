# Jenkins

## How artifacts get stored in Nexus.
Steps for Configuring Nexus with Maven in Jenkins DSL Pipeline
1. Set Up Nexus Repository:
   - Nexus repository needs to be configured to store your Maven artifacts (like .jar, .war, .pom).
     - Log into Nexus.
     - Create a Maven hosted repository (e.g., `maven-releases`, `maven-snapshots`).
     - For each repository, get the repository URL (you will need this for Jenkins to deploy artifacts).

2. Configure Maven in Jenkins:
   - Go to Jenkins > Manage Jenkins > Global Tool Configuration.
   - Set up Maven:
     - Name: maven3 (or any name you prefer).
     - Maven Version: Choose the installed Maven version (e.g., 3.6.3).
     - Install automatically (optional).
   - Add JDK Configuration:
     - Name: jdk17 (or your desired version).
     - Set up the JDK path or let Jenkins auto-install it.
            
3. Configure Maven Settings in Jenkins:
   - In Jenkins, go to Manage Jenkins > Configure System.
   - Add your Nexus credentials (Username and Password or token) under Maven Settings:
   - Maven Settings Config: This points to the settings.xml or pom.xml file, which contains Nexus credentials.
   - Use global-settings or create a custom one.
   - The settings.xml should contain the Nexus server URL and credentials.
```xml
<servers>
    <server>
        <id>nexus-releases</id>
        <username>${env.NEXUS_USERNAME}</username>
        <password>${env.NEXUS_PASSWORD}</password>
    </server>
</servers>

<profiles>
    <profile>
        <id>nexus</id>
        <repositories>
            <repository>
                <id>nexus-releases</id>
                <url>https://nexus.yourcompany.com/repository/maven-releases/</url>
            </repository>
        </repositories>
    </profile>
</profiles>
```
4. Create a Jenkins DSL Pipeline Script:
- Now, you will configure the Jenkins DSL pipeline that defines the build, test, and deploy stages for your Maven project.
```groovy
pipeline {
    agent any

    environment {
        NEXUS_URL = 'https://nexus.company.com/repository/maven-releases/'
        NEXUS_USERNAME = credentials('nexus-username')  // Use Jenkins Credentials Plugin
        NEXUS_PASSWORD = credentials('nexus-password')
    }

    tools {
        maven 'maven3'  // Use Maven tool installed in Jenkins
        jdk 'jdk17'     // Use JDK 17
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout source code from repository
                git 'https://github.com/yourorg/yourrepo.git'
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    // Run Maven build and tests
                    sh "mvn clean test"
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    // Configure Maven settings with Nexus repo details
                    withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                        // Deploy artifacts to Nexus repository
                        sh "mvn deploy -DskipTests"
                    }
                }
            }
        }

        stage('Clean up') {
            steps {
                // Clean up the workspace after deployment
                cleanWs()
            }
        }
    }

    post {
        success {
            echo "Build and deploy completed successfully!"
        }
        failure {
            echo "Build or deploy failed!"
        }
    }
}
```     
     
