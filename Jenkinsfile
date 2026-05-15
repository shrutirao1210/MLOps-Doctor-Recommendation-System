pipeline {
    agent any

    environment {
        BANDIT_IMAGE     = "shrutimrao/bandit"
        SPECIALITY_IMAGE = "shrutimrao/speciality"
        FRONTEND_IMAGE   = "shrutimrao/frontend"

        ANSIBLE_INVENTORY = "Ansible/hosts.ini"
        ANSIBLE_PLAYBOOK  = "Ansible/deploy.yml"

        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        PATH = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Code checked out."
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo "Building Docker images..."

                    sh "docker build -t ${BANDIT_IMAGE}:latest ./main1"
                    sh "docker build -t ${SPECIALITY_IMAGE}:latest ./main2"
                    sh "docker build -t ${FRONTEND_IMAGE}:latest ."
                }
            }
        }

        stage('Push Docker Images to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push shrutimrao/bandit:latest
                        docker push shrutimrao/speciality:latest
                        docker push shrutimrao/frontend:latest
                    '''
                }
            }
        }

        stage('Load Images into Minikube') {
            steps {
                sh """
                    minikube image load ${BANDIT_IMAGE}:latest
                    minikube image load ${SPECIALITY_IMAGE}:latest
                    minikube image load ${FRONTEND_IMAGE}:latest
                """
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh """
                ansible-playbook -i ${ANSIBLE_INVENTORY} ${ANSIBLE_PLAYBOOK} \
                    -e bandit_tag=latest \
                    -e speciality_tag=latest \
                    -e frontend_tag=latest
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }

        failure {
            echo "Deployment failed. Check logs above."
        }
    }
}
