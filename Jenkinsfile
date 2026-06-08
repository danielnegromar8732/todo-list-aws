pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                cleanWs()
                git branch: 'develop', url: 'https://github.com/danielnegromar8732/todo-list-aws.git'
            }
        }
        stage('Static Test') {
            steps {
                sh 'flake8 src/ --statistics --count --output-file=flake8-report.txt || true'
                sh 'bandit -r src/ -f html -o bandit-report.html || true'
            }
            post {
                always {
                    recordIssues tools: [
                        flake8(pattern: 'flake8-report.txt')
                    ]

                    publishHTML(target: [
                        reportName : 'Bandit Report',
                        reportDir  : '.',
                        reportFiles: 'bandit-report.html',
                        keepAll    : true,
                        alwaysLinkToLastBuild: true
                    ])
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'sam validate  --region us-east-1'
                sh 'sam build'
                sh 'sam deploy --config-env staging --resolve-s3 --no-fail-on-empty-changeset'
            }
        }
        // Stage necesaria para obtener la API URL para los tests
        stage('Get API URL') {
			steps {
				script {
					env.BASE_URL = sh(
						script: "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query \"Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue\" --output text",
						returnStdout: true
					).trim()

					echo "BASE_URL = ${env.BASE_URL}"
				}
			}
        }
        stage('Rest Test') {
            steps {
                echo "Testing against: $BASE_URL"
				sh "pytest test/integration/todoApiTest.py -v --junitxml=results-rest.xml"
				junit 'results-rest.xml'
            }
        }
        stage('Promote') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials', 
                                                  usernameVariable: 'GIT_USER', 
                                                  passwordVariable: 'GIT_PASS')]) {
                    sh '''
                        cd $WORKSPACE
                        git config user.email "danielnegromar8732@users.noreply.github.com"
                        git config user.name "Daniel NM"
                        git fetch origin
                        git checkout master
                        git merge origin/develop -m "Promote develop to master"
                        git remote set-url origin https://${GIT_USER}:${GIT_PASS}@github.com/danielnegromar8732/todo-list-aws.git
                        git push origin master
                    '''
                }
            }
        }
    }
}
