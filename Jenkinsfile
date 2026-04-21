pipeline {

    agent any

    tools {
        nodejs 'node24'   // Nom configuré dans Manage Jenkins → Tools → NodeJS
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

    

     

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }

    post {
        success {
            echo '✅ Build réussi !'
        }
        failure {
            echo '❌ Build échoué.'
        }
        always {
            cleanWs()
        }
    }
}