@Library('Shared') _
pipeline {
    agent any
    
    environment {
        SONAR_HOME = tool "Sonar"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (!params.FRONTEND_DOCKER_TAG || !params.BACKEND_DOCKER_TAG) {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }

        stage("Workspace Cleanup") {
            steps {
                cleanWs()
            }
        }
        
        stage('Git: Code Checkout') {
            steps {
                script {
                    code_checkout("https://github.com/GarvitdevOps/Wanderlust-Project.git", "main")
                }
            }
        }
        
        stage("Security Scans") {
            parallel {
                stage("Trivy: Filesystem Scan") {
                    steps {
                        script {
                            trivy_scan()
                        }
                    }
                }
                
                stage("OWASP: Dependency Check") {
                    steps {
                        script {
                            owasp_dependency()
                        }
                    }
                }
            }
        }
        
        stage("SonarQube: Code Analysis") {
            steps {
                script {
                    sonarqube_analysis("Sonar", "wanderlust", "wanderlust")
                }
            }
        }
        
        stage("SonarQube: Code Quality Gates") {
            steps {
                script {
                    sonarqube_code_quality()
                }
            }
        }
        
        stage('Exporting Environment Variables') {
            parallel {
                stage("Backend Env Setup") {
                    steps {
                        script {
                            dir("Automations") {
                                sh "bash updatebackendnew.sh"
                            }
                        }
                    }
                }
                
                stage("Frontend Env Setup") {
                    steps {
                        script {
                            dir("Automations") {
                                sh "bash updatefrontendnew.sh"
                            }
                        }
                    }
                }
            }
        }
        
        stage("Docker: Build and Push Images") {
            parallel {
                stage("Backend Build & Push") {
                    steps {
                        script {
                            dir('backend') {
                                docker_build("wanderlust-backend-beta", "${params.BACKEND_DOCKER_TAG}", "garvmodi")
                            }
                            docker_push("wanderlust-backend-beta", "${params.BACKEND_DOCKER_TAG}", "garvmodi")
                        }
                    }
                }
                
                stage("Frontend Build & Push") {
                    steps {
                        script {
                            dir('frontend') {
                                docker_build("wanderlust-frontend-beta", "${params.FRONTEND_DOCKER_TAG}", "garvmodi")
                            }
                            docker_push("wanderlust-frontend-beta", "${params.FRONTEND_DOCKER_TAG}", "garvmodi")
                        }
                    }
                }
            }
        }
    }
    
    post {
        success {
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "Wanderlust-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
    }
}

