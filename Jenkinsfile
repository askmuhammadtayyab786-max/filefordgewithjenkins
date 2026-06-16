pipeline {
    agent any
    environment {
        GIT_REPO   = 'https://github.com/askmuhammadtayyab786-max/filefordgewithjenkins.git'
        GIT_BRANCH = 'main'
    }
    stages {
        stage('Checkout') {
            steps {
                echo '>>> Pulling latest code from GitHub...'
                git branch: "${GIT_BRANCH}",
                    credentialsId: 'github-creds',
                    url: "${GIT_REPO}"
            }
        }

        stage('Stop Old Containers') {
            steps {
                echo '>>> Stopping existing containers...'
                sh """
                    cd ${WORKSPACE}
                    docker-compose down --remove-orphans || true
                """
            }
        }

        stage('Remove Old Images') {
            steps {
                echo '>>> Removing old images...'
                sh """
                    docker rmi frontend || true
                    docker rmi backend  || true
                    docker image prune -f || true
                """
            }
        }

        stage('Build Images') {
            steps {
                echo '>>> Building images...'
                sh """
                    cd ${WORKSPACE}
                    docker-compose build --no-cache
                """
            }
        }

        stage('Deploy') {
            steps {
                echo '>>> Starting all services...'
                sh """
                    cd ${WORKSPACE}
                    docker-compose up -d
                """
            }
        }

        stage('Health Check') {
            steps {
                echo '>>> Waiting for containers to stabilize...'
                sh 'sleep 15'
                sh '''
                    echo "=== Running Containers ==="
                    docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
                '''
                sh '''
                    echo "=== HTTP Health Check ==="
                    curl -f --retry 5 --retry-delay 3 http://3.17.129.133 \
                        && echo "SUCCESS: App is live!" \
                        || echo "WARNING: App not responding yet"
                '''
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
