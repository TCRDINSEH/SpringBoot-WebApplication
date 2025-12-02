pipeline {
    agent any

    tools {
        jdk 'jdk11'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        PROJECT_ID   = 'applied-pager-476808-j5'
        REGION       = 'us-central1'
        REPO         = 'webapp-repo'
        IMAGE        = 'webapp'
        TAG          = 'latest'
        MIG_NAME     = 'webapp-mig'
        TEMPLATE_NAME= 'webapp-template'
        ZONE         = 'us-central1-a'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/TCRDINSEH/SpringBoot-WebApplication.git'
            }
        }

        stage('Compile & Test') {
            steps {
                sh "mvn clean compile test"
            }
        }

        stage('SonarQube Analysis') {
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

        stage('Maven Build') {
            steps {
                sh "mvn package -DskipTests"
            }
        }

        stage('Docker Build & Push to Artifact Registry') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GCP_KEY')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=${GCP_KEY}
                        gcloud config set project ${PROJECT_ID}
                        gcloud auth configure-docker ${REGION}-docker.pkg.dev --quiet

                        docker build -t ${IMAGE}:${TAG} .
                        docker tag ${IMAGE}:${TAG} ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${TAG}
                        docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${TAG}
                    """
                }
            }
        }

        stage('Image Scan') {
            steps {
                sh "trivy image ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${TAG}"
            }
        }

        stage('Deploy to MIG') {
            steps {
                withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GCP_KEY')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=${GCP_KEY}
                        gcloud config set project ${PROJECT_ID}

                        NEW_TEMPLATE=${TEMPLATE_NAME}-\$(date +%Y%m%d%H%M%S) \

                        # Create instance template with startup script
                        gcloud compute instance-templates create $NEW_TEMPLATE \
                          --machine-type=e2-medium \
                          --region=${REGION} \
                          --metadata=startup-script='#!/bin/bash
                            apt-get update && apt-get install -y docker.io
                            docker run -d -p 80:8080 ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${TAG}'

                        # Update MIG to use new template
                        gcloud compute instance-groups managed set-instance-template ${MIG_NAME} \
                          --template=$NEW_TEMPLATE \
                          --zone=${ZONE}

                        # Rolling update MIG
                        gcloud compute instance-groups managed rolling-action start-update ${MIG_NAME} \
                          --zone=${ZONE} --max-surge=1 --max-unavailable=0
                    """
                }
            }
        }
    }
}
