pipeline {
    agent any
    options { skipDefaultCheckout() }
    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', credentialsId: 'git_user', url: 'https://github.com/adriasab3/UNIR-CP1.4.git'
                stash name:'code', includes:'**/*'
            }
        }
        stage('Static Test'){
            steps {
                unstash name:'code'
                sh '''
                    flake8 --format=pylint --exit-zero src > flake8.out
                    bandit --exit-zero -r src/ -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')]
                recordIssues tools: [pyLint(name:'Bandit', pattern: 'bandit.out')]
            }
        }
        stage('Deploy'){
            steps {
                unstash name:'code'
                sh '''
                    sam validate --region us-east-1
                    sam build
                    sam deploy --config-env staging --region us-east-1 --resolve-s3 --no-fail-on-empty-changeset
                '''
            }
        }
        stage('Rest Test'){
            steps {
                unstash name:'code'
                sh '''#!/bin/bash
                    export BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text)
                    pytest --junitxml=result-rest.xml test/integration/todoApiTest.py
                    '''
                junit 'result-rest.xml'
            }
        }
        stage('Promote'){
            steps {
                unstash name:'code'
                withCredentials([gitUsernamePassword(credentialsId: 'git_user', gitToolName: 'Default')]) {
                    sh '''
                        git fetch origin
                        git checkout master || git checkout -b master origin/master
                        git pull origin master
                        git merge develop --no-ff -m "Jenkins auto-merge develop into master"
                        git push origin master
                '''
                }
            }
        }
    }
}
