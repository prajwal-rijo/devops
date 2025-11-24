pipeline {
    agent any

    tools {
        jdk 'JDK17'          // your installed JDK name
        sonarScanner 'Sonar-Scanner-CLI'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=myproject \
                        -Dsonar.sources=./src \
                        -Dsonar.host.url=http://your-sonarqube:9000 \
                        -Dsonar.login=${SONAR_AUTH_TOKEN}
                    """
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
