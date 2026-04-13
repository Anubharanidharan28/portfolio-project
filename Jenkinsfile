pipeline {
    agent { label 'GCP-Jenkins-Agent' }

    environment {
        IMAGE_NAME = "anubharani/personal-portfolio-website"
        IMAGE_TAG = "${BUILD_NUMBER}"
        GCP_ARTIFACT_IMAGE_NAME = "asia-south1-docker.pkg.dev/testing-bharani/personal-portfolio-website/portfolio-website"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/Anubharanidharan28/portfolio-project.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('SonarQube') {
                        withCredentials([string(credentialsId: 'Sonar-Tokens', variable: 'SONAR_TOKEN')]) {

                            sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=portfolio-website \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://34.14.154.234:9000 \
                                -Dsonar.login=$SONAR_TOKEN
                            """
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                docker tag $IMAGE_NAME:$IMAGE_TAG $GCP_ARTIFACT_IMAGE_NAME:$IMAGE_TAG
                docker tag $IMAGE_NAME:$IMAGE_TAG $GCP_ARTIFACT_IMAGE_NAME:latest
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker push $IMAGE_NAME:latest
                    """
                }
            }
        }

        stage('Push to GCP Artifact Registry') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account-json', variable: 'GCP_KEY')]) {
                    sh """
                    echo "Authenticating with GCP..."

                    gcloud auth activate-service-account --key-file=$GCP_KEY

                    gcloud auth configure-docker asia-south1-docker.pkg.dev -q

                    echo "Pushing images..."

                    docker push $GCP_ARTIFACT_IMAGE_NAME:$IMAGE_TAG
                    docker push $GCP_ARTIFACT_IMAGE_NAME:latest
                    """
                }
            }
        }
    }
}