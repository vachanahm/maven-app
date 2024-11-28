pipeline {
    agent {
        label 'slave'
    }

    tools {
        maven "Maven3"
    }

    environment {
        DOCKER_TAG = "" // Placeholder for dynamic tag
    }

    stages {
        stage('Git Checkout - GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/vachanahm/maven-app.git'
            }
        }

        stage('Determine Docker Tag') {
            steps {
                script {
                    // Generate a dynamic tag using BUILD_NUMBER
                    def buildNumber = currentBuild.number
                    DOCKER_TAG = "1.${buildNumber}"
                    echo "Docker tag set to: ${DOCKER_TAG}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Create Docker Image') {
            steps {
                sh "docker build -t vachana999/base-image:${DOCKER_TAG} ."
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push vachana999/base-image:${DOCKER_TAG}"
            }
        }
    }
}
