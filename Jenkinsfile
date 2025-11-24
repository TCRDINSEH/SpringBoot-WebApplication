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


        
// stage('OWASP Dependency Check') {
//     steps {
//         dependencyCheck additionalArguments: '--scan ./   ', odcInstallation: 'DP'
//         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
//     }
// }


        
        stage('Maven Build') {
            steps {
                    sh "mvn clean package -DskipTests"

            }
        }
        
        // stage('Docker Build & Push') {
        //     steps {
        //            script {
        //                withDockerRegistry(credentialsId: 'Dockercred', url: 'https://index.docker.io/v1/') {
        //                     sh "docker build -t webapp ."
        //                     sh "docker tag webapp tcrd4323/webapp:latest"
        //                     sh "docker push tcrd4323/webapp:latest "
        //                 }
        //            } 
        //     }
        // }
        
      stage('Docker Build & Push to Artifact Registry') {
  environment {
    PROJECT_ID = 'applied-pager-476808-j5'
    REGION = 'us-central1'
    REPO = 'webapp-repo'
    IMAGE = 'webapp'
    TAG = 'latest'
  }
  steps {
    withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GCP_KEY')]) {
      sh """
        # Authenticate gcloud with service account
        gcloud auth activate-service-account --key-file=${GCP_KEY}
        gcloud config set project ${PROJECT_ID}

        # Ensure docker uses gcloud credential helper
        gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet

        # Build and push
        docker build -t ${IMAGE}:${TAG} .
        docker tag ${IMAGE}:${TAG} ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${TAG}
        docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${TAG}
      """
    }
  }
}

        stage('Docker Image scan') {
            steps {
                    sh "trivy image tcrd4323/webapp:latest "
            }
        }
        stage('Deploy to GKE') {
  steps {
    withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GCP_KEY')]) {
      sh '''
        gcloud auth activate-service-account --key-file=${GCP_KEY}
        gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${CLUSTER_ZONE} --project ${PROJECT_ID}
        kubectl apply -f k8s/deployment.yaml
      '''
    }
  }
}

        
    }
}

