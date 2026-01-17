pipeline {
    agent any

    parameters {
        string(
            name: 'sourceBranch',
            defaultValue: 'main',
            description: 'Branch of the source code'
        )
        string(
            name: 'depBranch',
            defaultValue: 'sigma',
            description: 'Client name (sigma or roko)'
        )
    }

    environment {
        AWS_REGION = 'eu-north-1'
        ECR_REGISTRY = '552162429839.dkr.ecr.eu-north-1.amazonaws.com/nodejs-app'
        ECR_REPO = 'nodejs-app'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                echo "Checking out source code from branch: ${params.sourceBranch}"
                git branch: params.sourceBranch,
                    url: 'https://github.com/ak-idrees/project-nodejs-.git'
            }
        }

        stage('Client Selection') {
            steps {
                script {
                    // Simple IF / ELSE logic
                    if (params.depBranch.toLowerCase() == 'sigma') {
                        env.CLIENT = 'sigma'
                        echo 'Building for SIGMA client'
                    } else {
                        env.CLIENT = 'roko'
                        echo 'Building for ROKO client'
                    }
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
                    aws ecr get-login-password --region ${AWS_REGION} |
                    docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh """
                    docker tag ${ECR_REPO}:${env.CLIENT} ${ECR_REGISTRY}/${ECR_REPO}:${env.CLIENT}
                    docker push ${ECR_REGISTRY}/${ECR_REPO}:${env.CLIENT}
                    """
                    echo "Image pushed: ${ECR_REPO}:${env.CLIENT}"
                }
            }
        }
    }
}
