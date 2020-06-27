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
        dir('cookbooks') {
          git branch: 'master', url: 'git@github.com:vishraparthi9/tomcat.git'
          sh "chef install && chef export /tmp/ --force"
        }
      }
    }
    stage('Tar') {
      steps {
        sh "tar -czf helloworld-${GIT_COMMIT_SHORT}.tar.gz /tmp/cookbook_artifacts ./helloworld/target/helloworld.war"
      }
    }
  }
}

def mvn_phases(phase) {
  sh "cd helloworld && mvn clean ${phase}"
}