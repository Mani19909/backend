pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout (time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment{
        appVersion = ''
        nexusUrl = "nexus.daws.info:8081"
        region   =  "us-east-1"
        account_id = "222634371492"
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
        stage('Docker build') {
            steps {
                sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-backend:${appVersion} .

                    docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-backend:${appVersion}

                """
            }
        }
        stage("Deploy") {
            steps {
                sh """
                    aws eks update-kubeconfig --region us-east-1 --name expense-dev
                    cd helm
                    sed -i 's/IMAGE_VERSION/${appVersion}/g' values.yaml
                    helm install backend .

                """
            }
        }
        // stage ('nexus artifact upload') {
            // steps {
                // script {
                    // nexusArtifactUploader(
                        // nexusVersion: 'nexus3',
                        // protocol: 'http',
                        // nexusUrl: "${nexusUrl}",
                        // groupId: 'com.expense',
                        // version: "${appVersion}",
                        // repository: "backend",
                        // credentialsId: 'nexus_auth',
                        // artifacts: [
                            // [artifactId: "backend",
                            // classifier: '', 
                            // file: "backend-" + "${appVersion}" + '.zip' , 
                            // type: 'zip']
                        // ]
                    // )
                // }
            // }
        // }
        // stage('Deploy') {
            // steps {
                // script{
                    // def params = [
                        // string(name: 'appVersion', value: "${appVersion}")
                    // ]
                    // build job: 'backend-deploy', parameters: params, wait: false
                // }
            // }
        // }      
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

