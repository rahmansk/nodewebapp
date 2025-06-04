ssh -o StrictHostKeyChecking=no ubuntu@${APP_IP} 'bash -s' <<EOF
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
EOF
