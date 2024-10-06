pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven'
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-cred') // AWS Access Key ID
        AWS_SECRET_ACCESS_KEY = credentials('aws-cred') // AWS Secret Access Key
        AWS_DEFAULT_REGION = 'us-east-2'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/CloudGeniuses/Boardgame.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
                archiveArtifacts artifacts: '**/target/*.jar', allowEmptyArchive: true
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImageName = 'Boardgame'
                    def dockerTag = 'latest'

                    echo "Building Docker image: ${dockerImageName}:${dockerTag}"
                    sh "sudo docker build -t ${dockerImageName}:${dockerTag} ."
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    // Docker Hub login
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo '${DOCKER_PASSWORD}' | sudo docker login --username '${DOCKER_USERNAME}' --password-stdin"
                    }
                }
            }
        }

        stage('Uploading to ECR') {
            steps {
                script {
                    // Login to AWS ECR
                    sh 'aws ecr get-login-password --region us-east-2 | sudo docker login --username AWS --password-stdin 211125403425.dkr.ecr.us-east-2.amazonaws.com'

                    // Push Docker image to ECR
                    sh 'sudo docker push 211125403425.dkr.ecr.us-east-2.amazonaws.com/Boardgame:latest'
                }
            }
        }
    }

    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

                def body = """
                    <html>
                    <body>
                    <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                    <h2>${jobName} - Build ${buildNumber}</h2>
                    <div style="background-color: ${bannerColor}; padding: 10px;">
                    <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                    </div>
                    <p>Check the <a href="${env.BUILD_URL}">console output</a>.</p>
                    </div>
                    </body>
                    </html>
                """

                emailext (
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: 'nikhilkhariya40@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html'
                )
            }
        }
    }
}
