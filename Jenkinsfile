
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
                git branch: "${params.BRANCH}", url: 'https://github.com/Harsh007-devops/jenkins-project3.git'
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
                      echo "Simulated Failure Triggered!"
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

    success {
        echo "Build Successful ✅"
    }

    failure {
        echo "Build Failed ❌ Sending Alert..."

        sh '''
        mkdir -p failed_logs
        echo "Failure occurred at $(date)" > failed_logs/error.log
        ls -lrt >> failed_logs/error.log
        '''

        archiveArtifacts artifacts: 'failed_logs/**'
    }

    always {
        echo "Cleaning workspace..."

        cleanWs()
    }
}
}
