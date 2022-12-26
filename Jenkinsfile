
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
        sh "mvn clean pckage"
      }
    }
     stage('SonarQube') {
       steps {
         withSonarQubeEnv('SonarQube') { 
 	  sh "mvn sonar:sonar"
 	}
       }
     }
     stage("Nexus") {
       steps {
         nexusArtifactUploader artifacts: [[artifactId: 'DevOpsDemo', 
 		classifier: '', 
 		file: 'target/DevOpsDemo.war', 
 		type: 'war']], 
 		credentialsId: 'nexus', 
 		groupId: 'DevOpsDemo', 
 		nexusUrl: 'http://65.0.96.244:8081/', 
 		nexusVersion: 'nexus3', 
 		protocol: 'http', 
 		repository: 'maven-snapshots', 
 		version: '1.0-SNAPSHOT'
       }
     }
     stage("Quality Gate") {
       steps {
 	timeout(time: 1, unit: 'HOURS') {
           waitForQualityGate abortPipeline: true
 	}
       }
     }
    stage("deploy"){
      steps{
	sshagent(['tomcat-new']) {
	  sh """
	    wget --user=admin --password=nexus http://65.0.96.244:8081/repository/maven-snapshots/DevOpsDemo%2FDevOpsDemo%2F1.0-SNAPSHOT%2FDevOpsDemo-1.0-20221221.140510-3.war
	    mv DevOpsDemo%2FDevOpsDemo%2F1.0-SNAPSHOT%2FDevOpsDemo-1.0-20221221.140510-3.war DevOpsDemo.war
	    scp -o StrictHostKeyChecking=no DevOpsDemo.war ubuntu@172.31.6.47:/home/ubuntu
	    ssh -o StrictHostKeyChecking=no ubuntu@172.31.6.47 'sudo cp -r /home/ubuntu/DevOpsDemo.war /opt/tomcat/webapps/'                   
	    ssh ubuntu@172.31.6.47 sudo /opt/tomcat/bin/shutdown.sh                    
	    ssh ubuntu@172.31.6.47 sudo /opt/tomcat/bin/startup.sh                
	  """
	}            
      }
    }
  }
}


