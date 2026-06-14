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
        
        stage('SonarQube-SAST') {
            steps {
            withSonarQubeEnv('SonarQube'){
                sh '''
                        mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
  -Dsonar.projectKey=numeric-Application \
  -Dsonar.projectName='numeric-Application' \
  -Dsonar.host.url=http://devsecdns.westeurope.cloudapp.azure.com:9000 \
  -Dsonar.token=sqp_e0f830a5a601fcffd68dc0486b6bb927834664a9
                    '''
            }
             
                timeout(time: 1, unit: 'MINUTES') {
                    script{
                         waitForQualityGate abortPipeline: true
                    }
                   
                }
             
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
                script {
                    env.IMAGE_TAG = env.GIT_COMMIT ?: "build-${env.BUILD_NUMBER}"
                    
                    echo "Starting deployment sequence..."
                    echo "Target Image Tag: ${env.IMAGE_TAG}"
                    
                    // 1. Modify the manifest file
                    sh "sed -i 's#replace#hgol42/omidfirsthub:${env.IMAGE_TAG}#g' k8s_deployment_service.yaml"
                    
                    // 2. FIXED: Explicitly point to the fresh config file on port 6443
                    echo "Applying manifest directly to the cluster..."
                    sh "kubectl apply -f k8s_deployment_service.yaml --kubeconfig=/home/jenkins/.kube/config"
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