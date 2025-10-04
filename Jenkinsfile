pipeline {
    agent any

    environment {
        // Docker image name with build number
        DOCKER_IMAGE = "bbenhur03/mdn-site:${env.BUILD_NUMBER}"
        // Jenkins credential ID for Docker Hub (username + token)
        DOCKER_HUB_CRED = 'dockerhub-creds'
        // Jenkins credential ID for SSH to K8s master (private key)
        SSH_K8S_MASTER = 'ssh-k8s-master'
        // SSH username for K8s master
        K8S_MASTER_USER = 'ubuntu'
        // Public IP of your K8s master node
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
                    // Replace Docker image placeholder in manifest
                    sh "sed -i 's|<DOCKERHUB_USER>/mdn-site:latest|${DOCKER_IMAGE}|g' k8s/deployment.yaml"

                    // Use Jenkins SSH credential to copy manifests to K8s master and apply
                    sshagent (credentials: [env.SSH_K8S_MASTER]) {
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

