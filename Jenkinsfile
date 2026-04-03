pipeline {
    agent {
        label 'agent-1'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    environment {
        APP_VERSION = ''
    }

    stages {

        stage ("read the version"){
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    env.APP_VERSION = packageJson.version
                    echo "application version: ${env.APP_VERSION}"
                }
            }
        }

        stage ("install Dependencies") {
            steps {
                sh """
                npm install
                ls -ltr
                echo "application version: ${APP_VERSION}"
                """
            }
        }

        stage ('build') {
            steps {
                sh """
                zip -q -r backend-${APP_VERSION}.zip * -x Jenkinsfile -x backend-${APP_VERSION}.zip
                ls -ltr
                """
            }
        }
    }

    post {
        always {
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success {
            echo 'I will run when pipeline is success'
        }
        failure {
            echo 'I will run when pipeline is failure'
        }
    }
}