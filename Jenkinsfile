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
                    set CONFIG_PATH=C:\\jenkins_docker_config
                    if not exist %CONFIG_PATH% mkdir %CONFIG_PATH%
            
                    echo {^"credHelpers^":{^"asia-south1-docker.pkg.dev^":^"gcloud^"}} > %CONFIG_PATH%\\config.json
            
                    type %CONFIG_PATH%\\config.json
                    """
                }
            }

        // stage('Authenticate gcloud') {
        //     steps {
        //         bat 'gcloud auth activate-service-account your-service-account@bamboo-diode-456912-p9.iam.gserviceaccount.com --key-file=C:\\path\\to\\key.json'
        //         bat 'gcloud auth configure-docker asia-south1-docker.pkg.dev'
        //     }
        // }

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
