pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'docker-hub-credentials'   
        SONARQUBE_CREDENTIALS_ID = 'sonarqube-token'   
        GITHUB_CREDENTIALS_ID = 'github-credentials'         
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
            post {
                success {
                    echo 'Checkout successful.'
                }
                failure {
                    echo 'Checkout failed.'
                }
            }
        }

        stage('Install Dependencies and Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
            post {
                success {
                    echo 'Build successful.'
                }
                failure {
                    echo 'Build failed.'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
            post {
                success {
                    echo 'Artifacts archived successfully.'
                }
                failure {
                    echo 'Artifact archiving failed.'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: "${SONARQUBE_CREDENTIALS_ID}", variable: 'SONAR_TOKEN')]) {
                    script {
                        // Echo the SonarQube URL for debugging
                        echo "SonarQube URL: ${params.SONARQUBE_URL}"

                        // Run the sonar-scanner command
                        sh """
                            /opt/sonar-scanner/sonar-scanner-6.2.1.4610-linux-x64/bin/sonar-scanner \
                                -Dsonar.projectKey=react-app \\
                                -Dsonar.sources=src \\
                                -Dsonar.host.url=http://52.3.63.167:9000 \\
                                -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
            post {
                success {
                    echo 'SonarQube analysis successful.'
                }
                failure {
                    echo 'SonarQube analysis failed.'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t srikanth6520/react-app:latest .'
            }
            post {
                success {
                    echo 'Docker image built successfully.'
                }
                failure {
                    echo 'Docker image build failed.'
                }
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
                success {
                    echo 'Docker image pushed to DockerHub successfully.'
                }
                failure {
                    echo 'Docker image push to DockerHub failed.'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
            echo 'Workspace cleaned up.'
        }
    }
}
