pipeline {
    agent any
    environment {
        SONARQUBE_SERVER = 'SonarQubeServer' // SonarQube server name in Jenkins
        SONARQUBE_SCANNER_HOME = '/opt/sonar-scanner/bin:$PATH' // Path to SonarQube Scanner
        SOURCE_CODE_REPO = "https://github.com/zamharrad/k8s-Deployment.git"
        IMAGE_NAME = "fastapi-app"

    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github-credentials', url: "${SOURCE_CODE_REPO}"
            }
        }

        stage('set image tag') {
            steps {
                script {
                    def shortHash = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.IMAGE_TAG = "${buildNumber}-${shortHash}"
                    echo "Image Tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // def scannerHome = tool 'SonarQube-Scanner' // SonarQube Scanner installation name in Jenkins
                    // withSonarQubeEnv(SONARQUBE_SERVER) {
                    //     sh "${scannerHome}/bin/sonar-scanner"
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh "sonar-scanner"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage("trivy security scan") {
            steps {
                script {
                    sh "trivy image ${IMAGE_NAME}:${IMAGE_TAG} > trivy-report-${IMAGE_TAG}.txt || true"
                }
            }
        }
    }
}