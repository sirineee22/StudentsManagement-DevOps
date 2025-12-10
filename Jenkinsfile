pipeline {
    agent any

    tools {
        maven 'Maven'   // Nom de ton Maven dans Jenkins
        jdk 'JDK'       // Nom de ton JDK dans Jenkins
    }

    environment {
        DOCKER_IMAGE = "siriinaa2233/alpine"   // Docker Hub image
        DOCKER_CRED  = "dockerhub-creds"       // Credentials Jenkins
        APP_PORT     = "8080"                  // Port de Spring Boot
        HOST_PORT    = "8082"                  // Port exposé pour accès externe
    }

    stages {

        /* 1) CHECKOUT */
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sirineee22/StudentsManagement-DevOps'
            }
        }

        /* 2) CLEAN + COMPILE */
        stage('Clean + Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        /* 3) UNIT TESTS */
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

        /* 4) PACKAGE */
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        /* 5) ARCHIVE ARTIFACT */
        stage('Archive JAR') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        /* 6) DOCKER BUILD */
        stage('Build Docker Image') {
            steps {
                script {
                    TAG = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    echo "Building Docker image: ${DOCKER_IMAGE}:${TAG}"

                    sh """
                        docker build -t ${DOCKER_IMAGE}:${TAG} .
                        docker tag ${DOCKER_IMAGE}:${TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        /* 7) PUSH DOCKER HUB */
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CRED}",
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    script {
                        sh """
                            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                            docker push ${DOCKER_IMAGE}:${TAG}
                            docker push ${DOCKER_IMAGE}:latest
                            docker logout
                        """
                    }
                }
            }
        }

        /* 8) RUN CONTAINER */
        stage('Run Container') {
            steps {
                script {
                    sh """
                        docker rm -f studentsapp || true
                        docker run -d --name studentsapp -p ${HOST_PORT}:${APP_PORT} ${DOCKER_IMAGE}:${TAG}
                    """
                }
            }
        }

        /* 9) ACCESS INFORMATION */
        stage('Show Access Info') {
            steps {
                script {
                    echo "👉 Application disponible sur : http://localhost:${HOST_PORT}"
                    echo "👉 Pour exposer via Ngrok : ngrok http ${HOST_PORT}"
                    echo "👉 Image Docker poussée sur Docker Hub : ${DOCKER_IMAGE}:${TAG}"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec succès !"
        }
        failure {
            echo "❌ Le pipeline a échoué. Vérifie les logs."
        }
    }
}
