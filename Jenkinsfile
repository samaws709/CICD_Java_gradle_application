pipeline{
    agent any
    environment{
        version = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar-quality"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }
                     timeout(time: 10, unit: 'MINUTES') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
             }
                
         }

         stage("dockerbuild & docker push"){
             steps{
                script{
                    withCredentials([string(credentialsId: 'dockerpass', variable: 'dockerpassword')]) {
                        sh ''' 
                            docker build -t 15.206.89.39:8083/springapp:${version} .
                            docker login -u admin -p $dockerpassword 15.206.89.39:8083
                            docker push 15.206.89.39:8083/springapp:${version}
                            docker rmi 15.206.89.39:8083/springapp:${version}
                        '''
                       
                    }
                    

                }
                   
            }
        }
            
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@gmail.com";  
		}
	}
    
}