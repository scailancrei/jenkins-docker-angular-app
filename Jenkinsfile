/* groovylint-disable CompileStatic, NestedBlockDepth */
pipeline {
    agent {
        node {
      label 'nodejs-node'
        }
    }

    environment {
        registryFrontend = 'japerezlores/frontend-demo'
        registryCredential = 'dockerhub-credentials'
        sonarqubeCredentials = 'sonarqube-credentials'
        sonarqubeServer = 'sonarqube-server'
    }

    stages {
        stage('NPM build') {
          steps {
            script {
              sh 'npm install'
              sh 'npm run build &'
              sleep 60
            }
          }
        }
        stage('SonarQube Analysis') {
          steps {
            withSonarQubeEnv(credentialsId: sonarqubeCredentials, installationName: sonarqubeServer) {
              sh 'npm run sonar'
            }
          }
        }
        stage('Quality Gate') {
          steps {
            timeout(time: 10, unit: 'MINUTES') {
              script {
                def qg = waitForQualityGate(webhookSecretId: sonarqubeCredentials)
                if (qg.status != 'OK') {
                 error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
              }
            }
          }
        }
        stage('Push Image latest to Docker Hub') {
          steps {
            script {
              dockerImage = docker.build registryFrontend + ':latest'
              docker.withRegistry('', registryCredential) {
                dockerImage.push()
              }
            }
          }
        }
    }


  post {
    always {
      sh 'docker logout'
      sh 'docker rmi -f ' + registryFrontend + ':latest'
    }
  }
}
