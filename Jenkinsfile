pipeline {
    agent { label "dev" }

    tools {
        maven 'M3'
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-11-openjdk-amd64' // Changed to Java 11
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
                sh '''
                    echo "----- BUILD LOG -----" > build_report.txt
                    echo "JAVA_HOME = $JAVA_HOME" >> build_report.txt
                    java -version >> build_report.txt 2>&1
                    mvn -version >> build_report.txt 2>&1
                    mvn clean package -DskipTests >> build_report.txt 2>&1
                '''
            }
        }

        stage('Unit Test') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    echo "----- TEST LOG -----" > test_report.txt
                    mvn test >> test_report.txt 2>&1
                '''
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('Code Analysis') {
            steps {
                echo 'Performing SonarQube analysis...'
                withSonarQubeEnv('sonar') {
                    sh '''
                        echo "----- SONAR ANALYSIS LOG -----" > sonar_report.txt
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.token=${SONAR_TOKEN} \
                            -Dsonar.projectName='my-maven-project' >> sonar_report.txt 2>&1
                    '''
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                echo 'Waiting for Quality Gate status...'
                timeout(time: 30, unit: 'MINUTES') { 
                    waitForQualityGate abortPipeline: true
                }
                sh 'echo "Quality Gate Passed Successfully!" > quality_gate_report.txt'
            }
        }
    }

    post {
        always { 
            echo 'Cleaning up workspace...'
            archiveArtifacts artifacts: '*.txt', fingerprint: true
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
                <p>Please find the attached stage reports (build, test, sonar, quality gate).</p>
                <p>Best regards,<br>Your Jenkins</p>
                """,
                mimeType: 'text/html',
                attachmentsPattern: '*.txt'
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
                <p>Please check the console output and attached logs for details.</p>
                <ul>
                    <li><b>Project:</b> ${env.JOB_NAME}</li>
                    <li><b>Build Number:</b> ${env.BUILD_NUMBER}</li>
                    <li><b>Status:</b> ${currentBuild.currentResult}</li>
                    <li><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
                </ul>
                <p>Best regards,<br>Your Jenkins</p>
                """,
                mimeType: 'text/html',
                attachmentsPattern: '*.txt'
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
                <p>Please check the console output and attached logs for details.</p>
                <ul>
                    <li><b>Project:</b> ${env.JOB_NAME}</li>
                    <li><b>Build Number:</b> ${env.BUILD_NUMBER}</li>
                    <li><b>Status:</b> ${currentBuild.currentResult}</li>
                    <li><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
                </ul>
                <p>Best regards,<br>Your Jenkins</p>
                """,
                mimeType: 'text/html',
                attachmentsPattern: '*.txt'
            )
        }
    }
}
