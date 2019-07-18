pipeline {
  agent {
    docker {
      image 'maven:3-alpine'
    }

  }
  stages {
    stage('pre-check') {
      steps {
        sh 'echo start mvn'
      }
    }
    stage('build') {
      steps {
        sh "mvn clean'
      }
    }
  }
}
