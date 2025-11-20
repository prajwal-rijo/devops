pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SERVER_IP   = "52.0.251.73"
        SERVER_USER = "ubuntu"
        TOMCAT_DIR  = "/opt/tomcat/webapps"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/prajwal-rijo/devops.git'
            }
        }

        stage('Build') {
            steps {
                dir('sample-app') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Upload to JFrog') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'jfrog-creds',
                    usernameVariable: 'JFROG_USER',
                    passwordVariable: 'JFROG_PASS'
                )]) {
                    sh '''
                        echo "Finding WAR file..."
                        WAR_FILE=$(ls sample-app/target/*.war)

                        echo "Uploading WAR to JFrog..."
                        curl -L -u $JFROG_USER:$JFROG_PASS \
                        -T $WAR_FILE \
                        "https://trial9krpxa.jfrog.io/artifactory/testrepo-generic-local/${JOB_NAME}-${BUILD_NUMBER}.war"
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(credentials: ['tomcat-ssh-key']) {
                    sh '''
                        echo "Starting deployment..."

                        WAR_FILE=$(ls sample-app/target/*.war)
                        WAR_NAME=$(basename $WAR_FILE)

                        echo "Copying WAR to server..."
                        scp -o StrictHostKeyChecking=no $WAR_FILE $SERVER_USER@$SERVER_IP:/tmp/

                        echo "Moving WAR to Tomcat..."
                        ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "sudo mv /tmp/$WAR_NAME $TOMCAT_DIR/"

                        echo "Restarting Tomcat..."
                        ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "sudo systemctl restart tomcat || sudo systemctl restart tomcat9"

                        echo "✅ Deployment Completed Successfully"
                    '''
                }
            }
        }
    }
}
