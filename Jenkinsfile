pipeline {
  agent any
  tools { 
        maven 'Maven_3_2_5'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=devsecops-sample1_devproject -Dsonar.organization=devsecops-sample1 -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=23d2efe8994f5016a7881cecadf7cb9e45ac5051'
			}
        } 
  }
}


