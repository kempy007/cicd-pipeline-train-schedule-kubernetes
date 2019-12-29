pipeline {
    agent any
//    agent {
//        kubernetes {
//            label 'dind'
//            label 'nested-pod'
//            yaml """
//spec:
//containers:
//- name: maven
//image: maven:3.3.9-jdk-8-alpine
//command:
//- cat
//tty: true
//    """
//        }
//    }
    environment {
        DOCKER_IMAGE_NAME = "ds2mk/train-schedule"
    }
    stages {
        stage('Build') {
            when { branch 'master' }
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when { branch 'master' }
            agent { kubernetes { label 'dind' } }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when { branch 'master' }
            agent { kubernetes { label 'dind' } }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
