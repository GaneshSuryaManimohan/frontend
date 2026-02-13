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
         NEXUS_URL = 'nexus.surya-devops.site:8081'
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

        stage('Nexus Artifact Upload') {
            steps {
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUS_URL}",
                        groupId: 'com.expense',
                        version: "${APP_VERSION}",
                        repository: "frontend",
                        credentialsId: 'nexus_auth',
                        artifacts: [
                            [artifactId: "frontend",
                            classifier: '',
                            file: "frontend-" + "${APP_VERSION}" + ".zip",
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
                        string(name: 'APP_VERSION', value: "${APP_VERSION}")
                    ]
                    // Triggers the job 'frontend-deploy'
                    build job: 'frontend-deploy', parameters: params, wait: false
                } 
            }
        }
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