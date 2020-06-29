pipeline {
    agent any
    stages {
        stage('Checkout to sub-dir') {
            steps {
                dir('subDir') {
                checkout scm
                }
            }
        }
        stage('Checkout repo') {
            steps {
                echo 'Checking out to helm..'
                git url: https://github.com/pradeep-puttarajaiah-jdas/helm.git
            }
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
