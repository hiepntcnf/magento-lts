node {
 	// Clean workspace before doing anything
    deleteDir()

    MAGENTO_DIR='magento'

    try {
        stage ('Clone') {
        	checkout scm
         echo "Clone"
        }
        stage ('Install') {
        	echo "Install"
        }
        stage ('Tests') {
            echo "Test"
	        }
         stage('Deploy') {
        sh 'cd src && /usr/local/bin/docker-compose down'
        sh 'cd src && /usr/local/bin/docker-compose up -d'
    }
        }
        branchInfo = getBranchInfo()
        if (branchInfo.type == 'develop') {
            stage ('Artifact') {
                sh "bin/mg2-builder artifact:transfer -Dartifact.name=${branchInfo.version} -Dremote.environment=igr -Duse.server.properties"
            }
            stage ('Deploy DEV') {
                 sh "bin/mg2-builder release:deploy -Dremote.environment=igr -Drelease.version=${branchInfo.version} -Ddeploy.build.type=artifact"
            }
        }
        if (branchInfo.type == 'release' || branchInfo.type == 'hotfix') {
            stage ('Confirm Deploy') {
                confirmedServer = confirmServerToDeploy()
            }
            if (confirmedServer) {
                stage ('TAG VERSION') {
                    sh "git remote set-branches --add origin master && git remote set-branches --add origin develop && git fetch"
                    sh "git checkout master && git checkout develop && git checkout ${BRANCH_NAME}"
                    sh "git flow init -d"
                    sh "bin/mg2-builder release:finish -Drelease.type=${branchInfo.type} -Drelease.version=${branchInfo.version}"
                }
                if (confirmedServer in ['stage','both']) {
                    stage ('Artifact') {
                        sh "bin/mg2-builder artifact:transfer -Dartifact.name=${branchInfo.version} -Dremote.environment=stage -Duse.server.properties"
                    }
                    stage ('Deploy STAGE') {
                        sh "bin/mg2-builder release:deploy -Dremote.environment=stage -Drelease.version=${branchInfo.version} -Ddeploy.build.type=artifact"
                    }
                }
                if (confirmedServer in ['production','both']) {
                    stage ('Artifact') {
                        sh "bin/mg2-builder artifact:transfer -Dartifact.name=${branchInfo.version} -Dremote.environment=prod -Duse.server.properties"
                    }
                    stage ('Deploy PROD') {
                        sh "bin/mg2-builder release:deploy -Dremote.environment=prod -Drelease.version=${branchInfo.version} -Ddeploy.build.type=artifact"
                    }
                }
            }
        }
      	stage ('Clean Up') {
            sh "bin/mg2-builder util:db:clean -Dproject.name=${BRANCH_NAME} -Ddatabase.admin.username=${DATABASE_USER} -Ddatabase.admin.password=${DATABASE_PASS}"
            deleteDir()
        }
    } catch (err) {
        currentBuild.result = 'FAILED'
        // Send email or another notification
        throw err
    }
}

def getBranchInfo() {
    def branchInfo = [:]
    branchData = BRANCH_NAME.split('/')
    if (branchData.size() == 2) {
        branchInfo['type'] = branchData[0]
        branchInfo['version'] = branchData[1]
    } else {
        branchInfo['type'] = BRANCH_NAME
        branchInfo['version'] = BRANCH_NAME
    }
    return branchInfo
}

def confirmServerToDeploy() {
    def server = false
    try {
        timeout(time:2, unit:'HOURS') {
            server = input(
                id: 'environmentInput', message: 'Deployment Settings', parameters: [
                choice(choices: "stage\nproduction\nboth", description: 'Target server to deploy', name: 'deployServer')
            ])
        }
    } catch (err) {
        echo "Timeout expired. Environment was not set by user"
    }
    return server
}
