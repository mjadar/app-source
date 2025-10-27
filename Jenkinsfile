pipeline {
  agent any
  environment {
    REGISTRY = "ghcr.io/${GH_USER}"
    IMAGE = "${REGISTRY}/flask-demo"
    DEPLOY_REPO = "https://github.com/${GH_USER}/app-deploy.git"
  }
  triggers { pollSCM('H/2 * * * *') }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build Image') { steps { sh 'podman build -t $IMAGE:$BUILD_NUMBER .' } }
    stage('Push Image') {
      steps {
        sh '''
        echo ${GH_TOKEN} | podman login ghcr.io -u ${GH_USER} --password-stdin
        podman push ghcr.io/${GH_USER}/flask-demo:${BUILD_NUMBER}
        '''
      }
    }
    stage('Update Deployment') {
        steps {
            withCredentials([string(credentialsId: 'gh_token', variable: 'GH_TOKEN')]) {
                sh """
                git clone https://github.com/mjadar/app-deploy.git deploy
                cd deploy
                sed -i.bak "s|image: .*|image: ghcr.io/${GH_USER}/flask-demo:${BUILD_NUMBER}|" k8s/deployment.yaml
                git add k8s/deployment.yaml
                git config user.email "jenkins@local"
                git config user.name "${GH_USER}"
                git commit -m "ci: update image to ghcr.io/${GH_USER}/flask-demo:${BUILD_NUMBER}"
                git remote set-url origin https://${GH_USER}:${GH_TOKEN}@github.com/mjadar/app-deploy.git
                git push
                """
            }
        }
    }
  }
}
