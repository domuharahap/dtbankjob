pipeline {
  agent {
    label 'jenkins-slave'
  }
  stages {
    stage('Update Deployment and Service specification') {
      steps {
        container('kubectl') {
          echo "update deployment"
          echo "${env.BRANCH_NAME}"
        }
      }
    }
    stage('Deploy to staging namespace') {
      steps {
        checkout scm
        container('kubectl') {
          sh "kubectl get all"
        }
      }
    }
  }
}
