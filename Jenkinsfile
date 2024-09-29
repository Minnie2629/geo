pipeline {
    agent any 

    tools {
        maven 'M2_HOME'
    }

    environment {
        AWS_REGION = 'us-east-1'
        BRANCH_NAME = 'main'
        SCANNER_HOME = tool 'sonar-tools'
        GIT_URL = 'https://github.com/Minnie2629/geo.git'
        QG_CONDITION = false
        GITHUB_CREDENTIALS = 'github-credentials' 
        SONAQUBE_CRED = 'Sonar-credentials'
        SONAQUBE_INSTALLATION = 'Sonar'
        APP_NAME = 'geoapp' 
        JFROG_CRED = 'jfrog-cred'
        ARTIFACTPATH = 'target/*.jar'
        ARTIFACTORY_URL = 'http://ec2-34-229-187-93.compute-1.amazonaws.com:8081/artifactory'
        REPO = 'devops'
        ARTIFACTTARGETPATH = "release_${BUILD_ID}.jar"
        DOCKER_REPO = '137766331670.dkr.ecr.us-east-1.amazonaws.com/devops_repository'
        REPO_URL = '137766331670.dkr.ecr.us-east-1.amazonaws.com/'
    }

    stages {
        stage('Checkout') {
            steps { 
                git branch: "${BRANCH_NAME}", credentialsId: "${GITHUB_CREDENTIALS}", url: "${GIT_URL}"
            }
        }

        stage('Unit Test') {
            steps { 
                sh 'mvn clean'
                sh 'mvn test'
                sh 'mvn compile'
            }
        }

        /*
        stage('Sonarqube Scan') {
            steps {
                withSonarQubeEnv(credentialsId: "${SONAQUBE_CRED}", installationName: "${SONAQUBE_INSTALLATION}") {
                    sh """${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=${APP_NAME} -Dsonar.projectKey=${APP_NAME} -Dsonar.java.binaries=."""
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: "${QG_CONDITION}", credentialsId: "${SONAQUBE_CRED}"
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy fs --format table -o maven_dependency.html ."
            }
        }
        */

        stage('Package App') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Upload Jar to Jfrog') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${JFROG_CRED}", usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    script {
                        sh """
                            curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_PASSWORD} \
                                 -T ${ARTIFACTPATH} \
                                 ${ARTIFACTORY_URL}/${REPO}/${ARTIFACTTARGETPATH}
                        """
                    }
                }
            }
        }

        stage('Docker Image Build') {
            steps {
                script {
                    sh "docker build --no-cache -t ${DOCKER_REPO}:latest ."
                    sh "docker build --no-cache -t ${DOCKER_REPO}:${BUILD_ID} ."
                }
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh "trivy image --format table -o docker_image_report.html ${DOCKER_REPO}:${BUILD_ID}"
            }
        }

        stage('Push Image to Registry') {
            steps {
                script {
                   // Use AWS CLI to log in to ECR
            def ecrPassword = sh(script: "aws ecr get-login-password --region ${AWS_REGION}", returnStdout: true).trim()
            sh "echo ${ecrPassword} | docker login --username AWS --password-stdin ${REPO_URL}"
            sh "docker push ${DOCKER_REPO}:latest"
            sh "docker push ${DOCKER_REPO}:${BUILD_ID}"
                }
            }
        }
    } // Closing the stages block
} // Closing the pipeline block


