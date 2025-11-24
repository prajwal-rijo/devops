pipeline {
    agent any

    tools {
        jdk 'jdk17'       // Must match your Jenkins JDK name
        maven 'maven3'    // Must match your Jenkins Maven name
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/panchami30/Java-mini-project.git'
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
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds',
                                                 usernameVariable: 'JFROG_USER',
                                                 passwordVariable: 'JFROG_PASS')]) {
                    sh '''
                        WAR_FILE=$(ls sample-app/target/*.war)
                        FILE_NAME="${JOB_NAME}-${BUILD_NUMBER}-sample.war"
                        curl -u $JFROG_USER:$JFROG_PASS -T "$WAR_FILE" \
                        "https://trial7n02kw.jfrog.io/artifactory/java_warfile_repo-generic-local/$FILE_NAME"
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(credentials: ['tomcat-ssh-key']) {
                    sh """
                        WAR_FILE=\$(ls sample-app/target/*.war)
                        FILE_NAME=\$(basename "\$WAR_FILE")
                        SERVER_IP=52.0.251.73
                        SERVER_USER=ubuntu
                        TOMCAT_DIR=/opt/tomcat/webapps

                        scp -o StrictHostKeyChecking=no "\$WAR_FILE" \$SERVER_USER@\$SERVER_IP:/tmp/
                        ssh -o StrictHostKeyChecking=no \$SERVER_USER@\$SERVER_IP "sudo mv /tmp/\$FILE_NAME \$TOMCAT_DIR/"
                        ssh -o StrictHostKeyChecking=no \$SERVER_USER@\$SERVER_IP "sudo systemctl restart tomcat"
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
