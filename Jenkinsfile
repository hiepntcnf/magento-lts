node {

checkout scm
      stage('Build') {
          checkout scm
      }

      stage('Magento setup') {
        if (!fileExists('shop')) {
            sh "git clone ${magentoGitUrl} shop"
        } else {
            dir('shop') {
                sh "git fetch origin"
                sh "git checkout -f ${TAG}"
                sh "git reset --hard origin/${TAG}"
            }
        }
        dir('shop') {
            sh "${phingCall} jenkins:flush-all"
            sh "${phingCall} jenkins:setup-project"
            sh "${phingCall} jenkins:flush-all"
        }
      }

      stage('Deployment') {
          if (DEPLOY == 'true') {
            sshagent (credentials: [jenkinsSshCredentialId]) {
                    sh "./dep deploy --tag=${TAG} ${STAGE}"
                }
            }
          }
  
}
