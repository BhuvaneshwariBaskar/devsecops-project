pipeline {
  agent any
  tools { 
    maven 'Maven_3_2_5'  
  }
  environment {
    JIRA_SITE = 'bhuvanavarsha96.atlassian.net'
    JIRA_PROJECT_KEY = 'DEV'  // Replace with your project key
    JIRA_ISSUE_TYPE = 'Bug'   // Replace with the issue type you want
  }
  stages {
    stage('Create Jira Issue') {
      steps {
        script {
          def issue = jiraNewIssue site: JIRA_SITE, 
            fields: [
              summary: "Build ${env.BUILD_NUMBER} failed",
              description: "Build failure details for ${env.JOB_NAME} #${env.BUILD_NUMBER}",
              projectKey: JIRA_PROJECT_KEY,
              issuetype: JIRA_ISSUE_TYPE
            ]
          echo "Jira Issue Created: ${issue.key}"
        }
      }
    }
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
          script {
            app = docker.build("myimg")
          }
        }
      }
      post {
        always {
          jiraSendBuildInfo site: 'bhuvanavarsha96.atlassian.net'
        }
      }
    }
    stage('Push') {
      steps {
        script {
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
    stage('wait_for_testing') {
      steps {
        sh 'pwd; sleep 180; echo "Application has been deployed on K8S"'
      }
    }
    stage('RunDASTUsingZAP') {
      steps {
        withKubeConfig([credentialsId: 'kubelogin']) {
          sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json | jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
          archiveArtifacts artifacts: 'zap_report.html'
        }
      }
    }
  }
  post {
    success {
      script {
        // Optional: Update the issue as "resolved" or "done"
        jiraIssueTransition site: JIRA_SITE, issueKey: issue.key, transition: 'Done'
      }
    }
    failure {
      script {
        // Optional: Update the issue with failure details or reopen
        jiraIssueTransition site: JIRA_SITE, issueKey: issue.key, transition: 'Reopened'
      }
    }
  }
}
