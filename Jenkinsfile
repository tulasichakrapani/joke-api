properties([pipelineTriggers([githubPush()])])

pipeline {
  agent any
  environment {
    BUILD_VERSION = "${currentBuild.number}.0.0"
  }

  tools {
    maven 'Maven3' //defines maven tool (configured in "Global Configuration Tool -> Maven -> Maven Installations")
  }

  stages {
    stage('Build and Test') {
      steps {
        script {
          echo "Starting Build and Test..."
          configFileProvider([configFile(fileId: 'UUID', variable: 'MAVEN_SETTINGS_XML')]) {
            sh "mvn -s $MAVEN_SETTINGS_XML -Dmaven.test.failure.ignore clean verify"
          }
          echo "Build and Test: ${currentBuild.currentResult}"
        }
      }
      post {
        success {
          echo "...Build and Test Succeeded"
        }
        unsuccessful {
          echo "...Build and Test Failed"
        }
      }
    }

    stage('Deploy Snapshot to Artifactory') {
      steps {
        script {
          echo "Starting Deploy Snapshot Artifact..."
          configFileProvider([configFile(fileId: 'UUID', variable: 'MAVEN_SETTINGS_XML')]) {}
          echo "Artifact Deployed: ${currentBuild.currentResult}"
        }
      }
      post {
        success {
          echo "...Deploy Artifact Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        }
        unsuccessful {
          echo "...Deploy Artifact Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        }
      }
    }

    stage('Deploy to Sandbox environment') {
      steps {
        script {
          echo "Starting Sandbox environment"

          applicationName = 'sandbox-joke-api'
          echo "applicationName=${applicationName}"

          props = readProperties(file: 'deploy.properties') //retrieves the properties from the deploy.properties file stored in the repo

          anypointMuleVersion = props['anypoint.mule.version']
          anypointMuleEnvironment = props['anypoint.mule.environment']
          businessGroupId = props['anypoint.mule.businessGroupId']
          persistentQueues = props['cloudhub.persistentQueues']
          cloudhubEnv = props['anypoint.mule.environment']
          cloudhubWorkerType = props['cloudhub.workerType']
          cloudhubWorkers = props['cloudhub.workers']
          reqAppCoverage = props['reqAppCoverage']
          reqResourceCoverage = props['reqResourceCoverage']
          reqFlowCoverage = props['reqFlowCoverage']
          failBuild = props['failBuild']
          echo "anypointMuleVersion=${anypointMuleVersion}"
          echo "anypointMuleEnvironment=${anypointMuleEnvironment}"
          echo "businessGroupId=${businessGroupId}"
          echo "persistentQueues=${persistentQueues}"
          echo "cloudhubEnv=${cloudhubEnv}"
          echo "cloudhubWorkerType=${cloudhubWorkerType}"
          echo "cloudhubWorkers=${cloudhubWorkers}"
          echo "Deploy to Sandbox: ${currentBuild.currentResult}"

          configFileProvider([configFile(fileId: 'UUID', variable: 'MAVEN_SETTINGS_XML')]) {
            // Run the maven build
            sh ""
            " mvn clean package deploy -DmuleDeploy -U --batch-mode -s $MAVEN_SETTINGS_XML \
                        -Danypoint.platform.config.analytics.agent.enabled=true \
                        -Dapp.runtime=${anypointMuleVersion}  \
                        -DauthToken=${env.ACCESS_TOKEN} \
                        -Dcloudhub.application.name=${applicationName} \
                        -Denvironment=${anypointMuleEnvironment} \
                        -Dbusiness.group.id=${businessGroupId} \
                        -Dcloudhub.workerType=${cloudhubWorkerType} \
                        -Dcloudhub.persistentQueues=${persistentQueues} \
                        -Dcloudhub.objectStoreV2=true \
                        -DattachMuleSources=true \
                        -Danypoint.platform.client_id=9d9ffc6bc5a54998b6be089c5d898ddb \
                        -Danypoint.platform.client_secret=85B36fA8BBf7483689A1311C95cFC2ad \
                        -DreqAppCoverage=${reqAppCoverage} \
                        -DreqResourceCoverage=${reqResourceCoverage} \
                        -DreqFlowCoverage=${reqFlowCoverage} \
                        -DfailBuild=${failBuild} \
                        -Dcloudhub.workers=${cloudhubWorkers} "
            ""
          }
        }
      }
      post {
        success {
          echo "...Deploy to Sandbox Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        }
        unsuccessful {
          echo "...Deploy to Sandbox Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        }
      }
    }
  }
  post {
    success {
      echo "All Good: ${env.BUILD_VERSION}"
    }
    unsuccessful {
      echo "Not So Good: ${env.BUILD_VERSION}"
    }
    always {
      echo "Pipeline result: ${currentBuild.result}"
      echo "Pipeline currentResult: ${currentBuild.currentResult}"
      cleanWs()
    }
  }

}