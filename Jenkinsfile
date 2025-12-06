pipeline {
    agent { label 'jenkins-agent' }

    environment {
        REGISTRY = "localhost:5000"
        IMAGE_NAME = "myapp"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build image (Docker or Podman)') {
            steps {
                sh """
                    if command -v docker >/dev/null 2>&1; then
                        echo "Using Docker engine"
                        docker build -t ${IMAGE_NAME}:${TAG} build_ubuntu
                    else
                        echo "Using Podman engine"
                        podman build -t ${IMAGE_NAME}:${TAG} build_ubuntu
                    fi
                """
            }
        }

        stage('Tag image for registry') {
            steps {
                sh """
                    if command -v docker >/dev/null 2>&1; then
                        docker tag ${IMAGE_NAME}:${TAG} ${REGISTRY}/${IMAGE_NAME}:${TAG}
                    else
                        podman tag ${IMAGE_NAME}:${TAG} ${REGISTRY}/${IMAGE_NAME}:${TAG}
                    fi
                """
            }
        }

        stage('Push image to registry') {
            steps {
                sh """
                    if command -v docker >/dev/null 2>&1; then
                        docker push ${REGISTRY}/${IMAGE_NAME}:${TAG}
                    else
                        podman push ${REGISTRY}/${IMAGE_NAME}:${TAG}
                    fi
                """
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh """
                    ansible-playbook -i inventory/hosts.ini deploy_app.yml \
                    --vault-password-file ~/.vault \
                    --extra-vars "build_number=${TAG}"
                """
            }
        }
    }
}

