pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "awamousene/examen_devops"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    tools {
        maven 'Maven 3.9.0'
        jdk 'JDK 21'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/AwaMoussaSene/examen_devops.git', branch: 'main'
            }
        }

        stage('Build Maven Project') {
            steps {
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat """
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% .
                    docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest
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
