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

        stage('Docker Build and Push') {
            steps {
                withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                    sh 'printenv'
                    
                    // FIXED: Corrected username typo (hgol42) to match build and push exactly
                    sh "docker build -t siddharth67/omidfirsthub:${env.GIT_COMMIT} ."
                    sh "docker push siddharth67/omidfirsthub:${env.GIT_COMMIT}"
                } 
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'

            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                jacoco(
                    execPattern: 'target/jacoco.exec',
                    classPattern: 'target/classes',
                    sourcePattern: 'src/main/java',
                    exclusionPattern: '**/*Test*.class'
                )
            }
        }

        success {
            echo 'Build completed successfully.'
        }

        failure {
            echo 'Build failed.'
        }
    }
}