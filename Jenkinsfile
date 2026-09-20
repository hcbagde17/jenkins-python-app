stage('Deploy to App VM') {
    steps {
        sshagent(credentials: ['gce-ssh']) {
            sh '''
                echo "Deploying with systemd..."

                ssh -o StrictHostKeyChecking=no videofilestill2024@35.192.31.192 \
                    "mkdir -p /home/videofilestill2024/app"

                scp -o StrictHostKeyChecking=no -r * \
                    videofilestill2024@35.192.31.192:/home/videofilestill2024/app/

                ssh -o StrictHostKeyChecking=no videofilestill2024@35.192.31.192 "
                    sudo systemctl daemon-reload &&
                    sudo systemctl restart flaskapp &&
                    sudo systemctl enable flaskapp
                "

                echo "Deployment complete."
            '''
        }
    }
}
