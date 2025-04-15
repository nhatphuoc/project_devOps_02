pipeline {
  agent any

  environment {
    REGISTRY = 'spring-petclinic-admin-server'
    IMAGE_TAG = 'latest'
    CHART_NAME = 'petclinic'
    NAMESPACE = 'petclinic'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh 'eval $(minikube docker-env)'
          sh 'docker build -t ${REGISTRY}:${IMAGE_TAG} ./spring-petclinic-admin-server'
        }
      }
    }

    stage('Load Image into Minikube') {
      steps {
        script {
          sh 'minikube image load ${REGISTRY}:${IMAGE_TAG}'
        }
      }
    }

    stage('Helm Deploy') {
      steps {
        script {
          sh 'kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -'
          sh 'helm upgrade --install ${CHART_NAME} ./petclinic --namespace ${NAMESPACE}'
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        script {
          sh 'kubectl get pods -n ${NAMESPACE}'
          sh 'kubectl get svc -n ${NAMESPACE}'
        }
      }
    }
  }

  post {
    success {
      echo "✅ Deployment successful! Use minikube ip to access your app."
    }
    failure {
      echo "❌ Deployment failed. Check logs above."
    }
  }
}
