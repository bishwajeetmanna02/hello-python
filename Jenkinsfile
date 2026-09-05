pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest -v
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    withEnv(["PATH+SONAR=${tool 'SonarScanner'}/bin"]) {
                        sh 'sonar-scanner'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['gce-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no bishwajeetmannas132418@34.93.14.63 \
                        "mkdir -p ~/app"

                        scp -o StrictHostKeyChecking=no \
                        app.py requirements.txt \
                        USER@APP_IP:/home/USER/app/

                        ssh -o StrictHostKeyChecking=no USER@APP_IP \
                        "cd ~/app && pip3 install -r requirements.txt && \
                        nohup python3 app.py > app.log 2>&1 &"
                    '''
                }
            }
        }
    }
}