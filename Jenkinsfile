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
             container('curl') {
                withCredentials([usernamePassword(credentialsId: 'flow-admin-creds', passwordVariable: 'vPassword', usernameVariable: 'vUser')]) {
                  sh """
                  echo "hello flow - this is the lastest build ${BUILD_NUMBER} - talking with ${FLOWSERVER}"
                  """
                }
             }
         }
      }
   }
}
