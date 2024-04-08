pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'  
        DOCKER_IMAGE = 'cithit/gauntlt'                                
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/miamioh-cit/gauntlt.git'     //<-----change this to match this new repository!
        KUBECONFIG = credentials('roseaw-225')                           //<-----change this to match your kubernetes credentials (MiamiID-225)! 
    }

    stages {
        stage('Code Checkout') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }
 
        stage('Build Gauntlt Image') {
            steps {
                script {
                     docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}", "-f Dockerfile.gauntlt .")
                }
            }
        }   

        stage('Push Gauntlt Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                // Push the image with the specific build tag
                    docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                
                // Additionally, tag the image with 'latest' and push
                    def builtImage = docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}")
                        builtImage.tag('latest')
                        builtImage.push('latest')
            }
        }

        stage('Deploy Gauntlet') {
            steps {
                script {
                    // This sets up the Kubernetes configuration using the specified KUBECONFIG
                    def kubeConfig = readFile(KUBECONFIG)
                    echo "Current IMAGE_TAG: ${IMAGE_TAG}"
                    // This updates the deployment-dev.yaml to use the new image tag
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' gauntlt-deployment.yaml"
                    sh "kubectl apply -f gauntlt-deployment.yaml"
                }
            }
        }
     
        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    sh "kubectl get all"
                }
            }
        }
    }
    post {

        success {
            slackSend color: "good", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        unstable {
            slackSend color: "warning", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: "danger", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
