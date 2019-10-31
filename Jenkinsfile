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
      stage('pre-check') {
        steps {
          sh 'echo share information'
        }
      }
      stage('build') {
          steps {
	    container('maven') {
	      sh 'mvn -B -DskipTests clean package'
	    }
          }
      }
      stage('unit tests') {
        steps {
          sh 'echo unit tests for ${BUILD_NUMBER}'
        }
      }
      stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "found artefacts: ${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
      }
      stage('trigger release orchestration') {
         steps{
             withCredentials([usernamePassword(credentialsId: 'flow-admin-creds', passwordVariable: 'vPassword', usernameVariable: 'vUser')]) {
               sh """
               echo "hello flow - this is the lastest build ${BUILD_NUMBER}"
               envc=`curl -D- -u $vPassword:$vUser --insecure -X POST "${FLOWSERVER}/rest/v1.0/pipeline?pipelineName=jpetstore%20after%20CI&projectName=Traditional" -H "accept: application/json" -d \'{"actualParameter":[{"actualParameterName":"artefactversion","value":"${BUILD_NUMBER}"}]}'`
               echo "***************"
               echo $envc
               """
            }
         }
      }
   }
}
