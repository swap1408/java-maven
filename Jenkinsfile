pipeline {
    agent any

    tools {
        jdk 'jdk21'         // Make sure these names match in Jenkins Global Tool Config
        maven 'maven3.9'
    }

    environment {
        REMOTE_USER = 'ubuntu'                      // change as needed
        REMOTE_HOST = '3.110.158.190'                     // target application server IP
        REMOTE_DIR  = '/home/ubuntu/app'            // remote directory
        SSH_KEY     = '/home/ubuntu/mumbai.pem'       // Jenkins server private key path
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                checkout scm  // This uses the Jenkins job’s configured Git repo
            }
        }

        stage('Build Project') {
            steps {
                sh 'mvn clean install'
                sh 'mvn clean package'
            }
        }

        stage('Copy JAR to App Server') {
            steps {
                script {
                    def jarFile = sh(script: "ls target/*SNAPSHOT.jar", returnStdout: true).trim()
                    sh """
                        echo "Copying ${jarFile} to ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}"
                        scp -i ${SSH_KEY} ${jarFile} ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build and deployment successful!'
        }
        failure {
            echo '❌ Build or deployment failed.'
        }
    }
}
