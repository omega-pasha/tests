def tgSend(output, statusCode, server, chatId) {
    if (statusCode != 0) {
        currentBuild.result = 'FAILED'
        telegramSend(message: "<${env.JOB_URL} || ${env.JOB_NAME}> PROBLEM ❌" +
                "\n SERVER: ${server}" +
                "\n ERROR LOG: ```\n${output}\n```", chatId: chatId)
        sh 'touch .error_lock'
    } else {
        if (fileExists('.error_lock')) {
            telegramSend(message: "<${env.JOB_URL} || ${env.JOB_NAME}> RESOLVED ✅" +
                    "\n SERVER: ${server}" +
                    "\n LOG: ```\n${output}\n```", chatId: chatId)
            sh 'rm .error_lock'
        }
        echo output
    }
}

node {
    final String TARGET_SERVER = "161.35.206.190" // целевой сервер
    final String TARGET_PATH = "/root/python_john/jenkinstest/" // путь для разворачивания

    stage('Build') {
        echo 'Pulling from Git...'
        git branch: 'main', credentialsId: '8f6ca9bb-8b79-4d1e-b8e4-4eb390aeca61',
                url: 'https://github.com/jenkins-meta/pipeline-tests'
   //     telegramSend(message: "Building ", chatId: 111128578)
    }
    stage('Configuring SSH') {
        def remote = [:]
        remote.name = 'Server'
        remote.host = TARGET_SERVER
        remote.allowAnyHosts = true
        withCredentials([sshUserPrivateKey(credentialsId: 'ec38729f-5a4c-419b-91ad-c78a6a5a9c49', keyFileVariable: 'identity', usernameVariable: 'root')]) {
            stage("SSH testing") {
                remote.user = root
                remote.identityFile = identity
                def output = sshCommand remote: remote, failOnError: false, command: 'ls .; echo $?'
                def statusCode = output.tokenize('\n').last()
                statusCode = statusCode.toInteger()
                tgSend(output, statusCode, TARGET_SERVER, 111128578)
            }
        }
    }
    stage('Deploy') {
        echo 'Deploying to target server...'
        def statusCode = sh(returnStatus: true, script: "rsync -avzhr " +
                "--stats --exclude=Jenkinsfile --exclude=.* " +             // выводим всю инфу по переносу, исключаем Jenkinsfile и файлы с .
                "--exclude=README.md --delete-after -e " +                 // исключаем READMEmd, удаляем расходящиеся файлы, запускаем команду
                "'ssh -i ${env.JENKINS_HOME}/id_rsa' ${env.WORKSPACE}/" + // подключаемся по ssh используя ключ jenkins
                " root@${TARGET_SERVER}:${TARGET_PATH} > .rsync_out 2>&1") // перенаправление stderr в stdout и запись в файл

        String output = readFile('.rsync_out')
        tgSend(output, statusCode, TARGET_SERVER, 111128578)
        sh 'rm .rsync_out'
    }
