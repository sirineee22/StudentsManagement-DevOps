pipeline {
    agent any
    
    tools {
        maven 'Maven'
        jdk 'JDK'
    }

    environment {
        DOCKER_IMAGE = "siriinaa2233/alpine"
        K8S_NAMESPACE = "devops"
        K8S_DEPLOYMENT = "students-app"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sirineee22/StudentsManagement-DevOps'
            }
        }

        stage('Clean + Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        /* NOUVEAU: SonarQube Analysis */
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def TAG = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${TAG} .
                        docker tag ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        /* Push Docker Image - Commenté */
        /*
        stage('Push Docker Image') {
            ...
        }
        */

        stage('Run Container') {
            steps {
                script {
                    def TAG = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    sh """
                        docker rm -f studentsapp || true
                        docker run -d --name studentsapp -p 8082:8080 ${DOCKER_IMAGE}:${TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        kubectl rollout restart deployment ${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE}
                        kubectl rollout status deployment ${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE} --timeout=5m
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec succès!"
        }
        failure {
            echo "❌ Le pipeline a échoué."
        }
    }
}