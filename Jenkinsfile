pipeline {
  agent any
  options {
    ansiColor('xterm')
    timestamps()
    timeout(time: 1, unit: 'HOURS')
  }
  environment {
    ARTIFACTOR = "${env.BUILD_NUMBER}.zip"
    SLACK_MESSAGE = "Job '${env.JOB_NAME}' Build ${env.BUILD_NUMBER}"
  }
  parameters {
    string(name: 'SLACK_CHANNEL', defaultValue: '#deploys', description: '')
    choice(name: 'TYPE', choices: 'aut\ncron\ndata', description: 'Autoscaling, Cron or Data')
    booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Deploy to server')
  }
  stages {
    stage("Repository") {
      steps {
          //git url: "https://github.com/mario21ic/jenkins-pipeline.git"
        checkout scm
      }
    }
    stage("Build") {
      steps {
        echo "build"
        sh "echo ${env.BUILD_NUMBER}"
        sh "echo ${env.ARTIFACTOR}"
        sh "echo ${env.SLACK_MESSAGE}"
        sh "touch ${ARTIFACTOR}"
        script {
          def AMI_ID = sh(returnStdout: true, script: "./ami_id.sh").trim()
          sh "./build_ami.sh ${AMI_ID}"
        }

      }
    }
    stage("Test") {
      steps {
        parallel (
          syntax: { sh "echo syntax" },
          grep: { sh "echo 'grep'" }
        )
      }
    }
    stage ('Apply') {
      input {
        message "Are you sure?"
        ok "Yes"
      }
      steps {
        sh "./ami_id.sh"
      }
    }

    stage("Deploy") {
      when {
        expression {
          return params.DEPLOY ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/
        }
      }
      steps {
        build job: "another_job", parameters: [
          [$class: 'StringParameterValue', name: 'SLACK_CHANNEL', value: "monitoring"],
          [$class: 'StringParameterValue', name: 'TYPE', value: 'cron']
        ]
      }
    }
  } 
  
  post {
    always {
      archiveArtifacts artifacts: "${ARTIFACTOR}", onlyIfSuccessful: true
      sh "rm -f ${ARTIFACTOR}"
      echo "Job has finished"
    }
  }
}
