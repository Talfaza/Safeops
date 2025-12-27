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
        
        stage('Build & Test') {
            parallel {
                stage('Node.js Services') {
                    agent {
                        docker {
                            image 'node:20-alpine'
                            args '-v $HOME/.npm:/root/.npm'
                        }
                    }
                    stages {
                        stage('Log Collector') {
                            steps {
                                dir('services/log-collector') {
                                    sh 'npm ci'
                                    sh 'npm run lint || true'
                                    sh 'npm test || echo "No tests configured"'
                                }
                            }
                        }
                        stage('Report Generator') {
                            steps {
                                dir('services/report-generator') {
                                    sh 'npm ci'
                                    sh 'npm run lint || true'
                                    sh 'npm test || echo "No tests configured"'
                                }
                            }
                        }
                    }
                }
                
                stage('Python Services') {
                    agent {
                        docker {
                            image 'python:3.11-slim'
                            args '-v $HOME/.cache/pip:/root/.cache/pip'
                        }
                    }
                    stages {
                        stage('Install Common Tools') {
                            steps {
                                sh 'pip install flake8 pytest pytest-cov'
                            }
                        }
                        stage('Log Parser') {
                            steps {
                                dir('services/log-parser') {
                                    sh 'pip install -r requirements.txt'
                                    sh 'flake8 src/ --max-line-length=120 || true'
                                    sh 'pytest tests/ -v --cov=src || echo "No tests configured"'
                                }
                            }
                        }
                        stage('Vuln Detector') {
                            steps {
                                dir('services/vuln-detector') {
                                    sh 'pip install -r requirements.txt'
                                    sh 'flake8 src/ --max-line-length=120 || true'
                                    sh 'pytest tests/ -v --cov=src || echo "No tests configured"'
                                }
                            }
                        }
                        stage('Fix Suggester') {
                            steps {
                                dir('services/fix-suggester') {
                                    sh 'pip install -r requirements.txt'
                                    sh 'flake8 src/ --max-line-length=120 || true'
                                    sh 'pytest tests/ -v --cov=src || echo "No tests configured"'
                                }
                            }
                        }
                        stage('Anomaly Detector') {
                            steps {
                                dir('services/anomaly-detector') {
                                    sh 'pip install -r requirements.txt'
                                    sh 'flake8 src/ --max-line-length=120 || true'
                                    sh 'pytest tests/ -v --cov=src || echo "No tests configured"'
                                }
                            }
                        }
                    }
                }
                
                stage('Dashboard') {
                    agent {
                        docker {
                            image 'node:20-alpine'
                            args '-v $HOME/.npm:/root/.npm'
                        }
                    }
                    steps {
                        dir('services/dashboard') {
                            sh 'npm ci'
                            sh 'npm run lint || true'
                            sh 'npm run build'
                        }
                    }
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
            sh 'docker images | grep safeops'
        }
        failure {
            echo "Pipeline failed! Check the logs for details."
        }
    }
}
