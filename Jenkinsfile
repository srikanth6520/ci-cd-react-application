pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'docker-hub-credentials'   
        SONARQUBE_CREDENTIALS_ID = 'sonarqube-token'   
        GITHUB_CREDENTIALS_ID = 'github-credentials'         
    }

    parameters {
        string(name: 'SONARQUBE_URL', defaultValue: 'http://localhost:9000', description: 'SonarQube Server URL')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
            post {
                success { echo 'Checkout successful.' }
                failure { echo 'Checkout failed.' }
            }
        }

        stage('Install Dependencies and Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
            post {
                success { echo 'Build successful.' }
                failure { echo 'Build failed.' }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
            post {
                success { echo 'Artifacts archived successfully.' }
                failure { echo 'Artifact archiving failed.' }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: "${SONARQUBE_CREDENTIALS_ID}", variable: 'SONAR_TOKEN')]) {
                    script {
                        echo "SonarQube URL: ${params.SONARQUBE_URL}"
                        sh """
                            /opt/sonar-scanner/sonar-scanner-6.2.1.4610-linux-x64/bin/sonar-scanner \
                                -Dsonar.projectKey=react-app \
                                -Dsonar.sources=src \
                                -Dsonar.host.url=${params.SONARQUBE_URL} \
                                -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
            post {
                success { echo 'SonarQube analysis successful.' }
                failure { echo 'SonarQube analysis failed.' }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t srikanth6520/react-app:latest .'
            }
            post {
                success { echo 'Docker image built successfully.' }
                failure { echo 'Docker image build failed.' }
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
            post {
                success { echo 'Docker image pushed to DockerHub successfully.' }
                failure { echo 'Docker image push to DockerHub failed.' }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'KUBECONFIG', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f kubernetes/Deployment.yaml -n testing'
                    sh 'kubectl apply -f kubernetes/Service.yaml -n testing'
                }
            }
            post {
                success { echo 'Deployment to Kubernetes successful.' }
                failure { echo 'Deployment to Kubernetes failed.' }
            }
        }
    }

    post {
        always {
            echo 'Build finished.'
        }
    }
}
