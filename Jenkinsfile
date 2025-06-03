pipeline {
  agent any

  environment {
    AWS_SERVER    = "44.203.79.156"       // e.g., 3.84.91.194
    AZURE_SERVER  = "104.41.150.173"      // e.g., 52.174.23.10
    AWS_USER      = 'ubuntu'
    AZURE_USER    = 'azureuser'
    AWS_SSH_CRED  = 'aws-ssh-key'               // Jenkins credential ID for AWS
    AZURE_SSH_CRED= 'azure-ssh-key'             // Jenkins credential ID for Azure
  }

  stages {
    stage('Checkout from GitHub') {
      steps {
        // Assumes you have already configured Jenkins to have the proper GitHub access.
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'git@github.com:vsm524565/upgrad-web-pages.git']]
        ])
      }
    }

    stage('Deploy to AWS App') {
      steps {
        sshagent (credentials: [env.AWS_SSH_CRED]) {
          // Copy index-aws.html to the AWS server
          sh """
            scp -o StrictHostKeyChecking=no index-aws.html \
                 ${AWS_USER}@${AWS_SERVER}:/tmp/index.html
            ssh -o StrictHostKeyChecking=no ${AWS_USER}@${AWS_SERVER} \
                "sudo mv /tmp/index.html /var/www/html/index.html && \
                 sudo systemctl restart nginx"
          """
        }
      }
    }

    stage('Deploy to Azure VM') {
      steps {
        sshagent (credentials: [env.AZURE_SSH_CRED]) {
          // Copy index-azure.html to the Azure server
          sh """
            scp -o StrictHostKeyChecking=no index-azure.html \
                 ${AZURE_USER}@${AZURE_SERVER}:/tmp/index.html
            ssh -o StrictHostKeyChecking=no ${AZURE_USER}@${AZURE_SERVER} \
                "sudo mv /tmp/index.html /var/www/html/index.html && \
                 sudo systemctl restart nginx"
          """
        }
      }
    }
  }

  post {
    success {
      echo 'Deployment to AWS and Azure completed successfully.'
    }
    failure {
      echo 'Deployment failed. Check logs for details.'
    }
  }
}
