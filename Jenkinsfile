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
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sjhajj/nano-kube.git' // Replace with your repository URL
            }
        }

        stage('Create .tar.gz Archive') {
            steps {
                script {
                    // Create a temporary directory to store the tarball
                    sh 'mkdir -p /tmp/tarball'
                    // Archive all files from the root directory, excluding the .git directory and move it to the temporary directory
                    sh 'tar --exclude=./.git -czvf /tmp/tarball/nano.tar.gz .'
                    // Move the tarball back to the workspace
                    sh 'mv /tmp/tarball/nano.tar.gz .'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
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
                    sh 'helm package mychart --destination .'
                }
            }
        }
        stage('Deploy Helm Chart') {
            steps {
                script {
                    sh 'helm upgrade --install myapp ./mychart-0.1.0.tgz --set image.tag=latest --namespace mynamespace'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Docker image has been successfully pushed to AWS ECR.'
        }
        failure {
            echo 'Failed to push the Docker image to AWS ECR.'
        }
    }
}