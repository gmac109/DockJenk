pipeline {
    agent any

    environment {
        APP_VERSION = 'v1.0.0'
        MYSQL_HOST  = 'mysql'
        MYSQL_USER  = 'root'
        MYSQL_DB    = 'devops'
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/gmac109/DockJenk.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-app:latest .'
            }
        }
        stage('Awaiting Approval') {
            steps {
                input message: 'Code has been built. Do you approve deployment to production?',
                      ok: 'Yes, Deploy it!'
            }
        }
        stage('Deploy with Docker Compose') {
            steps {
                withCredentials([string(credentialsId: 'mysql-root-password', variable: 'MYSQL_PASSWORD')]) {
                    sh '''
                        export MYSQL_ROOT_PASSWORD="$MYSQL_PASSWORD"
                        export GIT_COMMIT=$(git rev-parse --short HEAD)
                        docker compose down || true
                        docker compose up -d --build
                    '''
                }
            }
        }
    }
}