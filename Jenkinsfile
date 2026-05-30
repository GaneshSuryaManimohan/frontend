pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    environment{
         APP_VERSION = '' // Variable Declaration
         NEXUS_URL = 'nexus.surya-devops.online:8081'
         region = 'us-east-1'
         acc_id = '328342418911'
    }
    stages {
        stage('read the version'){
            steps{
                script{
                    def packageJson = readJSON file : 'package.json'
                    APP_VERSION = packageJson.version
                    echo "application version: ${APP_VERSION}"
                }
            }
        }
        stage('Build') {
            steps {
                sh """
                 zip -q -r frontend-${APP_VERSION}.zip * -x Jenkinsfile -x frontend-${APP_VERSION}.zip
                 ls -ltr
                """
            }
        }

        stage('Docker Build and Push'){
            steps{
                sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${acc_id}.dkr.ecr.${region}.amazonaws.com

                    docker build -t ${acc_id}.dkr.ecr.${region}.amazonaws.com/expense-dev-frontend:${APP_VERSION} .

                    docker push ${acc_id}.dkr.ecr.${region}.amazonaws.com/expense-dev-frontend:${APP_VERSION}

                """
            }
        }

        stage('Deploy'){
            steps{
                sh """
                    aws eks update-kubeconfig --region ${region} --name expense
                    cd helm
                    sed -i "s/IMAGE_VERSION/${APP_VERSION}/g" values.yaml
                    helm upgrade frontend .  
                """
            }
        }


        // stage('Nexus Artifact Upload') {
        //     steps {
        //         script{
        //             nexusArtifactUploader(
        //                 nexusVersion: 'nexus3',
        //                 protocol: 'http',
        //                 nexusUrl: "${NEXUS_URL}",
        //                 groupId: 'com.expense',
        //                 version: "${APP_VERSION}",
        //                 repository: "frontend",
        //                 credentialsId: 'nexus_auth',
        //                 artifacts: [
        //                     [artifactId: "frontend",
        //                     classifier: '',
        //                     file: "frontend-" + "${APP_VERSION}" + ".zip",
        //                     type: 'zip']
        //                 ]
        //             )
        //         }
        //     }
        // }

        // stage('Deploy') {
        //     steps {
        //         script{
        //             def params = [
        //                 string(name: 'APP_VERSION', value: "${APP_VERSION}")
        //             ]
        //             // Triggers the job 'frontend-deploy'
        //             build job: 'frontend-deploy', parameters: params, wait: false
        //         } 
        //     }
        // }
    }
    post {
        always {
            echo 'I will always say hello'
            deleteDir()
        }
        success {
            echo 'Shows Only upon success'
        }
        failure {
            echo 'shows upon failure'
        }
    }
}