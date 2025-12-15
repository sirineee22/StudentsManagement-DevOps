pipeline {
    agent any
    
    triggers {
        githubPush()  // Active le trigger GitHub webhook automatiquement
    }

    tools {
        maven 'Maven'   // Nom de ton Maven dans Jenkins
        jdk 'JDK'       // Nom de ton JDK dans Jenkins
    }

    environment {
        DOCKER_IMAGE = "siriinaa2233/alpine"   // Docker Hub image
        DOCKER_CRED  = "dockerhub-creds"       // Credentials Jenkins
        APP_PORT     = "8080"                  // Port de Spring Boot
        HOST_PORT    = "8082"                  // Port exposé pour accès externe
        K8S_NAMESPACE = "devops"               // Kubernetes namespace
        K8S_DEPLOYMENT = "students-app"        // Kubernetes deployment name
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
                    def TAG = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
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
                        def TAG = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                        sh """
                            echo "\$DH_PASS" | docker login -u "\$DH_USER" --password-stdin
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
                    def TAG = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    sh """
                        docker rm -f studentsapp || true
                        docker run -d --name studentsapp -p ${HOST_PORT}:${APP_PORT} ${DOCKER_IMAGE}:${TAG}
                    """
                }
            }
        }

        /* 9) DEPLOY TO KUBERNETES */
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "🚀 Déploiement sur Kubernetes..."
                    
                    sh """
                        # Vérifier la connexion au cluster
                        kubectl cluster-info
                        
                        # Vérifier que le deployment existe
                        if kubectl get deployment ${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE} > /dev/null 2>&1; then
                            echo "✅ Deployment trouvé, redémarrage en cours..."
                            
                            # Forcer le pull de la nouvelle image et redémarrer
                            kubectl rollout restart deployment ${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE}
                            
                            # Attendre que le rollout soit terminé
                            echo "⏳ Attente de la fin du déploiement..."
                            kubectl rollout status deployment ${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE} --timeout=5m
                            
                            # Afficher les pods
                            echo "📦 Pods actuels:"
                            kubectl get pods -n ${K8S_NAMESPACE} -l app=students-app
                            
                            echo "✅ Déploiement Kubernetes terminé avec succès!"
                        else
                            echo "❌ Deployment ${K8S_DEPLOYMENT} introuvable dans le namespace ${K8S_NAMESPACE}!"
                            echo "Veuillez déployer manuellement d'abord avec: kubectl apply -f spring-deployment.yaml"
                            exit 1
                        fi
                    """
                }
            }
        }

        /* 10) ACCESS INFORMATION */
        stage('Show Access Info') {
            steps {
                script {
                    def TAG = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    echo "=============================================="
                    echo "✅ DÉPLOIEMENT TERMINÉ"
                    echo "=============================================="
                    echo "🐳 Docker Container (local):"
                    echo "   → http://localhost:${HOST_PORT}"
                    echo ""
                    echo "☸️  Kubernetes (Minikube):"
                    echo "   → Namespace: ${K8S_NAMESPACE}"
                    echo "   → Deployment: ${K8S_DEPLOYMENT}"
                    echo "   → Obtenir l'URL: minikube service students-service -n ${K8S_NAMESPACE} --url"
                    echo ""
                    echo "🚀 Image Docker:"
                    echo "   → ${DOCKER_IMAGE}:${TAG}"
                    echo "   → ${DOCKER_IMAGE}:latest"
                    echo "=============================================="
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec succès !"
            echo "✅ Application déployée sur Docker ET Kubernetes !"
        }
        failure {
            echo "❌ Le pipeline a échoué. Vérifie les logs."
        }
    }
}