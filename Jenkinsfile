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
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        // Safety check: Use build number if Git Commit variable is missing
                        def imageTag = env.GIT_COMMIT ?: "build-${env.BUILD_NUMBER}"
                        
                        sh "docker build -t hgol42/omidfirsthub:${imageTag} ."
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                        sh "docker push hgol42/omidfirsthub:${imageTag}"
                        sh "docker logout"
                    }
                }
            }
        }

        stage('Kubernetes Deployment - DEV') {
            steps {
                withKubeConfig([credentialsId: 'Kubeconfig']) {
                    script {
                        // Safe fallback tag in case GIT_COMMIT is missing
                        def imageTag = env.GIT_COMMIT ?: "build-${env.BUILD_NUMBER}"
                        
                        // Dynamically updates the placeholder in your YAML file to use your new Docker image
                        sh "sed -i 's#replace#hgol42/omidfirsthub:${imageTag}#g' k8s_deployment_service.yaml"
                        
                        // Deploys the updated file to your Azure Kubernetes cluster
                        sh "kubectl apply -f k8s_deployment_service.yaml"
                    }
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
}