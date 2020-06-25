properties([pipelineTriggers([githubPush()])]
pipeline {
    agent any

    stages {
        stage('Build stage') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test stage') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deployment') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
