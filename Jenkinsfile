import groovy.json.JsonSlurper

sam_config_env = 'production';
github_branch = 'master';

pipeline {
    agent any;
    
    stages {
        stage('Get Code') {
            steps {
                git "https://github.com/yurifrezzato/todo-list-aws.git"
                
                sh 'ls -la'
                echo "WORKSPACE: ${WORKSPACE}"
                sh "git checkout ${github_branch}"
                sh "wget https://raw.githubusercontent.com/yurifrezzato/todo-list-aws-config/${sam_config_env}/samconfig.toml"
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Set variables
                    String sam_stack_name = 'todo-list-aws';
                    String curr_stack_name = sam_stack_name + '-' + sam_config_env;
                    
                    String sam_region = 'us-east-1';
                    
                    sh """
                        sam build
                        sam deploy --config-file samconfig.toml --config-env ${sam_config_env} --save-params --no-fail-on-empty-changeset
                    """

                    BASE_URL = sh( script: "aws cloudformation describe-stacks --stack-name ${curr_stack_name} --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region ${sam_region} --output text", returnStdout: true)
                }
            }
        }
        
        stage('Rest Test') {
            steps {
                sh"""
                    export PYTHONPATH=${WORKSPACE}
                    export BASE_URL=${BASE_URL}
                    python3 -m pytest -m readOnly --junitxml=result-rest.xml test/integration/todoApiTest.py
                """
                
                junit 'result-rest.xml';
            }
        }
    }
    post {
        cleanup {
            cleanWs();
        }
    }
}
