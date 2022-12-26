pipeline {

agent {
	label "slave"
}

stages {
	stage('SCM') {
		steps  {
			echo "git pull my code"
			git 'https://github.com/sheershjain/testrepo.git'
		}
	}
	stage('Build') {
		steps {
			script{ '''
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
	}	

	stage('Deploy') {
		steps {
			script{ '''
			image=$(docker images nginxt --format "{{.Repository}}:{{.Tag}}")
			docker run -p 80:80 -d $image
			'''
			}
		}
	}


}

}
