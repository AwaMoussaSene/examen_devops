pipeline {
    agent any

    environment {
        // Nom de l'image Docker : ton utilisateur Docker Hub et repo
        DOCKER_IMAGE = "awamousene/examen_devops"
        // Tag dynamique basé sur le numéro de build Jenkins
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                echo "Récupération du code depuis GitHub..."
                git url: 'https://github.com/AwaMoussaSene/examen_devops.git', branch: 'main'
            }
        }

        stage('Build Maven Project') {
            steps {
                echo "Compilation Maven et build du JAR..."
                // Exécute Maven depuis le PATH
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    echo "Connexion à Docker Hub..."
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"

                    echo "Construction de l'image Docker..."
                    bat """
                    docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% .
                    docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest
                    """

                    echo "Push de l'image Docker vers Docker Hub..."
                    bat """
                    docker push %DOCKER_IMAGE%:%DOCKER_TAG%
                    docker push %DOCKER_IMAGE%:latest
                    """
                }
            }
        }

        stage('Deploy to Render') {
            steps {
                withCredentials([
                    string(credentialsId: 'java-render-webhook', variable: 'RENDER_WEBHOOK'),
                    string(credentialsId: 'java-render-app-url', variable: 'RENDER_APP_URL')
                ]) {
                    echo "Déploiement sur Render..."
                    bat "curl -X POST \"%RENDER_WEBHOOK%\""
                    echo "App URL: %RENDER_APP_URL%"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline terminé avec succès ✅"
        }
        failure {
            echo "Le pipeline a échoué ❌"
        }
    }
}
