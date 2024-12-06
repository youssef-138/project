pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "youssef138/argocd"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        YAML_FILE = "kubernetes/frontend.yaml" // Path to your Kubernetes YAML file
        GIT_CREDENTIALS = 'github' // Replace with your Jenkins Git credentials ID
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh " cd frontend && docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} . "
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                    echo "${PASSWORD}" | docker login -u "${USERNAME}" --password-stdin
                    docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update YAML with New Image') {
            steps {
                script {
                    sh """
                    sed -i 's|image:.*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|g' ${YAML_FILE}
                    echo "Updated ${YAML_FILE} with new image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    """
                }
            }
        }

        stage('Commit and Push Changes to Git') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS}", usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh """
                    git config --global user.email "youssef138adel@gmail.com "
                    git config --global user.name "youssef-138"
                    git add ${YAML_FILE}
                    git commit -m "Update image to ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    git push https://${GIT_USERNAME}:${GIT_PASSWORD}@https://github.com/youssef-138/argocd.git 
                    """
                }
            }
        }
    }
}
