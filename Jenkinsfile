pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout ') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/TCRDINSEH/SpringBoot-WebApplication.git'
            }
        }
        
        stage('Code Compile') {
            steps {
                    sh "mvn compile"
            }
        }
        
        stage('Run Test Cases') {
            steps {
                    sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                  withSonarQubeEnv('sonar-server') {
    sh '''
   $SCANNER_HOME/bin/sonar-scanner \
      -Dsonar.projectKey=Java-WebApp \
      -Dsonar.projectName=Java-WebApp \
      -Dsonar.sources=. \
      -Dsonar.java.binaries=. \
      -Dsonar.exclusions=target/**
    '''


    
                }
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                   dependencyCheck additionalArguments: '--scan ./   ', odcInstallation: 'DP'
                   dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Maven Build') {
            steps {
                    sh "mvn clean compile"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'b289dc43-2ede-4bd0-95e8-75ca26100d8d', toolName: 'docker') {
                            sh "docker build -t webapp ."
                            sh "docker tag webapp tcrd4323/webapp:latest"
                            sh "docker push tcrd4323/webapp:latest "
                        }
                   } 
            }
        }
        
        stage('Docker Image scan') {
            steps {
                    sh "trivy image tcrd4323/webapp:latest "
            }
        }
        
    }
}

