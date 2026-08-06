stage('Deploy to Production') {
    steps {
        sshagent(credentials: ['ec2-ssh']) {
            sh """
            ssh -o StrictHostKeyChecking=no ${PROD_USER}@${PROD_HOST} '
                set -e

                cd ${PROD_DIR}

                echo "========== CURRENT COMMIT =========="
                git log --oneline -1

                echo "========== FETCH LATEST CODE =========="
                git fetch origin

                echo "========== RESET TO LATEST COMMIT =========="
                git reset --hard origin/main

                echo "========== UPDATED COMMIT =========="
                git log --oneline -1

                if [ ! -d "venv" ]; then
                    python3 -m venv venv
                fi

                source venv/bin/activate

                echo "========== INSTALLING REQUIREMENTS =========="
                pip install -r requirements.txt

                echo "========== RUNNING MIGRATIONS =========="
                python3 manage.py migrate

                echo "========== STOPPING OLD DJANGO SERVER =========="
                pkill -f "manage.py runserver" || true

                sleep 2

                echo "========== STARTING DJANGO SERVER =========="
                nohup python3 manage.py runserver 0.0.0.0:8000 > django.log 2>&1 &

                sleep 5

                echo "========== VERIFYING DJANGO SERVER =========="
                pgrep -f "manage.py runserver"

                echo "========== DEPLOYMENT SUCCESSFUL =========="
            '
            """
        }
    }
}