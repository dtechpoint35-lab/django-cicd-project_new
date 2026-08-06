pipeline {
    agent any

    environment {
        PROD_HOST = "3.109.200.176"
        PROD_USER = "ubuntu"
        PROD_DIR  = "/home/ubuntu/django-cicd-project_new"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Django Tests') {
            steps {
                sh '''
                    . venv/bin/activate
                    python3 manage.py test
                '''
            }
        }

        stage('Deploy to Production') {
            steps {
                sshagent(credentials: ['ec2-ssh']) {

                    sh """
                    ssh -o StrictHostKeyChecking=no ${PROD_USER}@${PROD_HOST} '

                    set -e

                    cd ${PROD_DIR}

                    echo "===== Updating Source Code ====="

                    git fetch origin
                    git reset --hard origin/main

                    if [ ! -d "venv" ]; then
                        python3 -m venv venv
                    fi

                    source venv/bin/activate

                    echo "===== Installing Dependencies ====="

                    pip install -r requirements.txt

                    echo "===== Running Migrations ====="

                    python3 manage.py migrate

                    echo "===== Stopping Old Django Server ====="

                    pkill -f "manage.py runserver" || true

                    sleep 2

                    echo "===== Starting Django Server ====="

                    nohup python3 manage.py runserver 0.0.0.0:8000 > django.log 2>&1 &

                    sleep 5

                    echo "===== Checking Django Process ====="

                    pgrep -f "manage.py runserver"

                    echo "===== Deployment Successful ====="

                    '
                    """
                }
            }
        }
    }

    post {

        success {
            echo "====================================="
            echo "CI/CD Pipeline Completed Successfully"
            echo "====================================="
        }

        failure {
            echo "====================================="
            echo "CI/CD Pipeline Failed"
            echo "====================================="
        }
    }
}