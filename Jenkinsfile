import groovy.json.JsonSlurper

pipeline {
    agent any;
    
    stages {
        stage('Get Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github_cred', passwordVariable: 'git_pass', usernameVariable: 'git_usr')]) {
                    git "https://$git_usr:$git_pass@github.com/yurifrezzato/todo-list-aws.git"
                }

                sh 'ls -la'
                echo "WORKSPACE: ${WORKSPACE}"
                sh 'git checkout develop'
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
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    // Set variables
                    String sam_stack_name = 'todo-list-aws';
                    String sam_config_env = 'staging';
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
                    python3 -m pytest --junitxml=result-rest.xml test/integration/todoApiTest.py
                """
                
                junit 'result-rest.xml';
            }
        }
        
        stage('Promote') {
            steps {
                println('Write changes in file');
                sh """
                    echo '<br />new line ${BUILD_ID}' >> CHANGELOG.md
                    git add .
                    git commit -m '${BUILD_ID}'
                    git push
                """
                
                println('Merge and push changelog')
                sh """
                    git checkout master
                    git merge develop
                    git push --set-upstream origin master
                """
            }
        }
    }
    post {
        cleanup {
            cleanWs();
        }
    }
}
