pipeline {
    agent { label "dev" }

    tools {
        maven 'M3'
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:${PATH}"
        
        SONAR_PROJECT_KEY = 'com.mycompany.app:my-maven-project'
        SONAR_ORGANIZATION = 'Aslah EA'
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('CHECKOUT CODE') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/aslahea/my-maven-project.git']])
                echo 'Code checked out successfully.'
            }
        }

        stage('BUILD') {
            steps {
                echo 'Building the project with Maven...'
                sh 'echo "JAVA_HOME = $JAVA_HOME"'
                sh 'java -version'
                sh 'mvn -version'
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Unit Test') {
            steps {
                echo 'Running unit tests...'
                sh 'mvn test'
                junit '**/target/surefire-reports/*.xml'
            }
        }
        stage('Code Analysis') {
            steps {
                echo 'Performing SonarQube analysis...'
                withSonarQubeEnv('sonar') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.token=${SONAR_TOKEN} -Dsonar.projectName='my-maven-project'"
                }
            }
        }
        stage('Quality Gate Check') {
            steps {
                echo 'Waiting for Quality Gate status...'
                timeout(time: 10, unit: 'MINUTES') { 
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    post {
        always { 
            echo 'Cleaning up workspace...'
            cleanWs() 
        }
        success { 
            echo 'Pipeline Succeeded! Sending success email...'
            emailext (
                to: 'aslahea068@gmail.com', 
                subject: "Jenkins Build ${currentBuild.fullDisplayName} - SUCCESS",
                body: """
                <p>Hello,</p>
                <p>The Jenkins pipeline for <b>my-maven-project</b> has finished successfully.</p>
                <ul>
                    <li><b>Project:</b> ${env.JOB_NAME}</li>
                    <li><b>Build Number:</b> ${env.BUILD_NUMBER}</li>
                    <li><b>Status:</b> ${currentBuild.currentResult}</li>
                    <li><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
                </ul>
                <p>Best regards,<br>Your Jenkins</p>
                """,
                mimeType: 'text/html'
            )
        }
        failure { 
            echo 'Pipeline Failed! Sending failure email...'
            emailext (
                to: 'aslahea068@gmail.com', 
                subject: "Jenkins Build ${currentBuild.fullDisplayName} - FAILED",
                body: """
                <p>Hello,</p>
                <p>The Jenkins pipeline for <b>my-maven-project</b> has failed!</p>
                <p>Please check the console output for details.</p>
                <ul>
                    <li><b>Project:</b> ${env.JOB_NAME}</li>
                    <li><b>Build Number:</b> ${env.BUILD_NUMBER}</li>
                    <li><b>Status:</b> ${currentBuild.currentResult}</li>
                    <li><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
                </ul>
                <p>Best regards,<br>Your Jenkins</p>
                """,
                mimeType: 'text/html'
            )
        }
        aborted { 
            echo 'Pipeline Aborted! Sending aborted email...'
             emailext (
                to: 'aslahea068@gmail.com', 
                subject: "Jenkins Build ${currentBuild.fullDisplayName} - ABORTED",
                body: """
                <p>Hello,</p>
                <p>The Jenkins pipeline for <b>my-maven-project</b> was aborted (e.g., due to timeout or manual cancellation).</p>
                <p>Please check the console output for details.</p>
                <ul>
                    <li><b>Project:</b> ${env.JOB_NAME}</li>
                    <li><b>Build Number:</b> ${env.BUILD_NUMBER}</li>
                    <li><b>Status:</b> ${currentBuild.currentResult}</li>
                    <li><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
                </ul>
                <p>Best regards,<br>Your Jenkins</p>
                """,
                mimeType: 'text/html'
            )
        }
    }
}
