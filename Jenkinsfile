pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK'
    }

    environment {
        DOCKER_IMAGE = "siriinaa2233/alpine"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sirineee22/StudentsManagement-DevOps'
            }
        }

        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
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
                    // Tag basé sur le commit
                    def tag = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()

                    sh "docker build -t ${DOCKER_IMAGE}:${tag} ."
                }
            }
        }

        stage('Run Container (8082)') {
            steps {
                script {
                    def tag = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()

                    sh """
                        docker rm -f studentsapp || true
                        docker run -d --name studentsapp -p 8082:8082 ${DOCKER_IMAGE}:${tag}
                    """
                }
            }
        }
    }
}
