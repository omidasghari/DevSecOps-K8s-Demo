pipeline {
    agent any

    stages {

        stage('Git Version') {
            steps {
                sh 'git --version'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Code Coverage') {
            steps {
                sh 'mvn jacoco:report'
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package -DskipTests=true'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        always {
            junit 'target/surefire-reports/*.xml'

            jacoco execpattern 'target/jacoco.exec'
        }

        success {
            echo 'Build completed successfully.'
        }

        failure {
            echo 'Build failed.'
        }
    }
}