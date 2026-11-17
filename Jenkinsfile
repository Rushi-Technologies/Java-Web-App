pipeline {
    agent any

    triggers {
        githubPush()
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    tools {
        maven 'Maven-3.10'
    }

    stages {
        stage('Run Unit Tests') {
            steps {
                 sh "mvn clean test"
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('Sonar Scan') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh "mvn sonar:sonar"
                }
            }
        }

      stage('Wait for Quality Gate Result') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                     waitForQualityGate abortPipeline: true
                } 
            }
        }

       stage("Deploy to Tomcat"){
             environment {
                 TOMCAT_SERVER_IP = "172.31.2.59"
             }
            steps {
                sshagent(credentials: ['Tomcat_Server_SSH_Cred']) {
                     sh """ssh -o StrictHostKeyChecking=no  ec2-user@${TOMCAT_SERVER_IP} sudo systemctl stop tomcat
                           ssh -o StrictHostKeyChecking=no  ec2-user@${TOMCAT_SERVER_IP} rm -f /opt/tomcat/webapps/java-web-app.war
                           sleep 10s
                           scp  -o StrictHostKeyChecking=no target/java-web-app*.war ec2-user@${TOMCAT_SERVER_IP}:/opt/tomcat/webapps/java-web-app.war
                           ssh -o StrictHostKeyChecking=no  ec2-user@${TOMCAT_SERVER_IP} sudo systemctl start tomcat"""
                }
            }
        }


    }

    post {
        always {
            sendEmail(currentBuild.currentResult)
        }
    }
}

def sendEmail(String buildStatus){
    emailext body:  """
                    <html>
                    <body style="font-family: Arial, sans-serif; background-color: #f4f4f4; padding: 20px;">
                        <h2 style="color: #2d87f0;">Jenkins Build Notification</h2>
                        <p><strong>Build Result:</strong> ${buildStatus}</p>
                        <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                        <p><strong>Project:</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        <p>For more details, please visit the Jenkins job page.</p>
                    </body>
                    </html>
                    """, 
        subject: "${env.JOB_NAME} - ${env.BUILD_NUMBER} - ${buildStatus}", 
        to: 'balajireddy.urs@gmail.com,rushitechnologiesbanglore@gmail.com',
        mimeType: 'text/html'
}