pipeline {
    agent any

    parameters {
        choice(
            name: 'ACTION',
            choices: ['DEPLOY', 'REMOVE'],
            description: 'Choose whether to deploy or remove the application'
        )
    }

    tools {
        maven 'maven'
    }

    environment {
        IMAGE_NAME = "foodfiesta-springboot"
        DOCKERHUB_REPO = "ro1hit/foodfiesta-springboot"
    }

    stages {

        stage('Build JAR') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Docker Login') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred-id',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {

                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                    '''
                }
            }
        }

        stage('Tag Image') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh "docker tag ${IMAGE_NAME}:latest ${DOCKERHUB_REPO}:latest"
            }
        }

        stage('Push Image') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh "docker push ${DOCKERHUB_REPO}:latest"
            }
        }

        stage('Deploy Containers') {
            when {
                expression { params.ACTION == 'DEPLOY' }
            }
            steps {
                sh 'docker compose down || true'
                sh 'docker compose pull'
                sh 'docker compose up -d'
            }
        }

        stage('Remove Application') {
            when {
                expression { params.ACTION == 'REMOVE' }
            }
            steps {
                sh 'docker compose down'
                sh 'docker image prune -af'
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully."
        }

        failure {
            echo "Pipeline execution failed."
        }

        always {
            sh 'docker logout || true'
            echo "Pipeline completed."
        }
    }
}