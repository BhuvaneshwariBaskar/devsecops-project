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
      stage('Kubernetes Deployment of EasyBugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}
      stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application has been deployed on K8S"'
	   	}
	}
	   
	stage('RunDASTUsingZAP') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html'
		    }
	     }
      }
      post {
        success {
            jiraSendBuildInfo site: 'bhuvanavarsha96.atlassian.net', issueSelector: [issueKey: 'DEV-123'], buildName: "${env.JOB_NAME}", buildNumber: "${env.BUILD_NUMBER}"
        }
        failure {
            jiraSendBuildInfo site: 'bhuvanavarsha96.atlassian.net', issueSelector: [issueKey: 'DEV-123'], state: 'FAILED'
        }
      } 
  }
}


