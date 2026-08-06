pipeline {
    agent any

    environment {
        PROD_HOST = "3.109.200.176"
        PROD_USER = "ubuntu"
        PROD_DIR  = "/home/ubuntu/django-cicd-project_new"
    }

    stages {

        stage('Checkout') {
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
                    ssh -o StrictHostKeyChecking=no ${PROD_USER}@${PROD_HOST} << 'EOF'

                    set -e

                    cd ${PROD_DIR}

                    echo "Current Commit:"
                    git log --oneline -1

                    echo "Fetching Latest Code..."
                    git fetch origin

                    git reset --hard origin/main

                    echo "Updated Commit:"
                    git log --oneline -1

                    if [ ! -d "venv" ]; then
                        python3 -m venv venv
                    fi

                    source venv/bin/activate

                    pip install -r requirements.txt

                    python3 manage.py migrate

                    python3 manage.py collectstatic --noinput

                    echo "Deployment Completed Successfully"

                    EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline Completed Successfully"
        }

        failure {
            echo "CI/CD Pipeline Failed"
        }
    }
}