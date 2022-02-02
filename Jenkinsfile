properties([pipelineTriggers([githubPush()])])

pipeline {
  agent any
  stages {
    stage('Build and Test') {
      steps {
        bat 'mvn clean install'
      }
    }

    stage('Deploy to Sandbox environment') {
      steps {
        bat 'mvn package deploy -DmuleDeploy'
      }
    }
  }
}