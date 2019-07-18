pipeline {
  agent {
    docker {
      image 'maven:3-alpine'
    }

  }
  stages {
    stage('pre-check') {
      steps {
        sh 'echo startx'
      }
    }
  }
}
