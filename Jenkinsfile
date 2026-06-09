pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                cleanWs()
                git branch: 'master', url: 'https://github.com/danielnegromar8732/todo-list-aws.git'
            }
        }
        stage('Deploy') {
            steps {
                sh 'sam validate  --region us-east-1'
                sh 'sam build'
                sh 'sam deploy --config-env production --resolve-s3 --no-fail-on-empty-changeset'
            }
        }
        stage('Get API URL') {
			steps {
				script {
					env.BASE_URL = sh(
						script: "aws cloudformation describe-stacks --stack-name todo-list-aws-production --query \"Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue\" --output text",
						returnStdout: true
					).trim()

					echo "BASE_URL = ${env.BASE_URL}"
				}
			}
        }
        stage('Rest Test') {
            steps {
                echo "Testing against: $BASE_URL"
				sh '''pytest test/integration/todoApiTest.py::TestApi::test_api_listtodos test/integration/todoApiTest.py::TestApi::test_api_gettodo -v --junitxml=results-rest.xml'''
				junit 'results-rest.xml'
            }
        }
	}
}
