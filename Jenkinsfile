@Library('JenkinsShared_Lib') _
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
            sendEmailNotification(currentBuild.currentResult,"balajireddy.urs@gmail.com")
        }
    }
}
