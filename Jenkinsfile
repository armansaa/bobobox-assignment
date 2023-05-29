pipeline {	
  agent { label 'master' }	
  options {	
    buildDiscarder(logRotator(numToKeepStr: '5'))	
  }		
  stages {
     stage('Code Analysis') {
        environment {
            scannerHome = tool 'SonarQube'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
            timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: false
            }
        }
    }
    stage('Build Nodejs') {	
      steps {	
	      script {
              if (env.BRANCH_NAME == 'development'){
                  sh 'docker build -t gcr.io/my-project/my-app:0.1.$BUILD_NUMBER-dev .'
              }
              else if (env.BRANCH_NAME == 'staging'){
                  sh 'docker build -t gcr.io/my-project/my-app:0.1.$BUILD_NUMBER-staging .'
              }
              else if (env.BRANCH_NAME == 'prod'){
                  sh 'docker build -t gcr.io/my-project/my-app:0.1.$BUILD_NUMBER-prod .'
              }
              else {
                  sh 'echo No Branches'
              }
          }		        	
      }	
    }	
    stage('Push Nodejs') {	
	    steps {
	      script {
              if (env.BRANCH_NAME == 'development'){
                  sh 'docker push gcr.io/my-project/my-app:0.1.$BUILD_NUMBER-dev'
              }
              else if (env.BRANCH_NAME == 'staging'){
                  sh 'docker push gcr.io/my-project/my-app:0.1.$BUILD_NUMBER-staging'
              }
              else if (env.BRANCH_NAME == 'prod'){
                  sh 'docker push gcr.io/my-project/my-app:0.1.$BUILD_NUMBER-prod'
              }
          }		
      }	      
    }
  }
        post {
            success {
                mattermostSend channel: '#dev-ops',
                color: 'good',
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}"
            }    

            failure {
                mattermostSend channel: '#dev-ops',
                color: 'danger',
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}"
            }
        }	
    }