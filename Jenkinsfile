import groovy.json.JsonSlurper

pipeline {
    agent any;
    
    stages {
        stage('Get Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github_cred', passwordVariable: 'git_pass', usernameVariable: 'git_usr')]) {
                    checkout scmGit(
                        branches: [[
                            name: 'develop'
                        ]],
                        userRemoteConfigs: [[
                            url: "https://$git_usr:$git_pass@github.com/yurifrezzato/todo-list-aws.git"
                        ]]
                    )
                }

                sh 'ls -la'
                echo "WORKSPACE: ${WORKSPACE}"
            }
        }
        
        stage('Static Test') {
            steps {
                // catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        python3 -m flake8 --exit-zero --format=pylint ./src > flake8.out
                    '''
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')];
                    
                    sh '''
                        python3 -m bandit --exit-zero -r ./src -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                    '''
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')];
                // }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    String sam_stack_name = 'todo-list-aws';
                    String sam_config_env = 'staging';
                    String sam_region = 'us-east-1';
                    curr_stack_name = sam_stack_name + '-' + sam_config_env;
                    
                    sh """
                        sam build
                        sam deploy --config-file samconfig.toml --config-env ${sam_config_env} --save-params --no-fail-on-empty-changeset
                    """

                    def stack_endpoint = sh(script: "sam list stack-outputs --stack-name ${curr_stack_name} --region ${sam_region} --output json", returnStdout: true).trim();
                    // println(stack_endpoint);
                    
                    def jsonParser = new JsonSlurper()
                    def stackOutputs = jsonParser.parseText(stack_endpoint)

                    BASE_URL = stackOutputs[0].OutputValue
                    // println(BASE_URL);
                }
            }
        }
        
        stage('Rest Test') {
            steps {
                sh"""
                    export PYTHONPATH=${WORKSPACE}
                    export BASE_URL=${BASE_URL}
                    python3 -m pytest --junitxml=result-unit.xml test/integration/todoApiTest.py
                """
            }
        }
        
        stage('Promote') {
            steps {
                println('Write changes in file');
                sh "echo 'new line ${BUILD_ID}' >> CHANGELOG.md";
                
                println('Merge and push changelog')
                sh """
                    git checkout master
                    git pull
                    git merge develop
                    git push --set-upstream origin master
                """
            }
        }
    }
    post {
        cleanup {
            // sh "sam delete --no-prompts --stack-name ${curr_stack_name}"; // delete sam stack
            cleanWs();
        }
    }
}
