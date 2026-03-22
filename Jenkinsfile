pipeline {
    agent any

    stages {

        stage('Git Checkout') {
            steps {
               checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'GitCreds', url: 'https://github.com/Subhash-Rokkala/shopping-cart.git']])
            }
        }

        stage('Build Process') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t subhashrokkala/shopping-cart:${BUILD_NUMBER} .
                docker tag subhashrokkala/shopping-cart:${BUILD_NUMBER} subhashrokkala/shopping-cart:latest
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DockerCred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push subhashrokkala/shopping-cart:${BUILD_NUMBER}
                    docker push subhashrokkala/shopping-cart:latest
                    docker logout
                    '''
                }
            }
        }

        stage('Test K8s Connection') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl get nodes'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl apply -f k8s/Deployment.yml
                    kubectl apply -f k8s/Service.yml
                    '''
                }
            }
        }

    }
}
