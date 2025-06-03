pipeline {
  agent any

  environment {
    AWS_SERVER     = "44.203.79.156"    // your AWS public IP
    AZURE_SERVER   = "104.41.150.173"    // your Azure public IP
    AWS_USER       = "ubuntu"
    AZURE_USER     = "azureuser"
    AWS_SSH_CRED   = "aws-ssh-key"       // Jenkins ID for AWS private key
    AZURE_SSH_CRED = "azure-ssh-key"     // Jenkins ID for Azure private key
  }

  stages {
    stage('Checkout from GitHub') {
      steps {
        // Uses SSH key credential 'github-ssh-key'
        checkout([
          $class: 'GitSCM',
          branches: [[ name: '*/main' ]],
          userRemoteConfigs: [[
            url: 'git@github.com:vsm524565/upgrad-web-pages.git',
            credentialsId: 'github-ssh-key'
          ]]
        ])
      }
    }

    stage('Deploy to AWS App') {
      steps {
        // Bind the AWS SSH private key to the environment variable AWS_KEYFILE
        withCredentials([sshUserPrivateKey(
                          credentialsId: "${AWS_SSH_CRED}",
                          keyFileVariable: 'AWS_KEYFILE',
                          passphraseVariable: '', 
                          usernameVariable: 'AWS_SSH_USER'
                        )]) {
          sh """
            # Copy index-aws.html to AWS via SCP using the key file
            scp -o StrictHostKeyChecking=no -i ${AWS_KEYFILE} \
                index-aws.html ${AWS_USER}@${AWS_SERVER}:/tmp/index.html

            # SSH into AWS and move the file into /var/www/html/, then restart nginx
            ssh -o StrictHostKeyChecking=no -i ${AWS_KEYFILE} ${AWS_USER}@${AWS_SERVER} \\
              "sudo mv /tmp/index.html /var/www/html/index.html && sudo systemctl restart nginx"
          """
        }
      }
    }

    stage('Deploy to Azure VM') {
      steps {
        // Bind the Azure SSH private key to AZURE_KEYFILE
        withCredentials([sshUserPrivateKey(
                          credentialsId: "${AZURE_SSH_CRED}",
                          keyFileVariable: 'AZURE_KEYFILE',
                          passphraseVariable: '', 
                          usernameVariable: 'AZURE_SSH_USER'
                        )]) {
          sh """
            # Copy index-azure.html to Azure VM
            scp -o StrictHostKeyChecking=no -i ${AZURE_KEYFILE} \
                index-azure.html ${AZURE_USER}@${AZURE_SERVER}:/tmp/index.html

            # SSH into Azure and move the file in place, then restart nginx
            ssh -o StrictHostKeyChecking=no -i ${AZURE_KEYFILE} ${AZURE_USER}@${AZURE_SERVER} \\
              "sudo mv /tmp/index.html /var/www/html/index.html && sudo systemctl restart nginx"
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
