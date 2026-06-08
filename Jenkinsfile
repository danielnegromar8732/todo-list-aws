pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/danielnegromar8732/todo-list-aws.git'
            }
        }
        stage('Static Test') {
            steps {
                sh '''
                    python -m flake8 src --exit-zero --output-file=flake8.out
                '''
            }
        }
    }
}
