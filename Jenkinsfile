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
                    curl -o samconfig.toml https://raw.githubusercontent.com/macorman06/CP1.4.-DevOps-UNIR-config/staging/samconfig.toml
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
                script {
                    def BASE_URL = sh(
                        script: """aws cloudformation describe-stacks \
                            --stack-name todo-list-aws-staging \
                            --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' \
                            --output text \
                            --region us-east-1""",
                        returnStdout: true
                    ).trim()
                    env.BASE_URL = BASE_URL
                }
                sh 'pytest test/integration/todoApiTest.py -v -m api'
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
                        git fetch origin master
                        git show origin/master:Jenkinsfile > /tmp/Jenkinsfile_cd
                        git show origin/master:Jenkinsfile_agentes > /tmp/Jenkinsfile_agentes_cd
                        git checkout master
                        git merge -X theirs develop --no-edit
                        cp /tmp/Jenkinsfile_cd Jenkinsfile
                        cp /tmp/Jenkinsfile_agentes_cd Jenkinsfile_agentes
                        git add Jenkinsfile Jenkinsfile_agentes
                        git diff --cached --quiet || git commit -m "chore: preserve CD Jenkinsfiles after CI merge"
                        git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/macorman06/CP1.4.-DevOps-UNIR.git master
                    '''
                }
            }
        }

    }

    post {
        always {
            cleanWs()
        }
    }

}
