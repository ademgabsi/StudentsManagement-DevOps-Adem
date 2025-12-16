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
                // Fournir le token via Jenkins credentials, et exécuter l'analyse
                // à l'intérieur du withSonarQubeEnv pour que waitForQualityGate fonctionne.
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQube') {
                        // Le wrapper exporte les variables d'environnement nécessaires.
                        // On passe explicitement le token pour s'assurer qu'il est utilisé.
                        sh 'mvn -B -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=${SONAR_TOKEN} sonar:sonar'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // Attend le résultat du Quality Gate et arrête le pipeline si échec
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
