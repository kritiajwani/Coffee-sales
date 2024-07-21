pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "my-image:latest" // Default image tag
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    // Set the DOCKER_IMAGE environment variable to use the build ID for unique tagging
                    env.DOCKER_IMAGE = "my-image:${env.BUILD_ID}"
                }
            }
        }
        stage('Check for Branch') {
            steps {
                script {
                    // Get the branch name
                    def branchName = env.GIT_BRANCH ?: sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()

                    if (branchName == 'main') {
                        currentBuild.description = "Changes detected in main branch. Building new image."
                        env.BUILD_IMAGE = "true"
                    } else {
                        currentBuild.description = "No relevant changes in main branch. Using existing image."
                        env.BUILD_IMAGE = "false"
                    }
                }
            }
        }
        stage('Build and Push Image') {
            when {
                expression { return env.BUILD_IMAGE == "true" }
            }
            steps {
                script {
                    // Build and push the new Docker image if changes are detected in the main branch
                    def image = docker.build(env.DOCKER_IMAGE)
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        image.push()
                    }
                }
            }
        }
        stage('Build Application') {
            steps {
                script {
                    // Use the specified Docker image to build the application
                    def image = docker.image(env.DOCKER_IMAGE)
                    image.inside {
                        // Add your application build steps here
                        sh 'echo "Building application with image ${env.DOCKER_IMAGE}"'
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
