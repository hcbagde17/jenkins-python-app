pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/hcbagde17/jenkins-python-app.git'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Installing dependencies..."
                    python3 -m pip install --user -r requirements.txt

                    echo "Compiling Python files..."
                    python3 -m compileall -q .
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."
                    python3 -m pytest -q
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {

                    withSonarQubeEnv('sonarqube') {

                        script {
                            def scannerHome = tool 'SonarScanner'

                            sh """
                                echo "Running SonarQube Scanner..."

                                "${scannerHome}/bin/sonar-scanner" \
                                  -Dsonar.projectKey=hello-python \
                                  -Dsonar.sources=. \
                                  -Dsonar.host.url="${SONAR_HOST_URL}" \
                                  -Dsonar.token="${SONAR_TOKEN}" \
                                  -Dsonar.python.version=3.10
                            """
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )
                ]) {

                    sh '''
                        echo "Checking SonarQube Quality Gate..."

                        STATUS=$(curl -s \
                            -u "$SONAR_TOKEN:" \
                            "http://34.67.125.36:9000/api/qualitygates/project_status?projectKey=hello-python" \
                            | jq -r '.projectStatus.status')

                        echo "Quality Gate Status: $STATUS"

                        if [ "$STATUS" != "OK" ]; then
                            echo "Quality Gate failed."
                            exit 1
                        fi

                        echo "Quality Gate passed."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline Succeeded"
        }

        failure {
            echo "Pipeline Failed"
        }
    }
}
