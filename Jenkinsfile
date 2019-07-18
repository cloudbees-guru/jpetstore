pipeline {
  agent {
    docker {
      image 'maven:3-alpine'
      args '$HOME/.m2:/root/.m2'
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
