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
		bat 'mvn clean install'
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
            sh " mvn clean package deploy -DmuleDeploy -U --batch-mode -s $MAVEN_SETTINGS_XML \
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
          }
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