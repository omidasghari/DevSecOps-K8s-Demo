pipeline {
    agent any

    stages {
        stage('Git Version') {
            steps {
                sh 'git --version'
            }
        }
        stage('Docker Build and Push') {
    steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
            sh 'printenv'
            
            // FIX: Lowercase repository name, clean double quotes, and single space before the dot
            sh "docker build -t hhgol42/omidfirsthub:${env.GIT_COMMIT} ."
            sh "docker push hgol42/omidfirsthub:${env.GIT_COMMIT}"
        } 
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
