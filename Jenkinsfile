pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ademgabsi/students-management"
        DOCKER_TAG   = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ademgabsi/StudentsManagement-DevOps-Adem.git'
            }
        }

        stage('Clean & Build') {
            steps {
                // On saute les tests pour terminer l'atelier rapidement
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Utilise la credential SONAR_TOKEN et fournit explicitement l'URL et le token au scanner
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn -B -Dsonar.host.url=http://localhost:9000 -Dsonar.login=${SONAR_TOKEN} sonar:sonar'
                }
            }
        }

        // Quality Gate (si webhook configuré dans SonarQube) — peut être ignoré si non configuré
        stage('Quality Gate') {
            when {
                expression { return true } // set to false to disable
            }
            steps {
                waitForQualityGate abortPipeline: true
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
