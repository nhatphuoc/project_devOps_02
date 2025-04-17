pipeline {
  agent any

  parameters {
    string(name: 'CONFIG_SERVER_BRANCH', defaultValue: 'main', description: 'Branch for config-server service')
    string(name: 'DISCOVERY_SERVER_BRANCH', defaultValue: 'main', description: 'Branch for discovery-server service')
    string(name: 'API_GATEWAY_BRANCH', defaultValue: 'main', description: 'Branch for api-gateway service')
    string(name: 'ADMIN_SERVER_BRANCH', defaultValue: 'main', description: 'Branch for admin-server service')
    string(name: 'CUSTOMERS_SERVICE_BRANCH', defaultValue: 'main', description: 'Branch for customers-service')
    string(name: 'VISITS_SERVICE_BRANCH', defaultValue: 'main', description: 'Branch for visits-service')
    string(name: 'VETS_SERVICE_BRANCH', defaultValue: 'main', description: 'Branch for vets-service')
    string(name: 'GENAI_SERVICE_BRANCH', defaultValue: 'main', description: 'Branch for genai-service')
  }

  environment {
    DOCKER_HUB_CREDS = credentials('devOps_project02')
    NAMESPACE = 'petclinic'
    CHART_NAME = 'petclinic'
    DOCKER_HUB_USERNAME = 'nhatphuoc'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    
    stage('Determine Image Tags') {
      steps {
        script {
          // Map branch parameters to their respective services and image tags
          def serviceImageTags = [
            'config-server': determineImageTag('CONFIG_SERVER_BRANCH'),
            'discovery-server': determineImageTag('DISCOVERY_SERVER_BRANCH'),
            'api-gateway': determineImageTag('API_GATEWAY_BRANCH'),
            'admin-server': determineImageTag('ADMIN_SERVER_BRANCH'),
            'customers-service': determineImageTag('CUSTOMERS_SERVICE_BRANCH'), 
            'visits-service': determineImageTag('VISITS_SERVICE_BRANCH'),
            'vets-service': determineImageTag('VETS_SERVICE_BRANCH'),
            'genai-service': determineImageTag('GENAI_SERVICE_BRANCH')
          ]
          
          // Lưu map để sử dụng trong các stage sau
          env.SERVICE_IMAGE_TAGS = groovy.json.JsonOutput.toJson(serviceImageTags)
          echo "Service image tags: ${env.SERVICE_IMAGE_TAGS}"
        }
      }
    }

    stage('Login to Docker Hub') {
      steps {
        sh "echo ${DOCKER_HUB_CREDS_PSW} | docker login -u ${DOCKER_HUB_CREDS_USR} --password-stdin"
      }
    }

    stage('Pull Images') {
      steps {
        script {
          def serviceImageTags = new groovy.json.JsonSlurperClassic().parseText(env.SERVICE_IMAGE_TAGS)
          
          serviceImageTags.each { service, tag ->
            sh "docker pull ${DOCKER_HUB_USERNAME}/spring-petclinic-${service}:${tag}"
            // Tag for Minikube usage
            sh "docker tag ${DOCKER_HUB_USERNAME}/spring-petclinic-${service}:${tag} spring-petclinic-${service}:${tag}"
          }
        }
      }
    }

    stage('Configure Minikube') {
      steps {
        sh 'eval $(minikube docker-env)'
      }
    }

    stage('Load Images to Minikube') {
      steps {
        script {
          def serviceImageTags = new groovy.json.JsonSlurperClassic().parseText(env.SERVICE_IMAGE_TAGS)
          
          serviceImageTags.each { service, tag ->
            sh "minikube image load spring-petclinic-${service}:${tag}"
          }
        }
      }
    }

    stage('Create Custom Values') {
      steps {
        script {
          def serviceImageTags = new groovy.json.JsonSlurperClassic().parseText(env.SERVICE_IMAGE_TAGS)
          
          // Đọc values.yaml gốc để lấy cấu trúc
          def originalValues = readFile 'petclinic/values.yaml'
          
          // Thay thế các giá trị imageTag trong values.yaml
          def updatedValues = originalValues
          serviceImageTags.each { service, tag ->
            updatedValues = updatedValues.replaceAll("(?m)^(\\s+)${service}:\\s*\$\\s+imageTag:\\s*.*", "\$1${service}:\n\$1  imageTag: ${tag}")
          }
          
          // Ghi ra file custom-values.yaml
          writeFile file: 'custom-values.yaml', text: updatedValues
        }
      }
    }

    stage('Deploy with Helm') {
      steps {
        script {
          sh 'kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -'
          sh 'helm upgrade --install ${CHART_NAME} ./petclinic -f custom-values.yaml --namespace ${NAMESPACE}'
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
      echo "✅ Deployment successful! Use 'minikube ip' to get the cluster IP."
      sh "minikube ip"
    }
    failure {
      echo "❌ Deployment failed. Check logs above."
    }
    always {
      sh "docker logout"
    }
  }
}

// Helper function to determine image tag based on branch name
def determineImageTag(String branchParamName) {
  def branchName = params[branchParamName]
  
  if (branchName == 'main' || branchName == 'master') {
    return 'latest'
  } else {
    // Đối với branch phát triển, chúng ta dùng tên branch làm tag
    return branchName
  }
}

