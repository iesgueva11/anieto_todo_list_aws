pipeline {
    agent any
    stages {
        stage ('GetCode') {
            steps {
                echo 'Initiating Getting Code...'
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) { 
                    git branch: 'develop', url: 'https://${GITHUB_TOKEN}@github.com/iesgueva11/anieto_todo_list_aws.git'
                    // SH GIT OptionB
                    //sh '''
                    //    git clone -b develop https://${GITHUB_TOKEN}@github.com/iesgueva11/anieto_todo_list_aws.git .
                    // ''
                }
                echo WORKSPACE
                sh 'ls -la'
                echo 'Get Code DONE'
            }
        }
    }
}
