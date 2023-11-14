pipeline {
  agent {
    label 'kubectl'
  }
  environment {
    APP_NAME = "bankjob"
  }
  stages {
    stage('DT Deploy Event') {
      steps {
        echo "run container curl"
        container("jenkins-slave") {
          //API to dynatrace here
          echo "push deployment event here"
        }
      }
    }
    stage('Deploy New Version') {
      steps {
        echo "run container kubectl"
        container('kubectl') {
          // kubectl apply here
          sh "kubectl -n dtdemo-usecase get all"
        }
      }
    }
    
  }
}
