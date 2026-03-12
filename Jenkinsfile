pipeline {
    agent any
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
                sh 'docker compose down || true'
                sh 'docker compose up -d --build'
            }
        }
    }
}