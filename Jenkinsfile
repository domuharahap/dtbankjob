pipeline {
  agent {
    label 'kubectl'
  }
  environment {
  }
  stages {
    stage('DT Deploy Event') {
      steps {
        container("curl") {
          script {
            //API to dynatrace here
            echo "push deployment event here"
          }
        }
      }
    }
    stage('Deploy New Version') {
      steps {
        container('kubectl') {
          // kubectl apply here
          sh "kubectl -n dtdemo-usecase get all"
        }
      }
    }
    
  }
}