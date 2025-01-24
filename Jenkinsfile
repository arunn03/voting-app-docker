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
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/arunn03/voting-app-docker.git'
            }
        }
        
        stage('Sonarqube analysis') {
            steps {
                
                withSonarQubeEnv('sonarqube-server') {
                    // script {
                    //     def sonarResult = sh(script: '''
                    //         $SCANNER_HOME/bin/sonar-scanner \
                    //         -Dsonar.java.binaries=**/*.java \
                    //         -Dsonar.projectName=Voting-App \
                    //         -Dsonar.projectKey=voting-app \
                    //         -Dsonar.sources=. \
                    //     ''', returnStatus: true)
                        
                    //     if (sonarResult != 0) {
                    //         currentBuild.result = 'FAILURE'
                    //         error("SonarQube analysis failed.")
                    //     }
                    // }

                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.java.binaries=**/*.java \
                        -Dsonar.projectName=Voting-App \
                        -Dsonar.projectKey=voting-app \
                        -Dsonar.sources=. \
                    '''
                }
            }
        }
        
        stage('Docker Images Build') {
            steps {
                sh 'docker build -t result-app ${WORKSPACE}/result/'
                sh 'docker build -t vote-app ${WORKSPACE}/vote/'
                script {
                    // Set platform and architecture for the build
                    def arch = sh(script: 'uname -m', returnStdout: true).trim()
                    if (arch == 'x86_64') {
                        sh '''
                            docker build -t worker-app \
                                --build-arg BUILDPLATFORM=linux/amd64 \
                                --build-arg TARGETPLATFORM=linux/amd64 \
                                --build-arg TARGETARCH=amd64 \
                                ${WORKSPACE}/worker/
                        '''
                    } else if (arch == 'aarch64') {
                        sh '''
                            docker build -t worker-app \
                                --build-arg BUILDPLATFORM=linux/arm64 \
                                --build-arg TARGETPLATFORM=linux/arm64 \
                                --build-arg TARGETARCH=arm64 \
                                ${WORKSPACE}/worker/
                        '''
                    }
                }
                // sh '''
                // docker build -t result-app ${WORKSPACE}/result/ && \
                // docker build -t worker-app \
                //     --build-arg BUILDPLATFORM=${BUILDPLATFORM} \
                //     --build-arg TARGETPLATFORM=${TARGETPLATFORM} \
                //     --build-arg TARGETARCH=${TARGETARCH} \
                //     ${WORKSPACE}/worker/ && \
                // docker build -t vote-app ${WORKSPACE}/vote/
                // '''
            }
        }
        
        stage('Docker Compose') {
            steps {
                step([$class: 'DockerComposeBuilder', dockerComposeFile: 'docker-compose.yml', option: [$class: 'StartAllServices'], useCustomDockerComposeFile: true])
            }
        }
 
    }
}
