pipeline {
    agent any

    environment {
        PROJECT_ID = 'bamboo-diode-456912-p9'
        CLUSTER = 'autopilot-cluster-1'
        ZONE = 'asia-south1'
        GCP_KEY = 'C:\\Users\\himan\\Downloads\\devops-lab-ci\\flask-gke-helm\\jenkins-sa-key.json'  // ⚠️ Ensure this path exists
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
                gcloud auth activate-service-account --key-file="${env.GCP_KEY}"
                gcloud config set project ${env.PROJECT_ID}
                gcloud auth configure-docker asia-south1-docker.pkg.dev --quiet
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                docker build -f ./app/Dockerfile -t asia-south1-docker.pkg.dev/${env.PROJECT_ID}/flask-repo/flask-app ./app
                docker push asia-south1-docker.pkg.dev/${env.PROJECT_ID}/flask-repo/flask-app
                """
            }
        }

        stage('Deploy to GKE') {
            steps {
                bat """
                gcloud container clusters get-credentials ${env.CLUSTER} --zone ${env.ZONE} --project ${env.PROJECT_ID}
                // helm version
                // helm upgrade --install flask-app ./flask-chart -n dev --set image.repository=asia-south1-docker.pkg.dev/${env.PROJECT_ID}/flask-repo/flask-app --set image.tag=latest 
                // kubectl get pods -n dev
                """
            }
        }

        stage('Deploy to GKE-1') {
            steps {
                configFileProvider([configFile(fileId: 'deploy_to_gke', targetLocation: 'deploy_to_gke.py')]) {
                        script {
                            def result = bat(
                                script: "python deploy_to_gke.py",
                                returnStdout: true
                            ).trim()
                            echo "Deployment Output:\n${result}"
                        }
    
                }
            }
        }




        
    }

    // post {
    //     success {
    //         echo '✅ Deployment to GKE succeeded!'
    //     }
    //     failure {
    //         echo '❌ Deployment failed. Please check the logs.'
    //     }
    // }
}
