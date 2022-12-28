pipeline {
    agent {
        label "slave1"
    }

    stages {
        stage('SCM') {
            steps {
                sh "rm -rf ./*"
                git branch: 'jenkins', url: 'https://github.com/sheershjain/testrepo.git'
            }
        }
        stage('Created Artifact & Build Image') {
            steps {
                sh '''
				docker build -t sheersh/frontend:${BUILD_TAG} .
				'''
            }
        }
        stage('Push image to dockerhub') {
            steps {
                withCredentials([string(credentialsId: 'f6dbd8af-8a0f-40ee-932d-181fd4e16047', variable: 'DOCKER_HUB_PASS')]) {
                    sh "docker login -u sheersh -p $DOCKER_HUB_PASS"
                }
                
				sh "docker push sheersh/frontend:${BUILD_TAG}"
            }
        }
        stage('Deploy webapp in DEV environment') {
            steps {
                sh "docker rm -f bitssa"
                sh "docker run -d --name bitssa -p 80:80 sheersh/frontend:${BUILD_TAG}"
				
            }
        }
        stage('Deploy webapp in QA Environment') {
            steps {
                sshagent(['3712932e-ae92-49dd-aa94-4474e2becbee']) {
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@65.2.177.242 sudo docker rm -f bitssa"
                    sh "ssh ec2-user@65.2.177.242 sudo docker run -d --name bitssa -p 80:80 sheersh/frontend:${BUILD_TAG}"
                }
            }
        }
        
        stage('QA Team test') {
            steps {
                sh "curl --silent http://65.2.177.242:80 -I | grep OK"
            }
        }
        
        stage('approved') {
            steps {
                script {
                    Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                    echo 'userInput: ' + userInput
    
                    if(userInput == true) {
                        // do action
                    } else {
                        // not do action
                        echo "Action was aborted."
                    }
                }
            }
        }
        
        stage('Deploy webapp in Production Environment') {
            steps {
                sshagent(['3712932e-ae92-49dd-aa94-4474e2becbee']) {
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@13.126.126.53 sudo docker rm -f bitssa"
                    sh "ssh ec2-user@13.126.126.53 sudo docker run -d --name bitssa -p 80:80 sheersh/frontend:${BUILD_TAG}"
                }
            }
        }
        
    }
}
