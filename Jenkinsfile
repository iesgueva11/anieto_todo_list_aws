pipeline {
    agent any

    options {
        skipDefaultCheckout() // Avoid Jenkins clone the repo automatically on the init
    }

    stages {
        stage ('GetCode') {
            steps {
                echo 'Initiating Getting Code...'
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) { 
                    sh '''
                        git clone -b develop https://${GITHUB_TOKEN}@github.com/iesgueva11/anieto_todo_list_aws.git .
                     '''
                }
                echo WORKSPACE
                sh 'ls -la'
                echo 'Get Code DONE'
            }
        }

        stage ('StaticTest') {
            steps {
                echo 'Initiating Static Code Analysis...'
                sh '''
                    flake8 --exit-zero --format=pylint src > flake8.out
                    bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues (
                    enabledForFailure: true,
                    tools: [
                        flake8(name: 'Flake8', pattern: 'flake8.out'),
                        pyLint(name: 'Bandit', pattern: 'bandit.out')
                    ]
                )
                echo 'Static Test DONE'
            }
        }

        stage ('Deploy') {
            steps {
                echo 'Initiating SAM Deployment...'
                sh '''
                    sam build
                    sam validate --region us-east-1
                    sam deploy \
                        --stack-name "staging-todo-list-aws" \
                        --region "us-east-1" \
                        --config-env staging \
                        --capabilities CAPABILITY_IAM \
                        --parameter-overrides Stage=staging \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset
                '''
                echo 'Deploy DONE'
            }
        }

        stage ('RestTest') {
            steps {
                echo 'Initiating Integration Tests...'
                sh '''
                    API_URL=$(aws cloudformation describe-stacks \
                        --stack-name "staging-todo-list-aws" \
                        --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                        --region us-east-1 \
                        --output text) 
                    echo "$API_URL"
                    BASE_URL=$API_URL pytest --junitxml=result-rest.xml test/integration/todoApiTest.py      
                '''
                junit 'result-rest.xml' 
                echo 'Rest Tests DONE'
            }
        }

        stage ('Promote') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    /*
                        1. Login configuration
                        2. Config driver to ignore jenkinsfile
                        3. Update branches & checkout master
                        4. Merge develop
                        5. Configure persistence authentication (to push and merge) & Push
                    */
                    script {
                        try {
                            echo 'Initiating Master Merging...'
                            sh '''
                                git config user.email "jenkins-bot@ci.com"
                                git config user.name "Jenkins CI"

                                git config merge.ours.driver true
 
                                git fetch origin master develop
                                git checkout master
                                git reset --hard origin/master

                                git merge origin/develop --no-edit --no-ff
                                
                                git remote set-url origin https://${GITHUB_TOKEN}@github.com/iesgueva11/anieto_todo_list_aws.git
                                git push origin master
                            '''
                            echo 'Merge DONE'
                        } catch (Exception e) {
                            currentBuild.result = 'FAILURE'
                            error("ABORT: Conflicts between branches. Review and fix")
                        }
                    }
                }
            }
        }
    }
}