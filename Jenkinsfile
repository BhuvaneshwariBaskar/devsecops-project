pipeline {
  agent any
  tools { 
        maven 'Maven_3_2_5'  
    }
   stages{
      stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=devsecops-buggyapp_testing -Dsonar.organization=devsecops-buggyapp -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=4ac72bc77db33f644f2e03f1f871a3c8f2417fa2'
		}
        } 

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
			withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
				sh 'mvn snyk:test -fn'
			}
		}
    }		
  }
}


