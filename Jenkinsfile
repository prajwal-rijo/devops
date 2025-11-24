pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'Maven'             // Your Maven tool name in Jenkins
        JAVA_HOME  = tool 'JDK17'             // Your JDK tool name in Jenkins
        PATH       = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/panchami30/Java-mini-project.git',
                    credentialsId: 'jfrog-creds'
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
                    dir('sample-app') {
                        withCredentials([string(credentialsId: 'prajwal-sonar', variable: 'SONAR_TOKEN')]) {
                            sh """
                                sonar-scanner \
                                -Dsonar.projectKey=java-mini-project \
                                -Dsonar.projectName=java-mini-project \
                                -Dsonar.sources=src \
                                -Dsonar.java.binaries=target \
                                -Dsonar.login=\$SONAR_TOKEN
                            """
                        }
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

        stage('Upload to JFrog') {
            steps {
                dir('sample-app/target') {
                    script {
                        // Replace 'my-repo-local' and 'your-artifactory-url' with your actual details
                        sh """
                            curl -u jfrog-creds-user:jfrog-creds-pass \
                            -T sample.war \
                            "https://your-artifactory-url/artifactory/my-repo-local/sample.war"
                        """
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                dir('sample-app/target') {
                    script {
                        sh """
                            curl -u tomcat-user:tomcat-pass \
                            -T sample.war \
                            "http://your-tomcat-server:8080/manager/text/deploy?path=/sample&update=true"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
        always {
            cleanWs()
        }
    }
}
