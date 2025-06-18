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
                    bat "gcloud auth configure-docker asia-south1-docker.pkg.dev -q"
                    bat "docker build -t $IMAGE ./app"
                    bat "docker push $IMAGE"
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                script {
                    bat "gcloud container clusters get-credentials $CLUSTER --zone $ZONE"
                    bat """
                    helm upgrade --install flask-app ./flask-chart \
                        --set image.repository=$IMAGE \
                        --set image.tag=latest
                    """
                }
            }
        }
    }
}
