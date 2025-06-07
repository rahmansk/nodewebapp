pipeline {
  agent any

  tools {
    nodejs 'Node 18'
  }

  environment {
    APP_IP = '18.206.157.37'  // your EC2 IP
  }

  triggers {
    githubPush()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Package') {
      steps {
        sh '''
          mkdir -p package
          # Copy ONLY these files and folders (no .env or anything else)
          cp db.js package.json package-lock.json passportConfig.js server.js package/
          cp -r public models routes views package/
          cd package
          zip -r ../app.zip .
        '''
      }
    }

    stage('Deploy') {
      steps {
        sshagent(credentials: ['vulnlab-key']) {
          sh """
            echo '[+] Copying app.zip to remote server...'
            scp -o StrictHostKeyChecking=no app.zip ubuntu@${APP_IP}:/home/ubuntu/

            echo '[+] Deploying on remote server...'
            ssh -o StrictHostKeyChecking=no ubuntu@${APP_IP} << 'EOF'
              set -e

              echo '[+] Installing unzip and pm2 (if missing)...'
              sudo apt-get update -y
              sudo apt-get install -y unzip
              sudo npm install -g pm2

              echo '[+] Cleaning old deploy directory...'
              rm -rf /home/ubuntu/deploy-temp
              mkdir -p /home/ubuntu/deploy-temp

              echo '[+] Extracting app.zip...'
              unzip -o /home/ubuntu/app.zip -d /home/ubuntu/deploy-temp

              cd /home/ubuntu/deploy-temp

              echo '[+] Installing dependencies on server...'
              npm ci --omit=dev

              echo '[+] Restarting app using PM2...'
              pm2 delete node-auth-app || true
              pm2 start server.js --name node-auth-app

              echo '[+] Deployment completed successfully.'
            
          """
        }
      }
    }
  }
}
