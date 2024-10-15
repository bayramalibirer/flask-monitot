pipeline {
    agent any

    stages {
        stage('Git Cloning') {
            steps {
                echo 'Cloning git repo'
                git url: 'https://github.com/bayramalibirer/flask-monitot.git',  branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                echo 'Building the image'
                sh 'docker build -t flask-monitot .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing to Docker Hub'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo "${PASS}" | docker login --username "${USER}" --password-stdin
                    docker tag flask-monitot ${USER}/flask-monitot:latest
                    docker push ${USER}/flask-monitot:latest
                    '''
                }
            }
        }
        stage('Integrate Remote kubernetess with Jenkins') {
            steps {
                withCredentials([string(credentialsId: 'SECRET_TOKEN', variable: 'KUBE_TOKEN')]) {
                    sh '''
                    curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"
                    chmod u+x ./kubectl
                    export KUBECONFIG=$(mktemp)
                    ./kubectl config set-cluster do-fra1-tech-istabul-odev-2 --server=https://987b5d0b-f856-4b46-b17f-9fc501bb027d.k8s.ondigitalocean.com --insecure-skip-tls-verify=true
                    ./kubectl config set-credentials jenkins --token=${KUBE_TOKEN}
                    ./kubectl config set-context default --cluster=do-fra1-tech-istabul-odev-2 --user=jenkins --namespace=default
                    ./kubectl config use-context default
                    ./kubectl get nodes
                    ./kubectl apply -f service.yaml
                    ./kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }
}
