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
        stage('Checkout') {
            steps {
                echo 'Checking out to helm..'
                git url: https://github.com/pradeep-puttarajaiah-jdas/helm.git
            }
        }
    }
}
