pipeline {
    agent { label 'GCP-Jenkins-Agent' }

    environment {
        IMAGE_NAME = "anubharani/portfolio-website"
        IMAGE_TAG = "${BUILD_NUMBER}"
        GCP_ARTIFACT_IMAGE_NAME = "asia-south1-docker.pkg.dev/testing-bharani/portfolio-artifact-registry/portfolio-website"
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
                                -Dsonar.host.url=http://34.14.154.234:9000\
                                -Dsonar.login=$SONAR_TOKEN
                            """
                        }
                    }
                }
            }
        }

    }   
}       