pipeline {
    agent any

    stages {
        stage ('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("fabricioveronez/web-live-app:${env.BUILD_ID}", "-f ./src/Dockerfile ./src")
                }
            }
        }

        stage ('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage ('Deploy no Kubernetes') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withAWS(credentials: 'jenkins-credential', region: 'us-east-1') {
                    sh 'aws eks update-kubeconfig --name live-eks'
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    sh 'kubectl apply -f ./k8s/deployment.yaml'
                }

            }
        }
    }
}