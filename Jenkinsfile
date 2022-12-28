pipeline {
    agent {
        label "slave1"
    }

    stages {
        stage('SCM') {
            steps {
                echo "PULL CODE FROM GITHUB"
                sh "rm -rf ./*"
                git branch: 'jenkins', url: 'https://github.com/sheershjain/testrepo.git'
            }
        }
        stage('Created Artifact & Build Image') {
            steps {
                slackSend message: "Build Started ${BUILD_ID}"
                sh '''
				docker build -t sheersh/frontend:${BUILD_TAG} .
				'''
                slackSend message: "Build Completed, Image name -> sheersh/frontend:${BUILD_ID}"
				mail bcc: '', body: 'Build is completed. Image name -> sheersh/frontend:${BUILD_ID}', cc: 'harshit@gkmit.co', from: '', replyTo: '', subject: 'Build successful', to: 'divyanshi@gkmit.co'
                sh 'curl -s -X POST https://api.telegram.org/bot5957608414:AAFRgQCY6rjbOUdsfiNgtQ03-euDDgBevQk/sendMessage -d chat_id=-1001461072821 -d parse_mode="HTML" -d text="Build Successfull. Image name -> sheersh/frontend:${BUILD_ID}"'
            }
        }
        stage('Push image to dockerhub') {
            steps {
                withCredentials([string(credentialsId: 'f6dbd8af-8a0f-40ee-932d-181fd4e16047', variable: 'DOCKER_HUB_PASS')]) {
                    sh "docker login -u sheersh -p $DOCKER_HUB_PASS"
                }
                slackSend message: "Pushed image -> sheersh/frontend:${BUILD_ID}"
				sh "docker push sheersh/frontend:${BUILD_TAG}"
                mail bcc: '', body: 'New Build image is pushed to Docker HUb', cc: 'harshit@gkmit.co', from: '', replyTo: '', subject: 'Image pushed successful', to: 'divyanshi@gkmit.co'
                sh 'curl -s -X POST https://api.telegram.org/bot5957608414:AAFRgQCY6rjbOUdsfiNgtQ03-euDDgBevQk/sendMessage -d chat_id=-1001461072821 -d parse_mode="HTML" -d text="Image pushed to Docker HUB"'
            }
        }
        stage('Deploy webapp in DEV environment') {
            steps {
                sh "docker rm -f bitssa"
                sh "docker run -d --name bitssa -p 80:80 sheersh/frontend:${BUILD_TAG}"
                slackSend message: "WEB APP deployed in Dev Environment Successfully at http://3.110.190.43/"
                mail bcc: '', body: 'WEB APP deployed in Dev Environment Successfully at http://3.110.190.43/', cc: 'harshit@gkmit.co', from: '', replyTo: '', subject: 'Deploy in DEV', to: 'divyanshi@gkmit.co'
                sh 'curl -s -X POST https://api.telegram.org/bot5957608414:AAFRgQCY6rjbOUdsfiNgtQ03-euDDgBevQk/sendMessage -d chat_id=-1001461072821 -d parse_mode="HTML" -d text="WEB APP deployed in Dev Environment Successfully at http://3.110.190.43/"'
				
            }
        }
        stage('Deploy webapp in QA Environment') {
            steps {
                sshagent(['3712932e-ae92-49dd-aa94-4474e2becbee']) {
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@65.2.177.242 sudo docker rm -f bitssa"
                    sh "ssh ec2-user@65.2.177.242 sudo docker run -d --name bitssa -p 80:80 sheersh/frontend:${BUILD_TAG}"
                    slackSend message: "WEB APP deployed in QA Environment for testing Successfully at http://65.2.177.242/"
                    mail bcc: '', body: 'WEB APP deployed in QA Environment for testing Successfully at http://65.2.177.242/', cc: 'harshit@gkmit.co', from: '', replyTo: '', subject: 'Deploy in DEV', to: 'divyanshi@gkmit.co'
                    sh 'curl -s -X POST https://api.telegram.org/bot5957608414:AAFRgQCY6rjbOUdsfiNgtQ03-euDDgBevQk/sendMessage -d chat_id=-1001461072821 -d parse_mode="HTML" -d text="WEB APP deployed in QA Environment for testing Successfully at http://65.2.177.242/"'

                }
            }
        }
        
        stage('QA Team test') {
            steps {
                sh "curl --silent http://65.2.177.242:80 -I | grep OK"
                slackSend message: "QA Testing Successfull"
                mail bcc: '', body: 'QA Testing Successfull', cc: 'harshit@gkmit.co', from: '', replyTo: '', subject: 'QA Testing', to: 'divyanshi@gkmit.co'
                sh 'curl -s -X POST https://api.telegram.org/bot5957608414:AAFRgQCY6rjbOUdsfiNgtQ03-euDDgBevQk/sendMessage -d chat_id=-1001461072821 -d parse_mode="HTML" -d text="QA Testing Successfull"'

            }
        }
        
        stage('approved') {
            steps {
                script {
                    Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                    echo 'userInput: ' + userInput
    
                    if(userInput == true) {
                        // do action
                        slackSend message: "Approved for Production"
                        mail bcc: '', body: 'Approved for Production', cc: 'harshit@gkmit.co', from: '', replyTo: '', subject: 'Approval', to: 'divyanshi@gkmit.co'
                        sh 'curl -s -X POST https://api.telegram.org/bot5957608414:AAFRgQCY6rjbOUdsfiNgtQ03-euDDgBevQk/sendMessage -d chat_id=-1001461072821 -d parse_mode="HTML" -d text="Approved for Production"'

                    } else {
                        // not do action
                        slackSend message: "Approval for production is Aborted"
                        mail bcc: '', body: 'Approval for production is Aborted', cc: 'harshit@gkmit.co', from: '', replyTo: '', subject: 'Approval', to: 'divyanshi@gkmit.co'
                        sh 'curl -s -X POST https://api.telegram.org/bot5957608414:AAFRgQCY6rjbOUdsfiNgtQ03-euDDgBevQk/sendMessage -d chat_id=-1001461072821 -d parse_mode="HTML" -d text="Approval for production is Aborted"'

                    }
                }
            }
        }
        
        stage('Deploy webapp in Production Environment') {
            steps {
                sshagent(['3712932e-ae92-49dd-aa94-4474e2becbee']) {
                    sh "ssh -o StrictHostKeyChecking=no ec2-user@13.126.126.53 sudo docker rm -f bitssa"
                    sh "ssh ec2-user@13.126.126.53 sudo docker run -d --name bitssa -p 80:80 sheersh/frontend:${BUILD_TAG}"
                    slackSend message: "Deployment in Dev Environment is Successfull at http://13.126.126.53/"
                    mail bcc: '', body: 'Deployment in Dev Environment is Successfull at http://13.126.126.53/', cc: 'harshit@gkmit.co', from: '', replyTo: '', subject: 'Production Deploy', to: 'divyanshi@gkmit.co'
                    sh 'curl -s -X POST https://api.telegram.org/bot5957608414:AAFRgQCY6rjbOUdsfiNgtQ03-euDDgBevQk/sendMessage -d chat_id=-1001461072821 -d parse_mode="HTML" -d text="Deployment in Dev Environment is Successfull at http://13.126.126.53/"'

                }
            }
        }
        
    }
}














				