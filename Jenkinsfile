pipeline {
    agent any
    environment {
        LABS = credentials('labcreds')
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk-amd64' // Set your JAVA_HOME path here.
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}" // Add Java binaries to PATH
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
                    // Install your project dependencies (e.g., requirements.txt or Pipfile)
                    sh './retail_pipeline_venv/bin/pipenv install'
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    // Ensure JAVA_HOME is set for PySpark to work
                    sh 'echo $JAVA_HOME'
                    sh 'echo $PATH'
                    // Run tests (assuming you are using pytest for tests)
                    sh './retail_pipeline_venv/bin/pipenv run pytest'
                }
            }
        }
        stage('Package') {
            steps {
                // Create the zip file but exclude the venv directory
                sh 'zip -r retailproject.zip . -x "retail_pipeline_venv/*"'
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
}
