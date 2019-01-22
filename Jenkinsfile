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
    }
}
