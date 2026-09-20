stage('Deploy to App VM') {
    steps {
        sshagent(credentials: ['gce-ssh']) {
            sh '''
                echo "Deploying with systemd..."

                ssh -o StrictHostKeyChecking=no managed-instance@35.192.31.192 \
                    "mkdir -p /home/managed-instance/app"

                scp -o StrictHostKeyChecking=no -r * \
                    APP_USER@APP_IP:/home/managed-instance/app/

                ssh -o StrictHostKeyChecking=no managed-instance@35.192.31.192 "
                    sudo systemctl daemon-reload &&
                    sudo systemctl restart flaskapp &&
                    sudo systemctl enable flaskapp
                "

                echo "Deployment complete."
            '''
        }
    }
}
