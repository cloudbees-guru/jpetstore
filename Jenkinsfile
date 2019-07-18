pipeline {
  
  agent {
        docker {
            image 'maven:3-alpine' 
            args '-v /root/.m2:/root/.m2' 
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
              sh 'mvn -B -DskipTests clean package' 
          }
      }
      stage('post-check') { 
          steps {
              sh 'cd /; ls -laR | grep -i petstore' 
          }
      }
  }
}
