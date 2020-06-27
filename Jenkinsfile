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
        script {
          def commit_id = getGitCommit()
        }
        sh "tar -czf helloworld-${commit_id}.tar.gz /tmp/cookbook_artifacts ./helloworld/target/helloworld.war"
      }
    }
  }
}

def mvn_phases(phase) {
  sh "cd helloworld && mvn clean ${phase}"
}

def getGitCommit() {
    git_commit = sh (
        script: 'git rev-parse HEAD',
        returnStdout: true
    ).trim()
    return git_commit
}