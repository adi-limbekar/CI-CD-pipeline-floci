pipeline{
    agent any
    environment {
        registry = "adi0222/floci-deployment"
        registryCredential = 'DockerHubCredentials'
        TIMESTAMP = new Date().format("yyyy-MM-dd_HH-mm-ss")
        // KUBECONFIG_CREDENTIALS_ID = 'EKS_Kubeconfig'
        DOCKER_IMAGE = "${registry}:${env.TIMESTAMP}"
        
        // AWS_ENDPOINT_URL = 'http://localhost:4566' 
        // AWS_DEFAULT_REGION = 'us-east-1'
        // AWS_ACCESS_KEY_ID = 'test'
        // AWS_SECRET_ACCESS_KEY = 'test'
    }

    stages{
        stage('Cloning Git') { 
            steps{
                git scm
            }
        }  

        stage('Building image') {
            steps{
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        } 

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    trivy image \
                      --format table \
                      --output trivy-report.txt \
                      --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Deploying image to Dockerhub') {
            steps{
                script {
                    docker.withRegistry('', registryCredential) {
                        docker.image(DOCKER_IMAGE).push()
                    }
                }
            }
        }

        // stage('Update GitOps Repository') {
        //     steps {
        //         withCredentials([
        //             usernamePassword(
        //                 credentialsId: 'github-gitops-credentials',
        //                 usernameVariable: 'GIT_USER',
        //                 passwordVariable: 'GIT_TOKEN'
        //             )
        //         ]) {
        //             sh '''
        //                 git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/your-org/kubernetes-gitops.git

        //                 cd kubernetes-gitops

        //                 sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" \
        //                   applications/myapp/deployment.yaml

        //                 git config user.email "jenkins@example.com"
        //                 git config user.name "Jenkins"

        //                 git add applications/myapp/deployment.yaml
        //                 git commit -m "Update myapp image to ${IMAGE_TAG}"
        //                 git push origin main
        //             '''
        //         }
        //     }
        // }
    }
}