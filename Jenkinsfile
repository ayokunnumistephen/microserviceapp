pipeline {
    agent any
    environment {
        IMAGE_NAME = "stephenadmin/cartservice"
        BUILD_TAG = "${BUILD_NUMBER}"
        DEPLOYMENT_MANIFEST = "deployment-service.yml"
        GIT_REPO_URL = "https://github.com/ayokunnumistephen/set1-microserviceapp.git"
        STAGE_BRANCH = "stage"
        MAIN_BRANCH = "main"
    }
    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t ${IMAGE_NAME}:${BUILD_TAG} ."
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push ${IMAGE_NAME}:${BUILD_TAG}"
                    }
                }
            }
        }
        stage('Update Deployment Manifest in Stage Branch') {
            steps {
                script {
                    // Using Git credentials
                    withCredentials([usernamePassword(credentialsId: 'git-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                            git clone ${GIT_REPO_URL}
                            cd set1-microserviceapp
                            git config --global user.email "jenkins@eamanzetec.com.ng"
                            git config --global user.name "Jenkins CI"
                            git checkout ${STAGE_BRANCH}
                            sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_TAG}|' ${DEPLOYMENT_MANIFEST}
                            git add ${DEPLOYMENT_MANIFEST}
                            git commit -m "Update image tag to ${IMAGE_NAME}:${BUILD_TAG} in stage branch"
                            git push https://\$GIT_USERNAME:\$GIT_TOKEN@github.com/ayokunnumistephen/set1-microserviceapp.git ${STAGE_BRANCH}
                        """
                    }
                }
            }
        }
        stage('Manual Approval for Main Branch Update') {
            steps {
                input message: "Approve updating deployment-service.yml in the main branch?"
            }
        }
        stage('Update Deployment Manifest in Main Branch') {
            steps {
                script {
                    // Using Git credentials
                    withCredentials([usernamePassword(credentialsId: 'git-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                            cd set1-microserviceapp
                            git checkout ${MAIN_BRANCH}
                            git config --global user.email "jenkins@eamanzetec.com.ng"
                            git config --global user.name "Jenkins CI"
                            sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_TAG}|' ${DEPLOYMENT_MANIFEST}
                            git add ${DEPLOYMENT_MANIFEST}
                            git commit -m "Update image tag to ${IMAGE_NAME}:${BUILD_TAG} in main branch"
                            git push https://\$GIT_USERNAME:\$GIT_TOKEN@github.com/ayokunnumistephen/set1-microserviceapp.git ${MAIN_BRANCH}
                        """
                    }
                }
            }
        }
    }
}
