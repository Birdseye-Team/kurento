pipeline {
    agent any

    environment {
        // Adjust if you use a specific Maven tool installation in Jenkins
        MAVEN_OPTS = "-Dmaven.test.skip=false"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn -B clean verify'
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    expression { return env.BRANCH_NAME == 'main' }
                    expression { return env.BRANCH_NAME == 'master' }
                    expression { return env.BRANCH_NAME == 'release' }
                    expression { return env.BRANCH_NAME ==~ /^release-.*/ }
                }
            }
            steps {
                // Uses <distributionManagement> from pom.xml
                sh 'mvn -B -Pdeploy deploy'
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        }
    }
}
