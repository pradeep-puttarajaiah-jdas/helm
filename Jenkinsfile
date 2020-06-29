pipeline {
    agent any
    stages {
        stage('Clone repository') {
            steps {
                echo 'Checking out to helm..'
                git clone https://github.com/pradeep-puttarajaiah-jdas/helm.git
            }
        stage('Helm Linting') {
            steps {
                helm lint C:\helm\windows-amd64
            }
        }
        stage('Chart-testing') {
            steps {
                helm test my-release
            }
        }
    }
}
