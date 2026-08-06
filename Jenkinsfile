pipeline {
    agent any

    environment {
        PROD_SERVER = "3.109.200.176"
        PROD_USER = "ubuntu"
        PROD_PATH = "/home/ubuntu/django-cicd-project_new"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Create Virtual Environment') {
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

        stage('Deploy') {
            steps {
                sshagent(credentials: ['ec2-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${PROD_USER}@${PROD_SERVER} '
                        set -e

                        cd ${PROD_PATH}

                        echo "===== Current Commit ====="
                        git log --oneline -1

                        echo "===== Pull Latest Code ====="
                        git fetch origin
                        git reset --hard origin/main

                        echo "===== New Commit ====="
                        git log --oneline -1

                        source venv/bin/activate

                        pip install -r requirements.txt

                        python3 manage.py migrate

                        python3 manage.py collectstatic --noinput

                        echo "Deployment Successful"
                    '
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