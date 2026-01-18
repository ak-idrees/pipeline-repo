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
            // Set client based on parameter
            env.CLIENT = params.depBranch.toLowerCase()
            echo "Building for client: ${env.CLIENT}"

            // Set image tag using client name + Jenkins build number
            env.IMAGE_TAG = "${env.CLIENT}-v${BUILD_NUMBER}"
            echo "Docker image tag will be: ${env.IMAGE_TAG}"
        }
    }
}

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${ECR_REPO}:${env.IMAGE_TAG}"
                    sh "docker build -t ${ECR_REPO}:${env.IMAGE_TAG} ."
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
            docker tag ${ECR_REPO}:${env.IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPO}:${env.IMAGE_TAG}
            docker push ${ECR_REGISTRY}/${ECR_REPO}:${env.IMAGE_TAG}
            """
            echo "Image pushed: ${ECR_REPO}:${env.IMAGE_TAG}"
        }
    }
}

 stage('Deploy Container') {
    steps {
        script {
            echo "Pulling Docker image: ${ECR_REGISTRY}/${ECR_REPO}:${env.IMAGE_TAG}"
            sh "docker pull ${ECR_REGISTRY}/${ECR_REPO}:${env.IMAGE_TAG}"

            echo "Stopping any existing container for client: ${env.CLIENT}"
            sh "docker rm -f ${env.CLIENT}-container || true"

            echo "Running container for client: ${env.CLIENT} on port 3000"
            sh """
            docker run -d --name ${env.CLIENT}-container -p 3000:3000 ${ECR_REGISTRY}/${ECR_REPO}:${env.IMAGE_TAG}
            """

            echo "Container for ${env.CLIENT} is now running. Access via EC2 public IP:3000"
        }
    }
}


