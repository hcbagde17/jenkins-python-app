pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/hcbagde17/jenk>
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Installing dependencies..."
                    python3 -m pip install --user -r requir>

                    echo "Compiling Python files..."
                    python3 -m compileall -q .
                '''
            }
        }

        stage('Test') {
