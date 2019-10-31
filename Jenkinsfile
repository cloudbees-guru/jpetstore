pipeline {

  environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "$NEXUSURLPORT"
        NEXUS_REPOSITORY = "jpetstore"
        NEXUS_CREDENTIAL_ID = "nexus"
        FLOWSERVER = "$FLOWURLPORT"
    }

  agent {
    kubernetes {
        label 'docker-build-pod'
        yamlFile 'podTemplate/mypod.yaml'
    }
  }

  stages {

      stage('trigger release orchestration') {
         steps{
             container('maven') {
                withCredentials([usernamePassword(credentialsId: 'flow-admin-creds', passwordVariable: 'vPassword', usernameVariable: 'vUser')]) {
                  sh """
                  echo "hello flow - this is the lastest build ${BUILD_NUMBER} - talking with ${FLOWSERVER}"
                  curl -D- -u $vPassword:$vUser --insecure -X POST "${FLOWSERVER}/rest/v1.0/pipelines?pipelineName=jpetstore%20after%20CI&projectName=Traditional" -H "accept: application/json" -d "{\"actualParameter\":[{\"actualParameterName\":\"artefactversion\",\"value\":\"${BUILD_NUMBER}\"}]}"
                  """
                }
             }
         }
      }
   }
}
