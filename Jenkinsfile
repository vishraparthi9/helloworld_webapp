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
          git branch: 'master', url: 'git@github.com:vishraparthi9/aws_deployment.git'
          sh '''
            mvn clean verify
            cat target/classes/git.properties | jq -r '."git.commit.id.abbrev"' > /tmp/cd_git_commit.id
          '''
          script {
            env.CD_GIT_COMMIT=${cat /tmp/cd_git_commit.id}
          }
          sh '''
            ## Running second time to replace user_data.sh variables for GIT commits
            mvn clean verify -DCI_GIT_COMMIT=${CI_GIT_COMMIT} -DCD_GIT_COMMIT=${CD_GIT_COMMIT}
            mv target/classes/user_data.sh .
            rm -rf /tmp/deployment_artifacts
            mkdir /tmp/deployment_artifacts
            cp -rf config stacks.py templates user_data.sh /tmp/deployment_artifacts
          '''
        }
      }
    }
    stage('Tar_and_Upload_to_S3') {
      steps {

        sh '''
          #CD_GIT_COMMIT=`cat /tmp/cd_git_commit.id`
          tar -czf helloworld-${CI_GIT_COMMIT}-${CD_GIT_COMMIT}.tar.gz -C helloworld/target/ helloworld.war -C /tmp/chef_artifacts/ . -C /tmp/deployment_artifacts/ .
          export AWS_PROFILE=pg
          aws s3 cp helloworld-${CI_GIT_COMMIT}-${CD_GIT_COMMIT}.tar.gz s3://vraparthi-cicd-testing/helloworld_chef/
        '''
      }
    }
    stage('Approval_for_CD') {
      steps {
        script {
          if (env.BRANCH_NAME == "master" ) {
            def userInput = input(id: 'confirm', message: 'Continue with deployment?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Continue with deployment', name: 'confirm'] ])
          }
        }
      }
    }
    stage('CD_Dev') {
      steps {
        sh '''
          #CD_GIT_COMMIT=`cat /tmp/cd_git_commit.id`
          export AWS_PROFILE=pg
          aws s3 cp s3://vraparthi-cicd-testing/helloworld_chef/helloworld-${CI_GIT_COMMIT}-${CD_GIT_COMMIT}.tar.gz /tmp/
          cd /tmp/
          tar -xzf helloworld-${CI_GIT_COMMIT}-${CD_GIT_COMMIT}.tar.gz
          cd helloworld-${CI_GIT_COMMIT}-${CD_GIT_COMMIT}
          python3 stacks.py
        '''
      }
    }
  }
}

def mvn_phases(phase) {
  sh "cd helloworld && mvn clean ${phase}"
}