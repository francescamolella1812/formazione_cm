pipeline {
    agent any

    environment {
        REGISTRY = "localhost:5000"
        IMAGE_NAME = "myapp"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${TAG} build_ubuntu
                """
            }
        }

        stage('Tag image for registry') {
            steps {
                sh """
                    docker tag ${IMAGE_NAME}:${TAG} ${REGISTRY}/${IMAGE_NAME}:${TAG}
                """
            }
        }

        stage('Push image to registry') {
            steps {
                sh """
                    docker push ${REGISTRY}/${IMAGE_NAME}:${TAG}
                """
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh """
                    BUILD_NUMBER=${TAG} ansible-playbook -i inventory/hosts.ini deploy_app.yml --vault-password-file ~/.vault
                """
            }
        }
    }
}

