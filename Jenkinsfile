pipeline {
    agent any

    tools {
        maven 'maven-3.9.0'
    }

    stages {
        stage('Git Checkout') {
            steps {
                notifyBuild('STARTED')
                git branch: 'main',
                    url: 'https://github.com/KanthivardhanK/maven-webapp-project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SQ REPORT') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'tomcat-cred',
                        usernameVariable: 'TOMCAT_USER',
                        passwordVariable: 'TOMCAT_PASS'
                    )
                ]) {
                    sh '''
                        curl --fail --show-error \
                        --user "$TOMCAT_USER:$TOMCAT_PASS" \
                        --upload-file target/maven-web-application.war \
                        "http://13.201.189.6:8080/manager/text/deploy?path=/maven-web-application&update=true"
                    '''
                }
            }
        }
    }

    post {
        success {
            notifyBuild('SUCCESS')
        }
        failure {
            notifyBuild('FAILURE')
        }
    }
}

// Notification method
def notifyBuild(String buildStatus = 'STARTED') {
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    def colorCode = [
        STARTED: '#FFFF00',
        SUCCESS: '#00FF00'
    ].get(buildStatus, '#FF0000')

    slackSend(
        color: colorCode,
        message: summary,
        channel: '#devops-alerts'
    )
}
