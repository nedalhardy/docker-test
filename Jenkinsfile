pipeline {
    agent any
    tools {
        maven 'localMaven'
        jdk 'localJDK'
        DockerTool {
            label 'windows'
            image 'mcr.microsoft.com/powershell'
        }
    }
    environment {
        registry = "nedaljed/myimage"
        registryCredential = 'dockerhub'
        dockerImage = ''
    }

    triggers {
        pollSCM('* * * * *') // Polling Source Control
    }

    stages {
        stage("build") {
            steps {
                bat 'mvn clean verify'
            }
        }
        stage("sonar") {
            steps {
                withSonarQubeEnv('sonarQube') {
                    bat 'mvn sonar:sonar'
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
            post {
                success {
                    echo 'SONAR PASSED'
                }
                failure {
                    echo 'SONAR FAILED'
                }
            }

        }
        stage("package") {
            steps {
                bat 'mvn clean package'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.jar'
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    echo "ici"
                    dockerImage = docker.build(registry + ":$BUILD_NUMBER")
                }
            }
        }

        stage('Deploy Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Remove Unused docker image') {
            steps {
                bat "docker rmi $registry:$BUILD_NUMBER"
            }
        }

    }
}