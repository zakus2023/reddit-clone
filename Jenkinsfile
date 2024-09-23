pipeline {
    agent any            // Specifies that the pipeline can run on any available Jenkins agent.
    
    tools {
        jdk 'jdk17'      // Declares that JDK 17 should be available for use in the pipeline.
        nodejs 'node16'  // Declares that Node.js version 16 should be available for the pipeline.
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'  // Sets the SonarQube scanner tool path as an environment variable.
        APP_NAME = "reddit-clone-pipeline"   // Defines the application name as "reddit-clone-pipeline".
        RELEASE = "1.0.0"                    // Sets the release version to "1.0.0".
        DOCKER_USER = "zakus2024"            // Stores the Docker username.
        DOCKER_PASS = 'docker'               // Stores the Docker password (typically should be securely stored).
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"  // Defines the Docker image name based on the Docker user and app name.
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"  // Creates a tag for the Docker image, combining release version and build number.
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()    // Cleans up the workspace before starting the pipeline to ensure no leftover files from previous builds.
            }
        }
        
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/zakus2023/reddit-clone'
                // Clones the 'main' branch of the Git repository where the Reddit clone project is stored.
            }
        }
        
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Reddit-Clone-CI \
                    -Dsonar.projectKey=Reddit-Clone-CI'''
                    // Runs the SonarQube scanner for static code analysis using the project name and key "Reddit-Clone-CI".
                }
            }
        }
        
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                    // Waits for SonarQube quality gate results to decide if the pipeline should proceed, using a stored token for authentication.
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
                // Installs the project's Node.js dependencies using npm.
            }
        }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
                // Runs a file system security scan using Trivy and outputs the results to a file called "trivyfs.txt".
            }
        }
         stage("Build & Push Docker Image") {
             steps {
                 script {
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image = docker.build "${IMAGE_NAME}"
                     }
                     docker.withRegistry('',DOCKER_PASS) {
                         docker_image.push("${IMAGE_TAG}")
                         docker_image.push('latest')
                     }
                 }
             }
         }
        
    }
}
