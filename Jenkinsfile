pipeline {

    agent any

    environment {
        IMAGE_NAME = "sreenivasulusamala108/flask-app"
        IMAGE_TAG = "latest"
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
                withCredentials([file( credentialsId: 'kubeconfig', variable: 'KUBECONFIG' )]) 
                { 
                    sh 
                    ''' echo "KUBECONFIG=$KUBECONFIG" 
                    echo "=== kubectl ===" 
                    which kubectl 
                    kubectl version --client 
                    echo "=== context ===" 
                    kubectl config current-context 
                    echo "=== cluster ===" 
                    kubectl cluster-info 
                    echo "=== nodes ===" 
                    kubectl get nodes 
                    ''' 
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file( credentialsId: 'kubeconfig', variable: 'KUBECONFIG' )])
                { 
                sh '''
                echo "=== kubectl version ===" 
                kubectl version --client 
                echo "=== Kubernetes context ===" 
                kubectl config current-context || true 
                echo "=== Kubernetes contexts ===" 
                kubectl config get-contexts || true 
                echo "=== Kubernetes nodes ===" 
                kubectl get nodes echo "=== Applying deployment ===" 
                kubectl apply -f flask-app.yaml
                
                '''
               }  
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([file( credentialsId: 'kubeconfig', variable: 'KUBECONFIG' )])
                {
                sh '''
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
