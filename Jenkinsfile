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
               sh 'docker build -t vachana999/base-image:tagname .'
               sh 'docker push vachana999/base-image:tagname'
            }
        }
        stage('Docker Container app') {
            steps {
               sh 'docker pull vachana999/base-image:tagname'
            }
        }
    }
}
