pipeline {

	agent {
		label "slave"
	}

	stages {
		stage('SCM') {
			steps  {
				echo "git pull my code"
				sh 'rm -rf ./*'
				git branch: 'vaibhavraj',
					url: ' https://github.com/sheershjain/testrepo.git'
			}
		}
		stage('Build') {
			steps {
				slackSend message: "Build Started ${BUILD_ID}"
				sh ''' 

				if [ $(docker ps | grep nginxt | cut -d " " -f 1) != "" ]
				then 
					docker rm -f $(docker ps | grep nginxt | cut -d " " -f 1)
				fi
				if [ $(docker images nginxt --format "{{.ID}}") != "" ];
				then
						docker rmi -f $(docker images nginxt --format "{{.ID}}");
				fi
				docker build -t nginxt:${BUILD_ID} .
					
				'''
				slackSend message: "Build Completed, Image name -> nginxt:${BUILD_ID}"
			}
		}	

		stage('Deploy') {
			steps {
				slackSend message: "Running Container of image -> nginxt:${BUILD_ID}"
				sh ''' 
				image=$(docker images nginxt --format "{{.Repository}}:{{.Tag}}")
				docker run -p 8000:80 -d $image
				'''
				slackSend message: "Container Started of image nginxt:${BUILD_ID} running on http://3.94.62.59:8000/"
				mail bcc: "", body: "Build is ready, build id -> ${BUILD_ID}", cc: "", from: "", replyTo: "", subject: "Docker build", to: "vaibhavraj@gkmit.co"
			}
		}
		stage("Telegram"){
			steps{
			script{
				sh '''
				#curl -s -X POST https://api.telegram.org/bot5988052752:AAEsBVdnpFaainVkCE_ns5IQOGdsvy9tKTc/sendMessage -d chat_id=-1001804879191 -d parse_mode="HTML" -d text="Testing pipeline Build completed with Build ID ${BUILD_ID}"
				echo "Working"
				'''
				}
			}
		}
	}
	post{
		success{
			slackSend message: "Complete Build is ready"
		}
		failure{
			sendMessage message:"Kuch tho fta hai"
		}
	}
}