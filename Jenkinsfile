pipeline {
    agent any

    stages {
        stage('checkout to helm') {
            steps {
                echo 'Checking out to helm..'
                cd C:\helm\windows-amd64
            }
        stage('Clone repository') {
            steps {
                echo 'clone repository to local'
                git clone https://github.com/pradeep-puttarajaiah-jdas/helm.git
            }
        }
        stage('Helm Linting') {
            steps {
                helm lint C:\helm\windows-amd64\helm
            }
        }
            stage('Chart-testing') {
                helm test my-release
            }
        }
    }
}
