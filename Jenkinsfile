pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git Branch')
        booleanParam(name: 'DEBUG', defaultValue: false, description: 'Enable failure')
        choice(name: 'ENV', choices: ['dev','staging','prod'], description: 'Environment')
    }

    environment {
        CACHE_FILE = "deps.txt"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}", url: 'YOUR_GIT_REPO_URL'
            }
        }

        stage('Cache Dependencies') {
            steps {
                script {
                    if (fileExists(env.CACHE_FILE)) {
                        echo "Using cached dependencies"
                    } else {
                        echo "Downloading dependencies"
                        sh "echo 'dependency-data' > ${CACHE_FILE}"
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Building application..."'
            }
        }

        stage('Test') {
            steps {
                retry(2) {
                    sh '''
                    echo "Running tests..."
                    
                    if [ "$DEBUG" = "true" ]; then
                      echo "Forcing failure"
                      exit 1
                    fi
                    
                    echo "Tests passed"
                    '''
                }
            }
        }

        stage('Archive') {
            steps {
                sh 'tar -czf build.tar.gz *'
                archiveArtifacts artifacts: '*.tar.gz'
            }
        }

        stage('Deploy') {
            when {
                expression { params.ENV == 'prod' }
            }
            steps {
                echo "Deploying to PROD environment"
            }
        }
    }

    post {
        failure {
            echo "Build Failed!"
            sh "mkdir -p failed_logs && cp -r * failed_logs/"
            archiveArtifacts artifacts: 'failed_logs/**'
        }

        always {
            echo "Cleaning workspace"
            cleanWs()
        }
    }
}
