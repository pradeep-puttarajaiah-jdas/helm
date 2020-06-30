pipeline {
   agent any
   stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[credentialsId: 'pradeep.puttarajaiah@blueyonder.com', url: 'https://github.com/pradeep-puttarajaiah-jdas/helm.git']]])
                sh "ls -lart ./*"
            }
        }     
    }
}
