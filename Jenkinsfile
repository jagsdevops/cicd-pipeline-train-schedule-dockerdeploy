pipeline {
    agent any
    environment {
        registry = "$dh_registry"
        registryCredential = "$cr_docker_hub"
        dockerImage = ''
    }
    stages {
        stage('Cloning Git') {
            steps {
                git 'https://github.com/jagsdevops/cicd-pipeline-train-schedule-dockerdeploy.git'
            }
        }
        stage('Building image') {
            steps{
                sh 'printenv'
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy Image') {
            steps{
                script {
                    docker.withRegistry( '', 'cr_docker_hub' ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prd_sys01 \"docker pull jagsdevops/trainschedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prd_sys01 \"docker stop trainschedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prd_sys01 \"docker rm trainschedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prd_sys01 \"docker run --restart always --name train-schedule -p 8080:8080 -d jagsdevops/trainschedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
        
    }
}
