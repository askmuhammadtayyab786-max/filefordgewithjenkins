pipeline {
    agent any

    environment {
        GIT_REPO   = 'https://github.com/askmuhammadtayyab786-max/filefordgewithjenkins.git'
        GIT_BRANCH = 'main'
        APP_IP     = '3.129.8.249'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}",
                    credentialsId: 'github-creds',
                    url: "${GIT_REPO}"
            }
        }

        stage('Stop Old Containers') {
            steps {
                sh 'docker-compose down --remove-orphans || true'
            }
        }

        stage('Remove Old Images') {
            steps {
                sh '''
                    docker rmi frontend || true
                    docker rmi backend  || true
                    docker image prune -f || true
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh 'docker-compose build --no-cache'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker-compose up -d'
            }
        }

        stage('Health Check') {
            steps {
                sh 'sleep 20'
                sh 'docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
                sh 'curl -f --retry 5 --retry-delay 5 http://3.129.8.249 && echo "SUCCESS" || echo "WARNING: not responding yet"'
            }
        }
    }

    post {
        success {
            echo '========== DEPLOYMENT SUCCESSFUL =========='
        }
        failure {
            sh '''
                docker logs frontend || true
                docker logs backend  || true
                docker logs nginx    || true
            '''
        }
        always {
            cleanWs()
        }
    }
}
