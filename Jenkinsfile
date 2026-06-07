node {
    try {
        stage('Git Version') {
            sh 'git --version'
        }

        stage('Unit Test') {
            sh 'mvn test'
        }

        stage('Code Coverage') {
            sh 'mvn jacoco:report'
        }

        stage('Build Artifact') {
            sh 'mvn clean package -DskipTests=true'
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }

        stage('Docker Build and Push') {
           
            withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                
              
                sh "docker build -t hgol42/omidfirsthub:${env.GIT_COMMIT} ."
                
              
                sh "echo \$DOCKER_HUB_PASSWORD | docker login -u \$DOCKER_HUB_USERNAME --password-stdin"
                

                sh "docker push hgol42/omidfirsthub:${env.GIT_COMMIT}"
                sh "docker logout"
            }
        }

        echo 'Build completed successfully.'

    } catch (Exception e) {
        echo "Build failed due to error: ${e.getMessage()}"
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        // Safe post-execution steps
        junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
        
        try {
            jacoco(
                execPattern: 'target/jacoco.exec',
                classPattern: 'target/classes',
                sourcePattern: 'src/main/java',
                exclusionPattern: '**/*Test*.class'
            )
        } catch (Exception je) {
            echo "Skipping JaCoCo reporting step due to version limits: ${je.getMessage()}"
        }
    }
}