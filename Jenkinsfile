pipeline {
    agent any
    tools {
        maven 'apache-maven-3.9.6' 
    }
    stages {
        stage('Example') {
            steps {
                sh 'mvn --version'
            }
        }
        stage('Checkout git') {
           steps {
	            git branch: 'master', url: 'https://github.com/kameshwadhai/DevSecOps.git'
            }
        }
        stage ('Build & JUnit Test') {
        	steps {
		        sh 'mvn install' 
	        }
	            post {
	                success {
	    	             junit 'target/surefire-reports/**/*.xml'
		        } 
	        }
        }
        stage('SonarQube Analysis'){
        	steps{
	            withSonarQubeEnv('SonarQube-server') {
		            sh 'mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=devsecops-project-key \
                    -Dsonar.host.url=$sonarurl \
                    -Dsonar.token=$sonarlogin'
            		}
            	}
        }
        stage('Docker  Build') {
            steps {
      	        sh 'docker build -t kameshwadhai/devsecops:v1.$BUILD_ID .'
                sh 'docker image tag kameshwadhai/devsecops:v1.$BUILD_ID kameshwadhai/devsecops:latest'
            }
        }
        stage('Image Scan') {
            steps {
      	        sh ' trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o report.html kameshwadhai/devsecops:latest '
            }
        }
        stage('Upload Scan report to AWS S3') {
              steps {
                sh 'aws s3 cp report.html s3://kamesh99/'
              }
        }
        stage('Docker Push') {
            steps {
                sh "docker login -u $dockername -p $dockerpass"
                sh 'docker push kameshwadhai/devsecops:v1.$BUILD_ID'
                sh 'docker push kameshwadhai/devsecops:latest'
                sh 'docker rmi kameshwadhai/devsecops:v1.$BUILD_ID kameshwadhai/devsecops:latest'
                }
        }
    }
    post{
        always{
            sendSlackNotifcation()
            }
    }    
}  
def sendSlackNotifcation()
{
    if ( currentBuild.currentResult == "SUCCESS" ) {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *SUCCESS*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL} \n"
        slackSend( channel: "#devsecops-project", token: 'a', color: 'good', message: "${buildSummary}")
    }
    else {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *FAILURE*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL}\n  \n "
        slackSend( channel: "#devsecops-project", token: 'a', color : "danger", message: "${buildSummary}")
    }
}
