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
                    sh 'mvn clean package -DskipTests -B'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                        dir('sample-app') {
                            // Fully qualified plugin invocation avoids "No plugin found" error
                            sh '''
                                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.1.2311:sonar -B \
                                    -Dsonar.login=$SONAR_TOKEN \
                                    -Dsonar.projectKey=JavaMiniProject \
                                    -Dsonar.projectName=JavaMiniProject
                            '''
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
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
                        if [ ! -f "$WAR_FILE" ]; then
                            echo "WAR file not found!"
                            exit 1
                        fi
                        FILE_NAME="${JOB_NAME}-${BUILD_NUMBER}-sample.war"
                        curl -f -u $JFROG_USER:$JFROG_PASS -T "$WAR_FILE" \
                            "https://trial7n02kw.jfrog.io/artifactory/java_warfile_repo-generic-local/$FILE_NAME"
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(credentials: ['tomcat-ssh-key']) {
                    sh '''
                        WAR_FILE=$(ls sample-app/target/*.war)
                        if [ ! -f "$WAR_FILE" ]; then
                            echo "WAR file not found!"
                            exit 1
                        fi
                        FILE_NAME=$(basename "$WAR_FILE")
                        SERVER_IP=52.0.251.73
                        SERVER_USER=ubuntu
                        TOMCAT_DIR=/opt/tomcat/webapps

                        # Copy WAR to server
                        scp -o StrictHostKeyChecking=no "$WAR_FILE" $SERVER_USER@$SERVER_IP:/tmp/
                        
                        # Move WAR to Tomcat webapps
                        ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "sudo mv /tmp/$FILE_NAME $TOMCAT_DIR/"

                        # Restart Tomcat
                        ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "sudo systemctl restart tomcat"

                        # Optional: check if Tomcat is running
                        ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "systemctl status tomcat --no-pager"
                    '''
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
