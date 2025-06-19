pipeline {
    agent any

    environment {
        PROJECT_ID = 'bamboo-diode-456912-p9'
        IMAGE = "asia-south1-docker.pkg.dev/%PROJECT_ID%/flask-repo/flask-app"
        CLUSTER = 'autopilot-cluster-1'
        ZONE = 'asia-south1'
        GCP_KEY = 'C:\\Users\\himan\\Downloads\\devops-lab-ci\\flask-gke-helm\\jenkins-sa-key.json'  //⚠️ Update this path
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Authenticate with GCP') {
            steps {
                bat """
                gcloud auth activate-service-account --key-file=%GCP_KEY%
                gcloud config set project %PROJECT_ID%
                gcloud auth configure-docker asia-south1-docker.pkg.dev --quiet
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -f ./app/Dockerfile -t %IMAGE% ./app"
                bat "docker push %IMAGE%"
            }
        }

        stage('Deploy to GKE') {
            steps {
                bat "gcloud container clusters get-credentials %CLUSTER% --zone %ZONE%"
                bat """
                helm upgrade --install flask-app ./flask-chart ^
                    --set image.repository=%IMAGE% ^
                    --set image.tag=latest
                """
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
