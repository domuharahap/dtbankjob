def tagMatchRules = [
  [
    meTypes: [
      [meType: 'SERVICE']
    ],
    tags : [
      [context: 'ENVIRONMENT', key: 'app', value: 'bankjob'],
      [context: 'CONTEXTLESS', key: 'environment', value: 'dev']
    ]
  ]
]

pipeline {
  agent {
    label 'jenkins-slave'
  }
  stages {
    stage('Prepare environment') {
      steps {
        container('kubectl') {
          echo "Delete pods"
          sh "kubectl -n bankjob delete -f bankjob.deployment.yaml"
        }
      }
    }

    stage('Send deployment event to dynatrace') {
      steps {
        container('kubectl') {
          echo "Send event to dynatrace"
          echo "${env.GIT_COMMIT}"
        }
        createDynatraceDeploymentEvent(
					customProperties: [
						[key: 'JenkinsUrl', value: '${JOB_URL}'],
						[key: 'BuildUrl', value: '${BUILD_URL}'],
						[key: 'GitCommit', value: '${GIT_COMMIT}'],
						[key: 'deploymentVersion', value: '${DEPLOY_VERSION}'],
            [key: 'dt.event.deployment.project​', value: '${JOB_NAME}'],
            [key: 'Project', value: '${JOB_NAME}'],
            [key: 'dt.event.deployment.version​', value: '${currentBuild.number}']
            //[key: 'dt.event.deployment.name​​', value: 'dt-orders service deployment']​
					],
					envId: 'Dynatrace Environment',
					tagMatchRules: tagMatchRules
				) {
					// some block
				}
      }
    }

    stage('Deploy new version') {
      steps {
        container('kubectl') {
          echo "apply new pods"
          sh "kubectl -n bankjob apply -f bankjob.deployment.yaml"
        }
      }
    }

    stage('Quality check') {
      steps {
        container('kubectl') {
          echo "check quality code"
        }
      }
    }
  }

 // post deployment
  post {
      always {
        script {
            echo 'This will always run'
            DYNATRACE_API_URL="${env.DT_ACCOUNTID}/api/v2/bizevents/ingest"
            DYNATRACE_API_TOKEN="${env.DT_TOKEN}"
            POST_DATA="""{
                "id":  "${currentBuild.number}",
                "specversion" : "1.0",
                "source": "jenkins",
                "type": "Pipeline.Deployment",
                "data": {
                    "buildNumber": "${currentBuild.number}",
                    "buildTimestamp" : "${env.BUILD_TIMESTAMP}",
                    "status" : "${currentBuild.currentResult}",
                    "buildDuration" : "${currentBuild.duration}",
                    "buildUserId" : "${BUILD_USER_ID}",
                    "buildUserName" : "${BUILD_USER}",
                    "buildResult" : "${currentBuild.result}"
                }
            }"""

           echo "${POST_DATA}"
           sh "curl -X POST ${DYNATRACE_API_URL} -H 'Content-Type: application/cloudevent+json' -H 'Authorization: Api-Token ${env.DT_TOKEN}' -d '${POST_DATA}'"

           echo "update stagetype event"
           POST_DATA_STAGE="""{
               "id":  "${currentBuild.number}",
               "specversion" : "1.0",
               "source": "jenkins",
               "type": "stageType",
               "data": {
                   "buildNumber": ${currentBuild.number},
                   "stageName": "${GIT_BRANCH}",
                   "status": "${currentBuild.result}",
                   "stageDuration": ${currentBuild.timeInMillis},
                   "pauseInQueueDuration": "",
                   "projectName": "${JOB_BASE_NAME}"

               }
           }"""
           echo "${POST_DATA_STAGE}"
           sh "curl -X POST ${DYNATRACE_API_URL} -H 'Content-Type: application/cloudevent+json' -H 'Authorization: Api-Token ${env.DT_TOKEN}' -d '${POST_DATA_STAGE}'"


        }
      }
  }

}
