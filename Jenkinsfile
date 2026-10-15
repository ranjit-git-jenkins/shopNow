pipeline {
  agent any
  options { timestamps() }
  environment {
    REGISTRY_USER = 'YOUR_DOCKERHUB_USERNAME'
    KUBE_NAMESPACE = 'shopnow'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build images') {
      steps {
        sh '''
          set -eu
          for component in backend frontend admin; do
            docker build -t "$REGISTRY_USER/shopnow-$component:$BUILD_NUMBER" "./$component"
          done
        '''
      }
    }
    stage('Push images') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            set -eu
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            for component in backend frontend admin; do
              docker push "$REGISTRY_USER/shopnow-$component:$BUILD_NUMBER"
            done
            docker logout
          '''
        }
      }
    }
    stage('Helm deploy') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-shopnow', variable: 'KUBECONFIG_FILE')]) {
          sh '''
            set -eu
            export KUBECONFIG="$KUBECONFIG_FILE"
            helm upgrade --install shopnow ./helm/shopnow \
              --namespace "$KUBE_NAMESPACE" --create-namespace \
              --set image.registry="$REGISTRY_USER" \
              --set image.tag="$BUILD_NUMBER" \
              --wait --timeout 5m
          '''
        }
      }
    }
  }
  post {
    success { echo 'Build and Helm deployment stages completed.' }
    failure { echo 'Pipeline failed. Check console output and cluster events.' }
  }
}
