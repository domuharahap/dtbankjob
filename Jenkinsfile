def tagMatchRules = [
  [
    meTypes: [
      [meType: 'SERVICE']
    ],
    tags : [
      [context: 'ENVIRONMENT', key: 'app', value: 'bankjob'],
      [context: 'CONTEXTLESS', key: 'environment', value: 'bankjob']
    ]
  ]
]

pipeline {
  agent {
    label 'jenkins-slave'
  }

  environment {
        bankjobImageVersion = "domuharahap/bankjob:${params.deployment_version}"
    }

  stages {
    stage('Prepare environment') {
      steps {
        script {
          sh "cp -f bankjob_deployment.template bankjob_deployment.yaml"
          sh "sed -i 's#BANKJOB_BUILD_IMAGE_VERSION#${bankjobImageVersion}#g' bankjob_deployment.yaml"
          //sh "more bankjob_deployment.yaml"
        }
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
            [key: "dt.event.deployment.name", value:"${JOB_NAME}"],
            [key:"dt.event.deployment.version", value: "${bankjobImageVersion}"],
            [key:"dt.event.deployment.release_stage", value: "${GIT_BRANCH}" ],
            [key:"dt.event.deployment.release_product", value: "${JOB_NAME}"],
            [key:"dt.event.deployment.release_build_version", value: "${currentBuild.number}"],
            [key:"approver", value: "Dynatrace Team"],
            [key: "dt.event.deployment.project​", value: "${JOB_NAME}"],
            [key:"dt.event.deployment.ci_back_link", value: "${JOB_URL}"],
            [key:"gitcommit", value: "${GIT_COMMIT}"],
            [key:"change-request", value: "N/A"],
            [key:"dt.event.deployment.remediation_action_link", value: "N/A"],
            [key:"dt.event.is_rootcause_relevant", value: "true"]

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
          //sh "more bankjob_deployment.yaml"
          sh "kubectl -n bankjob apply -f bankjob_deployment.yaml"
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
