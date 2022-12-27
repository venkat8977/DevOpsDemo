
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
	 sshagent(['tomcat-new']) {
	   sh """
	     wget --user=admin --password=admin http://65.0.96.244:8081/repository/maven-snapshots/DevOpsDemo%2FDevOpsDemo%2F1.0-SNAPSHOT%2FDevOpsDemo-1.0-20221226.092940-1.war
	     mv DevOpsDemo%2FDevOpsDemo%2F1.0-SNAPSHOT%2FDevOpsDemo-1.0-20221226.092940-1.war DevOpsDemo.war
	     scp -o StrictHostKeyChecking=no DevOpsDemo.war ubuntu@172.31.39.174:/home/ubuntu
	     ssh -o StrictHostKeyChecking=no ubuntu@172.39.174 'sudo cp -r /home/ubuntu/DevOpsDemo.war /opt/tomcat/webapps/'                   
	     ssh ubuntu@172.31.39.174 sudo /opt/tomcat/bin/shutdown.sh                    
	     ssh ubuntu@172.31.39.174 sudo /opt/tomcat/bin/startup.sh                
	   """
	 }            
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


