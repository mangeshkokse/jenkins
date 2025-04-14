pipeline {
 agent any

 environment {
 SCANNER_HOME = tool 'sonar-scanner'
 }

 tools {
 jdk 'jdk17'
 maven 'maven3'
 }
 stages {
 stage('Git Checkout') {
 steps {
 git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/ShubhamStunner/BoardGame.git'
 }
 }

 stage('Compile') {
 steps {
 sh "mvn compile"
 }
 }

 stage('Test') {
 steps {
 sh "mvn test"
 }
 }

 stage('Trivy File system scan') {
 steps {
 sh "trivy fs --format table -o trivy-fs-report.html ."
 }
 }

 stage('SonarQube Analysis') {
 steps {
 withSonarQubeEnv('sonar') {
 sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -
Dsonar.projectKey=BoardGame \
 -Dsonar.java.binaries=. '''
 }
 }
 }
19

 stage('Quality Ga
