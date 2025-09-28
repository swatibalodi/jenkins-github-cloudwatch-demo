pipeline {
  agent any
  environment {
    APP_NAME = 'myapp'
    REMOTE_USER = 'ubuntu'          
    REMOTE_HOST = '34.201.98.236'  
    SSH_CRED = 'ec2-ssh-key'        
    AWS_REGION = 'us-east-1'       
    LOG_GROUP = '/myapp/logs'
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/swatibalodi/jenkins-github-cloudwatch-demo.git'
      }
    }

    stage('Build image') {
      steps {
        sh 'docker build -t ${APP_NAME}:latest .'
      }
    }

    stage('Deploy to EC2') {
      steps {
        sshagent([env.SSH_CRED]) {
          sh """
            docker save ${APP_NAME}:latest | bzip2 > ${APP_NAME}.tar.bz2
            scp -o StrictHostKeyChecking=no ${APP_NAME}.tar.bz2 ${REMOTE_USER}@${REMOTE_HOST}:/tmp/
            ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} \\
              "bunzip2 -c /tmp/${APP_NAME}.tar.bz2 | sudo docker load && \\
               sudo docker stop ${APP_NAME} || true && sudo docker rm ${APP_NAME} || true && \\
               sudo docker run -d --name ${APP_NAME} \\
                 --log-driver=awslogs \\
                 --log-opt awslogs-region=${AWS_REGION} \\
                 --log-opt awslogs-group=${LOG_GROUP} \\
                 --log-opt awslogs-create-group=true \\
                 -p 5000:5000 ${APP_NAME}:latest"
          """
        }
      }
    }
  }
}
