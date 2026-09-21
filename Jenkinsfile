pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'

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

                    echo "Build completed successfully."
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."

                    python3 -m pytest -q

                    echo "Tests completed successfully."
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

                            withEnv([
                                "SCANNER_HOME=${scannerHome}"
                            ]) {

                                sh '''
                                    echo "Running SonarQube Scanner..."

                                    "$SCANNER_HOME/bin/sonar-scanner" \
                                        -Dsonar.projectKey=hello-python \
                                        -Dsonar.sources=. \
                                        -Dsonar.host.url="$SONAR_HOST_URL" \
                                        -Dsonar.token="$SONAR_TOKEN" \
                                        -Dsonar.python.version=3.10

                                    echo "SonarQube analysis submitted successfully."
                                '''
                            }
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
                        echo "Waiting for SonarQube analysis to finish..."

                        REPORT_FILE=".scannerwork/report-task.txt"

                        if [ ! -f "$REPORT_FILE" ]; then
                            echo "ERROR: report-task.txt was not found."
                            exit 1
                        fi

                        CE_TASK_ID=$(grep "^ceTaskId=" "$REPORT_FILE" | cut -d= -f2)

                        if [ -z "$CE_TASK_ID" ]; then
                            echo "ERROR: Could not find SonarQube task ID."
                            exit 1
                        fi

                        echo "SonarQube Task ID: $CE_TASK_ID"

                        STATUS="PENDING"

                        for i in $(seq 1 60)
                        do
                            STATUS=$(curl -s \
                                -u "$SONAR_TOKEN:" \
                                "$SONAR_HOST_URL/api/ce/task?id=$CE_TASK_ID" \
                                | jq -r '.task.status')

                            echo "SonarQube processing status: $STATUS"

                            if [ "$STATUS" = "SUCCESS" ]; then
                                break
                            fi

                            if [ "$STATUS" = "FAILED" ] || [ "$STATUS" = "CANCELED" ]; then
                                echo "SonarQube analysis processing failed."
                                exit 1
                            fi

                            sleep 5
                        done

                        if [ "$STATUS" != "SUCCESS" ]; then
                            echo "Timed out waiting for SonarQube."
                            exit 1
                        fi

                        echo "SonarQube analysis processing completed."

                        echo "Checking Quality Gate..."

                        QUALITY_STATUS=$(curl -s \
                            -u "$SONAR_TOKEN:" \
                            "$SONAR_HOST_URL/api/qualitygates/project_status?projectKey=hello-python" \
                            | jq -r '.projectStatus.status')

                        echo "Quality Gate Status: $QUALITY_STATUS"

                        if [ "$QUALITY_STATUS" != "OK" ]; then
                            echo "Quality Gate failed."
                            exit 1
                        fi

                        echo "Quality Gate passed successfully."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline Succeeded'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
