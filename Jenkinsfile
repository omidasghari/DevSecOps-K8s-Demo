pipeline {
    agent any

    stages {
        stage('Git Version') {
            steps {
                sh 'git --version'
            }
        }
        
        stage('Docker Build and Push') {
    // 1. Define the placeholder names (Keep them generic!)
    withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
        
        def imageTag = env.GIT_COMMIT ?: "build-${env.BUILD_NUMBER}"
        
        sh "docker build -t hgol42/omidfirsthub:${imageTag} ."
        
        // 2. Use the placeholder names in your shell commands
        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
        
        sh "docker push hgol42/omidfirsthub:${imageTag}"
        sh "docker logout"
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
    }

    post {
        always {
            // This works perfectly to ignore empty test folders
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'

            // FIXED: Using a safe structure for older JaCoCo plugin versions
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
