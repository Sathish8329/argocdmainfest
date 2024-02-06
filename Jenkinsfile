pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    // Clean up the workspace directory
                    deleteDir()

                    // Clone the Azure DevOps repository using the PAT without exposing it in the command
                    withCredentials([string(credentialsId: 'AzureCredential', variable: 'AzureCredential')]) {
                        sh 'git clone -b stage_10012024 https://${AzureCredential}@dev.azure.com/kovaionai/Generic-Workflows/_git/generic-workflows-frontend'
                    }
                }
            }
        }

        stage('Build and Run Docker Image') {
            steps {
                script {
                    // Move into the directory where the Dockerfile is located
                    dir('generic-workflows-frontend') {
                        // Build Docker image with a dynamic tag based on the Jenkins build number
                        def dockerImageTag = "sathish8329/priappfront:${BUILD_NUMBER}"
                        docker.build(dockerImageTag)

                        // Run Docker container (commented out for now)
                        // docker.image(dockerImageTag).run("--rm -d -p 8080:80")
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    def dockerImageTag = "sathish8329/priappfront:${BUILD_NUMBER}"

                    // Log in to Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', 'DockerHubCredentials') {
                        // Push the Docker image to Docker Hub
                        docker.image(dockerImageTag).push()
                    }
                }
            }
        }

        stage('Update GIT') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        withCredentials([usernamePassword(credentialsId: 'Github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                            sh "git config user.email sathish8329@gmail.com"
                            sh "git config user.name Sathish8329"
                            sh "cat pirappfront.yaml"
                            sh "sed -i 's+sathish8329/priappfront.*+sathish8329/priappfront:${BUILD_NUMBER}+g' pirappfront.yaml"
                            sh "cat pirappfront.yaml"
                            sh "git add ."
                            sh "git commit -m 'Triggered Build: ${BUILD_NUMBER}'"
                            sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/argocdmainfest.git HEAD:main"
                        }
                    }
                }
            }
        }
    }
}
