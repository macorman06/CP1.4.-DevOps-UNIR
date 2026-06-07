pipeline {
    agent any

    stages {

        stage('Get Code') {
            steps {
                git branch: 'develop',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/macorman06/CP1.4.-DevOps-UNIR.git'
            }
        }

        stage('Static Test') {
            steps {
                sh '''
                    flake8 --exit-zero --format=pylint src/ > flake8-report.txt
                    bandit -r src/ -f txt -o bandit-report.txt || true
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam validate --region us-east-1
                    sam deploy \
                        --config-env staging \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    export BASE_URL="https://1h0lihcnr3.execute-api.us-east-1.amazonaws.com/Prod"
                    pytest test/integration/todoApiTest.py -v -m api
                '''
            }
        }

        stage('Promote') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-credentials',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_PASSWORD'
                )]) {
                    sh '''
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins CI"
                        git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/macorman06/CP1.4.-DevOps-UNIR.git HEAD:master
                    '''
                }
            }
        }

    }

}
