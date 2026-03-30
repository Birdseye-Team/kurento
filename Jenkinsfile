def MVN = 'Maven 3.8.4'

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
            when {
                anyOf {
                    expression { return env.BRANCH_NAME == 'release' }
                    expression { return env.BRANCH_NAME ==~ /^release-.*/ }
                }
            }
            steps {
                withMaven(maven: MVN) {
                    sh 'mvn -B clean verify'
                }
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    expression { return env.BRANCH_NAME == 'release' }
                    expression { return env.BRANCH_NAME ==~ /^release-.*/ }
                }
            }
            steps {
                // Uses distributionManagement from kurento-parent-pom (command-line params don't work with Maven 3.8.4)
                withMaven(maven: MVN, globalMavenSettingsConfig: 'maven.birdseyesecurity.com') {
                    sh 'mvn -B -Pdeploy deploy'
                }
            }
        }
    }

    post {
        always {
            script {
                // Only publish JUnit results if Build & Test stage ran (i.e., on release branches)
                if (env.BRANCH_NAME == 'release' || env.BRANCH_NAME ==~ /^release-.*/) {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }
    }
}