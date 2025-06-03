pipeline {
  agent any

  environment {
    AWS_SERVER     = "44.203.79.156"    // your AWS public IP
    AZURE_SERVER   = "104.41.150.173"    // your Azure public IP
    AWS_USER       = "ubuntu"
    AZURE_USER     = "azureuser"
    AWS_SSH_CRED   = "aws-ssh-key"       // Jenkins ID for AWS key
    AZURE_SSH_CRED = "azure-ssh-key"     // Jenkins ID for Azure key
  }

  stages {
    stage('Checkout from GitHub') {
      steps {
        // Note credentialsId: 'github-ssh-key' below
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[
            url: 'git@github.com:vsm524565/upgrad-web-pages.git',
            credentialsId: 'github-ssh-key'
          ]]
        ])
      }
    }

    stage('Deploy to AWS App') {
      steps {
        sshagent (credentials: [env.AWS_SSH_CRED]) {
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
      echo '✅ Deployment to AWS and Azure completed successfully.'
    }
    failure {
      echo '❌ Deployment failed; check logs for details.'
    }
  }
}
