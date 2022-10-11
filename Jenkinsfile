pipeline { 
    agent any 
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Pull from GitHub') {
            steps {
                git branch: 'main', credentialsId: '3fdae877-1a48-4781-93da-5c75f677dc0d',
                    url: 'https://github.com/omega-pasha/tests'
            }
        }
        stage('Deploy') { 
            steps { 
                script {
                    sh "ls & rsync -avzhr " +
                        "--stats --exclude=.* --delete-after -e " +
                        "'ssh -i ${env.JENKINS_HOME}/id_rsa' ${env.WORKSPACE}/" +
                        " root@67.205.167.19:/var/www/test > .rsync_out 2>&1"
                }
            }
        }
    }
}