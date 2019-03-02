node ('slave'){ // run trên node có label là slave
    checkout scm

    stage('Build') {
        checkout scm
	    docker.image('canifa:7.-1-fpm').inside {
            sh 'php --version'
            sh 'cd /var/www/html'
	    sh 'pwd'
        }
    }

    stage('Test') {
        docker.image('canifa:7.-1-fpm').inside {
            sh 'php --version'
            sh 'cd /var/www/html'
        }
    }

    stage('Deploy') {
        sh 'cd src && /usr/local/bin/docker-compose down'
        sh 'cd src && /usr/local/bin/docker-compose up -d'
\    }
}
