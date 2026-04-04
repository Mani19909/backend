pipeline {
    agent {
        label 'agent-1'
    }
    options {
        timeout (time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment{
        appVersion = ''
        nexusUrl = "nexus.daws.info:8081"
    }
    stages {
        stage ("read the version"){
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"               
                }
            }
        }
        stage ("install Dependencies") {
            steps {
                sh '''
                npm install
                ls -ltr
                echo "application version: $appVersion"
                '''
            }
        }
        stage ('build') {
            steps {
                sh """
                zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                ls -ltr
                """
            }
        }
        stage ('nexus') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "backend",
                        credentialsId: 'nexus_auth',
                        artifacts: [
                            [artifactId: "backend",
                            classifier: '', 
                            file: "backend-" + "${appVersion}" + '.zip' , 
                            type: 'zip']
                        ]
                    )
                }
            }
        }
        stage('Deploy') {
            steps {
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'backend-deploy', parameters: params, wait: false
                }
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

