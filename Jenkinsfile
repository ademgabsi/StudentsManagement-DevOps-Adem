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
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn -B -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=${SONAR_TOKEN} sonar:sonar'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
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

        stage('Kubernetes Deploy') {
            steps {
                // Provide kubeconfig as a Jenkins "Secret file" credential with ID 'kubeconfig'
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                      export KUBECONFIG=${KUBECONFIG}
                      kubectl apply -f k8s/
                      kubectl rollout status deployment/students-management --timeout=120s || true
                      kubectl get pods -l app=students-management -o wide
                      kubectl get svc students-management -o wide
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminé (succès ou échec).'
        }
        success {
            echo 'Build réussi, image Docker poussée et déployée sur Kubernetes.'
        }
        failure {
            echo 'Le pipeline a échoué, vérifier les logs.'
        }
    }
}
