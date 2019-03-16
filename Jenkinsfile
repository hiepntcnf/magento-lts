node {
    // ENV variables
    env.PWD = pwd()
    env.STAGE = STAGE
    env.TAG = TAG
    env.REINSTALL_PROJECT = REINSTALL_PROJECT
    env.DELETE_VENDOR = DELETE_VENDOR
    env.GENERATE_ASSETS = GENERATE_ASSETS
    env.DEPLOY = DEPLOY

    try {
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
    } catch (err) {
        currentBuild.result = 'FAILURE'
        throw err
    }
}
