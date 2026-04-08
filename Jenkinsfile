def MVN = 'maven-3.9'

pipeline {
    agent any
    tools {
        jdk 'JDK25'
    }
    environment {
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
                    sh 'mvn -B -f clients/java/qa-pom/pom.xml clean verify'
                    sh 'mvn -B -f clients/java/pom.xml clean verify'
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
                withMaven(maven: MVN, globalMavenSettingsConfig: 'maven.birdseyesecurity.com') {
                    sh 'mvn -B -f clients/java/qa-pom/pom.xml clean verify'
                    sh 'mvn -B -f clients/java/pom.xml -Pdeploy deploy'
                }
            }
        }
    }
    post {
        always {
            script {
                if (env.BRANCH_NAME == 'release' || env.BRANCH_NAME ==~ /^release-.*/) {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }
    }
}