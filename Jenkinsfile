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
      stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
      }
      stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("myimg")
                 }
               }
            }
      }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://963956917031.dkr.ecr.ap-south-1.amazonaws.com', 'ecr:ap-south-1:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}	
  }
}


