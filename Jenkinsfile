pipeline {
    agent any;
    
    stages {
        stage('Get Code') {
            steps {
                echo 'Get Code'
                git 'https://github.com/yurifrezzato/todo-list-aws.git'
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
                    curr_stack_name = sam_stack_name + '-' + sam_config_env;
                    
                    sh """
                        sam build
                        sam deploy --config-file samconfig.toml --stack-name ${curr_stack_name} --config-env ${sam_config_env} --save-params
                    """

                    println(curr_stack_name);
                    stack_endpoint = sh(script: 'sam list stack-outputs --stack-name ${curr_stack_name} --region us-east-1 --output json', returnStdout: true);
                    println(stack_endpoint);
                }
            }
        }
    }
    post {
        cleanup {
            println(curr_stack_name);
            sh 'sam delete --no-prompts --stack-name ${curr_stack_name}' // delete sam stack
            cleanWs();
        }
    }
}
