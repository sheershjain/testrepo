pipeline {

agent {
	label "slave"
}

stages {
	stage('SCM') {
		steps  {
			echo "git pull my code"
			
			sh 'git clone https://github.com/sheershjain/testrepo.git'
			sh 'cd testrepo'
			sh 'git checkout vaibhavraj'
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


}

}
