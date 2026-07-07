pipeline {
    agent any 
    
    tools {
        jdk 'jdk17'
        maven 'maven'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE_PROD = "kosaraju333/ecommerce-app-prod:latest"
        DOCKER_IMAGE_DEV = "kosaraju333/ecommerce-app-dev:latest"
        CONTAINER_NAME_PROD = "ecommerce-app-prod"
        CONTAINER_NAME_DEV = "ecommerce-app-dev"
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/kosaraju3333/Ecommerce-App.git'
            }
        }
        
        stage('Maven Compail') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Maven Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                // some block
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=ECommerce \
                        -Dsonar.projectKey=ECommerce \
                        -Dsonar.java.binaries=target/classes
                        '''
                }
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Publish to Nexues') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk17', maven: 'maven') {
                    // some block
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script {
                    if (env.BRANCH_NAME == "master") {
                        // This step should not normally be used in your script. Consult the inline help for details.
                        withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                            // some block
                            sh "docker build -t ${DOCKER_IMAGE_PROD} ."
                        }
                    } 

                    if (env.BRANCH_NAME == "develop") {
                        // This step should not normally be used in your script. Consult the inline help for details.
                        withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                            // some block
                            sh "docker build -t ${DOCKER_IMAGE_DEV} ."
                        }
                    }    
                }
                
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                script {
                    if (env.BRANCH_NAME == "master") {
                        sh "trivy image --format table -o trivy-image-report.html ${DOCKER_IMAGE_PROD}"
                        archiveArtifacts artifacts: 'trivy-image-report.html', fingerprint: true
                    }

                    else if (env.BRANCH_NAME == "develop") {
                        sh "trivy image --format table -o trivy-image-report.html ${DOCKER_IMAGE_DEV}"
                        archiveArtifacts artifacts: 'trivy-image-report.html', fingerprint: true
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == "master") {
                        withCredentials([usernamePassword(
                            credentialsId: 'docker-cred',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            sh '''
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                docker push ${DOCKER_IMAGE_PROD}
                            '''
                        }
                    }

                    if (env.BRANCH_NAME == "develop") {
                        withCredentials([usernamePassword(
                            credentialsId: 'docker-cred',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )]) {
                            sh '''
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                docker push ${DOCKER_IMAGE_DEV}
                            '''
                        }
                    }
                }
                
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                     if (env.BRANCH_NAME == "main") {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@169.32.0.205 \
                            "docker pull kosaraju333/${DOCKER_IMAGE_PROD}:latest && \
                            docker stop ${DOCKER_IMAGE_PROD} || true && \
                            docker rm ${DOCKER_IMAGE_PROD} || true && \
                            docker run -d --name ${CONTAINER_NAME_DEV} -p 80:8080 ${DOCKER_IMAGE_PROD}"
                        '''
                    }

                     else if (env.BRANCH_NAME == "develop") {
                        sh '''
                            ssh -o StrictHostKeyChecking=no ubuntu@169.32.0.205 \
                            "docker pull kosaraju333/${DOCKER_IMAGE_DEV}:latest && \
                            docker stop ${DOCKER_IMAGE_DEV} || true && \
                            docker rm ${DOCKER_IMAGE_DEV} || true && \
                            docker run -d --name ${CONTAINER_NAME_DEV} -p 90:8080 ${DOCKER_IMAGE_DEV}"
                        '''
                    }
                }
                
            }
        }
    }
        
    //     stage('Deploy to Container') {
    //         steps {
    //             script {
    //                 sh "docker stop ecommerce-app-container || true && docker rm ecommerce-app-container || true"
    //                 sh "docker run -d --name ecommerce-app-container -p 8083:8080 ${DOCKER_IMAGE}"
    //             }
    //         }
    //     }
    // }
    
    post {
        always {
            emailext(
                to: 'kosaraju3333@gmail.com',
                subject: 'Test Pipeline Email',
                body: 'Hello from Jenkins Pipeline'
            )
        }
    }
}
    
    // post {
    //     always {
    //         script {
    //             def jobName = env.JOB_NAME
    //             def buildNumber = env.BUILD_NUMBER
    //             def pipelineStatus = currentBuild.result ?: 'SUCCESS'
    //             def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

    //             def body = """
    //             <html>
    //             <body>
    //             <div style="border: 4px solid ${bannerColor}; padding: 10px;">
    //             <h2>${jobName} - Build ${buildNumber}</h2>
    //             <div style="background-color: ${bannerColor}; padding: 10px;">
    //             <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
    //             </div>
    //             <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
    //             </div>
    //             </body>
    //             </html>
    //             """
                
    //             emailext(
    //                 to: 'kosaraju3333@gmail.com',
    //                 subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus}",
    //                 body: body,
    //                 mimeType: 'text/html'
    //             )

    //             // emailext (
    //             //     subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
    //             //     body: body,
    //             //     to: 'kosaraju3333@gmail.com',
    //             //     mimeType: 'text/html',
    //             //     attachmentsPattern: 'trivy-image-report.html'
    //             // )
    //         }
    //     }
    // }
// }
