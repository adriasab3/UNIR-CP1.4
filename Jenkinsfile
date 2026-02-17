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
    }
}
