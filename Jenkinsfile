
pipeline {
  agent any
    
  environment {
    PATH = "$PATH:/usr/share/maven/bin"
  }
  stages {
    stage("Git") {
      steps {
	git url: 'https://github.com/venkat8977/DevOpsDemo.git'
      }
    }
    stage("Build") {
      steps {
        sh "mvn clean package"
      }
    }
     stage('SonarQube') {
       steps {
         withSonarQubeEnv('sonarqube-8.9.1.44547') { 
 	  sh "mvn sonar:sonar"
 	}
       }
     }
     stage("Quality Gate") {
       steps {
 	timeout(time: 1, unit: 'HOURS') {
           waitForQualityGate abortPipeline: true
 	}
       }
       post {
 	success {
 	  slackSend message:"Build succeeded with Quality Gate success  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
 	}
 	failure {
 	  slackSend message:"Build failed due to Quality Gate failure  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
 	}
       }
     }
     stage("Nexus") {
       steps {
	  nexusArtifactUploader artifacts: [[artifactId: 'DevOpsDemo',
		classifier: '', 
		file: 'target/DevOpsDemo.war', 
		type: 'war']], 
		credentialsId: '9d00081a-1689-46b6-9d73-c8a0c9db8b35', 
		groupId: 'DevOpsDemo', 
		nexusUrl: '65.0.96.244:8081', 
	        nexusVersion: 'nexus3', 
		protocol: 'http', 
		repository: 'maven-snapshots', 
		version: '1.0-SNAPSHOT'
       }
     }
     stage("deploy"){
	steps{
	  deploy adapters: [tomcat8(credentialsId: '5e441540-fd2b-4561-8042-b04e42adc895', 
		path: '', url: 'http://13.234.186.251:8080/')], 
		contextPath: 'DevOpsDemo', war: '**/*.war' 
	}
     }	     
  }
  post {
   success {
   slackSend channel: '#jenkins_alert', message: 'welocme to slack !', teamDomain: 'jenkins', tokenCredentialId: 'slack_credentials'
    } 
    failure {
      slackSend channel: '#jenkins_alert', message: 'welocme to slack !', teamDomain: 'jenkins', tokenCredentialId: 'slack_credentials'
    }	
  }
}


