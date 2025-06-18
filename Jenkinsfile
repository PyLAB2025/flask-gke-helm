pipeline {
    agent any

    environment {
        PROJECT_ID = 'bamboo-diode-456912-p9'
        IMAGE = "asia-south1-docker.pkg.dev/$PROJECT_ID/flask-repo/flask-app"
        CLUSTER = 'autopilot-cluster-1'
        ZONE = 'asia-south1'
        DOCKER_CONFIG = 'C:\\jenkins_docker_config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Configure Docker Auth') {
            steps {
                bat """
                mkdir C:\\jenkins_docker_config 2>nul
                echo {^"credHelpers^": {^"asia-south1-docker.pkg.dev^": ^"gcloud^"}} > C:\\jenkins_docker_config\\config.json
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t %IMAGE% ./app"
                    bat "docker push %IMAGE%"
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    bat "gcloud container clusters get-credentials %CLUSTER% --zone %ZONE%"
                    bat """
                    helm upgrade --install flask-app ./flask-chart \
                        --set image.repository=%IMAGE% \
                        --set image.tag=latest
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment to GKE succeeded!'
        }
        failure {
            echo '❌ Deployment failed. Please check the logs.'
        }
    }
}
