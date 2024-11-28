pipeline {
    agent {
        label 'master-node'
    }

    tools {
        maven "Maven3"
    }

    environment {
        DOCKER_TAG = "" // Placeholder for dynamic tag
        REMOTE_SSH_KEY = credentials('remote-ssh-key') // SSH key credential ID from Jenkins
        REMOTE_HOST = '172.31.7.122'  // IP of the target Ubuntu machine
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
                 sshagent(credentials: [REMOTE_SSH_KEY])  {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
                        # Ensure Docker is installed and configured
                        if ! command -v docker &> /dev/null; then
                            echo "Docker not found, installing..."
                            sudo apt-get update -y
                            sudo apt-get install -y ca-certificates curl gnupg
                            sudo install -m 0755 -d /etc/apt/keyrings
                            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
                            sudo chmod a+r /etc/apt/keyrings/docker.gpg
                            echo \
                              "deb [arch=\$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
                              \$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
                            sudo apt-get update -y
                            sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-compose
                            sudo usermod -aG docker $USER
                            sudo systemctl enable docker
                            sudo systemctl start docker
                        else
                            echo "Docker is already installed."
                        fi

                        # Pull the image and start the container
                        docker pull vachana999/base-image:${DOCKER_TAG}
                        docker stop my_ci_cd_container || true
                        docker rm my_ci_cd_container || true
                        docker run -d --name my_ci_cd_container -p 9000:8080 vachana999/base-image:${DOCKER_TAG}

                        EOF
                    """
                }
            }
        }
    }
}
