pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven3.9'
    }

    environment {
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = '3.110.158.190'
        REMOTE_DIR  = '/home/ubuntu/app'
        SSH_KEY     = '/home/ubuntu/mumbai.pem'
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                checkout scm
            }
        }

        stage('Build Project') {
            steps {
                script {
                    // Force Maven to use Java 21
                    def javaHome = tool name: 'jdk21', type: 'hudson.model.JDK'
                    def mavenHome = tool name: 'maven3.9', type: 'hudson.tasks.Maven$MavenInstallation'

                    withEnv([
                        "JAVA_HOME=${javaHome}",
                        "PATH=${javaHome}/bin:${mavenHome}/bin:${env.PATH}"
                    ]) {
                        sh 'java -version'  // should show Java 21
                        sh 'mvn -version'   // should also show Java 21
                        sh 'mvn clean install'
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Copy JAR to App Server') {
            steps {
                script {
                    def jarFile = sh(script: "ls target/*SNAPSHOT.jar", returnStdout: true).trim()
                    sh """
                        scp -i ${SSH_KEY} ${jarFile} ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
                    """
                }
            }
        }
    }
}
