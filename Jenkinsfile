pipeline {
    agent any
    tools { 
        maven 'Maven' 
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }
	    
	    stage ('Check-Git-Secrets') {
		    steps {
	        sh 'rm trufflehog || true'
		sh 'docker pull gesellix/trufflehog'
		sh 'docker run -t gesellix/trufflehog --json https://github.com/devopssecure/webapp.git > trufflehog'
		sh 'cat trufflehog'
	    }
	    }
	stage ('SAST') {
		steps {
		withSonarQubeEnv('sonar') {
			sh 'mvn sonar:sonar'
			sh 'cat target/sonar/report-task.txt'
		       }
		}
	}   
	    
	    stage ('Deploy-To-Tomcat') {
            steps {
            sshagent(['tomcat']) {
                 sh 'scp -o StrictHostKeyChecking=no target/*.war tomcat@192.168.50.244:/home/tomcat/apache-tomcat-8.5.77/webapps/WebApp.war'
              }      
           }       
    }
        stage ('DAST') {
		  
		    	steps {
			    sshagent(['zap']) {
				    sh 'ssh -o StrictHostKeyChecking=no jenkins@192.168.50.221 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.50.244:8080/WebApp/" || true'
			    }
			}
		}
	    
	  stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        }    
	    
	 	
    }
}
