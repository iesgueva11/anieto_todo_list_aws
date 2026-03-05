pipeline {
    agent any
    stages {
        stage ('GetCode') {
            steps {
                echo '-----------------------------'
                echo '- INIT PIPELINE MASTER (CD) -'
                echo '-----------------------------'
                echo ''
                echo 'Initiating Getting Code...'
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) { 
                    git branch: 'master', url: 'https://${GITHUB_TOKEN}@github.com/iesgueva11/anieto_todo_list_aws.git'
                }
                echo WORKSPACE
                sh 'ls -la'
                echo 'Get Code DONE'
            }
        }

        stage ('Deploy') {
            steps {
                echo 'Initiating SAM Deployment...'
                sh '''
                    rm -f samconfig.toml
                    
                    sam build
                    sam validate --region us-east-1
                    sam deploy \
                        --config-file "" \
                        --stack-name "production-todo-list-aws" \
                        --region "us-east-1" \
                        --capabilities CAPABILITY_IAM \
                        --parameter-overrides Stage=production \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset \
                        --resolve-s3
                '''
                echo 'Deploy DONE'
            }
        }
        
        stage ('RestTest') {
            steps {
                echo 'Initiating Integration Tests...'
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        API_URL=$(aws cloudformation describe-stacks \
                            --stack-name "staging-todo-list-aws" \
                            --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                            --region us-east-1 \
                            --output text)
                        echo "$API_URL"
                        BASE_URL=$API_URL pytest -m readonly --junitxml=result-rest.xml test/integration/todoApiTest.py
                    '''
                    junit 'result-rest.xml'
                }
                echo 'Rest Tests DONE'
            }
        }

    }
}
