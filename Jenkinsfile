
pipeline {
    agent {
        label 'slave'
    }

    tools {
        maven "Maven3"
    }

    environment {
        DOCKER_TAG = "" // Placeholder for dynamic tag
        REMOTE_HOST = '3.35.53.77'  // IP of the target Ubuntu machine
        REMOTE_USER = 'ubuntu'  // Username on the target Ubuntu machine
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
                    DOCKER_TAG = "1.${env.BUILD_NUMBER}"
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

        stage('SSH into Remote Ubuntu Machine') {
            steps {
                sshagent(credentials: ['remote-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
                        # Pull the image and start the container
                        docker pull vachana999/base-image:${DOCKER_TAG}
                        docker run -d --name my_ci_cd_container -p 9000:8080 vachana999/base-image:${DOCKER_TAG}
                        EOF
                    """
                }
            }
        }
    }
}
