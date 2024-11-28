pipeline {
    agent any
    
    tools{
        maven "Maven3"
    }
       
    stages {
        stage('Git Checkout-github') {
            steps {
             git branch: 'main', url: 'https://github.com/vachanahm/maven-app.git'
            }
        }
        stage('Build') {
            steps {
               sh 'mvn clean package'
            }
        }
        stage('Create a image') {
            steps {
               sh 'docker build -t vachana999/base-image:1 .'
               sh 'docker run -d -p 9000:8080 vachana999/base-image:1 '
              
            }
        }
       stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker', usernameVariable: 'vachana999', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        // Log in to Docker Hub using the credentials
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    }
                }
            }
        stage('Docker Container app') {
            steps {
               sh 'docker push vachana999/base-image:1'
            }
        }
    }
}
