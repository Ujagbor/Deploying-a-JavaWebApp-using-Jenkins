pipeline {
    
    agent any
    tools {
        maven 'Maven3'
    }
    
    stages {
        
        stage('Checkout') {
            steps {
            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'ed7d77a0-5e0c-48e2-8b12-f3def5ab6599', url: 'https://github.com/Ujagbor/myJavaAppRepo']])
            }
        }
        
        stage ('Build') {
            steps {
                sh 'mvn clean install -f MyWebApp/pom.xml'
            }
        }
        
        stage ('Code Quality') {
            steps {
                withSonarQubeEnv('SonarQube') {
                sh 'mvn sonar:sonar -f MyWebApp/pom.xml'
                }
            }
        }
        
        stage ('Nexus Upload') {
           steps {
               nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '8b5623e2-0769-4b00-a62b-a4a5f66b4992', groupId: 'com.capone.af', nexusUrl: 'ec2-54-152-185-65.compute-1.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            } 
        }
        
        stage ('DEV Deploy') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'cbfb8ab9-cfd5-417d-bdb7-74965df660a5', path: '', url: 'http://ec2-3-88-128-16.compute-1.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ('Slack Notify') {
            steps {
                slackSend channel: 'application-migration', message: 'DEV Deployment was successful'
            }
        }
        
        stage ('DEV Approve') {
            steps {
                echo "Taking approval from DEV Manager for QA Deployment"
                timeout(time: 7, unit: 'DAYS') {
                input message: 'Do you want to deploy into QA?', submitter: 'admin'
                }
            }
        }
        
        stage ('QA Deploy') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'cbfb8ab9-cfd5-417d-bdb7-74965df660a5', path: '', url: 'http://ec2-3-88-128-16.compute-1.amazonaws.com:8080')], contextPath: null, war: '**/*.war'    
            }
        }
        
        stage ('Slack Notify-QA') {
            steps {
                slackSend(channel: 'application-migration', message: 'QA Deployment was successful, please start your testing')
            }
        }
    }
}
