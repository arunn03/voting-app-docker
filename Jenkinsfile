pipeline {
    agent any
    tools {
        jdk 'jdk17'
        dockerTool 'docker'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
 
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: true, url: 'https://github.com/arunn03/voting-app-docker.git'
            }
        }
        
        stage('Sonarqube analysis') {
            steps {
                
                withSonarQubeEnv('sonarqube-server') {
                    script {
                        def sonarResult = sh(script: '''
                            $SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.java.binaries=**/*.java \
                            -Dsonar.projectName=Voting-App \
                            -Dsonar.projectKey=voting-app \
                            -Dsonar.sources=. \
                        ''', returnStatus: true)
                        
                        if (sonarResult != 0) {
                            currentBuild.result = 'FAILURE'
                            error("SonarQube analysis failed.")
                        }
                    }
                }
            }
        }
        
        stage('Docker Images Build') {
            steps {
                sh '''
                docker build -t result-app ${WORKSPACE}/result/ && \
                docker build -t worker-app ${WORKSPACE}/worker/ && \
                docker build -t vote-app ${WORKSPACE}/vote/
                '''
            }
        }
        
        stage('Docker Compose') {
            steps {
                step([$class: 'DockerComposeBuilder', dockerComposeFile: 'docker-compose.yml', option: [$class: 'StartAllServices'], useCustomDockerComposeFile: true])
            }
        }
 
    }
}
