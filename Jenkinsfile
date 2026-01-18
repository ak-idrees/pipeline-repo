pipeline {
    agent any

    parameters {
        string(
            name: 'sourceBranch',
            defaultValue: 'main',
            description: 'Branch of the source code'
        )
        choice(
            name: 'depBranch',
            choices: ['sigma', 'roko'],
            description: 'Client name to build the Docker image for'
        )
    }

    environment {
        AWS_REGION = 'eu-north-1'
        ECR_REGISTRY = '552162429839.dkr.ecr.eu-north-1.amazonaws.com'
        ECR_REPO = 'nodejs-app'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                echo "Checking out source code from branch: ${params.sourceBranch}"
                git branch: params.sourceBranch,
                    url: 'https://github.com/ak-idrees/my-pipeline.git'
            }
        }

        stage('Select Client') {
            steps {
                script {
                    env.CLIENT = params.depBranch.toLowerCase()
                    echo "Building for client: ${env.CLIENT}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${ECR_REPO}:${env.CLIENT}"
                    sh "docker build -t ${ECR_REPO}:${env.CLIENT} ."
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Tagging and pushing Docker image to ECR"
                    sh """
                        docker tag ${ECR_REPO}:${env.CLIENT} ${ECR_REGISTRY}/${ECR_REPO}:${env.CLIENT}
                        docker push ${ECR_REGISTRY}/${ECR_REPO}:${env.CLIENT}
                    """
                    echo "Image pushed successfully: ${ECR_REPO}:${env.CLIENT}"
                }
            }
        }

    } // stages
}
