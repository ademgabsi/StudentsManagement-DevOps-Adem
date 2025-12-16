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

        stage('Clean & Build (with tests)') {
            steps {
                sh 'mvn -B -DskipTests=false clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Requires "Manage Jenkins -> Configure System -> SonarQube servers" with Name: SonarQube
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn -B sonar:sonar'
                }
            }
        }

        // Optional: wait for Quality Gate result (requires SonarQube webhook pointing to Jenkins)
        // Configure webhook in SonarQube:
        // Administration -> Configuration -> Webhooks -> Add:
        // Name: Jenkins, URL: http://127.0.0.1:8080/sonarqube-webhook/
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
