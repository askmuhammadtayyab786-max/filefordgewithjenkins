pipeline {
    agent any

    environment {
        GIT_REPO   = 'https://github.com/askmuhammadtayyab786-max/filefordgewithjenkins.git'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Checkout') {
            steps {
                node('built-in') {
                    echo '>>> Pulling latest code from GitHub...'
                    git branch: "${GIT_BRANCH}",
                        credentialsId: 'github-creds',
                        url: "${GIT_REPO}"
                }
            }
        }

        stage('Stop Old Containers') {
            steps {
                node('built-in') {
                    sh """
                        docker-compose -f ${WORKSPACE}/docker-compose.yml down --remove-orphans || true
                    """
                }
            }
        }

        stage('Remove Old Images') {
            steps {
                node('built-in') {
                    sh """
                        docker rmi frontend || true
                        docker rmi backend  || true
                        docker image prune -f || true
                    """
                }
            }
        }

        stage('Build Images') {
            steps {
                node('built-in') {
                    sh """
                        cd ${WORKSPACE}
                        docker-compose build --no-cache
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                node('built-in') {
                    sh """
                        cd ${WORKSPACE}
                        docker-compose up -d
                    """
                }
            }
        }

        stage('Health Check') {
            steps {
                node('built-in') {
                    sh 'sleep 15'
                    sh '''
                        docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
                    '''
                    sh '''
                        curl -f --retry 5 --retry-delay 3 http://3.17.129.133 \
                            && echo "SUCCESS: App is live!" \
                            || echo "WARNING: App not responding yet"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '========== DEPLOYMENT SUCCESSFUL =========='
        }
        failure {
            node('built-in') {
                sh '''
                    docker logs frontend || true
                    docker logs backend  || true
                    docker logs nginx    || true
                '''
            }
        }
        always {
            node('built-in') {
                cleanWs()
            }
        }
    }
}
