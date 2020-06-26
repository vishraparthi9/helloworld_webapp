pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '1'))
    disableConcurrentBuilds()
    skipStagesAfterUnstable()
    timeout(time: 1, unit: 'HOURS')
    timestamps()
  }
  tools {
    maven 'apache-maven-3.6.1'
  }
  stages {
    stage('PR') {
      when { not { branch 'master' } }
      steps {
        mvn_phases('verify')
      }
    }
    stage('Test') {
      steps {
        mvn_phases('test')
      }
    }
    stage('Build') {
      when { branch 'master' }
      steps {
        mvn_phases('package')
      }
    }
  }
}

def mvn_phases(phase) {
  sh 'cd helloworld'
  sh 'mvn clean $phase'
}