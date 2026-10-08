pipeline {

    agent any

    environment {
        IMAGE_NAME = "sreenivasulusamala108/flask-app"
        IMAGE_TAG = "v1"
        PATH = "/usr/local/bin:${env.PATH}"
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

       stage('Check kubectl') {
    steps {
        sh '''
            echo "PATH=$PATH"
            which kubectl
            kubectl version --client
        '''
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

                echo "===== kubectl ====="
                which kubectl
                kubectl version --client

                echo "===== KUBECONFIG ====="
                ls -l "$KUBECONFIG"

                echo "===== CONTEXT ====="
                kubectl config current-context

                echo "===== API SERVER ====="
                kubectl config view --minify \
                    -o jsonpath='{.clusters[0].cluster.server}'
                echo

                echo "===== CLUSTER INFO ====="
                kubectl cluster-info

                echo "===== NODES ====="
                kubectl get nodes
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
