pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-creds-id'   
        SONARQUBE_CREDENTIALS_ID = 'sonarqube-creds-id'   
        SONARQUBE_URL = 'http://your-sonarqube-server-url' 
        GITHUB_CREDENTIALS_ID = 'github-creds-id'         
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${GITHUB_CREDENTIALS_ID}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh 'git clone https://$GIT_USER:$GIT_PASS@github.com/srikanth6520/ci-cd-react-application.git'
                }
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
                        sonar-scanner \
                            -Dsonar.projectKey=react-app \
                            -Dsonar.sources=src \
                            -Dsonar.host.url=$SONARQUBE_URL \
                            -Dsonar.login=$SONAR_TOKEN
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
}
