pipeline {

    agent any

    environment {

        DOCKER_USER = 'mayuri67'

        IMAGE_NAME = 'python-app'

        IMAGE_TAG = "latest"

        DOCKER_CREDENTIALS = 'dockerhub-credentials'

        GITOPS_CREDENTIALS = 'github-credentials'

        GITOPS_REPO = 'https://github.com/mayuri67/python-argocd-gitops.git'

    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/mayuri67/python-argocd-cicd1.git'

            }
        }

        stage('Install Dependencies') {

            steps {

                sh '''
                    python3 --version

                    rm -rf venv

                    python3 -m venv venv

                    venv/bin/python -m pip install --upgrade pip

                    venv/bin/python -m pip install -r requirements.txt
                '''

            }
        }

        stage('Python Testing') {

            steps {

                sh '''
                   export PYTHONPATH=$WORKSPACE
                   venv/bin/python -m pytest -v
                '''

            }
        }

        stage('Docker Build') {

            steps {

                sh '''
                    docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_USER}/${IMAGE_NAME}:latest
                '''

            }
        }

        stage('Docker Push') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        docker push ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker push ${DOCKER_USER}/${IMAGE_NAME}:latest

                        docker logout
                    '''
                }

            }
        }

        stage('Update GitOps Repository') {

            steps {

                dir('gitops') {

                    git branch: 'main',
                        credentialsId: "github-credentials",
                        url: 'https://github.com/mayuri67/python-argocd-gitops.git'

                    sh '''
                        sed -i "s/^  tag:.*/  tag: \\"${IMAGE_TAG}\\"/" helm/python-app/values.yaml

                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"

                        git add helm/python-app/values.yaml

                        git commit -m "Update Python image to ${IMAGE_TAG}"

                        git push origin main
                    '''

                }

            }
        }

    }

    post {

        success {
            echo 'Python CI/CD Pipeline Completed Successfully!'
        }

        failure {
            echo 'Python CI/CD Pipeline Failed!'
        }

    }

}