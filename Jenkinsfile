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
            docker run --rm \
              -e SONAR_HOST_URL=$SONAR_HOST_URL \
              -e SONAR_LOGIN=$SONAR_AUTH_TOKEN \
              -v $WORKSPACE:/usr/src \
              sonarsource/sonar-scanner-cli \
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
        sh '''
        docker run --rm \
          -v $WORKSPACE:/src \
          -v $WORKSPACE/dependency-check-data:/usr/share/dependency-check/data \
          -v $WORKSPACE/dependency-check-report:/report \
          owasp/dependency-check \
          --scan /src \
          --format "XML" \
          --out /report
        '''
        dependencyCheckPublisher pattern: 'dependency-check-report/dependency-check-report.xml'
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
                       withDockerRegistry(credentialsId: 'Dockercred', url: 'https://index.docker.io/v1/') {
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

