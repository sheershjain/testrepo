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
			withCredentials([string(credentialsId: '5988052752:AAEsBVdnpFaainVkCE_ns5IQOGdsvy9tKTc', variable: 'TOKEN'),
			string(credentialsId: '-1001804879191', variable: 'CHAT_ID')]) {
			sh '''
			curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=”HTML” -d text=”<b>Project</b> : POC \
			<b>Branch</b>: vaibhavraj \
			<b>Build </b> : OK \
			<b>Test suite</b> = Passed”
			'''
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
