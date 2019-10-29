pipeline {
    agent any
    tools {
        maven 'localMaven'
        jdk 'localJDK'
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
                sh 'mvn clean verify'
            }
        }
        stage("sonar") {
            steps {
                withSonarQubeEnv('sonarQube') {
                    sh 'mvn sonar:sonar'
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
                sh 'mvn clean package'
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
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }

    }
}