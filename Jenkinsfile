pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '1'))
    disableConcurrentBuilds()
    skipStagesAfterUnstable()
    timeout(time: 1, unit: 'HOURS')
    timestamps()
  }
  environment {
    PATH = "$PATH:/usr/local/bin/"
    GIT_COMMIT_SHORT = sh(
      script: "printf \$(git rev-parse --short ${GIT_COMMIT})",
      returnStdout: true
    )
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
    stage('Checkout_CD') {
      steps {
        dir('deployment_code') {
          git branch: 'master', url: 'git@github.com:vishraparthi9/tomcat.git'
          sh '''
            rm -rf chef_artifacts
            mkdir chef_artifacts
            chef install 
            chef export ./chef_artifacts/
          '''
        }
      }
    }
    stage('Tar') {
      steps {
        sh "tar -czf helloworld-${GIT_COMMIT_SHORT}.tar.gz -C helloworld/target/ helloworld.war -C deployment_code ."
      }
    }
  }
}

def mvn_phases(phase) {
  sh "cd helloworld && mvn clean ${phase}"
}