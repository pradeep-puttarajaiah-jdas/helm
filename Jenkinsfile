pipeline {
   agent any
   stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[credentialsId: 'pradeep-puttarajaiah-jdas', url: 'https://github.com/pradeep-puttarajaiah-jdas/helm.git']]])
                sh "ls -lart ./*"
            }
        }     
    }
}
