pipeline {
    agent any
    options { skipDefaultCheckout() }
    stages {
        stage('Get Code') {
            steps {
                git branch: 'master', credentialsId: 'git_user', url: 'https://github.com/adriasab3/UNIR-CP1.4.git'
                stash name:'code', includes:'**/*'
            }
        }
        stage('Deploy'){
            steps {
                unstash name:'code'
                sh '''
                    sam validate --region us-east-1
                    sam build
                    sam deploy --config-env production --region us-east-1 --resolve-s3 --no-fail-on-empty-changeset
                '''
            }
        }
        stage('Rest Test'){
            steps {
                unstash name:'code'
                sh '''#!/bin/bash
                    export BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-list-aws-production --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text)
                    pytest -m read --junitxml=result-rest.xml test/integration/todoApiTest.py
                    '''
                junit 'result-rest.xml'
            }
        }
    }
}
