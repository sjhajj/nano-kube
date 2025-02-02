pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '168009156019'
        AWS_CREDS = 'ecr:us-east-1:awsjenkins'
        IMAGE_NAME = 'nanoapp-kube'
        ECR_REPO_NAME = 'nano-kube-repo'
        ECR_URI = "168009156019.dkr.ecr.us-east-1.amazonaws.com/nano-kube-repo"
        APP_REGISTRY="168009156019.dkr.ecr.us-east-1.amazonaws.com"
        CHART_NAME = 'nanoapp'
        CHART_VERSION = '0.1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sjhajj/nano-kube.git' // Replace with your repository URL
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} ."
                    sh "docker tag ${IMAGE_NAME}:latest ${ECR_URI}:latest"
                }
            }
        }

        stage('Upload App Image') {
            steps {
                script {
                    docker.withRegistry("${APP_REGISTRY}", "${AWS_CREDS}") {
                        def dockerImage = docker.image("${ECR_URI}:latest")
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Package Helm Chart') {
            steps {
                script {
                    // This command assumes the helm chart is in a directory called 'helm/nanoapp' within your repo
                    sh "helm package helm/${CHART_NAME} --version ${CHART_VERSION} --destination ."
                }
            }
        }

        stage('Deploy Helm Chart') {
            steps {
                script {
                    // Ensures the deployment uses the packaged chart from the previous stage
                    sh "helm upgrade --install ${CHART_NAME} ./${CHART_NAME}-${CHART_VERSION}.tgz --set image.tag=latest --namespace nano"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Docker image has been successfully pushed to AWS ECR and application deployed via Helm.'
        }
        failure {
            echo 'Failed to push the Docker image to AWS ECR or deploy the application via Helm.'
        }
    }
}
