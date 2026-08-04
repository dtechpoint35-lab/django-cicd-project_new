pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code checked out successfully.'
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
                python manage.py test
                '''
            }
        }

    stage('Deploy') {
    steps {
        sshagent(credentials: ['ec2-ssh']) {
            sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@3.109.200.176 "
                    cd /home/ubuntu/django-cicd-project_new &&
                    git pull origin main &&
                    source venv/bin/activate &&
                    pip install -r requirements.txt &&
                    python3 manage.py migrate
                "
            '''
        }
    }
}
    }
}