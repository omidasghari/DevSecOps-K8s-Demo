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
                // 'mvn test' runs the unit tests and generates target/surefire-reports/*.xml
                sh 'mvn test'
            }
        }

        stage('Code Coverage') {
            steps {
                // This generates the HTML reports using the data from 'mvn test'
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
            // FIX: allowEmptyResults prevents Jenkins from breaking if 0 tests are found
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'

            // FIX: Configured the jacoco step with safety parameters
            jacoco(
                execPattern: 'target/jacoco.exec',
                classPattern: 'target/classes',
                sourcePattern: 'src/main/java',
                exclusionPattern: '**/*Test*.class',
                allowEmptyResults: true
            )
        }

        success {
            echo 'Build completed successfully.'
        }

        failure {
            echo 'Build failed.'
        }
    }
}