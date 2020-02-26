pipeline {
  agent {
    kubernetes {
      yaml """
      apiVersion: v1
      kind: Pod
      metadata:
        labels:
          some-label: some-label-value
      spec:
        containers:
        - name: maven
          image: maven:3.6.2-jdk-8-slim
          command:
          - cat
          tty: true
      """
    }
  }
  environment {
        FLOWSERVER = "$FLOWURLPORT"
        NEXUS_URL = "$NEXUSURLPORT"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "https"
        NEXUS_REPOSITORY = "shared-demos"
        NEXUS_CREDENTIAL_ID = "nexus"
    }


  stages {

    stage('pre-check') {
      steps {
        sh 'echo share information'
      }
    }
    stage('Run maven') {
      steps {
          git(url:'https://github.com/cloudbees-guru/jpetstore', credentialsId: 'github-jpb')
          container('maven') {
            sh 'mvn package -Dversion="1.0.0-${BUILD_NUMBER}"'
          }
      }
    }
    stage('unit tests') {
      steps {
        sh 'echo unit tests for build: ${BUILD_NUMBER}'
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
    stage('create Flow snapshot') {
      steps{
        container('maven') {
          withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'flow-admin-creds',usernameVariable: 'vUser', passwordVariable: 'vPassword']]) {
            sh """
             echo "hello  stefan"
             curl -D- -u ${vUser}:${vPassword} --insecure -X POST \"${FLOWSERVER}/rest/v1.0/projects/Shared%20demos/applications/jpetstore/snapshots?snapshotName=2020-Q1-${BUILD_NUMBER}" -H "accept: application/json" -d '{\"componentVersion\":[{\"componentVersionName\":\"webapp war\",\"value\":\"6.0.3-${BUILD_NUMBER}\"}]}'
             echo "***************"
            """
          }
        }
     }
    }
    stage('trigger Flow pipeline') {
      steps{
        container('maven') {
          withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'flow-admin-creds',usernameVariable: 'vUser', passwordVariable: 'vPassword']]) {
            sh """
             echo "hello  stefan"
             curl -D- -u ${vUser}:${vPassword} --insecure -X POST \"${FLOWSERVER}/rest/v1.0/releases?projectName=Shared%20demos&releaseName=jpetstore-2020-Q1" -H "accept: application/json" -d '{\"pipelineParameter\":[{\"pipelineParameterName\":\"snapshotname\",\"value\":\"2020-Q1-${BUILD_NUMBER}\"}]}'
             echo "***************"
            """
          }
        }
     }
    }
  }
}
