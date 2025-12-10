pipeline {
    agent any

    tools {
        maven 'Maven'   // Nom de ton outil Maven configuré dans Jenkins
        jdk 'JDK'       // Nom de ton JDK configuré dans Jenkins
    }

    environment {
        DOCKER_IMAGE = "siriinaa2233/alpine"  // Ton image Docker Hub
        APP_PORT = "8080"        // Port interne de Spring Boot
        HOST_PORT = "8082"       // Port exposé sur l'hôte / pour ngrok
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sirineee22/StudentsManagement-DevOps'
            }
        }

        stage('Clean') {
            steps { sh 'mvn clean' }
        }

        stage('Compile') {
            steps { sh 'mvn compile' }
        }

        stage('Test') {
            steps { sh 'mvn test' }
            post { always { junit '**/target/surefire-reports/*.xml' } }
        }

        stage('Package') {
            steps { sh 'mvn package -DskipTests' }
        }

        stage('Archive JAR') {
            steps { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def tag = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    sh "docker build -t ${DOCKER_IMAGE}:${tag} -f dockerfile ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def tag = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        sh "docker push ${DOCKER_IMAGE}:${tag}"
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    def tag = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    sh """
                        docker rm -f studentsapp || true
                        docker run -d --name studentsapp -p ${HOST_PORT}:${APP_PORT} ${DOCKER_IMAGE}:${tag}
                    """
                }
            }
        }

        stage('Show Access Info') {
            steps {
                script {
                    echo "L'application Spring Boot est maintenant accessible sur http://localhost:${HOST_PORT}"
                    echo "Pour ngrok, lance : ngrok http ${HOST_PORT}"
                    echo "Ton image Docker est poussée sur Docker Hub sous : ${DOCKER_IMAGE}:${tag}"
                }
            }
        }
    }

    post {
        failure { echo "Le pipeline a échoué ! Vérifie les logs et Docker." }
        success { echo "Pipeline terminé avec succès !" }
    }
}
