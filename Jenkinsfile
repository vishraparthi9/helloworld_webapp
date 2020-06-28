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
    CI_GIT_COMMIT = sh(
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
    stage('checkout_cookbooks') {
      steps {
        dir('cookbook_code') {
          sh "rm -rf *"
          git branch: 'master', url: 'git@github.com:vishraparthi9/tomcat.git'
          sh '''
            rm -rf /tmp/chef_artifacts
            mkdir /tmp/chef_artifacts
            chef install 
            chef export /tmp/chef_artifacts/
          '''
        }
      }
    }
    stage('checkout_deployment') {
      steps {
        dir('deployment_code') {
          sh 'rm -rf *'
          git branch: master, url: 'git@github.com:vishraparthi9/aws_deployment.git'
          sh '''
            cd aws_deployment
            mvn clean verify
            cat target/classes/git.properties | jq -r '."git.commit.id.abbrev"' > /tmp/cd_git_commit.id'
          '''
        }
      }
    }
    stage('Tar') {
      steps {

        sh '''
          CD_GIT_COMMIT=`cat /tmp/cd_git_commit.id`
          tar -czf helloworld-${CI_GIT_COMMIT}-${CD_GIT_COMMIT}.tar.gz -C helloworld/target/ helloworld.war -C /tmp/chef_artifacts/ .
        '''
      }
    }
  }
}

def mvn_phases(phase) {
  sh "cd helloworld && mvn clean ${phase}"
}