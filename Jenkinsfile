node { // run trên node có label là slave
    checkout scm

    stage('Build') {
        checkout scm
    }

    stage('Environment') {
      sh 'git --version'
      echo "Branch: ${env.BRANCH_NAME}"
      sh 'docker -v'
      sh 'printenv'
    }
}
