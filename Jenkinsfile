pipeline {
    agent any

    tools {
        maven 'Maven'   // Nom de ton outil Maven configuré dans Jenkins
        jdk 'JDK'       // Nom de ton JDK configuré dans Jenkins
    }

    environment {
        DOCKER_IMAGE = "siriinaa2233/alpine"  // Ton image Docker Hub
        DOCKER_CRED  = "dockerhub-creds"     // Tes credentials Jenkins pour Docker Hub
        APP_PORT     = "8080"                 // Port interne de Spring Boot
        HOST_PORT    = "8082"                 // Port exposé sur l'hôte / pour ngrok
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
                    // Récupérer le commit git court pour tag
                    def tag = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    echo "Building Docker image: ${DOCKER_IMAGE}:${tag}"

                    // Construire l'image (Dockerfile à la racine)
                    sh "docker build -t ${DOCKER_IMAGE}:${tag} ."

                    // Tag latest
                    sh "docker tag ${DOCKER_IMAGE}:${tag} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    script {
                        def tag = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()

                        sh '''
                            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                            docker push ${DOCKER_IMAGE}:${tag}
                            docker push ${DOCKER_IMAGE}:latest
                            docker logout
                        '''
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
                    echo "L'application Spring Boot est accessible sur http://localhost:${HOST_PORT}"
                    echo "Pour ngrok : ngrok http ${HOST_PORT}"
                    echo "Ton image Docker est sur Docker Hub : ${DOCKER_IMAGE}:${tag}"
                }
            }
        }
    }

    post {
        success { echo "Pipeline terminé avec succès !" }
        failure { echo "Le pipeline a échoué ! Vérifie les logs et Docker." }
    }
}
