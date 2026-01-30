pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    // Pre-build environment variables
    environment {
        COURSE = "Jenkins"
        ACC_ID = "014641572640"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }

    // Timeout & concurrency options
    options {
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    // Tools section: NodeJS plugin ensures node & npm are available
    tools {
        nodejs 'node20' // Name must match NodeJS installation in Jenkins Global Tool Configuration
    }

    stages {

        // Stage 1: Read Version from package.json
        stage('Read Version') {
            steps {
                script {
                    // Use a local variable, not environment block
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "App version: ${appVersion}"
                }
            }
        }

        // Stage 2: Install Node.js dependencies
        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        node -v
                        npm -v
                        npm install
                    '''
                }
            }
        }

        // Stage 3: Build Docker Image & Push to ECR
        stage('Build Image') {
            steps {
                script {
                    withAWS(region:'us-east-1', credentials:'aws-creds') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                            docker images
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }

        // Stage 4: Deploy (Optional / gated by parameter)
        stage('Deploy') {
            when {
                expression { "$params.DEPLOY" == "true" }
            }
            steps {
                script {
                    sh '''
                        echo "Deploying application..."
                        # Add your deploy commands here
                    '''
                }
            }
        }
    }

    // Post-build actions
    post {
        always {
            echo 'I will always say Hello again!'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        aborted {
            echo 'Pipeline was aborted!'
        }
    }
}
