pipeline {
    agent any
    environment {
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk-amd64'
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    }
    stages {
        stage('Setup Virtual Environment') {
            steps {
                script {
                    sh '''
                        if [ ! -d "retail_pipeline_venv" ]; then
                            python3 -m venv retail_pipeline_venv
                            ./retail_pipeline_venv/bin/pip install --upgrade pip
                            ./retail_pipeline_venv/bin/pip install pipenv
                        fi
                    '''
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    sh './retail_pipeline_venv/bin/pipenv install --python python3'
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    sh 'echo $JAVA_HOME'
                    sh 'echo $PATH'
                    sh './retail_pipeline_venv/bin/pipenv run pytest'
                }
            }
        }
        stage('Package') {
            steps {
                sh 'rm -f retailproject.zip'
                sh 'zip -r retailproject.zip . -x "retail_pipeline_venv/*" -x "*.zip"'
            }
        }
        stage('Deploy') {
            steps {
                sshagent(credentials: ['deploy-ssh-key']) {
                    sh '''
                        set -e

                        ssh itv025741@g01.itversity.com \
                            "mkdir -p /home/itv025741/retailproject"

                        scp retailproject.zip \
                            itv025741@g01.itversity.com:/home/itv025741/retailproject/
                    '''
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline succeeded - retailproject.zip deployed to g01.itversity.com"
        }
        failure {
            echo "Pipeline FAILED - check the red stage above in console output"
        }
        always {
            sh 'rm -f retailproject.zip'
        }
    }
}