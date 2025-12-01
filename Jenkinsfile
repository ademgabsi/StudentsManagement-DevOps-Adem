pipeline {
    agent any

    environment {
        // Remplace par TON repo Docker Hub
        DOCKER_IMAGE = "ademgabsi/students-management"
        DOCKER_TAG   = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupérer la dernière version de TON fork
                git branch: 'main',
                    url: 'https://github.com/ademgabsi/StudentsManagement-DevOps-Adem.git'
            }
        }

        stage('Clean & Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminé (succès ou échec).'
        }
        success {
            echo 'Build réussi, image Docker poussée.'
        }
        failure {
            echo 'Le pipeline a échoué, vérifier les logs.'
        }
    }
}
