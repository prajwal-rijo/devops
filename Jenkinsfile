pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'Maven'     
        JAVA_HOME  = tool 'jdk-17'     // Use your actual JDK name
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

        // Other stages...
    }

    post {
        always {
            node {  // Wrap cleanWs in node
                cleanWs()
            }
        }
    }
}
