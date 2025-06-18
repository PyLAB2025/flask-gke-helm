pipeline {
    agent any

    environment {
        PROJECT_ID = 'bamboo-diode-456912-p9'
        IMAGE = "asia-south1-docker.pkg.dev/$PROJECT_ID/flask-repo/flask-app"
        CLUSTER = 'autopilot-cluster-1'
        ZONE = 'asia-south1'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "gcloud auth configure-docker asia-south1-docker.pkg.dev -q"
                    sh "docker build -t $IMAGE ./app"
                    sh "docker push $IMAGE"
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    sh "gcloud container clusters get-credentials $CLUSTER --zone $ZONE"
                    sh """
                    helm upgrade --install flask-app ./flask-chart \
                        --set image.repository=$IMAGE \
                        --set image.tag=latest
                    """
                }
            }
        }
    }
}
