pipeline {
   agent any
   stages {
    stage('Checkout') {
      steps {
        script {
           // The below will clone your repo and will be checked out to master branch by default.
           git credentialsId: 'pradeep-puttarajaiah-jdas', url: 'https://github.com/pradeep-puttarajaiah-jdas/helm.git'
           // Do a ls -lart to view all the files are cloned. It will be clonned. This is just for you to be sure about it.
           sh "ls -lart ./*"
        }
      }
    }
    stage('helm dependency update') {
       steps {
          script {
             //sh 'helm repo update'
             sh 'helm dependency update'
          }
       }
    }
    stage('Linting') {
       steps {
          sh 'helm lint'
       }
    }
    stage('List Releases') {
       steps {
          sh 'helm list'
       }
    }
    stage('Chart-testing') {
       steps {
          script {
             sh 'helm test logistics-refs'
          }
       }
    }
   }
}
