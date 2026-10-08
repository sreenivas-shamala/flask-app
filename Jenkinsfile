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

stage('Test Kubernetes Network') {
    steps {
        withCredentials([
            file(
                credentialsId: 'k8s_config',
                variable: 'KUBECONFIG'
            )
        ]) {
            sh '''
                echo "=== DNS ==="
                getent hosts kubernetes.docker.internal || true

                echo "=== Kubernetes API ==="
                curl -k -I https://kubernetes.docker.internal:6443 || true

                echo "=== Kubectl ==="
                kubectl config current-context
                kubectl get nodes
            '''
        }
    }
}
        
      stage('Test Kubernetes') {
    steps {
        withCredentials([
            file(
                credentialsId: 'k8s_config',
                variable: 'KUBECONFIG'
            )
        ]) {
            sh '''
                set -e

                echo "=== Kubeconfig ==="
                ls -l "$KUBECONFIG"

                echo "=== Context ==="
                KUBECONFIG="$KUBECONFIG" kubectl config current-context

                echo "=== User ==="
                KUBECONFIG="$KUBECONFIG" kubectl auth whoami

                echo "=== Nodes ==="
                KUBECONFIG="$KUBECONFIG" kubectl get nodes --request-timeout=10s
            '''
        }
    }
}
stage('Debug Kubernetes') {
    steps {
        withCredentials([
            file(
                credentialsId: 'k8s_config',
                variable: 'KUBECONFIG'
            )
        ]) {
            sh '''
                set -x

                echo "=== Server ==="
                kubectl config view --minify \
                    -o jsonpath='{.clusters[0].cluster.server}'
                echo

                echo "=== Context ==="
                kubectl config current-context

                echo "=== User ==="
                kubectl config view --minify \
                    -o jsonpath='{.contexts[0].context.user}'
                echo

                echo "=== API test ==="
                timeout 15s kubectl get --raw=/version
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
