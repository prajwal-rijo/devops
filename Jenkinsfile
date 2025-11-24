pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {

                    script {
                        // Load Sonar Scanner tool
                        scannerHome = tool name: 'Sonar-Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    }

                    dir('sample-app') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=java-mini-project \
                            -Dsonar.projectName=java-mini-project \
                            -Dsonar.sources=src \
                            -Dsonar.java.binaries=target \
                            -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload to JFrog') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds',
                                                 usernameVariable: 'JFROG_USER',
                                                 passwordVariable: 'JFROG_PASS')]) {
                    sh '''
                        echo "Uploading WAR to JFrog..."
                        WAR_FILE=$(ls sample-app/target/*.war)
                        FILE_NAME="${JOB_NAME}-${BUILD_NUMBER}-sample.war"
                        echo "Uploading file as: $FILE_NAME"
                        
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
                        echo "Deploying WAR to Tomcat server..."

                        WAR_FILE=\$(ls sample-app/target/*.war)
                        FILE_NAME=\$(basename "\$WAR_FILE")

                        SERVER_IP=52.0.251.73
                        SERVER_USER=ubuntu
                        TOMCAT_DIR=/opt/tomcat/webapps

                        echo "Using server IP: \$SERVER_IP"

                        scp -o StrictHostKeyChecking=no "\$WAR_FILE" \$SERVER_USER@\$SERVER_IP:/tmp/

                        ssh -o StrictHostKeyChecking=no \$SERVER_USER@\$SERVER_IP "sudo mv /tmp/\$FILE_NAME \$TOMCAT_DIR/"

                        ssh -o StrictHostKeyChecking=no \$SERVER_USER@\$SERVER_IP "sudo systemctl restart tomcat"

                        echo "Deployment completed successfully!"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
