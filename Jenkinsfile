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
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    checkout scm
                }
            }
        }

        stage('Install Dependencies and Build') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    archiveArtifacts artifacts: 'build/**', fingerprint: true
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    withCredentials([string(credentialsId: "${SONARQUBE_CREDENTIALS_ID}", variable: 'SONAR_TOKEN')]) {
                        script {
                            echo "SonarQube URL: ${params.SONARQUBE_URL}"
                            sh """
                                /opt/sonar-scanner/sonar-scanner-6.2.1.4610-linux-x64/bin/sonar-scanner \
                                    -Dsonar.projectKey=react-app \
                                    -Dsonar.sources=src \
                                    -Dsonar.host.url=${params.SONARQUBE_URL} \
                                    -Dsonar.login=$SONAR
                            """
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh 'docker build -t srikanth6520/react-app:latest .'
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh """
                            echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                            docker push srikanth6520/react-app:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    withCredentials([file(credentialsId: 'KUBECONFIG', variable: 'KUBECONFIG')]) {
                        sh 'kubectl apply -f kubernetes/Deployment.yaml -n testing'
                        sh 'kubectl apply -f kubernetes/Service.yaml -n testing'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Build finished.'
        }
    }
}
