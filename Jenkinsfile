pipeline {
    agent any
    
    environment {
        SONAR_HOME = tool "sonar"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }

        stage("Workspace cleanup") {
            steps {
                script {
                    cleanWs()
                }
            }
        }

        stage('Git: Code Checkout') {
            steps {
                script {
                    code_checkout("https://github.com/sauravgarg547/Wanderlust-Mega-Project.git","main")
                }
            }
        }

        stage("Run Tests") {
            steps {
                script {
                    sh 'mvn test'
                }
            }
        }

        stage("Check XML Files") {
            steps {
                script {
                    sh "ls -l **/*.xml || echo 'No XML files found'"
                }
            }
        }

        stage("SonarQube: Code Analysis") {
            steps {
                script {
                    sonarqube_analysis("sonar","wanderlust","wanderlust")
                }
            }
        }

        stage("Docker: Build Images") {
            steps {
                script {
                    dir('backend') {
                        docker_build("wanderlust-backend-beta","${params.BACKEND_DOCKER_TAG}","saurav547")
                    }
                    dir('frontend') {
                        docker_build("wanderlust-frontend-beta","${params.FRONTEND_DOCKER_TAG}","saurav547")
                    }
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'target/surefire-reports/*.xml', followSymlinks: false
            build job: "Wanderlust-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
    }
}
