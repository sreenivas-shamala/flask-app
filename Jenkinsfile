pipeline {

    agent any

    environment {
        IMAGE_NAME = "sreenivasulusamala108/flask-app"
        IMAGE_TAG = "v1"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', 
                url:'https://github.com/sreenivas-shamala/flask-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

                
        stage('Test Kubernetes') {
    steps {
        withCredentials([
            file(
                credentialsId: 'k8s-config',
                variable: 'KUBECONFIG'
            )
        ]) {
            sh '''
                set -x

                echo "=== Kubeconfig file ==="
                ls -l "$KUBECONFIG"

                echo "=== Kubeconfig content check ==="
                grep -E '^(apiVersion|kind|current-context|contexts:|clusters:|users:)' "$KUBECONFIG" || true

                echo "=== Explicit kubectl ==="
                KUBECONFIG="$KUBECONFIG" kubectl config get-contexts

                echo "=== Current context ==="
                KUBECONFIG="$KUBECONFIG" kubectl config current-context

                echo "=== Cluster ==="
                KUBECONFIG="$KUBECONFIG" kubectl cluster-info

                echo "=== Nodes ==="
                KUBECONFIG="$KUBECONFIG" kubectl get nodes
            '''
        }
    }
}
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file( credentialsId: 'k8s_config', variable: 'KUBECONFIG' )])
                { 
                sh '''
                               
                echo "=== Applying deployment ===" 
                kubectl apply -f flask-app.yaml
                '''
               }  
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([file( credentialsId: 'k8s_config', variable: 'KUBECONFIG' )])
                {
                sh '''
                
                kubectl get deployments

                kubectl get pods
               
                kubectl get services
                
                '''
                }    
           
           }
      }
    }
    post {
        success {
            echo "Deployment Successful!"
        }

        failure {
            echo "Pipeline Failed!"
        }
    }
}
