pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'docker-hub-credentials'   
        SONARQUBE_CREDENTIALS_ID = 'sonarqube-token'   
        SONARQUBE_URL = 'http://your-sonarqube-server-url' 
        GITHUB_CREDENTIALS_ID = 'github-credentilas'         
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Jenkins automatically checks out code from SCM
                checkout scm
            }
        }

        stage('Install Dependencies and Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: "${SONARQUBE_CREDENTIALS_ID}", variable: 'SONAR_TOKEN')]) {
                    sh """
                        sonar-scanner \\
                            -Dsonar.projectKey=react-app \\
                            -Dsonar.sources=src \\
                            -Dsonar.host.url=${SONARQUBE_URL} \\
                            -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t srikanth6520/react-app:latest .'
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh """
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push srikanth6520/react-app:latest
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean up workspace after pipeline run
        }
    }
}
