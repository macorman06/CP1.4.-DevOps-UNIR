pipeline {
    agent any

    stages {

        stage('Get Code') {
            steps {
                git branch: 'master',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/macorman06/CP1.4.-DevOps-UNIR.git'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    curl -o samconfig.toml https://raw.githubusercontent.com/macorman06/CP1.4.-DevOps-UNIR-config/staging/samconfig.toml
                    sam build
                    sam validate --region us-east-1
                    sam deploy \
                        --config-env production \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test') {
            steps {
                script {
                    def BASE_URL = sh(
                        script: """aws cloudformation describe-stacks \
                            --stack-name todo-list-aws-production \
                            --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                            --output text \
                            --region us-east-1""",
                        returnStdout: true
                    ).trim()
                    env.BASE_URL = BASE_URL
                }
                sh 'pytest test/integration/todoApiTest.py -v -m api -k "listtodos"'
            }
        }

    }

    post {
        always {
            cleanWs()
        }
    }

}
