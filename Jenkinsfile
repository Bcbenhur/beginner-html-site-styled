pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "bbenhur03/mdn-site:${env.BUILD_NUMBER}"
        DOCKER_HUB_CRED = 'dockerhub-creds'
        SSH_K8S_MASTER = 'ssh-k8s-master'
        K8S_MASTER_USER = 'ubuntu'
        K8S_MASTER_HOST = '54.160.127.73'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_HUB_CRED, usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                        sh '''
                            docker build -t ${DOCKER_IMAGE} .
                            echo $DH_PASS | docker login -u $DH_USER --password-stdin
                            docker push ${DOCKER_IMAGE}
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Update Kubernetes manifests and apply') {
            steps {
                script {
                    sh "sed -i 's|<DOCKERHUB_USER>/mdn-site:latest|${DOCKER_IMAGE}|g' k8s/deployment.yaml"

                    sshagent(credentials: [env.SSH_K8S_MASTER]) {
                        sh """
                            scp -o StrictHostKeyChecking=no k8s/deployment.yaml k8s/service.yaml ${K8S_MASTER_USER}@${K8S_MASTER_HOST}:/tmp/
                            ssh -o StrictHostKeyChecking=no ${K8S_MASTER_USER}@${K8S_MASTER_HOST} 'kubectl apply -f /tmp/deployment.yaml && kubectl apply -f /tmp/service.yaml'
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment complete — visit http://${K8S_MASTER_HOST}:30010"
        }
        failure {
            echo "❌ Pipeline failed — check Jenkins logs"
        }
    }
}
