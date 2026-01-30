pipeline {
    agent {
        node { label 'AGENT-1' }
    }

    environment {
        COURSE = "Jenkins"
        ACC_ID = "014641572640"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }

    options {
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        stage('Read Version') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    def appVersion = packageJSON.version
                    echo "App version: ${appVersion}"
                    env.APP_VERSION = appVersion
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh '''
                        # Load Node.js module
                        source /etc/profile.d/modules.sh || true
                        module load nodejs:20 || true

                        # Check Node.js
                        which node
                        node -v
                        npm -v

                        # Install dependencies
                        npm install
                    '''
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    withAWS(region:'us-east-1', credentials:'aws-creds') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${APP_VERSION} .
                            docker images
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${APP_VERSION}
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                expression { "$params.DEPLOY" == "true" }
            }
            steps {
                sh 'echo "Deploying application..."'
            }
        }
    }

    post {
        always {
            echo 'I will always say Hello again!'
            cleanWs()
        }
        success { echo 'Pipeline succeeded!' }
        failure { echo 'Pipeline failed!' }
        aborted { echo 'Pipeline was aborted!' }
    }
}
