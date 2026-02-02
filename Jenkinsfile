pipeline {
    agent any

    environment {
        COURSE    = "Jenkins"
        ACC_ID    = "014641572640"
        PROJECT   = "roboshop"
        COMPONENT = "catalogue"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Read Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    echo "app version: ${packageJson.version}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                  /usr/bin/npm install
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                  /usr/bin/npm test
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                  echo "Build stage completed"
                '''
            }
        }
    }

    post {
        always {
            echo "I will always say Hello again!"
            cleanWs()
        }
        failure {
            echo "I will run if failure"
        }
    }
}
