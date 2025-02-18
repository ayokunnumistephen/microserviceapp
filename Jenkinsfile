pipeline {
    agent any
    environment {
        APP_REPO_NAME = "microserviceapp"
        IMAGE_NAME = "stephenadmin/cartservice"
        BUILD_TAG = "${BUILD_NUMBER}"
        DEPLOYMENT_MANIFEST = "deployment-service.yml"
        GIT_REPO_URL = "https://github.com/ayokunnumistephen/microserviceapp.git"
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
                            rm -rf ${APP_REPO_NAME} || true
                            git clone ${GIT_REPO_URL}
                            cd ${APP_REPO_NAME}
                            git config --global user.email "jenkins@eamanzetec.com.ng"
                            git config --global user.name "Jenkins CI"
                            git checkout ${STAGE_BRANCH}
                            git pull origin ${STAGE_BRANCH} --rebase
                            sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_TAG}|' ${DEPLOYMENT_MANIFEST}
                            git add ${DEPLOYMENT_MANIFEST}
                            if git diff --cached --quiet; then
                                echo "No changes detected, skipping commit in stage branch."
                            else
                            git commit -m "Update image tag to ${IMAGE_NAME}:${BUILD_TAG} in stage branch"
                            git push https://\$GIT_USERNAME:\$GIT_TOKEN@github.com/ayokunnumistephen/microserviceapp.git ${STAGE_BRANCH}
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
                            cd ${APP_REPO_NAME}
                            git checkout ${MAIN_BRANCH}
                            git pull origin ${MAIN_BRANCH} --rebase
                            git config --global user.email "jenkins@eamanzetec.com.ng"
                            git config --global user.name "Jenkins CI"
                            sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${BUILD_TAG}|' ${DEPLOYMENT_MANIFEST}
                            git add ${DEPLOYMENT_MANIFEST}
                            if git diff --cached --quiet; then
                                echo "No changes detected, skipping commit in main branch."
                            else
                            git commit -m "Update image tag to ${IMAGE_NAME}:${BUILD_TAG} in main branch"
                            git push https://\$GIT_USERNAME:\$GIT_TOKEN@github.com/ayokunnumistephen/microserviceapp.git ${MAIN_BRANCH}
                        """
                    }
                }
            }
        }
    }
}
