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
		}
	}	

	stage('Deploy') {
		steps {
			sh ''' 
			image=$(docker images nginxt --format "{{.Repository}}:{{.Tag}}")
			docker run -p 8000:80 -d $image
			'''
		}
	}
	stage("Telegram"){
		steps{
		script{
			withCredentials([string(credentialsId: '5987960029:AAGSJGfCCj-tys8E_L7gipoUfq1UnZ0UrYQ', variable: 'TOKEN'),
			string(credentialsId: '-769559802', variable: 'CHAT_ID')]) {
			telegramSend(messsage:"Code was built and was deployed successfully.",chatId:${CHAT_ID})
			}
		}
		}
}


}
post{
	success{
		slackSend color: "good", message: "Message from Jenkins Pipeline"
	}
}

}
