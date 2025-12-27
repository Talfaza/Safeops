pipeline {
    agent any
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        timeout(time: 60, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log -1 --pretty=format:"%h - %s (%an, %ar)"'
            }
        }
        
        stage('Build Node.js Services') {
            steps {
                dir('services/log-collector') {
                    sh 'npm install'
                }
                dir('services/report-generator') {
                    sh 'npm install'
                }
            }
        }
        
        stage('Build Python Services') {
            steps {
                dir('services/log-parser') {
                    sh 'pip install -r requirements.txt || pip3 install -r requirements.txt'
                }
                dir('services/vuln-detector') {
                    sh 'pip install -r requirements.txt || pip3 install -r requirements.txt'
                }
                dir('services/fix-suggester') {
                    sh 'pip install -r requirements.txt || pip3 install -r requirements.txt'
                }
                dir('services/anomaly-detector') {
                    sh 'pip install -r requirements.txt || pip3 install -r requirements.txt'
                }
            }
        }
        
        stage('Build Dashboard') {
            steps {
                dir('services/dashboard') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }
        
        stage('Docker Build') {
            parallel {
                stage('Build log-collector') {
                    steps {
                        dir('services/log-collector') {
                            sh "docker build -t safeops-log-collector:${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build log-parser') {
                    steps {
                        dir('services/log-parser') {
                            sh "docker build -t safeops-log-parser:${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build vuln-detector') {
                    steps {
                        dir('services/vuln-detector') {
                            sh "docker build -t safeops-vuln-detector:${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build fix-suggester') {
                    steps {
                        dir('services/fix-suggester') {
                            sh "docker build -t safeops-fix-suggester:${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build anomaly-detector') {
                    steps {
                        dir('services/anomaly-detector') {
                            sh "docker build -t safeops-anomaly-detector:${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build report-generator') {
                    steps {
                        dir('services/report-generator') {
                            sh "docker build -t safeops-report-generator:${IMAGE_TAG} ."
                        }
                    }
                }
                stage('Build dashboard') {
                    steps {
                        dir('services/dashboard') {
                            sh "docker build -t safeops-dashboard:${IMAGE_TAG} ."
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully! Build: ${BUILD_NUMBER}"
            sh 'docker images | grep safeops || true'
        }
        failure {
            echo "Pipeline failed! Check the logs for details."
        }
    }
}
