import groovy.json.JsonSlurper

pipeline {
    agent any;
    
    stages {
        stage('Get Code') {
            steps {
<<<<<<< HEAD
                git "https://github.com/yurifrezzato/todo-list-aws.git"

                sh 'ls -la'
                echo "WORKSPACE: ${WORKSPACE}"
                sh 'git checkout master'
=======
                withCredentials([usernamePassword(credentialsId: 'github_cred', passwordVariable: 'git_pass', usernameVariable: 'git_usr')]) {
                    git "https://$git_usr:$git_pass@github.com/yurifrezzato/todo-list-aws.git"
                }
                
                script {
                    sam_config_env = 'staging';
                    String github_branch = 'develop';
                
                    sh 'ls -la'
                    echo "WORKSPACE: ${WORKSPACE}"
                    sh "git checkout ${github_branch}"
                    sh "wget https://raw.githubusercontent.com/yurifrezzato/todo-list-aws-config/${sam_config_env}/samconfig.toml"
                }
            }
        }
        
        stage('Static Test') {
            steps {
                sh '''
                    python3 -m flake8 --exit-zero --format=pylint ./src > flake8.out
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')];
                
                sh '''
                    python3 -m bandit --exit-zero -r ./src -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')];
>>>>>>> develop
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Set variables
                    String sam_stack_name = 'todo-list-aws';
<<<<<<< HEAD
                    String sam_config_env = 'production';
=======
>>>>>>> develop
                    String curr_stack_name = sam_stack_name + '-' + sam_config_env;
                    
                    String sam_region = 'us-east-1';
                    
                    sh """
                        sam build
                        sam deploy --config-file samconfig.toml --config-env ${sam_config_env} --save-params --no-fail-on-empty-changeset
                    """

                    def stack_endpoint = sh(script: "sam list stack-outputs --stack-name ${curr_stack_name} --region ${sam_region} --output json", returnStdout: true).trim();
                    def jsonParser = new JsonSlurper()
                    def stackOutputs = jsonParser.parseText(stack_endpoint)

                    BASE_URL = stackOutputs[0].OutputValue
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
